# node-heartbeats

[![Build Status](https://travis-ci.org/arjunmehta/node-heartbeats.svg?branch=master)](https://travis-ci.org/arjunmehta/node-heartbeats)

A simple node.js module to very efficiently track whether things are keeping up at a certain pace.

This library uses a much more efficient (yet less precise) method of testing system level event times as relativistic time differentials vs. universal time differentials. Think larger chunked time measures (a heart rate) instead of actual milliseconds.

Basically, you use this library to compare multiple "Pulse" objects to a single consistent "Heartbeat", and see how many beats it has missed.

This library is perfect if you have a large number of events that require heartbeats to be efficiently compared to certain thresholds without needing to be 100% precise.


## Basic Usage

```bash
npm install heartbeats
```

```javascript
var heartbeats = require('heartbeats');

var heart = new heartbeats.Heart(1000);
var pulseA = heart.newPulse();
var pulseB = heart.newPulse();

console.log( pulseA.missedBeats() ); // 0
console.log( pulseB.missedBeats() ); // 0

setInterval(function(){

  pulseB.beat();

  console.log( pulseA.missedBeats() ); // 2, 4, 6, 8
  console.log( pulseB.missedBeats() ); // 0

}, 2000);
```


## About Efficiency

Why is this library faster than more conventional methods? Basically, instead of using `Date.now()` or` new Date().getTime()` which are relatively very slow operations that give you very discrete, universal values for the **present time**, you use the present moment of a heartbeat to give your events a time relative to that heartbeat. This simple change results in extremely fast and efficient time difference calculations because it operates at a very low resolution compared to methods using the Date object. View the source to see details.


## API

The API is fairly straightforward, though it's good to be aware of a few things, particularly around precision issues that might arise for you.

### The Heart
To create a new heartbeat you, obviously, need to create a new heart. A heart has a core pulse, and beats at a specified interval. Hearts can also spawn new "Pulses" which are used to catch a "beat" from the Heart.

#### Constructor: new heartbeats.Heart(heartrate)
Creates a new "Heart" that beats at a certain heartrate in milliseconds.
```javascript
// a new heart that beats every 2 seconds
var heart = new heartbeats.Heart(2000);
```

#### Managed: heartbeats.createHeart(heartrate, name);
You can have multiple Hearts which are kept in a list in the heartbeats module. This is great if you want to access heartbeats from different modules.
```javascript
// a new heart that beats every 2 seconds named "global"
heartbeats.createHeart(2000, "global");
```

#### Managed: heartbeats.heart(name)
Returns a Heart from the managed list of hearts.
```javascript
// gets a heart named "global"
var heart = heartbeats.heart("global");
```

#### Managed: heartbeats.destroyHeart(name)
Removes the Heart from the internal managed list and clears the heartbeat interval.
```javascript
// destroys the "global" heart(beat)
heartbeats.destroyHeart("global");
```

#### heart.destroy()
Clears the heartbeat interval and removes the Heart from the internal managed list if it exists.


### The Pulse

#### heart.newPulse();
Returns a new Pulse object associated with the heart.
```javascript
// creates a new pulse from the "global" heart(beat)
var pulse = heartbeats.heart("global").newPulse();
```


#### pulse.beat()
Synchronizes the pulse with its main heart.
```javascript
// synchronizes the pulse to its heart
pulse.beat();
```

#### pulse.missedBeats()
Returns the number of heartbeats that have passed since it was synchronized using pulse.beat().
```javascript
// gets the number of beats the pulse is off from its heart
var beatoffset = pulse.missedBeats();
```

#### pulse.lag();
Returns an approximate number of milliseconds the pulse is lagging behind the main heartbeat. Basically this is `pulse.missedBeats*heart.heartrate`.
```javascript
// gets an approximate number of milliseconds the pulse is delayed from the heart
var delay = pulse.lag();
```


## The MIT License (MIT)

Copyright (c) 2014 Arjun Mehta

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal n the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
