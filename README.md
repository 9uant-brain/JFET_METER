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
  <img src= asset/diagram.png width="78%" height="78%">
</p>

<p align="center">
  <img src= asset/sch.png width="78%" height="78%">
</p>

The circuit is quite simple. Gate and source are tied to GND to ensure Vgs = 0, keeping the channel fully open. Then, A1 analog pin reads the drain voltage (Vd). Using Ohm’s law (I = V/R), the current (Idss) is calculated internally, where V = Vcc - Vd and R = R_Drain. The result is displayed on an OLED screen via the I²C protocol.

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
It is simple. I lined up the TO-92 leads, marked the spacing, and drilled the holes with thin drill bits. Then, I inserted leftover leads through the holes and soldered them in place. Even if they don’t grip the transistor leads tightly, tilting the transistor while it’s plugged in makes sufficient contact.

## Coding Process

The essential purpose of this hardware is measurement. However, Arduino measurements vary because they are only relative to VCC. And VCC itself fully depends on the USB port voltage, which comes from either the PC or USB adapters. These voltages are not always the same. Because of this, I had to add calibration code.

### Calibration

In the Arduino Nano (ATmega328P), there is an internal reference voltage of 1.1 V (±10%) that remains mostly constant under various conditions. By comparing this internal reference against VCC, we can estimate the actual VCC value.

Normally, all analog voltage measurements are referenced to VCC. Therefore, when we attempt to measure the internal reference itself, the result is influenced by VCC. Even though the internal reference is fixed at 1.1 V, without knowing the exact value of VCC, the measured result will not appear as 1.1 V.

However, this relationship can be used in reverse. If the measured value of the internal reference appears higher than expected, it indicates that VCC is higher (since the measurement ∝ 1.1 / VCC). Conversely, if the measured value is lower, it means VCC is lower. This inverse proportionality allows us to indirectly calculate the actual VCC.

---
Before implementing the calibration logic, I discovered that the internal reference voltage varies slightly between devices. To make the measurement more precise, I needed to determine the actual internal voltage. Since it cannot be measured directly through physical pins, I had to infer it using ADC raw data.

The relationship is: ADC measurement ∝ 1.1 (internal reference) / VCC

Unlike the internal reference, VCC can be measured directly through physical pins, and it stays constant unless a different USB port or power source is used. Therefore, by using the known VCC value, I can reverse the relationship to calculate the actual internal reference voltage for calibration.

```cpp
int readBandgapRaw() {
  ADMUX = _BV(REFS0) | 0x0E; // select internal 1.1V bandgap channel, reference = AVcc
  delay(2);                  // give it a moment to settle after switching channel
  ADCSRA |= _BV(ADSC);       // kick off conversion
  while (ADCSRA & _BV(ADSC)) { }  // wait until conversion is done
  return ADC;                // raw ADC value (0~1023)
}
}

void setup() {
  Serial.begin(9600);
}

void loop() {
  // 100 count averaging of internal bandgap raw
  long sum = 0;
  for (int i = 0; i < 100; i++) {
    sum += readBandgapRaw();
    delay(2);
  }
  int raw_avg = sum / 100;

  const float VCC_MEASURED = 4.37f;   // ── Directly measured VCC with multimeter
  float bandgap_V = (VCC_MEASURED * raw_avg) / 1023.0f;

  // ── Print result
  Serial.print("ADC raw(avg) = ");
  Serial.print(raw_avg);
  Serial.print(" , Bandgap(calc) = ");
  Serial.print(bandgap_V, 3);
  Serial.println(" V");

  delay(1000);
}
```
This is a simple code for measuring the internal voltage. It loads the internal voltage raw data (0–1023) and averages it over 100 samples. Using the formula ```ADC raw = Vinput / VCC * 1023```, we can rearrange it to calculate the internal reference. When reversed, the formula becomes:
```internal voltage = (VCC_measured * raw_avg) / 1023```

<p align="center">
  <img src= asset/print.png width="50%" height="50%">
</p>
This is the serial output from the code. It reports the bandgap voltage (the internal reference) at about 1.1 V. It is essentially the same as the nominal value, but it was worth confirming, since the actual value can vary from chip to chip.

### Main code

