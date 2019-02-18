---
title: "Week 1 Roundup: Project Introduction"
date: 2016-09-13 22:05:00
layout: post
---

After one week of preliminary research into the four ideas I had, I have decided
to keep my initial choice, and do something about space communications:

> *Long-Range Communications*: design the communication systems and software for
  low-budget, low-power payloads, like High-Altitude Balloons and NanoSats. This
  would involve investigating existing systems, implement (and potentially 
  design) the protocol used to transmit data over long distance and create the
  software needed at both ends of the transmission (payload and ground station).
>
> [A week ago][self-intro]

Since then, I have made some progress on choosing exactly why I'd like to do,
and how I want to do it.

## Context

For a few years now, a group of lecturer and students at Abertay has wanted to
build and launch a high-altitude balloon. To keep it simple, they are large
balloons filled with helium, carrying a polystyrene-enclosed payload, usually
recording at least its altitude and GPS co-ordinates, and usually some
instrument readings — temperature, images, anything really.

![A High-Altitude Balloon launch][img-hab]  
*Image by [Fraser Reid (CC-BY)][img-hab-cp]*

Balloons like these can reach up to 40 km in altitude, where the low pressure
lets the helium stretch the balloon more and more until it bursts. Then, the
payload falls back to the ground under a parachute, hopefully not too far from
where it took off.

To make sure the payload can be received, most balloons transmit their telemetry
(position, altitude and readings) to one or multiple ground stations through a
radio link. That's where it gets fun.

## Long-Range Radio

Balloons usually climb up to 30-40 km, and can drift quite a bit due to winds.
Depending on the conditions, it's not impossible for payload to be more than
200 km from the receiver, which makes reliable transmissions challenging.

The range of a transmitter-receiver couple depends on a few factors: the shape
of both antennae, the transmission power and the line-of-sight.

In the USA, the amateur radio license allows you to transmit from airborne
stations (which is exactly what the balloon is). This allows the use of fairly
powerful transmitter (up to 5 watts).

In the UK, we don't have such luck. A few frequency bands are considered
*license-exempt* and can be used airborne, as long as a certain power limit of
10 mW is not reached. Because of the low power, long-distance communications
require good antennae, and a data format that is not too vulnerable to noise,
or at least provides a way to correct errors.

## Flight Computers

In most High-Altitude Balloon projects, the payload is made one of two ways:
either a custom circuit board is designed for a specific mission, or low-cost
boards (Beaglebone, Raspberry Pi, Arduino) is used with custom software running
the data collection and transmission.

This goes against what is done for example on most satellites: save for
high-profile missions where everything needs to be controlled, most satellites
today are based on buses: the manufacturer sells a series of standard platforms
that host radio communications, power and control systems, and provide an
interface for the client's payload to tap into those systems.

## The Project

For the project, I'd like to work on communications, applied to low-cost systems
(High-Altitude Balloons, maybe nano satellites simulations). The project would
be split in two parts:

 * Design a modular flight computer based on low-cost computers (Raspberry Pi
   for example) that would provide power and communications to instruments. Any
   payload that follows the specification could be connected to the bus with as
   little work as possible.
   
 * Design the radio system used by the bus to transmit arbitrary data packets
   received from each instrument to the ground. Because of the limitations of
   license-exempt transmitters in the UK, it would have to work even with
   low-power radio links, which means low bandwidth: it will need to be
   efficient in speed and size of packets, and reasonably error-tolerant.

![Rough Concept][img-diagram]

## Preliminary Research Links:

 * PSAS uses amped WiFi to talk to their rockets and nanosats, at speeds beyond
   the speed of sound and ranges exceeding 130 km:
   [link budget evaluation][psas-link], [DxWifi update blog][psas-dxwifi] and
   [GitHub repo][psas-gh].
   
 * The UKHAS association provides numerous High-Altitude Balloon guides focused
   on the UK: [license-exempt transmitter list][ukhas-tx], [intro to long-range
   communications][ukhas-com].
   
 * A few CubeSat projects have public information available on the telemetry
   format they use (most of them seem custom-tailored): [LightSail-1][lsail-1],
   [University of Arizona][sat-uoa], [SEEDS CubeSat][sat-SEEDS].
  
 * CCSDS has established a standard for space communications, the Space Packet:
   [main website][ccsds], [space packet standard][ccsds-sp],
   [OSI-style space layers][ccsds-osi].

   [self-intro]: /2016/setting-up
   
   [psas-link]: http://psas.pdx.edu/research-notebooks/cubesat-linkbudget/cubesat-linkbudget
   [psas-dxwifi]: http://blog.psas.pdx.edu/2014/10/almost-there/
   [psas-gh]: https://github.com/psas/DxWiFi
   
   [ukhas-tx]: https://ukhas.org.uk/guides:radio_modules
   [ukhas-com]: https://ukhas.org.uk/communication:basics
   
   [lsail-1]: http://www.pe0sat.vgnet.nl/2015/lightsail-telemetry/
   [sat-uoa]: ftp://128.196.250.12/pub/pub/cubesat/cubesat_papers/itc.pdf
   [sat-SEEDS]: http://cubesat.aero.cst.nihon-u.ac.jp/image/telemetry/FM_e/FM_Packet_Telemetry_Format_For_SEEDS_English.pdf
   
   [ccsds]: https://public.ccsds.org/
   [ccsds-sp]: https://public.ccsds.org/Pubs/133x0b1c2.pdf
   [ccsds-osi]: https://public.ccsds.org/Pubs/130x0g3.pdf
   
   [img-hab]: /static/img/2016-09/w1-hab.jpg
   [img-hab-cp]: https://www.flickr.com/photos/62569532@N00/7424880108
   [img-diagram]: /static/img/2016-09/w1-diagram.png