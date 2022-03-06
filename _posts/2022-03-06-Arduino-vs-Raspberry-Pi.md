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
You can't write Arduino code and just throw it on any microcontroller. An Arduino really is more than just a microcontroller. In fact, an Arduino _comes with_ a microcontroller, specifically an ATMega. The Arduino is the extra bells and whistles around it that make it easy to prototype things. If you have an electric motor for example, you can just plug its wires into the receptacles on the Arduino board. You could accomplish the exact same thing by putting the ATMega into a breadboard and connecting to it that way.

I want to understand microcontrollers from top to bottom as much as I can. I know that code I write ultimately ends up pushing electrons around but in between gets blurry several times.

- why can't I write python for microcontrollers?
- how do microcontrollers differ from normal computers?
- how would I build a microcontroller myself? (TODO this is just an druino and thoe ATMega is the fucking controller lol)
Mind you, building a "microcontroller" is a pretty general thing. A microcontroller at its heart, is a processor that is attached to several input/output connections that allow it to read and/or drive information from other devices / signals. For this, we'll focus on specifically building an Arduino clone.

- You need a processor. You can use an AVR or ARM processor. 
- You need a crystal. This acts as an oscillator which is basically the clock for the processor. So how does this part work? It's really a crycal and a circuit together. The technical definition of a crystal is a uniform/homogenous substance structured in a geometric pattern. Oscillators specifically use quartz crystals. Quartz is good because it deforms under an electric field. So if you voltage to the crystal and then remove it, the crystal will deform and then release the voltage as it regains its shape. 
As a physical fact, quartz crystals can repeat this process at a consistent frequency, which makes them good for digital clocks. 
- you need some capacitors. Why? In a perfect world, you wouldn't. But they're good for regulating voltage. If you had a perfectly consistent voltage applied to your circuit, you wouldn't need them. But in the real world, voltage is very difficult to keep consistent. If you had a fluctuating voltage, it could fry your system. Capacitors act as tiny little reservoirs to handle the spikes and dips of voltage so that the rest of the system gets a consistent amount.


- microcontroller vs microprocessor?
A processor is what you normally think of when you think of a computer / CPU. It's the "brain" of the computer, and it's where all the instructions route through. A microcontroller _has_ a microprocessor. It also has some specifically chosen extra components that make it easier to embed in a larger system, like robots or appliances. A processor is the brain of the controller, and the extra components are like the limbs. So if you attached those same limbs to a normal computer, you could definitely drive whatever appliances you want. 
The reason we don't do that in practice is because computers are more expensive and come with overhead that we don't need for these specific purposes. 
E.g if you're writing a program for a washing machine, you wouldn't expect to plug it into a car. It doesn't need to be flexible.