```cpp
#include <Arduino.h>          
#include <Wire.h>             // I2C communication library
#include <U8g2lib.h>          // U8g2 library for OLED display

// Pin definitions
const int PIN_DRAIN = A1;     // Analog pin A1: measure drain voltage
const int R_D       = 999;    // Resistor value (Ohm) between VCC and drain
float VCC;                    // Supply voltage (to be estimated)

// OLED display object (SSD1306 128x64, I2C, no reset pin)
U8G2_SSD1306_128X64_VCOMH0_F_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE);

// Function to read internal 1.1V bandgap reference
int readBandgapRaw() {
  ADMUX = _BV(REFS0) | 0x0E;      // Select internal 1.1V bandgap channel, reference = AVcc
  delay(2);                       // Allow settling time after switching channel
  ADCSRA |= _BV(ADSC);            // Start ADC conversion
  while (ADCSRA & _BV(ADSC)) { }  // Wait until conversion is finished
  return ADC;                     // Return raw ADC value (0~1023)
}

void setup() {
  Serial.begin(115200);           // Initialize serial communication at 115200 baud

  u8g2.begin();                   // Initialize OLED display
  u8g2.enableUTF8Print();         // Enable UTF-8 print (safe even if not used)

  VCC = (1.1 * 1023.0) / readBandgapRaw();  // Estimate VCC using bandgap method
}

void loop() {
  delay(100);                     // Delay for stability after FET setup

  int raw = analogRead(PIN_DRAIN);            // Read ADC value from drain node
  float Vd = raw * (VCC / 1023.0);            // Convert raw ADC value to voltage

  float Idss = (VCC - Vd) / R_D;              // Calculate drain current (A)
  float Idss_mA = Idss * 1000.0;              // Convert to mA

  Serial.print("Vd = "); Serial.print(Vd, 3);               // Print drain voltage
  Serial.print(" V, Idss = "); Serial.print(Idss_mA, 2);    // Print drain current
  Serial.println(" mA");

  u8g2.clearBuffer();             // Clear OLED buffer
  u8g2.setFont(u8g2_font_ncenB08_tr);  // Set font

  u8g2.setCursor(0, 12);          // Set cursor position
  u8g2.print("JFET Idss Test");   // Print title

  u8g2.setCursor(0, 28);          
  u8g2.print("Vd: "); u8g2.print(Vd, 2); u8g2.print(" V");   // Print drain voltage

  u8g2.setCursor(0, 44);
  u8g2.print("Idss: "); u8g2.print(Idss_mA, 1); u8g2.print(" mA"); // Print Idss

  u8g2.sendBuffer();              // Send buffer content to OLED

  delay(200);                     // Small delay before next loop
}

```
<!--This main code includes  measuring process, voltage calibration I explained by this far. It reads internal voltage, calibrates VCC with it. And with calibrated VCC, It can measure Vd correctly, to calculate Idss. And print it on the OLED module. The OLED display object(U8G2_SSD1306_128X64_VCOMH0_F_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE)) is preference from "https://github.com/olikraus/u8g2/wiki/u8g2setupcpp". Even If they are same display driver(SSD1306), same resolution (128X64), there is three versions of them (NONAME / VCOMH0 / ALT0). As SSD1306 modules slightly differ by manufacturer, and the seller don't know about which version works, I tried them all. And, VCOMH0 worked for me.-->

This main code includes the **measuring process and voltage calibration** I described earlier.
It reads the internal bandgap reference voltage and uses it to calibrate VCC.
With this calibrated VCC, the drain voltage (Vd) can be measured accurately,
which allows the calculation of Idss.
The results are then printed on the OLED module.

The OLED display object:

    U8G2_SSD1306_128X64_VCOMH0_F_HW_I2C u8g2(U8G2_R0, /* reset=*/ U8X8_PIN_NONE);

was chosen based on the reference from [the official U8g2 setup page](https://github.com/olikraus/u8g2/wiki/u8g2setupcpp).

Even though these modules all use the same display driver (**SSD1306**) and the same resolution (**128×64**),
the library provides three versions of the constructor: **NONAME / VCOMH0 / ALT0**.
This is because SSD1306 modules differ slightly depending on the manufacturer,
and the seller often cannot specify which version will work.
I tested all three, and for my module the **VCOMH0** version worked correctly.

## Hardware Demonstration
You can watch the demonstration video through the image link below. I combined two videos into one.

In the first part, I demonstrate VCC calibration. Since no JFET is inserted in the socket, Vd directly represents VCC. The first cable shows a slightly lower voltage because it is powered by a USB hub, while the second cable shows a higher voltage as it is powered by a DC adapter.

In the second part, you can see how Idss varies among different JFETs. I demonstrated both types of sockets: one for bent leads and the other for straight leads.
[![Video Label](http://img.youtube.com/vi/WLm-msQ3tJI/0.jpg)](https://youtu.be/WLm-msQ3tJI)
