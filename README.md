[![Build Status](https://travis-ci.org/aullman/opentok-camera-filters.svg?branch=master)](https://travis-ci.org/aullman/opentok-camera-filters)

# opentok-camera-filters
Library which lets you add visual filters to your OpenTok Publisher.

![opentok-camera-filters collage](https://github.com/aullman/opentok-camera-filters/raw/master/images/Collage.png)

# [Demo](https://aullman.github.io/opentok-camera-filters/)

# [Blog Post](http://www.tokbox.com/blog/camera-filters-in-opentok-for-web/)

# Browser Support

* Chrome 51+
* Firefox 49+

These filters require the Canvas [captureStream API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/captureStream) which works in Chrome 51+ and Firefox 43+. Unfortunately adding audio to the stream doesn't work until Firefox 49+. This is why currently [the tests](https://travis-ci.org/aullman/opentok-camera-filters.svg?branch=master) only run in Firefox nightly.

# Usage

You can view the [source code for the demo](https://github.com/aullman/opentok-camera-filters/blob/gh-pages/src/demo.js) for an example of how to use this library.

**Note:** Make sure you include the opentok-camera-filters code before you include opentok.js.

Include the filters and then initialise with the filter you want to use.

```javascript
const filters = require('opentok-camera-filters/src/filters.js');
const filter = require('opentok-camera-filters')(filters.none);
```

Then when you have a Publisher you need to set it, eg.

```javascript
const publisher = session.publish();
filter.setPublisher(publisher);
```

If you want to change the filter you can use the change method, eg.

```javascript
filter.change(filters.red);
```

If you're using the face filter you will need to setup the web worker. The worker expects a file at `'./js/faceWorker.bundle.js'`. The root of that JS file is [src/faceWorker.js](/src/faceWorker.js). So you need to point WebPack or Browserify at that file and put the output in the /js directory of your webserver.

# Available Filters

A lot of the filters were taken from [tracking.js](https://trackingjs.com).

## red
Give the video a red tint

![red](https://github.com/aullman/opentok-camera-filters/raw/master/images/red.png)

## green
Give the video a green tint

![green](https://github.com/aullman/opentok-camera-filters/raw/master/images/green.png)

## blue
Give the video a blue tint

![blue](https://github.com/aullman/opentok-camera-filters/raw/master/images/blue.png)

## grayscale
Converts a colour from a colorspace based on an RGB color model to a grayscale representation of its luminance.

![grayscale](https://github.com/aullman/opentok-camera-filters/raw/master/images/grayscale.png)

## blur
A Gaussian blur (also known as Gaussian smoothing) is the result of blurring an image by a Gaussian function.

![blur](https://github.com/aullman/opentok-camera-filters/raw/master/images/blur.png)

## sketch
Computes the vertical and horizontal gradients of the image and combines the computed images to find edges in the image.

![sketch](https://github.com/aullman/opentok-camera-filters/raw/master/images/sketch.png)

## invert
Inverts the colour in every pixel of the video.

![invert](https://github.com/aullman/opentok-camera-filters/raw/master/images/invert.png)

## face
Does face detection using [tracking.js](https://trackingjs.com) and draws an image on top of the face.

![face](https://github.com/aullman/opentok-camera-filters/raw/master/images/face.png)

# Custom Filters

If you want to create your own custom filter you just need to create a function that looks like one of the functions in the [filters.js](src/filters.js) file. These functions accept a videoElement and a canvas parameter and they take the data out of the videoElement which is rendering the unfiltered video from the camera and they draw it onto the canvas after applying a filter. It should return an object with a stop method which when called will stop the filter from processing. For example creating a simple filter which draws a new random colour every second would look something like:

```javascript
const randomColour = () => {
  return Math.round(Math.random() * 255);
};

filter.change((videoElement, canvas) => {
  const interval = setInterval(() => {
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = `rgb(${randomColour()}, ${randomColour()}, ${randomColour()})`;
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }, 1000);
  return {
    stop: () => {
      clearInterval(interval);
    }
  };
});
```

You can also use the [filterTask](src/filterTask.js) which handles transforming image data from the videoElement and just lets you pass it a filter function which takes ImageData and transforms it returning new ImageData. The [invert function](https://github.com/aullman/opentok-camera-filters/blob/a845d2f4eec8a8a6bea86c3a785ef089656d861f/src/filters.js#L92) is a good example of a simple filter which uses this. 
