# FCHAO-Inverter
Hacking the FCHAO Inverter a.k.a. Peter inverter

After watching [this video](https://www.youtube.com/watch?v=I5rBVGSszBY) regarding the affortable mains inverter I decided to buy it for a project which needed a solar charger for an electrical outboard engine. The engine unfortunately had a propriatary charger with additional communication so I needed 230Vac@1250Watt. Therfore I purchased the 24v 2500Watt FCHAO to make sure it was only loaded 50% which usually greatly increases the operating time.

To allow switching the inverter on/off based on the load (low when the outboard engine battery was charged), solar storage battery condition and temperature I needed these values as sensor input for a simple Homeassistant system. The information is available on the remote display which is connected via an RS485 connection. I suspected Modbus but unfortunately the protocol was propritary so I had to reverse engeneer it.

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

I used ESPhome with an ESP32-C1 do to de RS485 commnication, decode the data and create the sensors fro home assistant. 


