# FCHAO-Inverter
Hacking the FCHAO Inverter a.k.a. Peter inverter

After watching [this video](https://www.youtube.com/watch?v=I5rBVGSszBY) regarding the affortable mains inverter. 
Thank you [OffGridGarageAustralia](https://www.youtube.com/@OffGridGarageAustralia) for providing this usefull information.
I decided to buy it for a project which needed a solar charger for an electrical outboard engine.

# The project 
A seascout group decided to replace one of their gasoline engins with and EPropulsion motor which came with this [battery](https://www.epropulsion.com/e-series-batteries) and this 25A/48V [charger](https://www.epropulsion.com/product-page/e-battery-charger-25a/)
The battery unfortunately would only work with its own charger and I did not want to mess with this beacuse of the warrenty
So I needed 230Vac@1250Watt. Therefore I purchased a 24V/2500Watt FCHAO (Peter inverter) to make sure it was only loaded 50% which usually greatly increases the operating time.

To allow switching the inverter on/off based on the load (low when the outboard engine battery was charged), solar storage battery condition and temperature I needed these values as sensor input for a simple Homeassistant system. The information is available on the remote display which is connected via an RS485 connection. I suspected Modbus but unfortunately the protocol was propritary so I had to reverse engeneer it.

# The RS485 protocol 

Wiring Diagram, I used a standard Ethernet cable
| **Wire** | **Pin** | **Function** | **Description** |
| :------------- | :----: | :--------- | :---------------------------------- |
| Orange/White   |   1   | A+         | Positive signal RS485             |
| Orange         |   2   | B-         | Negative signal RS485             |
| Green/White    |   3   | ?          |                                     |
| Blue           |   4   | On/Off     | On when connected to GND with a relay |
| Blue/White     |   5   | ?          |                                     |
| Green          |   6   | 24V when off |                                     |
| Brown/White    |   7   | GND        | Ground connection                   |
| Brown          |   8   | GND        | Ground connection                   |


So switching on/off was easy simply short the blue wire to e.g. the brown.

Snooping on the RS485 communication (with is bi-directional) while the display was connected showed,the display sends this string to get info from the inverter
> AE01010305EE

The inverter reply looked like
> AE011283023100000277001600000997EE

With changing input voltage output power and temperater I found tis was the encoding
| Input voltage | Temperature | Output voltage | power | PayloadHex | Header   | Output voltage | Power | Input voltage | Temperature | Fill | check | Close |
|---------------|-------------|----------------|-------|------------|----------|----------------|-------|---------------|-------------|------|-------|-------|
|               |             |                |       | byte0      | byte4-5  | byte6-7        | byte8-9 | byte-10-11    | byte12-13   | byte14-15 | byte 16 |       |
| 27.7          | 16          | 231            | 0     | AE01       | 1283     | 0231           | 0000  | 0277          | 0016        | 0000 | 0997  | EE    |
| 27.6          | 16          | 231            | 0     | AE01       | 1283     | 0231           | 0000  | 0276          | 0016        | 0000 | 0996  | EE    |
| 26.4	        | 17          |	225	           | 339	  |AE01       | 1283     | 0225           | 0339  | 0263          | 0017        | 0000 | 0825  | EE    |

So infact the decimal number ar represented by nibbles of the Hex code. They only range between 0 .. 9.

# The HW

I used ESPhome with an ESP32-C1 do to de RS485 commnication, decode the data and create the sensors for home assistant.
To convert the UART at pins GPIO20 + 21 to RS485 I used a MAX3485 module powered by 3.3V from the ESP32-C3.

# The code
Not being a SW engineer coding this in C++ was a chalange. Using ESPhome gives you the full framework but decoding the received data needs to be done in C++ using the Lamba function of ESP home. With the help of AI I got something working although there is for sure room for improvement. The code is available in the file [FCHAO-Inverter.yaml](https://github.com/taste66/FCHAO-Inverter/blob/main/FCHAO-Inverter.yaml)


