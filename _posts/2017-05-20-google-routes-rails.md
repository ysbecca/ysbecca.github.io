---
layout: post
comments: true
title:  "Drawing and saving overlays on Google maps with Google Maps API in a Rails application"
date:   2017-05-20 21:18:00 +0100
categories: programming
---

Google Maps has a well-documented API for customising maps with your own imagery and content. For a side project built with Ruby on Rails, I wanted to **allow users to freehand highlight routes on a map and save them for future viewing and modifying.** The best way seemed to be to use [complex polylines](https://developers.google.com/maps/documentation/javascript/examples/polyline-complex) and save the lines as an overlay paired with the map window information. Here is a straightforward way of doing it cleanly within the Rails framework.

<!--excerpt-->

The premise of the project is allowing people (whether Post employees, menu/flyer distributors, etc.) doing door-to-door distribution to track where they've distributed, or to plot out where they need to distribute, on a Google map. While the project is currently live, it is still in progress and being tested, so I won't share the link at this time.

![photo]({{site.baseurl}}/assets/post-images/2017-05-20-a.png "Highlighting routes")

As pictured above, to draw on the map, users tap a starting point, which will be joined to any consecutive point they tap with a straight yellow line. To end a path, users tap twice, and then they can begin new paths as they like.

<h4>Setting up the view templates</h4>

JavaScript files within Rails are easily included as ```html.erb``` templates. First, I created a new partial template called ```_googlemap.html.erb``` within my views folder under the relevant model. For now, the template contains a single line:

```html
<div id="googlemap"></div>
```

My model saves both the routing/map information, as well as some additional fields. All the routes information will be stored in a hidden ```routes``` field. Next, in the ```_form.html.erb``` template for my model, I rendered the partial template where I wanted the map to appear.

```html
<!-- The generated form field for my model field called routes; hidden -->
<div class="form-group hidden-field" id="distribution-routes">
  <%= f.label :routes %>
  <%= f.text_field :routes %>
</div>

<div id="editing-field" class="hidden-field"><%= @editing %></div>
<div id="latitude-field" class="hidden-field"><%= @latitude %></div>
<div id="longitude-field" class="hidden-field"><%= @longitude %></div>

<%= render :partial => "map" %>
```

Using css, I hide the ```editing-field```, ```latitude-field```, and ```longitude-field``` divs. These values are passed from my controller to tell me whether the user is viewing or editing, and the latitude and longitude of the map to display. When the user is editing, I want to allow drawing and to display the drawing tools. While perhaps not the most elegant solution, putting these parameters in divs with unique ID's makes them easy to access from the JavaScript.

<h4>Drawing and viewing routes as overlays</h4>

The [Google Maps API](https://developers.google.com/maps/documentation/javascript/examples/polyline-complex) for complex polylines has good documentation with sample code to get you started. I registered my app, got a unique API key, then took their sample code as the base. Here is my code, with commentary.

```javascript
<script type="text/javascript">

	// When viewing a saved model, the routes in JSON format are in a hidden field.
	// I write all new routes to this field as well, so that Rails automatically takes care
	// of saving.
	var routesHandle = document.getElementById("distribution_routes");
	var routes = [];

	// All paths, for clearing map. Each non-connected route is a separate path.
	var allOverlays = [];

	// Determine whether we are editing the map or not.
	var editing = false;
	if(document.getElementById("editing-field").textContent == "true") {
		editing = true;
	}

	var map, drawingManager;

	function initMap() {

		// Load the default values from the hidden divs.
		var defaultLatitude = document.getElementById("latitude-field");
		var defaultLongitude = document.getElementById("longitude-field");

		if(defaultLongitude && defaultLatitude) {
			defaultLatitude = defaultLatitude.textContent;
			defaultLongitude = defaultLongitude.textContent;
		}

		var defaultZoom = 12;
		var defaultCenter;
		if(defaultLatitude && defaultLongitude) {
			defaultCenter = new google.maps.LatLng(defaultLatitude, defaultLongitude);
		} else {
			defaultCenter = new google.maps.LatLng(53.8008, -1.5491);
		}

		// Create map and add controls.
		var mapOptions = {
			center: defaultCenter,
			zoom: defaultZoom,
			mapTypeId: google.maps.MapTypeId.ROADMAP,
			scrollwheel: true,
			scaleControl: true
		};

		map = new google.maps.Map(document.getElementById('googlemap'), mapOptions);
		
		if(editing) {
			// Instantiate a new drawing manager and add the drawing controls.
			drawingManager = new google.maps.drawing.DrawingManager({
				drawingMode: google.maps.drawing.OverlayType.Polyline,
				drawingControl: true,
				drawingControlOptions: {
					position: google.maps.ControlPosition.TOP_CENTER,
					drawingModes: ['polyline'],
				},
				// For a highlighting effect, so chose a light yellow colour with 0.5 opacity.
				polylineOptions: {
					strokeColor: '#FFFF00',
					strokeOpacity: 0.5,
					strokeWeight: 10,
					clickable: false,
					editable: true,
					zIndex: 1
				}
			});

			drawingManager.setMap(map);
	
			// Add a listener for the completed polyline event, whenever a user ends a path.
	  	google.maps.event.addListener(drawingManager, 'polylinecomplete', addLine);
		}
	
		displayPaths();
	}

	// Handles the saving of a polyline once it is completed.
	function addLine(polyline) {
		allOverlays.push(polyline);

		var path = polyline.getPath().getArray();
		var new_path = [];

		// Construct the new array of paths.
		for (var i = path.length - 1; i >= 0; i--) {
			new_path.push([path[i].lat(), path[i].lng()]);
		}

		routes.push(new_path); // Array of arrays [[lat, lng], [lat2, lng2]... ]
		routesHandle.value = JSON.stringify(routes); // Convert it all to JSON!
	}

	// Fetches the paths and displays them.
	function displayPaths() {
    	if(routesHandle.value == "") {
    		return;
    	}

		routes = JSON.parse(routesHandle.value);
		
		// Now iterate through all the polylines and draw them on the map.
		for(var i = 0; i < routes.length; i++) {
			var newPath = [];
			for (var j = 0; j < routes[i].length; j++) {
				// Format of routes[i][j] [[lat, lng], [lat2, lng2]...]
				newPath.push({ lat: routes[i][j][0], lng: routes[i][j][1] });
			}
			// Draw saved paths on the map with the same settings as they were drawn.
			var poly = new google.maps.Polyline({
				path: newPath,
				strokeColor: '#FFFF00',
				strokeWeight: 10,
				strokeOpacity: 0.5,
				editable: false,
				clickable: false,
				zIndex: 5,
			});

			allOverlays.push(poly);
			poly.setMap(map);
		}
 	}

	// Removes the routes and clears the map.
	function clearMap() {
		routesHandle.value = "";
		routes = [];
		for (var i = allOverlays.length - 1; i >= 0; i--) {
			allOverlays[i].setMap(null);
		}

		allOverlays = [];
	}
	
</script>
```

<h4>Saving routes as JSON as a Rails model field</h4>

In the script above, all the routes are parsed and saved in JSON format. Rather than convert the JSON to a String in the controller before saving the models, I made the ```routes``` field in my schema of type JSON.
 
```ruby
create_table "distributions", force: :cascade do |t|
    t.jsonb    "routes"
```
I don't need to touch the controller or model; Rails automatically now saves my routes! As mentioned previously, while this may not be the most elegant solution, doing the parsing in the JavaScript allows for seamless integration with the rest of the Rails components. Side note: if you're wondering why I use **jsonb** instead of **json**, [this post describes the difference](http://nandovieira.com/using-postgresql-and-jsonb-with-ruby-on-rails).

<h4>Improvements</h4>

A few features I am currently working on are:

* sorting distributions based on map routes
* merging overlays from multiple maps and saving
* multiple highlighting colours
* opening the map in a whole new window for easier navigation (on mobile or tablet, scrolling and zooming can be tricky because the map is embedded in the page)

If I've missed something or parts are not clear, please let me know and I'll do my best to clarify!
