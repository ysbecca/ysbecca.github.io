---
layout: post
comments: true
title:  "Managing memory with large datasets in TensorFlow"
date:   2018-01-30 21:18:00 +0100
categories: programming
tags:
- tensorflow
- gpu
- parallelism
- memory
---

In the past nine months or so I've been working primarily with high-resolution, high-magnification digital histopathology images, which are hundreds of thousands of pixels in diameter and gigabytes in size. Anyone working with large, content-rich datasets and TensorFlow has developed an ambivalent reaction to the following error message:

```
tensorflow.python.framework.errors_impl.ResourceExhaustedError: OOM when allocating tensor with shape[4096]
```

Here are three simple ways to help maximise usage of available resources. Alternatively, you could go work at [Google DeepMind](https://deepmind.com/) where memory is not an issue.

<!--excerpt-->

# 1. Use all the available resources - multiple GPUs.

Running on a single GPU node is built-in and pretty much transparent in TensorFlow. Using multiple devices requires a bit more work, but has both performance benefits and increases the available memory. Keep in mind that there is a latency associated with GPU-to-CPU and CPU-to-GPU data transfers.

#### Data parallelism

The data parallelism approach is to split each training batch into equal sets, replicate the TensorFlow model across all available devices, then give each device a split of each batch to process. All the outputs are gathered and optimised on a single device (usually a CPU), then the weight updates returned backwards across all the devices. [Vahid Kazemi @vahidk has an excellent simple example with code here](https://github.com/vahidk/EffectiveTensorflow#multi_gpu). His ```make_parallel``` function can easily be extended to process multiple outputs:

```python
def make_parallel(fn, num_gpus, num_outputs, **kwargs):
    in_splits = {}
    for k, v in kwargs.items():
        in_splits[k] = tf.split(v, num_gpus)

    out_splits = [[]]*num_outputs
    for i in range(num_gpus):
        with tf.device(tf.DeviceSpec(device_type="GPU", device_index=i)):
            with tf.variable_scope(tf.get_variable_scope(), reuse=i > 0):
                out_ = fn(**{k : v[i] for k, v in in_splits.items()})
                for j in range(num_outputs):
                	out_splits[j].append(out_[j])

    # tf.stack may be needed instead of tf.concat depending on the shape of some of your variables.
    return [tf.concat(x, axis=0) for x in out_splits]

# Model returning several outputs
def model(a, b):
    return a + b, a*b, a/b

added, multiplied, divided = make_parallel(model, num_gpus, 3, a=a, b=b)
```

**Note 1:** If your dataset size is not evenly divisible by the batch size, then the remainder must be divisible by the number of devices or ```tf.split()``` will throw an error. A quick example:

```python
dataset_size = 100
num_gpus = 4

# How much data we want to process per device, per batch
device_batch_size = 8
# Actual combined batch size for all our resources
# 32
batch_size =  num_gpus * device_batch_size
```

This will throw a ```tf.split``` error, because ```dataset_size``` mod ```device_batch_size``` is NOT divisible by ```num_gpus```; i.e., 105 mod (8\*4) = 9, which cannot be evenly split among the four gpus. Here, 6 or 10 would both be acceptable for ```device_batch_size```.


**Note 2:** If you decide to run your restore a model trained on multiple devices to a single CPU, there is no problem; however, do not remove any code. Even if not used, the restored model requires the ```tf.split``` and ```tf.concat``` graph elements to run.

**Note 3:** In the loss function, we need to set ```colocate_gradients_with_ops=True``` to ensure the gradients get passed back to the same device the op was run on.

#### A Concrete Example - Small Convolutional Neural Network with Data Parallelism

```python
# Finding how many devices are available
gpus = [x.name for x in device_lib.list_local_devices() if x.device_type == 'GPU']
num_gpus = len(gpus)

def model(x_image_, y_true_):
    # Building the model architecture - a single convolutional layers and a fully connected layer.
    model, _ = cn.new_conv_layer(input=x_image_,
                                  num_input_channels=num_channels,
                                  filter_size=filter_size,
                                  num_filters=num_filters,
                                  use_pooling=True,
                                  max_pool_size=max_pool_size,
                                  use_relu=True)
    model, num_fc_features = cn.flatten_layer(model)
    model, _ = cn.new_fc_layer(input=model,          
                             num_inputs=num_fc_features,
                             num_outputs=fc_size,
                             use_relu=True)
    model = tf.nn.dropout(model, keep_prob)
    model, _ = cn.new_fc_layer(input=model,
                             num_inputs=fc_size,
                             num_outputs=num_classes,
                             use_relu=True)

    y_pred = tf.nn.softmax(model)
    y_pred_cls = tf.argmax(model, dimension=1)
    cost = tf.reduce_mean(tf.losses.softmax_cross_entropy(logits=model, onehot_labels=y_true_, weights=weights_))

    # Outputs to be returned to CPU
    return y_pred, y_pred_cls, cost
    
if num_gpus > 0:
    y_pred, y_pred_cls, cost = make_parallel(model, num_gpus, 3, x_image_=x_image, y_true_=y_true)
else:
    # CPU-only version
    y_pred, y_pred_cls, cost, f_vector = model(x_image_=x_image, y_true_=y_true)

# Notice colocate_gradients_with_ops flag set to TRUE.
optimizer = tf.train.AdagradOptimizer(learning_rate=2e-4).minimize(cost, colocate_gradients_with_ops=True)

correct_prediction = tf.equal(y_pred_cls, y_true_cls)
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))

# Start the session logging placement.
session = tf.Session(config=tf.ConfigProto(allow_soft_placement=True, log_device_placement=True))

# During training, run optimiser as usual. Each batch will be split up across as many devices were detected.
feed_dict_train = {x: x_batch, y_true: y_true_batch, keep_prob: 0.85}
session.run(optimizer, feed_dict=feed_dict_train)

```

#### Distributed TensorFlow

While not a subject of this post, Distributed TensorFlow definitely merits a mention; it requires more low-level knowledge of TensorFlow, but TensorFlow provides a [guide](https://www.tensorflow.org/deploy/distributed) with examples. In my limited experience, this method is more complicated than the parallelism mentioned above, as setting up the clusters and servers must be done manually, and deciding how to allocate tasks requires more thought. If you do have a large number of resources available, a master-worker setup will maximise resource usage.


# 2. Sub-divide your data.

Python does not clear variables until the end of the script, so make sure to re-use variables such as references to dataset objects to free up memory. A post is coming soon about **quickly loading image data in batches using Lightning Memory-Mapped Databases (LMDB).**


# 3. Save intermediate results to disk.

This sounds unnecessarily slow, but when memory is tight, every little bit helps! Anything that only needs to be processed once and then used later can be saved and loaded as needed. This frees up memory during all the intermediate operations. Intermediate values such as feature vectors, meta data, etc, can be quickly saved to a CSV, HDF5 file, or just dumped using ```pickle``` or ```json``` or the like.









