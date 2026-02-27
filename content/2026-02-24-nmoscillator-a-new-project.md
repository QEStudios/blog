---
draft: true
title: "NMOScillator: A New Project!"
description: An overview of my latest project, the NMOScillator, which is an SN76489-based chiptune music player using 74 series logic ICs.
date: 2026-02-26T03:35:00+11:00
taxonomies:
  tags:
    - nmoscillator
    - electronics
    - low-level
    - arduino
    - music
    - golang

---

# An overview of my latest project, the NMOScillator.

It seems like every time I start to get into the weeds with a project, a new one springs up in its place and occupies my attention.
This time is no different.

Over the past few weeks, I've been fixating on a brand new project, which I've been calling the NMOScillator.
The basic idea is to drive an [SN76489 sound chip](https://en.wikipedia.org/wiki/Texas_Instruments_SN76489) using rudimentary 74 series logic ICs to produce chiptune music.
It's a very simple concept, which is why I found it so appealing. I've always had an attraction to creating complexity from simplicity, especially in electronic circuits.

Of course, I wanted to avoid writing a blog post about something that only existed as an idea. I originally ordered the sound chips back in September of last year, and I first wrote down plans for the NMOScillator in November. It's taken up until now for me to be ready to share my proof-of-concept.

The plan was to use a ROM (of sorts) to store song data, and then some logic could read that ROM data, parse out any special instructions it contained, and then pass the rest of the data on through to the SN76489. Thanks to the simple protocol used by the SN76489, piping commands to it from the ROM would be the easy part. The hard part would be the rest of the parsing logic.

*If you want to learn more about the SN76489, you can find some [useful links](#useful-links) at the bottom of this post.*

---

## Part 1: The ROM format

*(If I write something in **bold** in this section, that means it's a recurring term I came up with to describe the NMOScillator's behaviour.)*

One of the requirements I set for myself was to make the parsing logic smart enough to fit lengthy songs into the ROM. The most convenient way to do this was to store the songs in **frames**, similar to rows in a tracker. These frames would be played back at a given **tempo**, which could be specified in the ROM data.

{{ img(src="2026-02-26-furnace-rows-1.png", alt="Screenshot showing a row highlighted in Furnace tracker", caption="Frames in the ROM data are like rows in music trackers. (Pictured: Furnace tracker)", size_landscape="small", process="false") }}

Each frame was designed to have one byte of header data, and then a stream of **commands** to be either interpreted by the NMOScillator or piped to the SN76489. The frame headers contain flags for song looping, and specify the number of commands each frame contains. Each command in a frame can either be an **SN76489 command** (of which the byte is sent directly to the SN76489), a **tempo change** command, or a **frame delay** command (which delays the next frame by a multiple of the tempo). The type of each command is specified by its **command index**, counting down from the number of commands present in the frame. Specific indexes are defined as specific command types, so you can infer what each byte means just by counting how many bytes you've already read in the frame.

That's a lot to take in, but if you're interested, I've written up a (mostly) complete description of the ROM format [on my GitHub](https://github.com/QEStudios/NMOScillatorCompiler/blob/master/ROM_FORMAT.md), along with some example frames. It'll probably change as I develop the project further.

---

## Part 2: The Logic Simulation

I decided that a good way to test this ROM format's viability would be to build up a logic circuit for the NMOScillator in Logisim. That way, I could get an intuitive sense of what is/isn't possible with basic logical components.

This resulted in the following initial logic circuit:

{{ img(src="2026-02-26-logic-diagram-1.webp", alt="Screenshot showing a logic circuit in Logisim", caption="The initial logic circuit designed in Logisim. (Click to open in new tab)", size_landscape="large") }}

{{ file_with_mirror(text="Download this circuit here (28.02 KB)", path="2026-02-26-logic-circuit-1.circ", mirror="https://mega.nz/file/ZvNxXQgb#fxqgNYK9E5f5GCnFPk4NYhTR_TDH6CqTrvVhbD1efc4") }}

The circuit consists of 5 counters, 2 registers, and some glue logic. It appeared to correctly play the basic ROM files I tested it with, and I felt that I had reached a good balance between capability and size with this design. There are some weird quirks of the circuit which may need looking into, such as the Address Counter being clocked at the same time the Command Index is loaded from ROM, which may lead to a race condition if the ROM manages to update its outputs *before* the Command Index is loaded. However, this may never be an issue, due to the inherent propagation delay the Address Counter and ROM have.

As of writing this post, I'm porting the design over to [Digital](https://github.com/hneemann/Digital) with simulations of the real logic chips I'll be using, for faster performance and a more accurate depiction of the logic.

I put together this video annotating a simple example ROM running in the Logisim simulator. You can pause to read the text explaining the different stages of playback.

{{ video(src="2026-02-26-logic-simulation-annotated-1.webm", alt="Annotated recording of the Logisim simulation", caption="", size="large" type="video/webm") }}
[Click to watch this video in higher quality on YouTube.](https://youtube.com/watch?v=9cukEJ25jUs)  
(Note that the highest bit of the Tempo Counter is set to 0 in this video, not 1, to make it run significantly faster for demonstration purposes)

---

## Part 3: The Compiler

Now that I had a working circuit, I needed some songs to test it with. I had planned on using [Furnace](https://tildearrow.org/furnace) to compose them, but I had no way of turning them into NMOScillator ROMs. So, I wrote my own compiler to do just that.

I decided to write the compiler in [Go](https://go.dev), partly because I wanted it to be faster than a Python script would be, and partly because I wanted to familiarise myself with the language. I was still very new to Go at the time, and I thought this would be a good opportunity to learn it.

After spending a little over a month developing the compiler, I was able to generate ROM files for any song I composed in Furnace. The compiler took such a long time to develop because of my desire to make it modular. It uses multiple internal representations for the song data in different parts of the program, meaning it should be easy to write additional parsers for other input filetypes.

The source code for the compiler is available [on github here](https://github.com/QEStudios/NMOScillatorCompiler). I feel it would be a bit out of scope for this blog post to explain everything in the code, so if you want to learn more about the program you can read through the source code.

# TODO set part number
## Part X: Music Showcase

I've recorded a few songs I made for the NMOScillator, for you to enjoy. None of these are original compositions (I'm not very good at writing music, unfortunately), but they were all arranged for the NMOScillator by hand.

### Undertale - Rude Buster



# TODO: 
  - the name NMOScillator is because the SN76489 is an NMOS device
  - compiler
  - furnace composing
  - arduino testing (breadboard) + arduino code
  - doing amplifier circuitry for the first time
  - making it into a pcb for better noise and because it looks pretty

## Useful Links

- [Video Game Music Preservation Foundation page for the SN76489](https://www.vgmpf.com/Wiki/index.php?title=SN76489)
- [Wikipedia page for the SN76489](https://en.wikipedia.org/wiki/Texas_Instruments_SN76489)
- [SN76489AN Manual](https://wiki.console5.com/tw/images/b/b0/SN76489.pdf) (The best scan I could find of the datasheet.)
- [SN76494 / SN76496 Manual](https://www.vgmpf.com/Wiki/images/9/9c/SN76494_-_Manual.pdf) (Similar to the SN76489, but with different clock rates and an additional Audio In pin)
- [Furnace homepage](https://tildearrow.org/furnace) / [Furnace GitHub](https://github.com/tildearrow/furnace)

To stay up to date with my latest posts, you can subscribe to my blog's [rss](/rss.xml) or [atom](/atom.xml) feeds.

-skm

*All photos in this post were taken by me.*