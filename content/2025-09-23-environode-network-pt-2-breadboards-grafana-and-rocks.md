---
title: "Environode Network Pt. 2: Breadboards, Grafana, and ROCKs"
description: Updates on my current Environode network setup, featuring my new ROCK 4C+.
date: 2025-09-23
taxonomies:
  tags:
    - environode
    - electronics
    - iot
    - raspberry pi pico
    - radxa rock
    - grafana
---

# An update on my Environode project.
Updates on my current Environode network setup, featuring my new ROCK 4C+.

---

My [previous blog post](/2025-05-20-environode-network-introduction) covered the initial idea for the Environode project. Since then, I've made some progress and am now ready to make a second blog post detailing the latest updates. I've had two breadboarded Environodes running for the past 3 months now with only some minor issues, and I'm very happy with how it's all turned out so far.

After writing that first blog post, I spent some time on the interface and code for the Environodes. The code running on the Environodes themselves is written in [MicroPython](https://micropython.org), an implementation of Python designed to run on microcontrollers. The server code that the Environodes communicate with is written in Python. I chose these languages because I'm very familiar with them, and although they might not be the most widely respected choice for larger projects, it sped up development quite significantly.

The breadboard Environode design looks like this:

{{ img(src="2025-09-23-pico-breadboard-1.webp", alt="Image of an Environode on a breadboard", caption="The breadboarded Environode using the Raspberry Pi Pico W clone", size="medium") }}

I chose to use a blue-and-yellow I<sup>2</sup>C OLED display for the screen, as the top 16 pixels are yellow, meaning I can show a constant status bar at the top while the rest of the display is reserved for other information. I also added an LDR to allow for dimming the screen when the environment brightness is lower, and the screen can also shut off after a certain amount of time to reduce power and prolong the screen's lifespan.

There are still issues with the BMP280 sensor, so I haven't enabled pressure logging yet. There's a chance it may be caused by the conflicting I<sup>2</sup>C pullups on the screen and sensor modules. However, I haven't noticed any issues with the AHT20 sensor at all, and it's been running without issue the whole time.

I initially built the Environodes on breadboards as a proof-of-concept, and I was not planning on sharing the breadboard design, hoping that I would quickly move them onto dedicated PCBs. I ended up procrastinating moving them to PCBs for 3 months, and those breadboard Environodes have been consistently running the whole time. Because of this, I feel confident in releasing the (very simple) schematic for them now:

{{ img(src="2025-09-23-pico-schematic-breadboard-1.webp", alt="Schematic drawing for the original breadboard version of the Raspberry Pi Pico Environode", caption="Original schematic for the breadboarded Raspberry Pi Pico Environode", size="medium") }}

*[Download this schematic here (97.71 KB)](/files/2025-09-23-pico-schematic-breadboard-1.kicad_sch)&nbsp;&nbsp;&nbsp;[(Mirror)](https://mega.nz/file/N68lwQxY#_WO86jBCGoBulecV6tE1MAH59I2Yc3qlqqoCdPWa0pY)*

This schematic is a 1:1 diagram of how *I* prototyped the Environodes. However, this original design has some flaws:
- I<sup>2</sup>C modules are powered with 5V while the data lines are driven at 3.3V, which might cause the Pico to misbehave or get damaged if the data lines are pulled to 5V.
- Both I<sup>2</sup>C modules have their own pullups on the breakout boards, which can lead to bad signal integrity.
- The buttons rely on internal pullups in the Pico, which may not be as reliable as hardware pullups.

As such, I would recommend using this adjusted schematic if you plan on building your own Environodes on breadboards:

{{ img(src="2025-09-23-pico-schematic-breadboard-adjusted-1.webp", alt="Schematic drawing for the adjusted breadboard version of the Raspberry Pi Pico Environode", caption="Adjusted schematic for the breadboarded Raspberry Pi Pico Environode", size="medium") }}

*[Download this schematic here (116.82 KB)](/files/2025-09-23-pico-schematic-breadboard-adjusted-1.kicad_sch)&nbsp;&nbsp;&nbsp;[(Mirror)](https://mega.nz/file/171B1LRR#8RMsxCGgbbrFwqAU_ElscEDCsH1nw7QTBsazQOunJnI)*

---

As mentioned in [the previous blog post](/2025-05-20-environode-network-introduction), I was using my Raspberry Pi 3B+ as the central server that the Environodes communicate with and send data to. However, a month later I stumbled across a great deal for a [Radxa ROCK 4C+](https://radxa.com/products/rock4/4cp) from [Jaycar for $39 AUD](https://www.jaycar.com.au/p/XC9300) (excluding shipping). This was under half of the RRP at the time and cheaper than any other comparable SBC, so I immediately bought one. This turned out to be a *great* decision as it easily outperforms my Raspberry Pi 3B+ in every aspect. I soon after moved the server program onto the ROCK and it's been running faithfully ever since. I almost wish I grabbed a second one while I could!

{{ img(src="2025-09-23-pi-rock-1.webp", alt="Picture showing my new Radxa ROCK 4C+ sitting next to my Raspberry Pi on my shelf", caption="My new ROCK 4C+ (right) sitting next to my Raspberry Pi (left) in an identical case", size="small") }}

For visualising the data coming in from the Environodes, I opted to use [Grafana](https://grafana.com), as it's powerful and allows for advanced data visualisation. This is something that the Raspberry Pi 3B+ would struggle at, but the ROCK was up for the task. It took a while to setup, as the latest precompiled version of Grafana at the time used instructions that didn't exist on the RK3399-T that the ROCK 4C+ uses. I ended up using an older version of Grafana (specifically v8.5.22) which didn't rely on those instructions. I could have recompiled Grafana from source, but I decided it wasn't worth the effort at the time.

After a little configuration, my Grafana dashboard looks like this:

{{ img(src="2025-09-23-grafana-1.webp", alt="Grafana dashboard showing data from the past 7 days", caption="Grafana dashboard showing 7 days of historical data", size="large", quality=100) }}

And thanks to the flexibility of Grafana, I can easily add more visualisations or data points without much hassle.

---

Running Environodes on breadboards is fine, but having dedicated PCBs is even better. I started work on custom PCBs for the Environodes a little while ago, and I'm currently waiting on the first boards to arrive now. I won't release any of the schematics or PCB designs just yet, as there may be errors that I want to iron out before sharing them. Once it's all ready, I plan to release the design files under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) and the code under [MIT](https://opensource.org/license/mit/).

I have plenty more planned for this project, so if you want to stay up to date you can subscribe to my blog's [rss](/rss.xml) or [atom](/atom.xml) feeds.

-skm

*All images in this post were taken by me.*