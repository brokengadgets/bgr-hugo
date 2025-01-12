---
date: '2025-01-11T19:16:03-05:00'
draft: false
title: 'Mumen Controller'
featured_image: '/images/controller.webp'
type: page

---

Checking my git logs, I've worked on what would become mumen-controller for nearly 3 years at the time of writing this and have never stopped to record my progress in a useful way (though I have written about it in various private chat with friends and posted photos of build progress to a few group chats). For the beginning of this post, I'll focus on a historical recap to the best of my available records and recollection. Some details may be slightly deviated from the way things went, but it should give a useful-enough background on the project, some of the hurtles to cross, and where the current goals were generated. 

### 🥊 The Motivation 🥊
I'm not any good at fighting games, but I admire the dedication excellent players demonstrate. Even the casual players I have associated with take the game very seriously and using their controllers is invaluably fun. Upon looking into buying or building my own fight stick, hitbox, or arcade pad, I was appalled at how expensive the whole process was and sought to perform a lot of the work by repurposing junk around the house. In repurposing junk, I ended up with a $5 devkit the sparkfun pro micro driven by the Atmega 32U4. I had a few in my drawer and thought I'd just make one dance as a fightstick. 
![Sparkfun Pro Micro](/images/promicro.webp)

### 📦 We came from Nothing 📦
I ordered a joystick unit and once it arrived, I disassembled an old gaming keyboard and stuck it all into a cardboard box with no concern whatsoever for rectifier diodes. After a short period, it became clear the cardboard case was insufficient for the amount of force used in fighting games, so the center was reinforced with a series of large toy blocks hot glued in place. The stick was repurposed for the next iteration before this photo was captured.
![Unlicensed Cardboard: a Beautiful Success.](/images/unlicensed-cardboard50.webp)

