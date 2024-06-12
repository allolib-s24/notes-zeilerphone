# Sound Design:
To replicate the classic “acid bass” sound, I needed a reference point so I could determine which components were necessary to create it. I found [this video](https://www.youtube.com/watch?v=M3FdELMpDIA), and decided to try to replicate it in Allolib, and more generally replicate Ableton’s August Synthesizer 1.1 by Katsuhiro Chiba. 

[August Synthesizer 1.1 by Katsuhiro Chiba](https://cdn-resources.ableton.com/resources/filer_thumbnails/public/2012/01/09/katsuhiro-chiba-classic-synths-august.png__556x176_q85_crop_subsampling-2_upscale.jpg)

This synthesizer is made up of a few components:
OSC1: a square wave oscillator, I used Gamma’s built in `gam::square<>` struct. 
OSC2: a secondary oscillator that can either be sawtooth or noise, but is unnecessary for the acid bass sound so I left it unimplemented.
FILTER: a low-pass filter with a toggle between 12 and 24 dB/octave cutoff slope. Again, I only needed to use the 12dB slope, but this could very easily be changed to 24 by chaining two together. Luckily, Gamma’s built-in biquadratic filter, `gam::biquad<>`, is a second order infinite impulse response filter with a 12dB/octave cutoff. This was perfect for my needs. 
FILTER EG: an envelope for the filter’s resonant frequency. This envelope changes the cutoff frequency of the low-pass filter over time. This envelope is what allows the timbre, or harmonic content, of the synthesized output to change. I used Gamma’s Attack Delay Sustain Release envelope, `gam::ADSR<>`, to match the knobs on the Ableton synthesizer.
AMP: a simple amplitude control. The August Synthesizer has an additional setting for a chorus effect, but it was unnecessary to create an acid bass sound so has been left unimplemented. 
AMP EG: an envelope for the amplitude. This allows for the amplitude of the output to change over time. This also used `gam::ADSR<>` to match the video.
The LFO and MIXER controls were unnecessary for this demonstration so they were not implemented. 

The method of synthesis used for this sound is some mix between simple additive synthesis and subtractive synthesis. Unlike the subtractive synthesis example in the Allolib playground, which uses a band-pass filter with envelope controlled cutoff frequency and bandwidth and a noise generator, this synth applies a low-pass filter to a square wave generator. 

The characteristic ‘squelchy’ sound of the acid bass is due to the resonance of the filter. This diagram shows the frequency response of lowpass filters with resonance. In signal processing/electrical engineering terms this is usually called the Q factor. 

[Low-pass filter with various Q factors](https://qtxasset.com/files/sensorsmag/nodes/2009/5861/Figure3_big.jpg)

Notice that with higher Q factor, meaning higher resonance, there is greater than unity gain for frequencies around the cutoff frequency. This means that when the cutoff frequency is modified by an envelope, it actually amplifies different frequencies along the way. The “acid” sound of an acid bass synth comes from the cutoff frequency being modulated over time and at that frequency the various harmonics of the bass oscillator being amplified.

This is also why there is sometimes a clicking sound during the attack of a note. Because the envelope for this synth has a short attack, it applies the amplification due to resonance to a wide range of frequencies in a very short time. When we hear what is essentially a wide-frequency range amplitude burst, we perceive it as a click. 

# Spectrogram (Final Project):
For my final project for CMPSC 190D, I wanted to use the tools in Allolib to create a real time frequency spectrogram display. 
A spectrogram is a standard audio display that shows the frequency components of a signal over time. A spectrogram has three dimensions of data–time, frequency, and intensity–so there are different ways each of these are represented. I chose to use the y-axis for frequency, x-axis for time, and color to represent intensity. 

Gamma is a submodule for Allolib. It includes a built-in Short-Time Fourier Transform, called `gam::STFT`, that uses a sliding window approach. It takes a steady flow of samples and in regular intervals uses them to calculate the Fourier transform. 

I wrote a header file that can be included in any Allolib project. It can be found at [this Github repository](https://github.com/zeilerphone/allolib_spectrogram/). This link also contains a guide on how to use it.

The Spectrogram is implemented using a couple custom classes. The wrapper class is `al::Spectrogram`, and it is the only thing a user should need to interact with. Internally, I created a `Grid` class sub-class `Strip`. 
Knowledge about the implementation of these functions is unnecessary for use of the spectrogram. The `Spectrogram` class is designed to user-friendly.

The `Strip` object inherits from `al::Mesh` and is a two vertex wide vertical triangle strip. It contains all of the color information for each spectrum slice. It also contains a protected member function that maps values between 0 and 1 to various colors. This values are pulled directly from MatLab's "jet" color scheme.

The `Grid` object contains a vector of these `Strip` objects and functions to manipulate them. There are two main functions:
- `void write_data(std::vector<float> vals)`: this function takes a float vector and passes its values to the `Strip` object in `Grid`'s internal vector. The index is chosen based on a `writePointer` integer and allows the `Strip` vector to act as a circular buffer.
- `Strip* read_data(int index)`: this function returns a pointer to the `Strip` that is `index` places after `writePointer`. 




