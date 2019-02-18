title: Week 6 Roundup: Presentation & Risks
template: post.html
date: 2016-10-19

Last Thursday I presented my [project proposal][1] to the rest of the Computing
degree class. I still have a few weeks to write the formal academic proposal,
but explaining my aims and presenting my preliminary research to our lecturers
and the rest of the class was a good reality check. Quick thoughts:

 * Both lecturers seemed to agree with my research so far, and seemed okay with
   my aim, objectives and research question.
   
 * I might need to find more papers relating directly to High-Altitude Balloons
   -- a lot of my literature so far concerns nano-satellites -- and papers on
   testing methods for space things.
   
 * Some of my classmates made me realise that the final testing round -- in the
   case where we get to fly the payload -- will depend on much more than just
   my work, but also temperature, pressure, weather conditions… I need to make
   sure that I can evaluate my work and write the dissertation even if a flight
   is not possible, or does not go well.

I also got confirmation yesterday that Ian Ferguson will be my project
supervisor. His advice for the scope concerns is to slice the project in three
"levels": The lower level is the amount of work I should aim for to get a
passing grade if everything goes wrong, level three is the best case scenario.
What I have so far is:
 
 1. The radio binary protocol is fully designed and can be demonstrated in a
    demo environment, the data bus's software is able to collect data from
    different payloads and forward them to the transmission software.

 2. The radio link is complete and can be demonstrated on short-to-mid-range
    distances, the flight software handles the whole data chain (fetches from
    payloads and transmits through binary radio protocol).
    
 3. The data bus and radio link prototype is built and can be flown, multiple
    demonstration payloads are built and can be connected to the bus, a
    ground-side application can decode packets in real time (command-line or
    GUI). If funding is sufficient, a demo mission is launched.
 
 [1]: /static/files/prop-slides.pdf
