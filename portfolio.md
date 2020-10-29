---
layout: post
title: Portfolio
permalink: /portfolio/
---

_This is a rough draft of my engineering portfolio. Please don't judge too harshly. When I have the time, I intend to add images, diagrams, and code snippets. Oh yes, and proof-reading. There is sure to be a number of grammar or spelling mistakes in here. As is, I'm hoping it will at least provide a general overview of the projects I've worked on in the past._

## JLM Energy Inc.
I worked at JLM from August 2012 through June 2018. This was my first job as an engineer. These are some of the projects I've worked on.

### Zefr
#### Arrayed micro wind turbine
These were small roof mounted wind turbines capable of generating up to 300W in the right conditions. They intended to be installed in an array, similar to solar panels, and even used microinverted originally designed for solar applications. The electronics included a rectifier, DC-DC converter, rheostatic breaking, and telemetry. The electronics were originally packed into the nacelle, but were moved to an external module due to thermal runaway issues. An energy meter called the Smart Gear Box (SGB) measured to total output of the array, and collected telemetry data from each turbine via a ISM band radio. 

Due to it's high cost, and some unresolved mechanical issues (where the hub would disintigrate at high angular velocity, sending turbine blades flying in all directions) Zefr was never a market success, and the project was eventually canceled.

I came into the project not long before the first articles shipped. My first task was to develop drivers, and communication protocol for the ISM band radio. Development was done on a Silicon Labs 8051 microcontroller using the Keil C51 compiler. 

After I had the radios communicating reasonably well ()despite being caged inside a metal box), I moved on to a complete redesign of the Smart Gear Box. The original array topology was to have the DC outputs stacked, and connected to a string inverter similar to a solar array. Just before shipping, it became apparent that this wasn't possible, and we needed to use microinverters instead, and thus needed an AC energy meter to measure the total output of the array. I had complete ownership of this project from the PCB design and layout, BOM management and assembly, and the entire firmware stack including all the drivers and communication protocols. Not bad for some kid straight out of university. Start to finish the SGB took about 4 months to complete. I made a ton of mistakes, but learned a lot along the way. In the end, it wasn't perfect, but it was able to collect and aggregate data just as intended.

After the first articles shipped, we began getting reports of nacelles catching fire in high wind conditions. I mounted an investigation and found that the induction generator in the nacelle got much hotter that anticipated. It was always assumed that the wind would provide cooling, so heat would never be a problem. Instead, the nacelle basically became a wind-powered toaster, hot enough to burn the powder coating. At this time I had moved on to another product called Chargrz which was a small module installed behind a solar panel that could charge a battery directly from the panel (see below), but it's electronics were almost identical to that of Zefr. I was able to advocate for replacing the electronics in the nacelle with a Chargrz module. So the nacelle contained the generator and rectifier, and the electronics weren't affected by the heat.


## Gyezr
#### Smart solar thermal array
This product was developed for commercial building owners who wanted to offset their natural gas heating bill while getting highly granular performance data. Each module contained a solar powered MCU with the same radio used in Zefr (see above). The MCU collected temperature data for each pipe inside each module, and relayed it to a control box where it was processes, and forwarded along to a back-end web service. The control box made use of this data to decide how water should flow through the system by turning the pump on or off, and adjusting the position of an electromechanical valve.

At the time this product was released, solar thermal was already out of vogue, and therefore was something of a flop, which is a shame; it was really amazing to watch this system kick into action as the morning sun hit the panels and the reservoir temperature would quickly rise. We only sold one system before discontinuing the product.

When I got to this project the electronics design had been started and abandoned due to the small size of the hardware/embedded team at JLM. A prototype PCB had been built, but it had many issues, and there was no firmware to speak of. The board, which was intended for the control box, had an 8051 MCU, an ISM band radio, and four relays. The relay coils were interfaced to the MCU via a shift register. Two relay were connected to each phase of a split-phase pump, while the other two were connected to a valve; one relay turned the valve left, while the other turned it right.

This is probably one of the weirdest and most interesting problems I've faced, but when a relay transitioned from closed to open, the one of the neighboring relays would inexplicably close. So when the pump turned off, the valve would begin to rotate, and when rotation stopped, the pump would turn back on. It was as though the system was suffering a demonic possession. After several days of debugging, I found nothing in the code that would cause this behavior, so I started probing points on the board with a scope. I discovered that when a relay's coil was deenergized, noise would appear on the clock line of the shift register. This would cause all the bits to shift, potentially closing a neighboring relay. The board's layout was done somewhat haphazardly  with no ground plane, and no thought taken to current paths for the relays.

I redesigned the board, taking a little more care this time, and the problem was solved. This bug left me traumatized, and to this day I'm still a little nervous when it comes to driving relays off a shift register. We installed our first and only system at a swimming pool on a military base, and after a few months we received complaints that the pool was too hot! The original requirements model specified a maximum temperature, which I hard-coded into the controller. I had to go back and implement some configuration parameters that the customer could use to set the maximum temperature via a USB dongle that communicated over the ISM radio. I designed, tested, and shipped the dongle, but the entire project what put to a halt by bureaucratic shenanigans as the Army officials would not allow the use of the ISM band radio on the base.

