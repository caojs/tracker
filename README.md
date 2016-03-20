# Tracker

Tracker is an incredibly tiny (~1k) but incredibly powerful
library for **transparent reactive programming** in JavaScript. (It
is a fork of [Meteor Tracker](https://github.com/meteor/meteor/tree/devel/packages/tracker))

Tracker gives you much of the power of a full-blown
[Functional Reactive Programming](http://en.wikipedia.org/wiki/Functional_reactive_programming) (FRP) system without requiring you to rewrite your program as a FRP data flow graph. Combined with Tracker-aware libraries, this lets you build complex event-driven programs without writing a lot of boilerplate event-handling code.

Tracker is essentially a simple _convention_, or interface, that lets reactive data sources (like your database) talk to reactive data consumers (such as a live-updating HTML templating library) without the application code in between having to be involved. Since the convention is very simple, it is quick and easy for library authors to make their libraries Tracker-aware, so that they can participate in Tracker reactivity.

This README has a short introduction to Tracker. For a complete guide
to Tracker, consult the thorough and informative [Tracker
Manual](https://github.com/meteor/meteor/wiki/Tracker-Manual), which
is five times longer than the Tracker source code itself. You can also browse the API reference on the [main Meteor docs page](http://docs.meteor.com/#tracker).

## Installation

npm:
```
npm install --save tracker
```

## Example

Take this ordinary JavaScript function:

```javascript
function currentTemperatureFahrenheit () {
  return currentTemperatureCelsius() * 9/5 + 32;
}

```

We can call it for its value (assuming there's a `currentTemperatureCelsius` function):

```
> currentTemperatureFahrenheit()
71.8
```

But, if the `currentTemperatureCelsius` function is Tracker-aware (or even if it's not, but as long it reads the current temperature ultimately from some Tracker-aware data source), then we can also call `currentTemperatureFahrenheit` _reactively_.

```javascript
var Tracker = require('tracker');

// Reactive function
function currentTemperatureFahrenheit () {
  return currentTemperatureCelsius() * 9/5 + 32;
}

var handle = Tracker.autorun(function () {
  console.log('The current temperature is', currentTemperatureFahrenheit(), 'F.');
});

```
```
The current temperature is 113 F.
The current temperature is 75.2 F.
The current temperature is 39.2 F.
...                      

```

The function passed to `Tracker.autorun` is called once immediately, and then it's called again whenever there are any changes to any of the _reactive data sources_ that it referenced. To make this work, `currentTemperatureCelsius` just needs to register with Tracker as a reactive data source when it's called, which takes only a few lines of code.

```javascript
var Tracker = require('tracker');
var dep = new Tracker.Dependency();
var temperature = randomTemperature();

// create random temperature
function randomTemperature () {
  var min = 0;
  var max = 100;
  var range = max - min;
  var temperature = Math.ceil(Math.random() * range) + min;
  return temperature;
}

// Data source
function currentTemperatureCelsius () {
  dep.depend();
  return temperature;
}

// dep.changed() will cause Tracker-aware recompute.
setInterval(function () {
  temperature = randomTemperature();
  dep.changed();
  console.log('Temperature changed.');
}, 1000);

```

The function Tracker.autorun return a computation. And we can stop "Reactive function" recomputing everytime data source change by using stop function in computation object.

```javascript
handle.stop();

```

## Code

```javascript
var Tracker = require('tracker');
var dep = new Tracker.Dependency();

// create random temperature
function randomTemperature () {
  var min = 0;
  var max = 100;
  var range = max - min;
  return Math.ceil(Math.random() * range) + min;
}

var temperature = randomTemperature();

// Data source
function currentTemperatureCelsius () {
  dep.depend();
  return temperature;
}

// dep.changed() will cause Tracker-aware recompute.
setInterval(function () {
  temperature = randomTemperature();
  dep.changed();
  console.log('Temperature changed.');
}, 1000);

// Reactive function
function currentTemperatureFahrenheit () {
  return currentTemperatureCelsius() * 9/5 + 32;
}

var handle = Tracker.autorun(function () {
  console.log('The current temperature is', currentTemperatureFahrenheit(), 'F.');
});

// stop recomputing
setTimeout(function () {
  console.log('Stop recomputing.')
  handle.stop();
}, 3000);
```
```
The current temperature is 75.2 F.
Temperature changed.
The current temperature is 93.2 F.
Temperature changed.
The current temperature is 123.8 F.
Temperature changed.
Stop recomputing.
Temperature changed.
Temperature changed.
```
