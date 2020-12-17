
# Alex Neiman's Real time diagnostics OBD Scan gauge tool

This is the code for an Arduino-powered tachometer/scan gauge. It uses the CAN bus interface to send requests to the ECU using OBD protocol. Currently configured for TOYOTA Prius 2020.

## Resources

Please [visit this project on Instructables](https://www.instructables.com/TachometerScan-Gauge-Using-Arduino-OBD2-and-CAN-Bu/) for further instructions.

Desktop utility is [available here on GitHub](https://github.com/neiman3/obd_rtd_desktop).

## Disclaimer
YOU ARE OPENING A SERIOUS CAN OF WORMS HERE...

Messing around with your vehicle/can bus can cause damage and/or unintentional operation of your vehicle! I am providing this code and instruction to you with no warranty or guarantee. You assume all risk. Try this while the vehicle is PARKED FIRST! This project is actually fairly safe if you simply do a little research about what you are doing. If you don't know what any part of the instructions or code means, it's probably a sign that you should not be messing  with your car's electrical system! Don't take this project on. Buy a premade bluetooth OBD dongle and use that instead.

I wish you good luck...

# Operation and Explanation
Works like this: Arduino sends out a msg on the CAN bus with a CAN ID of `0x7DF`. That means "request". Encoded in that message is your PID that you are requesting.Look at function called requestDataOBD(). See the third byte set to var pid? That's the PID you are asking for. The ecu will then send a return message on the CAN bus with an ID of `0x7eA` with your requested PID encoded. That is what the Arduino is listening for! 

Each PID corresponds to a different measurement. Look at the #define starting w/ PID to see some various different values that can be requested from the ecu. If you're getting wrong/no data, try changing the PID that you are requesting. To find PIDs for your vehicle, you can visit [Excel sheet of codes for Prius and some Subaru](https://docs.google.com/spreadsheets/d/1QYWdWkLg0O4tg-ANYdTwwEMhpjkYAI_5JpBfTR1JfQ0/edit?hl=en&hl=en#gid=6) (switch to appropriate tab on sheet for your subi/toy) and [Wikipedia: general codes for most vehicles](https://en.wikipedia.org/wiki/OBD-II_PIDs#Service_01). Keep in mind that you must adjust the draw functions for size/position so that the values fit on the screen.
| Device | Pin | Bus |
|--|--|--|
| MCP2515 | 13 | SCK |
|  | 11 | MOSI |
|  | 12 | MISO |
|  | 10 | CS |
| OLED | A4 | SCL |
| |A5 | SDA|
| Both | VCC | 3.3v / 5v
| | GND | GND|
 
 OLED not showing up? First upload an I2c scanner sketch. Google it if you need the code. If it shows up on the scanner, make sure your i2c addres in THIS sketch has been changed to match the address of your oled. That could be the issue!
 
Hook the CAN Hi and Lo up to the CAN bus of your car. Next, hook the GND into the vehicle. There are signal gnd and chassis gnd in the obd port by default. I used both of those as ground. Next, you need to apply power to system. I used a fuse tap on the windshield wiper circuit since my prius has many circuits fused in a box right next to OBD port. You can use the Vbatt on the OBD port, but beware! That is hooked to the battery, so the device will never turn off. Depending on the current draw of the arduino, it may kill your battery in a matter  of days or weeks if you let your car sit! You may have a vehicle with an ACC/ignition power pin on the OBD port. Lots of pins are manufacturer specific. However, don't just try the multimeter  and hook your arduino up to anything with 12 volts! Your vehicle may have a J1699 or other serial bus that uses 12v logic level. Who knows what a current draw would do! Check with oscilliscope  to make sure your "12volt source" is a steady DC power supply, not a signal, before you hook arduino power to any other pin besides batt on the OBD port. Once you hook everything up, run a wire up the steering column (or even thru the dash) for your OLED. Mind knee curtain airbags! Parking brake. gas and brake pedal...don't be stupid. Don't run the wire in front of anything important. I have 3d printable housing designs. one for the microcontroller and one for the oled. It is nicely sloped so that the screen is perfectly angled with my dashboard above the steering wheel. Try designing your own for a custom place in your vehicle! NOTE ON OPERATION: The device will poll for RPM data every 1,430 ms. It will poll for engine temp every 9,930 ms. I chose these values so that there is a low chance of two signals being sent within a short time of each other. I have found that the CAN bus transciever has a hard time reporting multiple responses when multiple CAN bus frames are recieved at one time. The important takeaway is this: the engine temp only updates at roughly ten second intervals. When  you turn the device on, there will be no data for temp. I set the default value for temp to be 999. If the arduino sees this value, it will not be drawn. However, the RPM updates once a second. By the time the welcome screen has dissapeared, the 1430ms period has elapsed, and the arduino will immediately request engine RPM. My point is this: if you turn your car on and see 9999 RPMs, something is wrong! That means that the arduino is not communicating with your vehicle. You should see engine temp come on the screen after about ten seconds.
