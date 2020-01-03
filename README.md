# 48-V Input Affordable 1.2-A Output Converter / SY8502

## Overview

![](/tinypol-board.png "Photo of the Fully Automated 48-V Tinypol 1.0")

This design is based around the Silergy SY8502FCC converter.  
[IC Datasheet](/1809231105_Silergy-Corp-SY8502FCC_C87752.pdf). [Board Schematic](/tinypol_color.pdf).  
For layout, see the .kicad_pcb file in the repository.

![](/schematic.png "Schematic of the board.")

## Use

Connect the input side to the power source, and the output side to the load. The converter powers on automatically above approx. 6.2V input. Output is programmed to 5.2 volt, can be changed by replacing R3 and/or R4. Reference voltage is 1.2 V. Supported input voltage is around 56-V, above which the D1 TVS diode will start to clamp. You can remove this to increase maximum input voltage to 85V abs max, but then you should avoid hot-plugging the board.

There are two ways to shut down the converter while input power is present. 
 - Either apply a logic high to input labled OFF (J4) *don't exceed +/- 12V on this pin*, or 
 - pull EN input low. The EN pin is pulled up on-board to VIN with 100kOhm, therefore the circuit you use to discharge EN must withstand the maximum VIN you plan to apply.

 Alternatively, you can disconnect the input.

## Properties

|                          Specifications |                                                                    |
|----------------------------------------:|--------------------------------------------------------------------|
|                             Dimensions: | 25x42x6mm (without pin headers)                                    |
|                                   Mass: | 4.8g (without pin headers)                                         |
|                          Mounting Hole: | M3 at (-6;-4)mm from bottom right corner                           |
|                          Input Voltage: | 6.8V - 56V                                                         |
|                         Output Voltage: | 5.2V (default)<br>Optionally between 1.8V and 24V                  |
|             Internal Reference Voltage: | 1.2V                                                               |
|                          Output Ripple: | 500mVpp with no load<br>80mVpp in Continuous Conduction Mode       |
|                         Output Current: | 1.8A peak<br>1.2A continuous                                       |
|                            Power Stage: | Synchronous buck with 500mOhm (HS) and 240mOhm (LS) nMOS switches  |
|             Overtemperature Protection: | Yes, at 150 degrees Celsius                                        |
|               Short Circuit Protection: | Yes, at 2A                                                         |
|              Fixed Switching Frequency: | No. <br>In Continuous Conduction Mode fSW is approx. 340 kHz       |
|                             Soft Start: | 2ms, fixed in silicon                                              |
| Transient Input Overvoltage Protection: | Yes, with on-board TVS diode                                       |
|          Output Overvoltage Protection: | No.                                                                |
|                       Hot-Plug Capable: | Yes.                                                               |
|                          Soft Shutdown: | Yes (logic high on OFF pin) or<br>Yes (pulling EN pin low)         |
|                    Output ON Indicator: | No.                                                                |
|                      Power Good Output: | No.                                                                |
|                                   RoHS: | Yes, plus tantalum-free.                                           |

![](/render.png "Render of the 48-V PoL with explanations")

## Planned Future Changes

 - Add a capacitor at SYE pin to de-glitch.
 - Add an output indicator LED.

## Test report

The IC looks quite OK! Output has little noise (thanks to the plentiful output MLCCs) but high ripple because of low switching frequency and hysteretic operation. Rectifier MOSFET control looks good. Short circuit test looks good. Startup after short is good.

The SYE (enable) pin does not seem to be properly deglitched internally. I can make it behave erronously by feeding noise into EN. The on-board 100k pullup is strong enough to counteract spurious noise, but you can drive it externally to make it do weird things.

I can tickle the IC to make the output oscillate, but it settles rather quickly. Would not use it to power sensitive circuits directly, without a second regulator.

Driving it close to 2 A output makes it run very hot. The thermal pad below the IC seems to be well soldered and the board is sinking the heat. Inductor heats up quite a bit; a better one would probably help increase efficiency, but it is okay for its typical use case, and you would not get much more current out of this anyway. The board has nice thermal mass!

Tried adding 2200uF electrolytic cap to the output, this did not reduce ripple (because it is determined by the hysteresis of the comparator) but pushed the switching frequency down to the audible range with low load. 

One weirdness I observed: when running it with a hard short at its output the output current is initially clamped at 2A, at which the converter heates up, which changes the overcurrent comparator's threshold. After thermal shutdown kicked in, I let the IC cool down, and repeated the test. Now the output current into a short was only 1A. This did not change after power cycling the IC, and seems to be permanent. I will repeat this test on different boards too, but I want to set up a test script first so I can compare chips before-after, and that needs more work on our lab instrumentation.

Either way, this IC looks very usable, while being much cheaper than any of its alternatives.

### Planned Future Tests:
 - Characterize:
   - Iq over Vin with EN=low and EN=high
   - Efficiency curve (needs work on lab equipment)
 - Increasing Cff to optimize load step behavior
 - Adding C to SYE to test glitch improvement
