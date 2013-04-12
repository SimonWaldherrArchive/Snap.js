# Snap.js
A Library for creating beautiful mobile shelfs in Javascript

<a href="http://www.screenr.com/embed/0EJ7" target="_blank">View Video Preview</a>

<a href="http://jakiestfu.github.io/Snap.js/" target="_blank">View Demo</a>

<a href="http://www.screenr.com/embed/0EJ7" target="_blank"><img src="http://i.imgur.com/t3mGcEx.gif"></a>

[Tap here](http://jakiestfu.github.io/Snap.js/demo/apps/ratchet/template.html) or scan the QR-Code for the mobile demo

<img src="https://f.cloud.github.com/assets/198920/347680/5e8461a8-9eea-11e2-9dba-aa464825e033.png" height="250px" width="250px">

## Whats a Shelf?
This UX idea has been called many things. Shelf, drawer, droor, slider, backpane, side-menu, but I like to call it a shelf.

The idea is a user (on a mobile app, mostly iOS) may slide the actual app to the left or right to reveal a menu that was hidden underneath the app. Snap.js makes it a breeze to interact with your app pane element with dragging and flicking, making it easy for you to recreate what has been popularized by Facebook and Path.

## Support
Snap.js has been tested on the following devices:

* iPhone 5 (Mobile Safari/Chrome)
* iPhone 4S  (Mobile Safari/Chrome)
* Galaxy S3

## Features
* Library Independent
* High Customization
* Flick Support
* User Intent Detection
* Event Hooks
* CSS3 Powered Animations
* Drag Support
* Programatic API
* "No-Drag" Elements
* Definable Easing Mode
* Supports Ratchet (with templates!)

## Usage

```javascript
var snapper = new Snap({
  element: document.getElementById('content')
});
```

## Settings and Defaults
```javascript
settings = {
    element: null,
    resistance: 0.5,
    flickThreshold: 50,
    transitionSpeed: 0.3,
    easing: 'ease',
    maxPosition: 266,
    minPosition: -266,
    tapToClose: true,
    minDragDistance: 5,
    slideIntent: 40
}
```

* **`element`**: The element which the user will be sliding side to side
* **`resistance`**: The cooeficcient used to slow sliding when user has passed max or min threshold
* **`flickThreshold`**: Number of pixels the user needs to swiftly travel to activate a "flick" open
* **`transitionSpeed`**: The speed at which the pane slides open or closed
* **`easing`**: The CSS3 Easing method you want to use for transitions
* **`maxPosition`**: Maximum number of pixels the pane may be slid to the right
* **`minPosition`**: Maximum number of pixels the pane may be slid to the left
* **`tapToClose`**: If true, tapping an open pane will close it
* `minDragDistance`: The minimum amount of pixels the user needs to drag within the `slideIntent` degrees to move the pane 
* **`slideIntent`**: The number of degrees the user must initiate sliding in towards the left or right (see diagram below)

Notes on Slide Intent: The slide intent is an int between 0 and 90, and represents the degrees in the first quadrant of a circle that you would like to have mirrored on the X *and* Y axis. For example, if you have 40 set as your `slideIntent` value, the user would only be able to slide the pane by dragging in the blue area in the diagram below. Once intent has been defined, it will not change until the user releases.

<img src="http://i.imgur.com/uG2CNR8.png">


## Public Methods

### `open`: Opens the pane to the specified side

```javascript
snapper.open('left');
// OR
snapper.open('right');
```

### `close`: Closes the pane

```javascript
snapper.close();
```

### `on`: Adds an event hook

```javascript
snapper.on('start', function(){
  // Do Something
});
```
The available methods to hook into are as follows:

* `start`: Fired when touching down on the draggable pane and it begins to move
* `drag`: Fired when the pane has been moved or slid
* `stop`: Fired when the pane has been let go od
* `animating`: Fired when the pane is animating
* `animated`: Fired when the pane is finished it's animations
* `ignore`: Fired when trying to drag the pane but ended up dragging on an ignored element

### `off`: Removes an event hook

```javascript
snapper.off('drag');
```
The event names listed above apply for the `off` method.


### `state`: Returns detailed information about the state of the pane

```javascript
var data = snapper.state();
```
The data returned from the `state` method will look like the following:

```javascript
{
    state: "closed", // State of the Pane
    info:{
        opening: "left", // Side which user intends to open
        towards: "right", // Direction user is dragging towards
        hyperExtending: false, // True if user is pulling past predefined bounds
        halfway: false, // True if pane is at least halfway open
        flick: false, // True if user has moved pane X amount of pixels in the open/close direction without changing directions
        translation:{
            absolute: 20, // Pixels pane has translated
            relative: 21, // Pixels pane has translated relative to starting translation
            sinceDirectionChange: 10, // Pixels pane has translated since the direction of the pane has changed
            percentage: 40.571649 // The percentage that the Pane is open. Good or animating other things
        }
    }
} 
```

## Gotchas

### Layout
The layout itself is what most people will have a hard time emulating, so the simplest approach I have found is as follows:

Two absolute elements, one to represent *all* the content, and another to represent *all* the drawers. The content has a higher z-index than the drawers. Within the drawers element, it's direct children should represent the containers for the drawers, these should be `fixed` or `absolute`. Assigning classes to your drawers to specify which side it is on is recommended. All absolutely positioned elements should have 0 for `top, left, right, bottom` properties, excluding your panes which will have `auto` set to their respective sides and a width assigned. The width of your drawers is usually the same number you want to use for `minPosition` and `maxPosition`

```html
div.drawers {position: absolute;}
    div.left-drawer  {position: absolute;}
        [content]
    div.right-drawer  {position: absolute;}
        [content]
div#content {position: absolute;}
    [top-bars]
    [content] {overflow: auto}
    [bottom-bars]
```

A sample layout is found in demo/apps/default.html.

### Independent Scrolling
Some CSS is required to get some smooth ass scrolling. Utilize the CSS below to apply this to any of your elements:
```css
.scrollable{
	overflow: auto;
	-webkit-transition-property: top, bottom;
	transition-property: top, bottom;
	-webkit-transition-duration: .2s, .2s;
	transition-duration: .2s, .2s;
	-webkit-transition-timing-function: linear, linear;
	transition-timing-function: linear, linear;
	-webkit-overflow-scrolling: touch;
}
```

### Z-Indexes and Display
Because of the nature of this code, drawers are just kind of stacked behind the content. To bring the proper drawer to the fron, you can hook into Snaps event system:

```javascript
var UpdateDrawers = function(){
  var state = snapper.state(),
		towards = state.info.towards,
		opening = state.info.opening;
	if(opening=='right' && towards=='left'){
		// Revealing Right Drawer, apply CSS to that droor to bring it to the front.
    // Usually, display:block works if you set all panes to display:none;
	} else if(opening=='left' && towards=='right') {
		// Revealing left Drawer
	}
};

// Bind Events
snapper.on('drag', UpdateDrawers);
snapper.on('animating', UpdateDrawers);
snapper.on('animated', UpdateDrawers);
```

An example of this code in action can be found in demo/apps/ratchet/template.html

## Modularity

You may use Require.js to include this script. Exports are supported, too

```javascript
var Snap = require('snap'),
    snapper = new Snap();
```

## License

Copyright (c) 2013 Jacob Kelley

MIT License

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
