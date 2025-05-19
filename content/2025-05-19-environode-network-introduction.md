---
title: Environode Network Introduction
description: An introduction to my Environode project.
date: 2025-05-19
draft: true
---
# An introduction to my Environode project.
This blog post discusses my ongoing Environode project. Note that as this project is not in a completed state yet, some things may change and not everything is set in stone.

---

When I found out I could buy a Raspberry Pi Pico clone for around $2 AUD on AliExpress, I knew that would be the microcontroller I'd be using for most of my projects. When I found out I could buy a *Raspberry Pi Pico W* clone for around *$6* AUD, I knew something was up. Microcontrollers like that with proper WiFi support are rarely that cheap.

So, intrigued, I purchased one. Now there's always a risk when buying online, and this time was no different. What I received appeared to be exactly what I had hoped for: a Raspberry Pi Pico with WiFi support. But I soon realised that what I had purchased was, in fact, not the same as an *official* Raspberry Pi Pico W. This board, instead of having an **Infineon CYW43439** chip handling its WiFi connectivity, sported an **Espressif ESP8285** chip. Not to be deterred, I looked up documentation online for this specific flavour of Pico clone. What I found was pretty much just "Buy an official Pico instead."
(I later discovered that the listing's images clearly showed that the Pico had an Expressif chip. Always make sure to double-check before buying something!)

So, naturally, I decided to use this clone Pico W to design my own locally hosted environmental sensor network, and this is the topic of this blog post!

---

The clone Raspberry Pi Pico "W" is configured to use AT commands over UART to communicate with the ESP8285. This is the same as how those "WiFi enabled" Arduino UNO clones work. This means you have to first flash the firmware of the ESP8285 by first flashing the RP2040 with a basic program to pass the serial commands through to the ESP8285, then plugging in the clone whilst holding down the ESP's dedicated BOOT button (lovingly mislabeled as *DOOT* on my clone). I only figured this out from watching Chinese videos from a manufacturer of these boards. So far, the user experience is lacking.

Then, I had to learn how to use AT commands. This wasn't too difficult, as Espressif has it all documented on their website. However, what *was* difficult was finding out that, for some reason, I couldn't get https communication working! After flashing the firmware maybe 6 times, I decided it wasn't worth my time and opted to use exclusively TCP packets to send the data.

The data in question would be temperature and humidity readings from an AHT20 module, also purchased from AliExpress for around $1.50 AUD. The specific module I bought had both an AHT20 (temperature + humidity) and a BMP280 (pressure) sensor on it, as both of these sensors communicate over I²C and thus were connected over the same I²C lines. I later found that the BMP280 was too unreliable and kept failing to communicate after a while, so I stopped using it. The project is designed to be flexible, and so I can always add more sensors if I decide to later on.

The Pico is programmed to keep a log of the sensor readings, and then periodically send these readings to my Raspberry Pi 3B+ over TCP. The Pi runs a small server script written in Python which listens for these TCP packets and saves the readings into a database. And as the Pico is barely breaking a sweat running it's code, I am able to connect an I²C 128x64 OLED Display module to it too, for displaying the readings and such.

Now multiply this by 5 times, and you have an entire network of small (and cheap!) WiFi-connected nodes sending environmental readings to a server.

---

Of course, there are a number of downsides to running the project this way. One big issue is that these clone Pico "W" boards aren't nearly as well-documented as the official boards using the Infineon chip; some reverse-engineering is required. Another major problem is that the communication between the Pico's and the Pi is *completely unsecured* and in fact can be *very easily hijacked*. This is why the Environode network runs only over LAN and should never be exposed to the internet (and even then you run the risk of a bad actor *on* your LAN messing with the network.)

However, for a cheap and easy project, it works great. I got to learn many things while making this project, and these skills will carry on into my other projects too. I plan on integrating some of my other Pico-based projects into the Environode network too, notably my Heximal Clock (which I plan to talk about in a later blog post! 👀).

-skm