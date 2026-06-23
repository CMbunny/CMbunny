# Modbus RTU (Remote Terminal Unit)
Modbus RTU is designed for serial communication. It is the most common protocol used in industrial automation for simple, point-to-point or multi-drop communication between a controller (Master) and field devices (Slaves).

## How it works:
It encodes data as binary, making it compact and efficient for low-bandwidth environments.

***Physical Connection:*** It uses two wires (or four) for RS-485. Because it is a "bus" architecture, you can connect multiple devices in a daisy-chain.

***Error Handling:*** It relies on a CRC field at the end of every message to ensure data integrity. If a device receives a corrupted message, it simply ignores it.

**Modbus TCP (Transmission Control Protocol)**
Modbus TCP is essentially Modbus RTU "wrapped" inside a standard TCP/IP packet. It is designed to work over modern Ethernet networks.

***How it works:*** The Modbus message is encapsulated within a TCP frame. Because Ethernet is a reliable transport layer, the CRC field used in RTU is removed; the TCP/IP stack handles error detection and retransmission if necessary.

***Addressing:*** Instead of a simple Slave ID, the device is identified by its IP Address. This allows multiple masters to communicate with the same device simultaneously.

***Infrastructure***: It takes advantage of existing IT infrastructure, such as switches, routers, and fiber optics, making it ideal for long-distance, high-speed, and complex network architectures.


|**Feature**|	**Modbus RTU**|**Modbus TCP**|
|---|---|---|
|**Medium**|  Serial (RS-485/232)|Ethernet (Network cables)|
|**Error Handling**|	CRC (Application level) |	Multi-layer (TCP/IP/Ethernet)|
|**Reliability**  |"Best Effort" (ignores bad data)  |"Guaranteed" (automatic re-send)|
|**Addressing**   |	Slave ID |IP Address|
|**Speed** |Slow (serial limitations)  |	Fast (network bandwidth)|


## Question:
"TCP/IP checksum + IP layer error correction: give more explanation on this."

## The Answer:
Think of the difference like sending a package:<br>
1.Modbus RTU (The Postcard): You write a message and stick a stamp on it (the CRC). If the postcard gets wet or ripped in the mail, it’s ruined. The post office doesn't care; it just delivers the scrap of paper to the recipient, who then realizes it’s unreadable and throws it away.<br>
2.Modbus TCP (The Certified Courier): You hand the package to a professional service.<br>
i)The Ethernet layer checks the physical "box" to ensure it wasn't crushed during transport.<br>
ii)The IP layer checks the address label to make sure the package is going to the right house.<br>
iii)The TCP layer is the "courier." It keeps a checklist. If any part of your package is missing or damaged, the courier automatically goes back to the sender and says, "This didn't arrive correctly, please send it again."

## Question:
Which one should you use?
## Answer:<br>
1.Choose Modbus RTU if you are working with short-range, low-cost field devices, have limited bandwidth, or are retrofitting older systems where serial infrastructure is already installed.<br>
2.Choose Modbus TCP if you need high speed, long distances, integration with IT/Cloud systems, or if you need multiple controllers to access data from the same device simultaneously.

## NOTES:
**Why is it better?**
In the RTU world, if there is a tiny bit of "noise" (interference) on the wire, your data is corrupted, and you have to manually code your software to ask for a retry. <br>
In the TCP world, you don't even have to think about it. The network hardware and the computer's operating system handle all the error-checking, fixing, and re-sending in the background. Your software only ever sees the "perfect" final data.
