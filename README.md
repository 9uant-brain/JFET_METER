# JFET_METER
## Why I made this
JFETs are tricky—and that very quirkiness is what makes them irreplaceable, especially in audio. How do I deal with it? I usually measure and bin parts by Idss. Even devices from the same production lot can vary widely.

That said, there’s a parameter more informative than Idss: Vgm (i.e., transconductance, gm). Because a JFET is a voltage-controlled device, gm—the slope of the Id–Vgs curve—captures the device’s sensitivity to gate-voltage control. Idss is still the more commonly cited spec, even though it’s just the maximum drain current at Vgs = 0. There are good reasons for that:

1. Idss is much easier to measure. It’s simply the drain current at zero gate bias. Estimating gm requires measuring several Id points at different Vgs values to determine the slope.
2. Idss correlates with gm, so it’s a useful and valid proxy for quick screening, despite being simpler.


To measure Idss, you need to set up a basic JFET circuit, measure the voltage, and calculate the current using Ohm’s law. I could have done this with just a multimeter and a breadboard, but I wanted a more convenient and sophisticated way to measure Idss — especially when dealing with multiple JFETs.

I could have simply bought a cheap Chinese tester, but I also wanted to test myself to see if I could actually build it. The cost saving was just a bonus. 

## Hardware setup
To automate the Idss measuring process, I needed a microcontroller. The Arduino Nano was a perfect fit for this purpose due to its low cost, versatility, and ease of use — I could get one for just $2.

I also needed a display to easily show the measurement results. This setup, consisting of only an Arduino and a display, is sufficient for measuring Idss. The diagram and circuit below illustrate the hardware configuration:

<p align="center">
  <img src= asset/diagram.png width="80%" height="80%">
</p>

<p align="center">
  <img src= asset/sch.png width="80%" height="80%">
</p>

The circuit is quite simple. D3 digital pin pulls the gate to ground, turning the JFET on. Then, A1 analog pin reads the drain voltage (Vd). Using Ohm’s law (I = V/R), the current (Idss) is calculated internally, where V = Vcc - Vd and R = R_Drain. The result is displayed on an OLED screen via the I²C protocol.

## Hardware Implementation

Since this is not a complex circuit, I built it on a prototype PCB, connecting the components with jumper wires. The completed circuit can be seen in the picture below. You may also notice the red module and parallel header sockets, which are used for an op-amp tester I am currently working on. The Arduino and other modules are attached using header sockets.

<p align="center">
  <img src= asset/C1.jpg width="50%" height="50%">
</p>

<p align="center">
  <img src= asset/C2.jpg width="50%" height="50%">
</p>

There is a special feature on this board. I highlighted three holes on the PCB, which actually serve as an improvised TO-92 transistor socket, since I didn’t have proper sockets at the time I built it. Just above it, you can also see a three-pin socket that was soldered later to accommodate transistors with bent leads. I will explain how I improvised this in the next paragraph with pictures.

<p align="center">
  <img src= asset/C3.jpg width="50%" height="50%">
</p>

<p align="center">
  <img src= asset/C4.jpg width="50%" height="50%">
</p>
It is simple. I lined up the TO-92 leads, marked the spacing, and drilled the holes with thin drill bits. Then, I inserted leftover leads through the holes and soldered them in place. They don’t hold the transistor leads tightly, but if I tilt the transistor while it’s plugged in, they make sufficient contact.

## Coding Process

What is essential purpose of this hardware? It is measuring. But, arduino measurement varies because it only can measure relatively with VCC. And VCC, it fully depends on usb port voltage which is powered by PC or usb adapters. They aren't same. Because of it, I had to add calibration code.

### Calibration

In arduino nano chip (ATmega328P), there is internal reference voltage (1.1V ±10%) mostly constant regardless of condition. So, if I compare this reference with VCC, I can figure out VCC value. If I measure internal voltage, it varies with VCC, as I explained, every voltage measurement are relativtiy with VCC. So, let's assume we measure internal voltage. Even internal 1.1V is still 1.1V, if exact VCC value didn't defined, its measurement never shown as 1.1V. But, we can work backwards with it. 
If the measurement is higher than 1.1V, VCC have to be higher (measurement∝1.1/VCC). On the other hand, if the measurement is lower than 1.1V, VCC have to be lower (measurement∝1.1/VCC).
