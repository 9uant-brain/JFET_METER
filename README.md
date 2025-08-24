# JFET_METER
## Why I made this
JFETs are tricky—and that very quirkiness is what makes them irreplaceable, especially in audio. How do I deal with it? I usually measure and bin parts by Idss. Even devices from the same production lot can vary widely.

That said, there’s a parameter more informative than Idss: Vgm (i.e., transconductance, gm). Because a JFET is a voltage-controlled device, gm—the slope of the Id–Vgs curve—captures the device’s sensitivity to gate-voltage control. Idss is still the more commonly cited spec, even though it’s just the maximum drain current at Vgs = 0. There are good reasons for that:

1. Idss is much easier to measure. It’s simply the drain current at zero gate bias. Estimating gm requires measuring several Id points at different Vgs values to determine the slope.
2. Idss correlates with gm, so it’s a useful and valid proxy for quick screening, despite being simpler.


To measure Idss, you need to set up a basic JFET circuit, measure the voltage, and calculate the current using Ohm’s law. I could have done this with just a multimeter and a breadboard, but I wanted a more convenient and sophisticated way to measure Idss — especially when dealing with multiple JFETs.

I could have simply bought a cheap Chinese tester, but I also wanted to test myself to see if I could actually build it. The cost saving was just a bonus. 

## Hardware setup
To automate the Idss measuring process, I needed a microcontroller. The Arduino Nano was a perfect fit for this purpose due to its low cost, versatility, and ease of use—I could get one for just $2.

I also needed a display to easily show the measurement results. This setup, consisting of only an Arduino and a display, is sufficient for measuring Idss. The diagram and circuit below illustrate the hardware configuration:

<p align="center">
  <img src= asset/diagram.png width="80%" height="80%">
</p>

<p align="center">
  <img src= asset/sch.png width="80%" height="80%">
</p>

The circuit is quite simple. D3 digital pin pulls the gate to ground, turning the JFET on. Then, A1 analog pin reads the drain voltage (Vd). Using Ohm’s law (I = V/R), the current (Idss) is calculated internally, where V = Vcc - Vd and R = R_Drain. The result is displayed on an OLED screen via the I²C protocol.

## Hardware Implementation

Since this is not complex circuit, I built this on prototype pcb. And components are connected with jumper wire. You can see the completion in the picture below. You might see red module and header parrell sockets too, they are used for opamp tester (I am working on). Arduino and modules are attached with header socket.

<p align="center">
  <img src= asset/C1.jpg width="50%" height="50%">
</p>

<p align="center">
  <img src= asset/C2.jpg width="50%" height="50%">
</p>

There is a special gimmick on this board. You can see I highlighted 3 holes on the board, it is actually improvised TO92 transistor socket. Because at the time I built this board, I didn't have sockets. Above on it, you also can see 3 pin socket, but it was soldered later to attach bent lead transistors.



