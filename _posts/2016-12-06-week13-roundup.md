---
title: "Week 13 Roundup: Feasibility Demo"
layout: post
date: 2016-12-06
---

Last week I had my feasibility demonstration with Ian. The goal was to show
the preliminary research I've been doing since September. Since we were supposed
to show a finite number of "artefacts", I picked:

 * A demo of image transmission through radio, from an Arduino, using the
   interrupt-based code I'd developed in [week 7][1]. Not much discussion there
   apart from the difficulty of working with a 50 bauds connection.
 
 * A proof that bytes can be intercepted from the decoded RTTY stream in
   dl-fldigi (code from [week 9][2]). We discussed the alternatives to printing
   hex values (sockets seem like the best way to go about sending the byte
   stream to another piece of software), and the crashes due to the old GUI
   system used by dl-fldigi (timer conflict, it would seem).
   
 * Diagrams of the preliminary packet and frame format from [last week][3] and
   of the [flight software architecture][4].
 
 * Pictures of the hardware used for development (Arduino Uno, radio transmitter
   and the associated circuitry).

The four artefacts are available in the demonstration's [GitHub repository][5].

 [1]: /2016/interrupt-driven-rtty
 [2]: /2016/week9-roundup
 [3]: /2016/week12-roundup
 [4]: /static/files/flightcore-systems.pdf
 [5]: https://github.com/AHABus/feasibility-demo