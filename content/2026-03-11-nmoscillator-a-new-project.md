---
title: "The NMOScillator: A New Project!"
description: An overview of my latest project, the NMOScillator, which is an SN76489-based chiptune music player using 74 series logic ICs.
date: 2026-03-11T06:46:00+11:00
taxonomies:
  tags:
    - arduino
    - electronics
    - golang
    - low-level
    - music
    - nmoscillator
    - pcb

---

# An overview of my latest project, the NMOScillator.

It seems like every time I start to get into the weeds with a project, a new one springs up in its place and occupies my attention.
This time is no different.

Over the past few weeks, I've been fixating on a brand new project, which I've been calling the NMOScillator.
The basic idea is to drive an [SN76489 sound chip](https://en.wikipedia.org/wiki/Texas_Instruments_SN76489) using rudimentary 74 series logic ICs to produce chiptune music. Hence the name 'NMOScillator' - a portmanteau of 'NMOS', because the SN76489 uses NMOS technology, and 'oscillator'.
It's a very simple concept, which is why I found it so appealing. I've always had an attraction to creating complexity from simplicity, especially in electronic circuits.

Of course, I wanted to avoid writing a blog post about something that only existed as an idea. I originally ordered the sound chips back in September of last year, and I first wrote down plans for the NMOScillator in November. It's taken me until now to be ready to share my proof-of-concept.

The plan was to use a ROM (of sorts) to store song data, and then some logic could read that ROM data, parse out any special instructions it contained, and then pass the rest of the data on through to the SN76489. Thanks to the simple protocol used by the SN76489, piping commands from the ROM would be the easy part. The hard part would be the rest of the parsing logic.

*If you want to learn more about the SN76489, you can find some [useful links](#useful-links) at the bottom of this post.*

---

# Table of Contents

- [Part 1: The ROM format](#part-1-the-rom-format)
- [Part 2: The Logic Simulation](#part-2-the-logic-simulation)
- [Part 3: The Compiler](#part-3-the-compiler)
- [Part 4: The Breadboard Circuit](#part-4-the-breadboard-circuit)
- [Part 5: The Test Rig PCB](#part-5-the-test-rig-pcb)
- [Part 6: Music Showcase](#part-6-music-showcase)
- [Part 7: Next Steps](#part-7-next-steps)
- [Useful Links](#useful-links)

---

# Part 1: The ROM format

*(If I write something in **bold** in this section, that means it's a recurring term I came up with to describe the NMOScillator's behaviour.)*

One of the requirements I set for myself was to make the parsing logic smart enough to fit lengthy songs into the ROM. The most convenient way to do this was to store the songs in **frames**, similar to rows in a tracker. These frames would be played back at a given **Tempo**, which could be specified in the ROM data.

{{ img(src="2026-03-11-furnace-rows-1.png", alt="Rows of song data in Furnace tracker", caption="Frames in the ROM data are like rows in music trackers. (Pictured: Furnace tracker)", size="small", noprocess=true) }}

Each frame was designed to have one byte of header data, and then a stream of **commands** to be either interpreted by the NMOScillator or piped to the SN76489. The frame headers contain flags for song looping, and specify the number of commands each frame contains. Each command in a frame can either be an **SN76489 Command** (of which the byte is sent directly to the SN76489), a **Tempo Change** command, or a **Frame Delay** command (which delays the next frame by a multiple of the Tempo). The type of each command is specified by its **Command Index**, counting down from the number of commands present in the frame. Specific indexes are defined as specific command types, so you can infer what each byte means just by counting how many bytes you've already read in the frame.

That's a lot to take in, but if you're interested, I've written up a (mostly) complete description of the ROM format [on my GitHub](https://github.com/QEStudios/NMOScillatorCompiler/blob/master/ROM_FORMAT.md), along with some example frames. It'll probably change as I develop the project further.

---

# Part 2: The Logic Simulation

I decided that a good way to test the viability of this ROM format would be to build up a logic circuit for the NMOScillator in Logisim. That way, I could get an intuitive sense of what is/isn't possible with basic logical components.

This resulted in the following initial logic circuit:

{{ img(src="2026-03-11-logic-diagram-1.webp", alt="The logic circuit for the NMOScillator in Logisim", caption="The initial logic circuit designed in Logisim. (Click to open in new tab)", size="large") }}

{{ file_with_mirror(text="Download this circuit here (28.02 KB)", path="2026-03-11-logic-circuit-1.circ", mirror="https://mega.nz/file/ZvNxXQgb#fxqgNYK9E5f5GCnFPk4NYhTR_TDH6CqTrvVhbD1efc4") }}

The circuit consists of 5 counters, 2 registers, and some glue logic. It appeared to correctly play the basic ROM files I tested it with, and I felt that I had reached a good balance between capability and size with this design. There are some weird quirks of the circuit which may need looking into, such as the Address Counter being clocked at the same time the Command Index is loaded from ROM, which may lead to a race condition if the ROM manages to update its outputs *before* the Command Index is loaded. However, this may never be an issue, due to the inherent propagation delay the Address Counter and ROM have.

As of writing this post, I'm porting the design over to [Digital](https://github.com/hneemann/Digital) with simulations of the real logic chips I'll be using, for faster performance and a more accurate depiction of the logic.

I put together this video annotating a simple example ROM running in the Logisim simulator. You can pause to read the text explaining the different stages of playback.

{{ video(src="2026-03-11-logic-simulation-annotated-1.webm", poster="2026-03-11-logic-simulation-annotated-1.webp", alt="Annotated recording of the Logisim simulation", caption="Note that the highest bit of the Tempo Counter is set to 0 in this video, not 1, to make it run significantly faster for demonstration purposes.", youtube="https://youtube.com/watch?v=9cukEJ25jUs", size="large" type="video/webm") }}

---

# Part 3: The Compiler

Now that I had a working circuit, I needed some songs to test it with. I had planned on using [Furnace](https://tildearrow.org/furnace) to compose them, but I had no way of turning them into NMOScillator ROMs. So, I wrote my own compiler to do just that.

I decided to write the compiler in [the Go programming language](https://go.dev), partly because I wanted it to be faster than a Python script would be, and partly because I wanted to familiarise myself with the language. I was still very new to Go at the time, and I thought this would be a good opportunity to learn it.

After spending a little over a month developing the compiler, I was able to generate ROM files for any song I composed in Furnace. The compiler took such a long time to develop because of my desire to make it modular. It uses multiple internal representations for the song data in different parts of the program, meaning it should be easy to write additional parsers for other input filetypes.

To compile a Furnace song into a ROM file, the compiler follows this process:

1. The input text file is read, and parsed into equivalent internal structures. Unimportant / unused metadata is dropped at this stage.

2. The parsed text file structures are processed into a lower-level structure format used to describe NMOScillator songs. Here, note pitches are converted to period values, song speed is converted into Tempo values and Frame Delays, and any skipped notes due to jumping are removed. All song data is now stored as a series of frames containing commands.

3. Binary ROM data is generated from the processed song data. This is a relatively simple operation, as the NMOScillator song data structure is very similar to the raw ROM data it describes.

<!-- Use noprocess=true so transparency is preserved. The png is small anyway. -->
{{ img(src="2026-03-11-compiler-flow-chart-1.png", alt="Flow chart showing the internal stages of the NMOScillator compiler", caption="", size="medium") }}

This 3-step process keeps the program modular, meaning that, if I wanted to, I could add additional input file formats without much trouble. One interesting format might be MIDI files, though it would need significantly more processing to make it work with the NMOScillator's requirements.


The source code for the compiler is available [on GitHub here](https://github.com/QEStudios/NMOScillatorCompiler).

{{ video(src="2026-03-11-compiler.webm", poster="2026-03-11-compiler.webp", alt="Example of the compiler being used in a terminal", caption="Example compilation of a Furnace song.", youtube="https://youtube.com/watch?v=UV6K-tGLZv4", size="medium" type="video/webm" loop=true) }}

---

# Part 4: The Breadboard Circuit

As the compiler was working, and the logic simulation showed promise, I decided that the next step would be testing the real SN76489 chips. I had done only minimal prior testing to ensure they weren't DOA, so I was excited (but also a little nervous) to see how well they would work.

I put one of the sound chips on a breadboard, alongside an Arduino Nano. I had ordered the SN76489s in a pack of five, so I wasn't *too* concerned about breaking one through experimentation (which, thankfully, didn't happen). After reading the datasheet extensively, I connected the Arduino to the control and data inputs, and wrote some simple code in the Arduino IDE to drive the sound chip with some basic commands. Surprisingly, it worked first try, though I was only looking at waveforms on my oscilloscope. 

After that success, I spent some time writing a more complex program to interpret NMOScillator ROMs and play them on the real SN76489.

{{ img(src="2026-03-11-breadboard-1.webp", alt="NMOScillator test rig with the sound chip and Arduino Nano on a breadboard", caption="", size="large") }}
{{ img(src="2026-03-11-breadboard-top-1.webp", alt="Top-down view of the breadboard test rig", caption="", size="large") }}

*Unfortunately, during this time I wasn't focused on documenting my progress as much, so I don't have many photos of the breadboard test rig.*

It was around this point where I realised my need for an amplifier circuit to drive a speaker, as the sound chip can't supply enough current to drive one directly off the audio output pin. I decided on using an [LM386 Audio Power Amplifier IC](https://en.wikipedia.org/wiki/LM386), which is very standard and is designed exactly for this purpose. With this being my first time experimenting in audio electronics, I felt quite lost looking through datasheets. Some things felt almost like black magic, like seeing filter circuitry thrown in every place imaginable. I did my best to follow the circuit recommendations I found online, and managed to get the amplifier working well enough, though it was still noticeably noisy.

I was content with this for a while, and I ended up using the circuit to test a number of songs on real hardware. However, mostly due to using a cheap breadboard, the circuit was both noisy and unstable, which can make it extremely annoying to debug. Because of this, I decided to migrate the circuit onto a PCB, hoping that with proper routing I’d be able to negate most of the interference noise.

The Arduino code isn't very well-written so I don't feel like sharing it at this point. I might release the code if I ever work it into a state I'm happy with, but I don't plan on spending my time on that right now.

*(I'm planning to eventually invest in some high quality breadboards from [BusBoard](https://busboard.com/BB830), which would fix most of my issues with intermittent connections.)*

---

# Part 5: The Test Rig PCB

The circuit I was using on the breadboard wasn't particularly complex, so drawing the schematic for the PCB went relatively smoothly. I took extra care to make sure my amplifier circuitry was suitable, and did as much as I could to minimise noise; if I wasn't happy with something *now*, I wouldn't be happy with it on the actual NMOScillator either.

{{ img(src="2026-03-11-test-rig-schematic-1.webp", alt="Schematic drawing for the NMOScillator test rig PCB", size="medium") }}

Annoyingly, I had somehow managed to burn out the D4 pin on the Arduino Nano I was using, so I decided to design the PCB around it by using A0 instead. I didn't see this as much of a compromise, because the PCB was only designed to replace the noisy breadboard anyway.

I designed the PCB to have a small speaker mounted on the back side of it, facing through the slots in the board towards the front. I added a button and an LED, to allow for a basic user interface. Of course, I also included a header to use an external speaker if the built-in speaker didn't sound good. I also tried rounding my traces for the first time, as inspired by [this project by mitxela](https://mitxela.com/projects/melting_kicad). It probably doesn't make much of a difference to interference or noise, but I thought it looked nice.

{{ img(src="2026-03-11-test-rig-pcb-1.webp", alt="Layout for the NMOScillator test rig PCB", size="medium") }}

If you're interested in building this PCB for yourself, the design files are available [here on GitHub](https://github.com/QEStudios/SN76489-Test-Rig-PCB).

---

Once I was happy with the PCB layout, I ordered the boards from [JLCPCB](https://jlcpcb.com). They arrived after around 2 weeks, and as always I was pleased with the production quality. The rounded traces looked very aesthetically pleasing, so I'll definitely be using that technique on PCBs in the future. I took some glamour shots of the bare board, and then got to assembling it.

{{ img(src="2026-03-11-blank-pcb-1.webp", alt="The blank PCB", size="medium") }}
{{ img(src="2026-03-11-melty-traces-1.webp", alt="Closeup of the rounded traces on the PCB", caption="Melty traces!", size="medium") }}

I was relieved that the on-board speaker was very easy to solder and fit perfectly onto the PCB. In fact, almost everything on the PCB worked exactly as I had hoped. There was *significantly* less noise than the breadboard had, and the soldered joints were much more reliable than the touchy cheap breadboard contacts. The Arduino could control the sound chip exactly as I wanted, and the music I could play on it sounded quite good. The built-in speaker on the PCB didn't do it many favours, though, and I ended up using a larger external speaker whenever I wanted to hear the songs in their best quality. I did accidentally wire the logarithmic potentiometer in reverse on v1.0 of the PCB (which is the version I ordered), but it was an easy fix with two bodge wires (and I fixed it on v1.1 of the board.) Overall, the PCB turned out just as I had wanted, which made me very happy.

{{ img(src="2026-03-11-assembled-pcb-1.webp", alt="The assembled PCB", size="medium") }}

{{ img(src="2026-03-11-speaker-and-bodge.webp", alt="Test", caption="The speaker mounted on the back, and the bodge fixing the potentiometer connections.", size="medium") }}


---

# Part 6: Music Showcase

I've recorded a few songs I made for the NMOScillator, in both Furnace and on the test rig, for you to enjoy. None of these are original compositions (I'm not very good at writing music, unfortunately), but they were all arranged for the NMOScillator by hand.

## Undertale - Rude Buster
{{ video(src="2026-03-11-rude-buster-furnace-1.webm", poster="2026-03-11-rude-buster-furnace-1.webp", alt="Screen recording of Rude Buster in Furnace tracker", caption="Furnace screen recording.", youtube="https://youtube.com/watch?v=an-_REktqtA", size="medium", type="video/webm") }}

{{ audio(src="2026-03-11-rude-buster-recording.ogg", caption="Real test rig audio recording (fade out added in post).") }}

## Erik Satie - Gymnopédie No. 1
{{ video(src="2026-03-11-gymnopedie-furnace-1.webm", poster="2026-03-11-gymnopedie-furnace-1.webp", alt="Screen recording of Gymnopédie No. 1 in Furnace tracker", caption="Furnace screen recording.", youtube="https://youtube.com/watch?v=cHtsFqRrfxQ", size="medium", type="video/webm") }}

{{ audio(src="2026-03-11-gymnopedie-recording.ogg", caption="Real test rig audio recording.") }}

## Scott Joplin - Maple Leaf Rag
{{ video(src="2026-03-11-maple-leaf-rag-furnace-1.webm", poster="2026-03-11-maple-leaf-rag-furnace-1.webp", alt="Screen recording of Maple Leaf Rag in Furnace tracker", caption="Furnace screen recording.", youtube="https://youtube.com/watch?v=HaUrlRjwuvk", size="medium", type="video/webm") }}

*I didn't record any audio of the Maple Leaf Rag on the test rig, sorry!*

I would have liked to record a *video* of the test rig playing songs, but it's really hard to get good audio quality with little background noise out of a video recording.

---

# Part 7: Next Steps

Whew! That ended up being a long blog post. Now that I've had some experience with the SN76489, I feel more confident that I'll be able to see this project through.

For now, I'll do my best to finish designing the real IC-based NMOScillator circuit in Digital. I'll have to hunt down any stray race conditions in the circuit, because I'm worried they could become a problem down the road. I also want to figure out a system for pre-setting the NMOScillator's Address Counter, so multiple songs can be packed into the same ROM with their own address offsets.

Stay tuned for more on this project! If you'd like, you can subscribe to my blog's [rss](/rss.xml) or [atom](/atom.xml) feeds to stay updated on the NMOScillator, and all my other posts.

-skm

---

## Useful Links

- [NMOScillator Compiler GitHub repository](https://github.com/QEStudios/NMOScillatorCompiler) (Source for the NMOScillator Compiler program)
- [NMOScillator Test Rig PCB GitHub repository](https://github.com/QEStudios/SN76489-Test-Rig-PCB) (Design files for the NMOScillator Test Rig PCB)
- [Video Game Music Preservation Foundation page for the SN76489](https://www.vgmpf.com/Wiki/index.php?title=SN76489)
- [Wikipedia page for the SN76489](https://en.wikipedia.org/wiki/Texas_Instruments_SN76489)
- [SN76489AN Datasheet/Manual](https://wiki.console5.com/tw/images/b/b0/SN76489.pdf) (The best scan I could find of this datasheet)
- [SN76494 / SN76496 Datasheet/Manual](https://www.vgmpf.com/Wiki/images/9/9c/SN76494_-_Manual.pdf) (Similar to the SN76489, but with different clock rates and an additional Audio In pin)
- [LM386 Audio Power Amplifier IC Datasheet](https://www.ti.com/lit/ds/symlink/lm386.pdf)
- [Furnace homepage](https://tildearrow.org/furnace) / [Furnace GitHub](https://github.com/tildearrow/furnace)

*All photos in this post were taken by me.*  
*The songs featured here are my own arrangements. The original compositions belong to their respective composers.*