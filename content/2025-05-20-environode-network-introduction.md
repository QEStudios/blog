---
title: Environode Network Introduction
description: This blog post discusses my ongoing Environode project. Since it's still a work in progress, some things may change, as nothing is fully set in stone.
date: 2025-05-20
taxonomies:
  tags:
    - environode
    - electronics
    - iot
    - raspberry pi pico
    - raspberry pi
---

# An introduction to my Environode project.
This blog post discusses my ongoing Environode project. Since it's still a work in progress, some things may change, as nothing is fully set in stone.

---

When I found out I could buy a [Raspberry Pi Pico clone](https://www.aliexpress.com/item/1005007393927221.html?pdp_ext_f=%7B"sku_id":"12000040565295947"%7D) for around $3 AUD on AliExpress, I knew that would be the microcontroller I'd be using for most of my projects. When I found out I could buy a [*Raspberry Pi Pico W* clone](https://www.aliexpress.com/item/1005007393927221.html?pdp_ext_f=%7B"sku_id":"12000040565295948"%7D) for under *$8* AUD, I knew something was up. Microcontrollers like that with proper WiFi support are rarely that cheap.

Intrigued, I purchased one. Now, there's always a risk when buying online, and this time was no different. What I received appeared to be exactly what I had hoped for: a Raspberry Pi Pico with WiFi support. But I soon realised it wasn’t a drop-in replacement for an *official* Raspberry Pi Pico W. This board, which I expected to have an [**Infineon CYW43439**](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/wi-fi-4-802.11n/cyw43439/) chip handling its WiFi connectivity, instead sported an [**Espressif ESP8285**](https://www.espressif.com/sites/default/files/documentation/0a-esp8285_datasheet_en.pdf) chip. Not to be deterred, I looked up documentation online for this specific flavour of Pico clone. What I found was pretty much just "Buy an official Pico instead."  
*(I later discovered that the listing's images clearly showed that the Pico had an Espressif chip. Always make sure to double-check before buying something!)*

Naturally, instead of following the advice, I decided to use this clone Pico W to design my own locally hosted environmental sensor network, and this is the topic of this blog post!

{{ img(src="2025-05-20-pico-1.jpg", alt="Picture showing the clone Raspberry Pi Pico W", caption="The clone Raspberry Pi Pico W", size="medium") }}

---

The clone Raspberry Pi Pico "W" is configured to use AT commands over UART to communicate with the ESP8285. This is the same as how [those "WiFi enabled" Arduino UNO clones](https://www.jaycar.com.au/p/XC4411) work. This means you have to first flash the firmware of the ESP8285 by first flashing the RP2040 with a basic program to pass the serial commands through to the ESP8285, then plugging in the clone whilst holding down the ESP's dedicated BOOT button (lovingly mislabeled as *DOOT* on my clone). I figured this out from [a few manufacturer videos in Chinese](https://forums.raspberrypi.com/viewtopic.php?t=361532#p2174428), which some kind internet soul had uploaded to their MEGA account. So far, the user experience is lacking.

*([I've reuploaded the files to my own MEGA account to preserve them, you can download them here.](https://mega.nz/folder/VqMg2YQC#seTJR8HW_Eu3lB-ttfzXyQ))*

{{ img(src="2025-05-20-doot-1.jpg", alt="Picture showing the word 'DOOT' printed on the Pico clone", caption="...lovingly mislabeled as DOOT on my clone", size="small") }}

Then, I had to learn how to use AT commands. AT commands (short for *Attention commands*) are simple text-based instructions used by modems and certain modules (such as WiFi or Bluetooth chips) to communicate over a serial connection. Learning to use them wasn't too difficult, as [Espressif has it all documented on their website](https://docs.espressif.com/projects/esp-at/en/latest/esp32/AT_Command_Set/index.html). However, what *was* difficult was finding out that, for some reason, I couldn't get https communication working! After flashing the firmware maybe 6 times with different configurations, I decided it wasn't worth the trouble and opted to use exclusively TCP packets to send the data.

The data in question would be temperature and humidity readings from an AHT20 module, also purchased from AliExpress for around $1.50 AUD. [The specific module I bought](https://www.aliexpress.com/item/1005007394116053.html) had both an [AHT20](https://cdn.sparkfun.com/assets/d/2/b/e/d/AHT20.pdf) (temperature + humidity) and a [BMP280](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmp280-ds001.pdf) (pressure) sensor on it, as both of these sensors communicate over I<sup>2</sup>C and thus were connected over the same I<sup>2</sup>C lines.

I later found that the BMP280 was too unreliable and kept failing to communicate after a while, so I stopped using it. The project is designed to be flexible, and so I can always add more sensors if I decide to later on.

{{ img(src="2025-05-20-modules-1.jpg", alt="Picture showing the environment sensors and small OLED screen", caption="The AHT20/BMP280 sensor module and the 128x64 OLED display", size="medium") }}

The Pico is programmed to keep a log of the sensor readings, and then periodically send these readings to my Raspberry Pi 3B+ over TCP. The Pi runs a small server script written in Python, which listens for these TCP packets and saves the readings into a database. And as the Pico is barely breaking a sweat running its code, I am able to connect an [I<sup>2</sup>C 128x64 OLED Display module](https://www.aliexpress.com/item/1005007895120525.html?pdp_ext_f=%7B"sku_id":"12000042749372460"%7D) to it too, for displaying the readings and such.

Now multiply this setup by five, and you have an entire network of small (and cheap!) WiFi-connected nodes sending environmental readings to a server.

{{ img(src="2025-05-20-pi-1.jpg", alt="Picture showing my Raspberry Pi in its case sitting on my shelf", caption="My Raspberry Pi in its case sitting on my shelf", size="small") }}

---

Of course, there are a number of downsides to running the project this way. One big issue is that these clone Pico "W" boards aren't nearly as well-documented as the official ones using the Infineon chip; some reverse-engineering is required. Another major problem is that the communication between the Picos and the Pi is *completely unsecured* and in fact can be *very easily hijacked*. This is why the Environode network runs only over LAN and should never be exposed to the internet (and even then, there's a risk of someone on your LAN interfering with the network.)

However, for a cheap and easy project, it works great. I got to learn many things while making this project, and these skills will carry on into my other projects too. I plan on integrating some of my other Pico-based projects into the Environode network too, notably my Heximal Clock (which I plan to talk about in a later blog post! 👀).

---

The code for the Environode network will likely be released as Open Source, and if it does I'll include information on how to get it running on your own. However, in its current state it is not ready for release (I'm still just working on breadboards!) and so I won't be releasing the source code alongside this blog post. Sorry about that! You can, however, subscribe to the [rss](/rss.xml) or [atom](/atom.xml) feed to be updated on my future blog posts.

-skm

*All images in this post were taken by me.*