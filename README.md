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

## Example

Take this ordinary JavaScript function:

```javascript
var currentTemperatureFahrenheit = function () {
  return currentTemperatureCelsius() * 9/5 + 32;
};

```

We can call it for its value (assuming there's a `currentTemperatureCelsius` function):

```
> currentTemperatureFahrenheit()
71.8
```

But, if the `currentTemperatureCelsius` function is Tracker-aware (or even if it's not, but as long it reads the current temperature ultimately from some Tracker-aware data source), then we can also call `currentTemperatureFahrenheit` _reactively_:

```
> var handle = Tracker.autorun(function () {
  console.log("The current temperature is", currentTemperatureFahrenheit(),
              "F");
  });
The current temperature is 71.8 F       (printed immediately)
The current temperature is 71.9 F       (printed a few minutes later)
> handle.stop();                        (stop temperature changes from printing)

```

The function passed to `Tracker.autorun` is called once immediately, and then it's called again whenever there are any changes to any of the _reactive data sources_ that it referenced. To make this work, `currentTemperatureCelsius` just needs to register with Tracker as a reactive data source when it's called, which takes only a few lines of code.

Or, instead of calling `Tracker.autorun` ourselves, we might use `currentTemperatureFahrenheit` in a [Blaze](https://www.meteor.com/blaze) template:

```handlebars
<!-- In demo.html -->
<template name="demo">
  The current temperature is {{currentTemp}} degrees Fahrenheit.
  {{#if belowFreezing}}
    That's below freezing!
  {{/if}}
</templates>
```

```javascript
// In demo.js
Template.demo.helpers({
  currentTemp: function () {
    return currentTemperatureFahrenheit();
  },
  belowFreezing: function () {
    return currentTemperatureFahrenheit() < 32.0;
  }
});
```

When this template is shown, the temperature shown on the screen will update live as the weather changes. The "below freezing" message will appear when the temperature dips below freezing, and disappear when it rises above freezing, all without having to write any additiona logic. Blaze makes this work simply by calling `Tracker.autorun` around its calls out to our helper functions.


What does it look like on the other side, for package authors that are creating new reactive data sources? Here's what the implementation of `currentTemperatureCelsius` might look like (supposing you had an object `Thermometer`, with methods `read` and `onChange`):

```javascript
var temperatureDep = new Tracker.Dependency;

var currentTemperatureCelsius = function () {
  temperatureDep.depend();
  return Thermometer.read();
};

Thermometer.onChange(function () {
  temperatureDep.changed();
});
```

As you can see, it only takes a few lines of code to make `Thermometer` Tracker-aware so that it can participate in transparent reactivity.

It's easy for a library to detect if Tracker is available, and cooperate with it if so (by making the appropriate `depend` and `changed` calls, which have virtually no overhead except when called in a reactive context), and function normally if not. So adding Tracker support doesn't break compatibility with existing users of a library, and the support doesn't have any performance impact if it's not used. (The impact when it *is* used is small as well.)