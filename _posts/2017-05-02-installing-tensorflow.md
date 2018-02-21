---
layout: post
comments: true
title:  "Installing TensorFlow: fixing easy-install.pth and setuptools issues"
date:   2017-05-02 21:18:00 +0100
categories: programming
tags:
- tensorflow
---

I recently installed TensorFlow on my Mac OS laptop and ran into some small issues. Maybe this will be useful to someone in the future, so here's how they were resolved.

The straightforward ```sudo pip3 install --upgrade tensorflow``` threw errors, so I took TensorFlow's suggestion to try installing directly from the Python package URL (I am installing the CPU only version).

<!--excerpt-->

```
sudo pip3 install --upgrade https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.1.0-py3-none-any.whl
```


This threw the following two errors&#58;

```
PermissionError: [Errno 13] Permission denied: '/Users/ysbecca/anaconda/lib/python3.5/site-packages/easy-install.pth'

FileNotFoundError: [Errno 2] No such file or directory: '/Users/ysbecca/anaconda/lib/python3.5/site-packages/setuptools-33.1.1-py3.5.egg'
```

I fixed the first issue by navigating into the site packages folder and changing the permissions&#58;

```
sudo chown ysbecca easy-install.pth 
chmod +x easy-install.pth
curl https://bootstrap.pypa.io/ez_setup.py | python
```

Then I looked for the missing .egg file, and found only&#58;

```
-rw-r--r--    2 ysbecca  staff      30 15 Sep  2016 setuptools.pth
```

I uninstalled and then re-installed the proper setuptools package as follows&#58;

```
sudo pip install setuptools==33.1.1
```

Still missing the .egg file, but it looked promising&#58;

```
drwxr-xr-x   37 root     staff    1258  2 May 16:43 setuptools
drwxr-xr-x   12 root     staff     408  2 May 16:43 setuptools-33.1.1.dist-info
-rw-r--r--    1 ysbecca  staff  702770  2 May 16:41 setuptools-33.1.1.zip
-rw-r--r--    2 ysbecca  staff      30  2 May 16:41 setuptools.pth
```

Now the initial ```sudo pip3 install --upgrade tensorflow```works.


