# JFET_METER
## Why I made this
JFETs are tricky—and that very quirkiness is what makes them irreplaceable, especially in audio. How do I deal with it? I usually measure and bin parts by Idss. Even devices from the same production lot can vary widely.

That said, there’s a parameter more informative than Idss: Vgm (i.e., transconductance, gm). Because a JFET is a voltage-controlled device, gm—the slope of the Id–Vgs curve—captures the device’s sensitivity to gate-voltage control. Idss is still the more commonly cited spec, even though it’s just the maximum drain current at Vgs = 0. There are good reasons for that:

1. Idss is much easier to measure. It’s simply the drain current at zero gate bias. Estimating gm requires measuring several Id points at different Vgs values to determine the slope.
2. Idss correlates with gm, so it’s a useful and valid proxy for quick screening, despite being simpler.


To measure Idss, you need to set up a basic JFET circuit, measure the voltage, and calculate the current using Ohm’s law. I could have done this with just a multimeter and a breadboard, but I wanted a more convenient and sophisticated way to measure Idss — especially when dealing with multiple JFETs.

I could have simply bought a cheap Chinese tester, but I also wanted to test myself to see if I could actually build it. The cost saving was just a bonus. 

## System Overview
To automate Idss measuring process, I needed MCU. Arduino nano was seemed perfect for the purpose, because it is very cheap, versatile and easy to learn to use. I could get it just for $2. Also I needed a display to make the measurement values easy to read. Those are sufficient just for measuring Idss.