### LiFePO<sub>4</sub> Battery Management System (BMS)
#### Because lithium batteries misbehave sometimes
During 2013, JLM Energy pivoted away from energy generation, and toward energy storage. We launched a residential energy storage unit called Energizr a year ahead of Tesla's Powerwall, and began planning a commercial shipping container base solution called Gridz. Both of these products were based on LiFePO<sub>4</sub> battery technology, which needed a BMS to monitor and balance the individual cells. Most off-the-shelf BMS's are either expensive, or didn't scale well to meet our needs, so we designed our own to use in-house with both our commercial and residential products. Each BMS board could monitor and balance up to 24 cells. Our residential product had 15 cells, so it only needed one board. Our commercial product supported over 300 cells, so multiple boards needed to be stacked in a daisy-chain. The boards measured individual cell voltages and temperatures, and clocked the data back to a single controller over an isolated serial connection. This was our first product where the processing was done on an embedded Arm device with ethernet running linux(Beaglebone), instead of the 8051 / ISM radio combination.
 
For this project I was again the sole designer. The CEO handed me a block diagram scribbled on a piece of paper, and asked me to build it into a product. This project represents a shift in how I approached software design. At this stage in my career, I was becoming confident in the C language, and well, confidence can quickly become over confidence. Before, when I was working in the highly constrained 8051 environment, I took an extremely cautious design-forward approach to everything I did. But with linux, and dynamic memory allocation at my fingertips, I was more that happy to abuse the fragile constructs that the C language has to offer. Code written during this stage in my career was carelessly peppered with mallocs and reallocs, and also (and as a result) memory leaks, dangling pointers, and buffer overruns. All this because something in my head told me that this code would be 'faster' and 'more efficient'. All this micro-optimization lead to some truly regrettable code, and this is were I learned a valuable lesson about heap corruption.

When trying to debug a segmentation fault with printf, I found that printing a long message in the area of the suspected fault made the problem go away, but with the short message the program would crash as expected. The crash was occurring in an SPI read/write function, and after quite a lot of digging, I found that a buffer overrun was occurring in a function that processed cell voltage data (in a completely different module). The cell processing had overwritten values used by the SPI function, causing a crash. This is a hard lesson that every engineer must learn the hard way.

The BMS shipped after about 6 months of development, and quickly became an integral part of JLM's product line. Of course, my careless approach lead to many early adopters calling with complaints, but I was happy to take their feedback and used it to improve the product over time. It was in use for the duration of the company's existence.


### Chargrz
#### Solar battery charger
Although Chargrz never shipped as a standalone unit, I'm including it here since it served as the basis for what would become JLM's flagship product. The idea behind Chargrz was to eliminate a bulky expensive battery charger in an solar storage system, and replace it with small modules (about 6 inches in diameter and 1 inch deep) that would mount directly under a solar panel. The module contained a DC-DC converter with Maximum Power Point Tracker (MPPT), the now familiar 8051 MCU for controls and communications coupled with an ISM band radio, and an RGB LED to indicate status. Each panel would have it's own module, so the battery could be charged directly by the panel. With a few small modifications, Chargrz was also used with the Zefr wind turbine after it became apparent the electronics needed to be put outside the nacelle. This also gave the wind turbine the ability to directly charge a battery.

Like with most projects at JLM, I was given the specs and expected to make it into a product. The development cycle was mostly uneventful, and it was ready for production in about 2 months after conception.

### Powrz
#### Commercial and industrial controller
Starting in 2015, Powrz became the lifeblood to JLM, finding it's way into nearly every product shipped, and constantly being modified to fit more applications. It was a general purpose linux based platform intended to control and monitor power inverters and energy meters. At it's core was a Beaglebone Black, an embedded Arm computer running linux. Connected to the Beaglebone were 5 relays, 8 analog inputs, a pulse counter, RTC with battery backup, RS-485 transceiver, something called isoSPI for interfacing with the BMS, and most importantly, a big friendly reset button. Powering all with was a 10W AC-DC converter rated up to 300Vrms so it could be connected directly to a single phase of three-phase power.

It started as a way to interface a Beaglebone to an Acuvim energy meter. The first prototype was a tangled mess of off the shelf modules mostly from Adafruit and Sparkfun all hacked together by the software team, who had been trying to pull the data out of the energy meter, and send it to the web back-end. I was called in to an end-of-week meeting with the software team, and was asked to design a board that incorporated all the components they were working with. Since I had no life at the time, I worked through the night and finalized the design of what would be the Powrz board.

I worked with Powrz board for three years, and have a lot more to say about it. Check back for updates.


### Gridz
#### Commercial and Industrial Energy Storage
This was JLM's commercial and industrial energy storage offering. The systems ranged in size form with the largest being distributed among several shipping containers and providing hundreds of kWh of storage. The upfront design for these systems was largely driven by the sales team, so each system was unique in some way, but they all featured some form of power converter, energy meter, and BMS. Most of the system were installed to reduce demand charges, but some were built for time-of-use mitigation, or PV smoothing applications. Powrz served as the controller for all systems, communicating with all the subsystems via its many interfaces.

Gridz occupied the latter half of my time at JLM, and I have so much to say about it. Too much, in fact, and I'm having trouble organizing it all into prose. Since this is just a rough draft, I'm not too worried about it. It's not like anybody is going to read this anyway. If by chance you are reading this, and you are not me, check back soon. I'll have some updates. This was a really cool project and I did a lot of clever trick to solve some weird problem that you may enjoy reading about.

## Simpl Global Inc.

JLM Energy went under in 2018, and Simpl Global was founded by it's former CEO in an attempt to revive and improve upon some of JLM's more promising products. I worked at Simpl from it's founding to March, 2020.

### SimplBox
This company is still around, so I'll just link to their website for a general overview of the product. [https://simplglobal.com/products/simplbox](https://simplglobal.com/products/simplbox)

This being a three person company, I was highly involved in every step of this product's development. Starting with the usual stuff like PCB design and firmware architecture, but also venturing into less familiar terrain with back-end web development, and interfacing with contract manufacturers. Check back soon for more details.

