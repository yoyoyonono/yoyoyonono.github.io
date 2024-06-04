# The DRS Pad Write-up

Last updated: 2024-06-04

Yes, finally, this is it.

This is not really a "How to build my pad" guide, but more a "How to design and build the pad ***you*** want" guide. There are things I would do differently, and probably things you would want differently as well. I've tried to include enough information so that you know what you need to consider when designing and building your own.

## Several exposition things that are important but you might already know

This is the bare minimum to achieve playability.

### The IR frame

This setup will get you playing as quickly and as cheaply as possible.

#### Materials required

- IR frame (98")

The original game uses an IR Frame with a non-standard aspect ratio. Off-the-shelf IR frames are almost always 16:9, so you will need to a frame of a different size. A 98" frame is the recommended size as it has the closest width while having extra length. A 55" frame placed in landscape would also have the correct width but would be much shorter.

#### Software

Spice2x has a hook which takes the Windows Touch events and sends those to the game. This should be on by default. If using the IR Frame in portrait orientation, then enable `Swap X/Y`.

The IR frames have great amount of latency (~70ms) so use the in-game offset options or a DLL hook to calibrate and set that properly for your setup.

### Adding a depth sensor

This is technically not part of the pad but useful information nonetheless.

#### Materials required

- Intel RealSense SR300/D415
- USB 3 Extender (SR300)/USB 3 Type-C cable (D415)

SR300 units are out of production but are available relatively cheaply secondhand, and work plug-and-play. D415 units are used in the original cabinet but require special firmware and drivers to be used.

Both cameras require a USB 3 connection with the computer to function, so ensure that any cables support USB 3.

#### Software

SR300 drivers and the testing tool are available from Intel's website. D415 drivers for DRS are not publicly available.

## The part you're here for

Everything to this point should have been review or at least not too much to learn. Ensure that the game is playable as described above before continuing.

Most of the work from here is related to adding the lights and having them work with Spice2x.

### Wood and Glass

This part is reasonably simple.

Find a sheet of tempered glass or polycarbonate that is roughly the right size. I used 4' by 6' by 12 mm tempered glass. This is not quite wide enough for the IR frame, but it works fine as long as you have something else to hold it up.

For the wood, I used plywood but MDF would probably work better. A 4' by 8' sheet was more than enough to make the pad.

Use a router or CNC machine to cut 38 Grooves slightly longer than each LED strip, about 1 cm wide, and 0.5 cm deep into the wood. Then, cut the excess completely off. Paint the wood if desired.

### Electronics

#### Materials required

- Raspberry Pi Pico
- WS281x (Neopixel) LED Strip (30 LEDs/m)
- 24V/12V/5V Power Supply

Pi Pico clones are recommended over original units as they usually have reset buttons and USB Type-C plugs which offer great convenience over the original design. Otherwise, there is no functional difference.

All WS281x strips will work with the same input, as long as you use RGB and not RGBW. WS2815 is recommended because it runs on 12V and thus uses less current. WS2013 is recommended over WS2812 for 5V because it includes the backup data line.

Apparently a 24V version exists as well. That would be desirable for the current savings and even further reduced voltage drop. However, all I have been able to find suggests that these are only addressable three at a time.

Calculate the required current for your power supply based on the operation voltage of the LEDs. For 12V, I used a 30A power supply and that has worked well.

#### Wiring

The pad has 38 columns of LEDs, 49 rows tall.

Most Pi Pico Neopixel libraries use one PIO state machine per strip. Operating under this assumption, my pad uses 7 groups of 5 strips, and 1 group of 3. This works, but requires a lot of extra wiring unless you can get your strips pre-cut to 49 with connectors.

The solution I ended up using would allow controlling up to 26 strips, and possibly more if your board exposes the other GPIO pins. So, I would recommend using 12 groups of 3 strips, and one group of two. This allows you to use a 5 m reel by only cutting 3 LEDs off the end. Doing this also saves a considerable amount of RAM on the Pico.

The relevant GPIO pins on the Pico should be connected to the DI lines on the first strips of each group.

At 12V, power injection at multiple points within the group did not seem to be necessary, but you will likely need it at 5V.

#### Assembly

Cut the LEDs to strips of 49 each. Then, arrange them inside the grooves in the wood, alternating orientation, but remember that every strip in a new group should be pointing downwards. Solder the strips together, making sure that the wires are cut to different lengths so that they can bend properly.

Create, as a PCB or on perfboard, a circuit which routes the relevant GPIO pins to a connector of some kind. I used a JST XH connector because I had those lying around. Each wire coming out of the other end of that connector will go to each of the LED strips. They usually use JST SM connectors, but I didn't have any, so I used normal 2.54 mm pin headers and hot glue.

Connect power and ground to your power supply. I used a terminal block. If you have crimp connectors you should use them, especially for the lines coming from the power supply. Connect the Pi Pico's ground to the ground of the power supply here as well.

#### Software

Clone the [drs-pad](https://github.com/yoyoyonono/drs-pad) repository.

Open the `pico` folder and compile the project. Upload the resulting UF2 file to the Pico.

In the `py` folder, edit `main.py` and change the serial port to whichever one the Pico is using.

Set Spice2x to use port 9001 for the API.

#### Notes

Be aware of the USB controller that the Pi Pico is plugged in to. Its USB serial connection has been known to be unreliable if it shares a controller with another high-bandwidth device. There may also be issues if using it behind a hub. Try different USB ports on the computer or installing a PCI card.

## Other background information

### Pico data format

The data is 5586 bytes in GRB order and bit-reversed for each LED. Light data is compressed with Brotli.

On startup the Pico waits for any byte from the computer.

#### Main loop

- Pico sends byte to signal ready
- Computer sends compressed length as two bytes big-endian.
- Computer sends compressed message

## Things that may or may not come in the future

Diagrams and technical drawings for everything don't really exist because this was mostly designed and planned in my head. I might make some.

A PCB design for the Pico board would be pretty easy to make as well. You would probably do best to design it yourself to fit your needs though.

The python script isn't great. It uses a ton of resources and could probably be made a lot faster. A Rust rewrite is in the works.
