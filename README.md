
This is a differential temperature controller with 8 temperature sensor inputs, 4 digital inputs and 8 opto coupled solid state relay outputs.  The controller is compatible with the [LMT01](https://www.ti.com/product/LMT01) temperature sensor.  Each temperature sensor is connected to the controller with one pair of a cat 5 or 6 cable.  The temperature sensor uses the same 2 wires for both power and signaling.  Each temperature sensor has a max length of 45 meters.

The LMT01 is an ideal temperature sensor for differential temperature controllers as it

* uses digital signaling so no noise is added to the measurement over the cable back to the controller.
* allows for differential signaling for long cables.
* needs only 2 wires (twisted pair) per sensor.  Power and signaling is done over the same wires.

### Sensor Operation

The LMT01 uses a Pulse Train Interface to communicate the temperature where the number of output pulses is proportional to temperature.  The 1 or 0, high or low of each pulse is communicated by switching the device current consumption between two values, aprox 34uA and 125uA.

The controller measures each LMT01 sensor one at a time via U1, ADG707 differential 8-to-1 multiplexer.  When a sensor is muxed, its positive pin is connected to a 3.3kΩ pull up (R17) and its negative pin is connected to a 3.3kΩ pull down (R18).  Each resistor sees about a 300mV change in voltage between the pulse’s high / low 34uA and 125uA change in device current.  When R17’s change is positive, R18’s is negative.  These signal pulses are AC coupled to a differential op-amp circuit (U3A) via C20 and C39.  680pF was chosen to be about half the time constant for the 88Khz pulse.

1 / 88Khz = 11.4uS.  680pF * 10K = 6.8uS.

C20 and C39 must charge before they pass the pulses.  Too large a value looses pulses at the beginning.  680pF was measured to loose about 1 pulse, or about 0.1 C.

The differential op-amp circuit separates common noise from the differential signal that the twisted pair has picked up from noise sources over it’s length.  The differential op-amp circuit has gain to compensate for the signal loss in the cable.  The differential op-amp circuit has a gain of about 5 which is the max value not to saturate the strongest signal, a sensor connected directly to the controller without a length of wire.

[300mV - (-300mV)] * 5 <br>
= 600mV * 5 <br>
= 3v  (U1 has 3.3v supply)

The gain can be increased to just below the saturation point of the shortest cable.  C1 and C2 filter high frequency noise.  The output of the differential op-amp circuit is feed to a comparator with hysteresis (U3B).

The comparator has a hysteresis of 314mV.  A cat-5 twisted pair of up to 45 meters still had enough margin above the 314mV hysteresis for reliable operation.  The output of the comparator is feed to the microcontroller’s external timer counter clock input.

The microcontroller’s external timer counter clock (TCLK0) input feeds both TC0 and TC1 timer counters in the SAM4S2 mcu.  TC0 uses TCLK0 as a clock source and thus counts the pulses from the sensor.  The value of TC0 can be read and converted to a temperature.  TC1 is used to trigger when the LMT01 has finished sending pulses.  TC1 is clocked from the main mcu clock.  TC1 is configured so that its value is reset each time there is an edge on TCLK0.  TC1 compare C register is configured to a value about 2x the normal pulse period of the LMT01.  Each pulse from the LMT01 resets the count in TC1.  When the LMT01 stops sending pulses, TC1’s value grows to match compare C register which triggers an interrupt for the software to read TC0 and then change the mux to the next sensor and start all over again.

### Opto Coupler Ouptputs

Opto coupler AQH0213 is used to drive the coil of a relay that controls the load.  The AQH0213 is apart of a family of opto couplers with the same footprint.  The appropriate opto coupler can be chosen to match the voltage and current requirements of the specific relay for the application.