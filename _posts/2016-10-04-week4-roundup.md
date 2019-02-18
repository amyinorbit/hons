---
title: "Week 4 Roundup: Literature"
date: 2016-10-04 18:12:00
layout: post
---

This week I've worked on refining my research question and looking for papers
that support the project better.

## Research Question

"What are the advantages, issues and obstacles in designing a generalised
intra-payload and long-range communications platform for High-Altitude Balloon
scientific missions?"

## Literature

  * ECSS Secretariat. 2010. [_SpaceWire - Remote memory access protocol_][1].
    European Cooperation for Space Standardization.
   
    Spacewire as a whole, and the remote memory access protocol are standard
    used in high-end space probes and satellites to create complete networks
    between instruments. I would have to design a similar, albeit simpler system
    for the intra-payload part of the project
    
  * Hinschelwood et al. 2015. [_A Raspberry Pi Weather Balloon_][2]. Young
    Scientists Journal (17), pp. 20-24.
    
    This paper is a report of a high-school Raspberry Pi-based HAB project. The
    students adopted moderately modular approach – the payloads' code and the
    flight software are still fairly tightly coupled. It shows low-power
    computer platforms are usable as flight computers.
    
  * Volstad, M. 2011. [_Internal Data Bus of a Small Student Satellite_][3].
    Norwegian University of Science and Technology.
    
    In this paper, the author goes over the design (mostly electronic, but also
    software-wise) of the internal bus of a CubeSat, over which all payloads
    communicate with the flight computer.
    
    Some parts are specific to satellite operations (radiation tolerant design),
    while others could be used in High-altitude balloon: I2C data bus, reset
    circuitry to allow the flight computer to isolate a faulty payload.

    
   [1]: http://forums.ecss.nl/forums/ecss/dispatch.cgi/standards/showFile/100770/d20100209121656/No/ECSS-E-ST-50-52C(5February2010).pdf
   [2]: http://search.proquest.com/openview/56b604650b11f0465d595c558052f79b/1?pq-origsite=gscholar&cbl=226453
   [3]: http://www.diva-portal.org/smash/get/diva2:473592/FULLTEXT01.pdf