Somewhere in parallel with that effort (it looks like all 11 commits were on May 1, 2022) I rolled together [UnlicProController](https://github.com/nulvox/UnlicProController). For that project, I initially planned to only install the firmware provided by the [similar project from fluffymadness](https://github.com/fluffymadness/ATMega32U4-Switch-Fightstick), but whickly found myself changing enough to make it my own repo and developing features other users would not want. Once it was working and I had no sane way to upstream my changes, I did what any sany person would: 

`rm -rf .git ; git init UnlicProController`.

### 🤥 I'm a real controller! 🤥
My obvious next step was to make a better case and use nicer keys, so I got to work with some old pallet wood from an appliance I purchased and a $15 laminated oak board from Lowes making the true form of the UnlicProController case. 

![Unlicensed Wood: a boat of a controller](/images/unlicensed-wood50.webp)

While this case feels great to play on, the controller weighs 8.7lbs and takes up an unruly amount of desk space. To top it off the firmware is this mostly ripped off thing that I didn't love a lot of the backing libraries for (but was unwilling to clean up in the existing codebase). 

So I started down the pathway of a new firmware from the ground up and thought I'd just give rust a swing because I could find some low level board support for the Atmega 32u4 and I was having a good time with the language in a few other projects: 

```
cargo new mumen-controller 
cd mumen-controller  
git init MumenController
```

### 🦀 Feeling a little Rusty 🦀
The [main branch](https://github.com/nulvox/MumenController) underwent a rocky lifetime so far. Even on such a small and low power devkit, I found that the admittedly sloppy C++ implementation was surprisingly low latency as far as the industry is concerned (maybe someday there will be a post about assembling the standard controller latency tools). That coupled with learning about the [GP2040-CE](https://github.com/OpenStickCommunity/GP2040-CE)'s superior I/O array and feature~~bloat~~set, it was clear MumenController should aim for speed. In that realm, the top competitor is the [Brook P5 Mini](https://focusattack.com/brook-p5-mini-fight-board-pcba/) which has ~8ms latency and firmware-driven SOCD-cleaning (saving the extra component and any additional latency that might introduce) but also brings some interesting shortcomings:
 - Only supports connecting as a wired PS3/4/5 controller
 - No support for the [Brook Universal Fighting Board Wiring Harness](https://focusattack.com/bundle-japan-wiring-for-brook-ufb-gp2040-ce/)
   - 🧐 curious that they would create a standard then break it themselves 🧐
   - Users have to solder themselves. Nothing I'm not used to, but not what I expect from a product at $60
 - No support for analog inputs 
   - Though it does support pretty standard analog emulation
   - and neaT PS5 touchpad emulation!
![Brook P5 Mini front: notice the potting compound on the small chip in the bottom left](/images/brook-front50.webp) ![Brook P5 Mini back: this footprint suggests the small chip is the MCU](/images/brook-back50.webp)

In the process of making the rust firmware more efficient, I eventually uncovered a messy showstopper: the C++ firmware was running with an unavoidable race condition this whole time and rustc was having no part of it even with extensive coaxing. Specifically, the hardware atomics on the ProMicro were only half as wide as needed to write the HID Descriptor table for the Switch Pro Controllers. 😵 Time to change. 

### ✨Wastefully and Whimsically Migrating Hardware✨
Knowing we had to change and speed was a goal, it seemed clear to look for a platform with more robust support in the embedded rust community. A quick search through [crates.io](https://crates.io/search?q=bsp) showed quite a few better options, but already having a teeny4.1 sitting on the desk next to me (and having a few other projects on the platform to tinker with), I thought I'd take a peek at the rust support (which is [pretty awesome](https://crates.io/crates/teensy4-bsp)). While the cost is absurd, it's still about half the cost (at this point) of buying a Brook board and a lot more flexible. Promising to remove my atomics race headache and provide notably faster cores (likely faster than what Brook is using if I had to guess what is behind the delicious potting compound) it seemed prudent to build the funnycar of fightsticks. The redesign also went along with a hitbox style and ditched the old concept of the shift key (at least for now) to provide a faster processing goal and cut out some of the fluff unrelated to the project's target niche in the ecosystem. 
![Teensy4.1](/images/teensy4.1-70.webp) ![Teensy 4.0](/images/teensy4.0-25.webp)

While working those hardware angles, the mess of a wiring harness was another piece of techdebt to address. With the 22awg coated wires we had on-hand and the various kinds of perfboards we tried working with, this was all around a nightmare. Once a few Brook harness headers came in, we thought we would just solder across a perfboard to those and it was still a nightmare. After a short period of research, the next action was clear: `sudo pacman -S kicad`. Having never designed a pcb before and seeing an enticing ~~bait~~sale on [PCBWay](https://www.pcbway.com/Member/Login/?from=bingSE06), KiCad was launched and a new board was being designed. 

![Final Printed Layout](/images/pcb-layout50.webp)

These simple boards provide a pinout to the 20-pin brook header, some extra pins for our use on other features, through-holes for rectifier diodes, a footprint for the teensy4.0, and now a significantly higher cost that the Brook board . 10 of them were ordered from PCBW and they showed up surprisingly quickly and We quickly populated a few.
![Stax](/images/board-stack.webp)
![WIP](/images/populating.webp)

While waiting on those boards to arrive, the new case needed to be built and the clear top face needed some art. With some cheap acrylics laying on a shelf, some pretty neat imagery was layered onto the acrylic clear sheet. The cheap paints didn't stick well to the surface and took a long time to dry coupled with the need to layer the pain in an unusual way (front layers need to come first to show up closest to the underside of the clear panel), this took a long time to get onto the page. While waiting for the paint layers to dry, it was easy to get a new wooden frame made from an oak 1x2 and a plain acrylic sheet from Lowes.  
![Laying out the design](/images/paint-layout.webp) ![A few layers in](/images/paint-wip1.webp) ![More layers](/images/paint-wip2-50.webp) ![The frame and bottom layer mostly sanded and smoothed](/images/frame50.webp) ![mumen-controller face](/images/controller.webp)


### 🥸 Standing Out to Fit-in 🥸
As already mentioned in brief, there are a lot of competing fighting game controller implementations out there and having a place that works for any product in any market means having something to stand apart from the rest with.
 - The Gold standard for builders: [Brook Universal Fighting](https://focusattack.com/brook-gen-5x-multi-console-pc-fighting-board/) $74.95
 - An older classic: [Brook ZeroPi](https://focusattack.com/brook-zero-pi-fighting-board-pi-ps3-ps2-psx-pc-switch/) $24.95
 - The modern Speed demon: [Brook P5 Mini](https://focusattack.com/brook-p5-mini-fight-board-pcba/) $59.95
    - 🤔 Interesting that 1 company has done so well for so long. 
 - Feature-rich DIY/FOSS Solution: [GP2040-CE](https://focusattack.com/gp2040-ce-v5-6e-usb-b-usb-c-open-source-multi-console-fight-board/) $34.99
There are a few additional options available, but those I am aware of are either lacking a stand-out quality or don't ship with a key component built-in (ie, they are not presented meaningfully as a product).

Looking at this market, I see that the Brook offerings are going to stay and more or less be a baseline. The GP2040-CE does an excellent job of providing a wider feature set than the Brook options and has the benefit of running the same firmware on a number of other boards (with just some small changes to build options). The project is mature, large, and dense; a feature set like that is not a realm to compete with and the RP2040 is [a little pricy](https://octopart.com/102110545-seeed+studio-116237358), but [not terribly out of sight](https://octopart.com/mimxrt1062cvl5b-nxp+semiconductors-105790568). The GP2040-CE is also available ready to go at a very low price as far as this market is concerned. 

The commercial competitor there is the Universal Fighting Board which does a lot of great things for the user:
 - SOCD cleaning
 - Console support switching
 - Signal rectifier diodes
 - A standard header for buttons
 - Brook touchpad emulation