# super-bufrd
UGens for accessing long buffers with subsample accuracy:
- **SuperBufRd** A modification of BufRd to be able to access samples with double precision
- **SuperPhasor** A new phasor UGen to drive a SuperBufRd
- **SuperPhasorX** A new phasor UGen to drive multiple SuperBufRds with crossfading to allow for click-free looping and position jumping
- **SuperPair** A sclang class to communicate position information with these UGens
- **SuperPlayBuf** A pseudo-ugen wrapper around a SuperPhasor and SuperBufRd, similar to PlayBuf
- **SuperPlayBufDetails** Same thing but outputs the phasor information as well as audio signal
- **SuperPlayBufX** / **SuperPlayBufXDetails** Like SuperPlayBuf/SuperPlayBufDetails but offers crossfading at loop points and on jumping to a new position
- **SuperBufFrames** A modification of BufFrames

[Full spec / documentation of classes here](https://gist.github.com/esluyter/53597bed464d16fdb603c9db8405e3a9)

This implementation is a work in progress. If you find bugs or have feature requests please submit an issue!

## Simple example usage of SuperPlayBufX pseudo-ugen
```supercollider
// sound file up to 2139095040 samples long (i.e. up to 12 hours long at 48k)
~buf = Buffer.read(s, "path/to/long/soundfile.wav", action: { "OK, loaded!".postln });

// wait for buffer to load then play it on a loop, mouse controls speed from 5x forward to 5x in reverse
x = {
  SuperPlayBufX.ar(1, ~buf, MouseX.kr(-5, 5), \trig.tr(0), start:\pos.skr(0).poll)
}.play;

// jump to a certain position in seconds:
x.set(\pos, ~buf.atSec(230.704), \trig, 1);
```

## More elaborate example with playhead
```supercollider
~buf = Buffer.read(s, "path/to/long/soundfile.wav", action: { "OK, loaded!".postln });

(
~synth = {
    var pos = \pos.skr(0);
    var trig = \trig.tr(0);
    var rate = MouseX.kr(-12, 12);
    var lpf_freq = rate.abs.linlin(1, 3, 20000, 5000); // make fast forward less grating on ears
    var sig, playhead, isPlaying;
    #sig, playhead, isPlaying = SuperPlayBufX.arDetails(2, ~buf, rate, trig, start: pos);
    SendReply.ar(Impulse.ar(10), '/playhead', playhead.components); // send both components of playhead
    LPF.ar(sig, lpf_freq);
}.play;

// print the elapsed time:
OSCdef(\playhead, { |msg|
    ("Playhead: %       BufDur: %".format(
        ~buf.atPair(*msg[3..4]).asTimeString,
        ~buf.duration.asTimeString)
    ).postln;
}, '/playhead');
)

~synth.set(\pos, ~buf.atSec(60 * 10), \trig, 1) // 10 minutes in
~synth.set(\pos, ~buf.atSec(60 * 40), \trig, 1) // 40 minutes in
~synth.set(\pos, ~buf.atSec(60 * 60), \trig, 1) // 1 hour in
```

## Mac / Linux build instructions
- On a Mac, make sure you have Xcode command-line tools installed
- Make sure you have a copy of the SuperCollider source code

Run the following commands in a terminal:
```
git clone https://github.com/esluyter/super-bufrd.git
cd super-bufrd
mkdir build
cd build
cmake -DSC_PATH=/path/to/sc3source/ ..
cmake -DCMAKE_BUILD_TYPE=RELEASE ..
make
```
Move the super-bufrd folder into your `Platform.userExtensionDir` and recompile sclang.

## Windows build instructions
I'm not a Windows expert, but these are the steps that worked for me on a 64-bit version of Windows 10. If you have a 32-bit version, omit the "Win64" from the first cmake line below.

- Install git
- Install cmake
- Install visual studio community 2017 / Desktop development with C++
- Make sure you have a copy of the SuperCollider source code

Run the following commands in git bash:
```
git clone https://github.com/esluyter/super-bufrd
cd super-bufrd
mkdir build
cd build
cmake -G "Visual Studio 15 2017 Win64" -DSC_PATH=/path/to/sc3source ..
cmake --build . --config Release
```
Move the super-bufrd folder into your `Platform.userExtensionDir` and recompile sclang.
