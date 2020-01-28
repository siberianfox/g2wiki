This walk through will show how to assemble an Arduino Due with 4 lower power stepper motors, configure them for operation, install g2core on the Due, and then install + configure useful extra hardware (limit switches).

The same principle can be applied to higher power ("external") stepper motors, though it's a much better idea to [buy a  shield for the Due specifically for this purpose](https://webshop.djuke.nl/kits/g2shield-external-kit).

The significant pieces we'll be working with in this guide:

* [Arduino Due](https://store.arduino.cc/usa/due)
* 4 x [lower power stepper motors](https://www.omc-stepperonline.com/nema-17-stepper-motor/nema-17-bipolar-45ncm-63-74oz-in-1-5a-42x42x39mm-4-wires-w-1m-cable-and-connector.html)
* 4 x [lower power stepper motor drivers](https://www.pololu.com/product/1182)
* A [power supply](https://www.omc-stepperonline.com/power-supply/150w-24v-65a-115230v-switching-power-supply-stepper-motor-cnc-router-kits-s-150-24.html), for the stepper motors
* 3 x limit switches (TBD)

General supporting pieces we need for the above to work:

* Something to mount the stepper motor drivers on.  We're using a [common breadboard](https://www.jaycar.com.au/arduino-compatible-breadboard-with-830-tie-points/p/PB8815) in this instance.
* [Jumper leads](https://www.jaycar.com.au/jumper-lead-mixed-pack-100-pieces/p/WC6027) to connect things
* A USB-A to micro-USB cable, for connecting my computer to the Arduino Due 