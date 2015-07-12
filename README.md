# beet.js
Polyrhythmic Sequencer library for Web Audio API.

The sequencer can have multiple layers where each layer has a different step count. This feature allows the user to create complex [polyrhythms](https://en.wikipedia.org/?title=Polyrhythm) and [euclidean rhythms](http://cgm.cs.mcgill.ca/~godfried/publications/banff.pdf) using a simple api.
### Installation
You can install `beet` from npm.
````
npm install beet.js
````
Or grab the latest version from [build folder](https://github.com/zya/beet.js/tree/master/build) and include it in your html.
````html
<script src='beet.min.js'></script>
````
### Example Usage
````js
var context = new AudioContext();

// initialize beet
var beet = new Beet({
  context: context,
  tempo: 100
});

// create a euclidean pattern - 5 pulses distrubted in 7 steps
var pattern = beet.pattern(5,7);

// create a beet layer - pass it the pattern and a callback
var layer = beet.layer(pattern, callback);

function callback(time){
  var osc = context.createOscillator();
  osc.connect(context.destination);
  osc.start(time);
  osc.stop(time + 0.1);
}

// add the layer
beet.add(layer);

// start the sequencer
beet.start();
````
## API

### Beet
#### `beet.start(time)`

Starts the sequencer after a given time in seconds.

#### Parameters
* `time` - a number in seconds - defaults to 0

#### `beet.stop(time)`

Stops the sequencer after a given time in seconds and resets the current step number.

#### Parameters
* `time` (optional) - a number in seconds - defaults to 0

#### `beet.pause(time)`

Pauses the sequencer after a given time in seconds but keeps the current step number.

#### Parameters
* `time` (optional) - a number in seconds - defaults to 0

#### `beet.add(layer)`

Adds a given `beet.layer` to the list of layers to play.

#### Parameters
* `layer` (Required) - a `beet.layer` object.

#### `beet.remove(layer)`

Removes a given `beet.layer` from the list of layers to play.

#### Parameters
* `layer` (Required) - a `beet.layer` object.

### Pattern
#### `beet.pattern(pulses, steps)`
Returns a `pattern` object containing a [bjorklund](https://github.com/zya/bjorklund) sequence with equally distributed number of `pulses` over `steps`.

#### Parameters
* `pulses` (required) - number of pulses (active steps) in the sequence
* `steps` (optional) - number of steps in the sequence - value of `pulses` will be used steps is not specified.

#### example
````js
var pattern = beet.pattern(3, 7);
// a pattern object containing a sequence as below
// '1010100'

var pattern2 = beet.pattern(4);
// a pattern object with a sequence as below
// '1111'
````

#### `.update(pulses, steps)`
Updates the pattern object with the new values. You can also update the values directly. See example below.

#### Parameters
* `pulses` (required) - number of pulses (active steps) in the sequence.
* `steps` (optional) - number of steps in the sequence. value of `pulses` will be used steps is not specified.

#### example
````js
var pattern = beet.pattern(2, 5);
pattern.update(5,8); // updates the pattern to '10100'

// you can also set the values separately
pattern.pulses = 4;
pattern.steps = 8;
````

#### `.shift(offset)`
Shifts the sequence by the offset and returns the pattern object.

#### Parameters
* `offset` (required) - number of steps to shift the sequence by

#### example
````js
var pattern = beet.pattern(1, 4); // returns '1000'
pattern.shift(1) // updates the sequence to '0100'
````

### Layer
#### `layer(pattern, onCallback, offCallback)`
Creates a `beet.layer` object and returns it.

#### Parameters
* `pattern` (required) - a `beet.pattern` object.
* `onCallback` (required) - a function to call on pattern's pulses e.g. 1's
* `offCallback` (optional) - a function to call on pattern's empty slots e.g. 0's.

#### example
````js
var pattern = beet.pattern(1, 4); // returns '1000'
var layer = beet.layer(pattern, on, off); 
// on will be called on 1
// off will be called on 0's
````
#### Callback format
The callbacks are functions that will be called from the scheduler.
The functions will be called with a `time` and a `step` parameter.
You can use the `time` parameter for web audio methods. `step` can be used to change audio behaviour according to the current step.

#### example
````js
function callback (time, step) {
    var osc = context.createOscillator();
    osc.connect(context.destination);
    if(step === 1){
      osc.frequency.value = 880;
    }
    osc.start(time);
    osc.stop(time + 0.2);
}
````

### Utils
Beet comes with a series of useful utilities for web audio api. These might me moved to a separate module in the future as these are not specific to beet's functionality. But for now, features will be added gradually.

#### `.envelope(audioparam, now, opts)`
Applies an envelope to the audio param given a series of options.
* `audioparam` (required) -  an [AudioParam](https://developer.mozilla.org/en-US/docs/Web/API/AudioParam) object to be automated.
* `now` (required) - the time for the automation to start, e.g. the time parameter in `beet` callback.
* `opts` (optional) - an JavaScript object that contains the envelope parameters.
  * `start` (optional) - the start value for the automation curve. the audio param will be set to this value at the beginning of the automation.
  * `peak` (optional) - the maximum value for the automation curve that the attack stage will end up in.
  * `attack` (optional) - the time in seconds for the attack stage.
  * `decay` (optional) - the time in seconds for the decay stage.
  * `sustain` (optional) - the value for that the decay stage will end up in.
  * `release` (optional) - the time in seconds for the release stage.
