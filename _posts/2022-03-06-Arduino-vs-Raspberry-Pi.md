Every now and then I get an itch to do some lower level programming and physical stuff, and every time I forget the difference between a Raspberry Pi and an Arduino. 

My basic understanding is that the Pi is basically just a small computer.
It has its own unix flavored OS and you can use a GUI. 
An arduino on the other hand is something different, it's probably more what we think about when we think about low level embedded systems. 

The difference basically is that an Arduino is a microcontroller and a Pi is a microprocessor. (Great, but what does that mean?)
A microprocessor is a processor that's all in one integrated circuit. 
And a processor (CPU) is composed of an arithmetic linear unit (ALU), control unit (CU), and memory.

** TODO what actually do those do and how do they interact? how do we make it more real? what does assembly code do w/r/t them? **

One thing that confused me as I was learning was that a microcontroller contains a microprocessor. 
And yet if you look at the prices for Arduino and Pi, you might think it was the other way around.
Arduino is expensive because you're not just paying for a microcontroller. Really you're paying for a good prototyping experience
by supporting the team that makes the IDE. I took it for granted that you can buy an Arduino and get a LED flashing within 5 minutes. 
That's not a typical microcontroller experience. 

Microcontrollers are not a one-size-fits-all kind of thing. I've been working with python and JVM languages for so long that I've just totally taken
the abstraction layer for granted at this point.

You can write Arduino code using essentially C++. The Arduino system compiles that C++ to assembly that targets their Arduino microcontroller. 
You can't write Arduino code and just throw it on any microcontroller.

