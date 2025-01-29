# A2OVC
Arduino-to-OBS (Open Broadcaster Software) Volume Controller, or A2OVC is kind of a short project, that is supposed to be modular at it's root, and simple to get running.
Whoever comes across this repo, know, that I have little to basic knowledge in both programming languages. And this wasn't exactly basic.

## How it works
It can be broken into 3 separate parts: Arduino, Rust, and OBS.

### Arduino side

There are 4 things in Arduino code that should be changed: <br /><br />
`MAX_PINS` (the amount things your arduino will read),<br />
`int pins[MAX_PINS] = {}` (which are what pins are being read), <br />
`JITTER_TRESH` (which is used to ignore small changes, but has it's issues),<br />
and `d` (delay on how frequently to call this function).<br />
<br />
`MAX_PINS` should always be equal to how many pins you define in `int pins[MAX_PINS]`.

The rest essentially all it does is "initialize pins as Input, read input, compare to prior, change is more that +-`THRES`, output in serial".<br />
The problem with `TRESH` is that if set too high or low, you lose precision.<br />
Another issue is `d`. You pull too fast, the voltages become unstable, and give unreliable results.


### Rust side
This is the nastiest side, because Rust is like sticking my feet into tar -- I have no real experience with it, or anything of sorts.<br />
The gist of it is, we use obws library to initialize a connection to WebSocket, and control it through that library. We also open a connection to serial port (check whichever port it uses through Device Manager).<br />
Now, `read_serial` function takes in `port`, `baudrate` and `obs_client` (the WebSocket we opened).<br />
We do some checking and long story short, we read the serial. Then we normalize those values to the range of 0 to 1, and multiply by -100 (which is the minimum dB), which effectively gets us the whole range of OBS volume.<br />
Now we got an array of elements, ready to set volumes on an input.<br /><br />

Since this is compiled, making any changes to code won't be a thing, so the easiest thing to do was to write the names of sources you want to change into a file, then make it read it. That's where the file `audio-sources` comes into play. Each source is on it's own line. Rust will check if the source exists, and will get the UID to it (which it will change). An oversight is probably re-reading the file over and over again, and should just be moved to somewhere during app launch.

Now we essentially loop over list, and set volume to each, which is found.

### OBS side
For some reason, OBS fails if I have a password set on WebSocket, but you can play around with it yourself.

## In retrospect
A lot of things could have been changed, a lot of things could have been better, but my inability, and inexperience is the short-coming here.<br />
And for most of all, this was just done as a proof of concept, since I noticed Deej doing something similar (just saw a quick glimpse of the youtube video).
