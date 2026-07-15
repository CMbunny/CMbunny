# CANbus Protocol
CANbus (Controller Area Network) is the "nervous system" of modern battery packs. While Modbus is like a post office sending letters, CANbus is like a high-speed, crowded meeting room where everyone shouts at once, but only the right people listen.<br>
## 1. What is CANbus?
Originally designed for cars, CANbus is a ***message-based protocol***. Unlike Modbus, where a master asks, "What is your voltage?", CANbus devices simply "broadcast" their data to the entire network whenever they want.<br>

## 2. How it works?
1.) **The Broadcast:** When a Battery Management System (BMS) has an update, it puts a message on the bus. This message has a **CAN ID** (*a number that identifies what the data is, like "Battery Voltage" or "Cell Temperature"*).
2.)**The Filter:** Every other device on the network (like an Inverter or your monitoring system) listens to the bus. They look at the CAN ID. If they don't care about "Cell Temperature," they simply ignore the message.
3.)**Arbitration:** If two devices try to talk at the exact same time, the one with the "higher priority" (the lowest CAN ID number) wins, and the other waits. This happens in microseconds.

## 3. CANbus vs. Modbus vs. DLMS
|**Feature**|Modbus|CANbus|DLMS (COSEM)|
|--|--|--|--|
|**Logic**|Master/Slave (Polling)|Multi-Master (Broadcasting)|Object-Oriented (Complex)|
|**Speed**|Slow (RS485)|Fast (Up to 1 Mbps)|Moderate|
|**Complexity**|Low|Medium|High|
|**Best For**|PLC/Industrial sensors|Automotive/Battery packs|Smart Meters/Utility billing|

- **Difference:** Modbus requires you to "ask" for data. CANbus is "event-driven"—you get data as soon as the battery decides to send it.

## 4. Things to notice in CANbus
- **Termination:** Just like RS485, CANbus requires a ***120Ω resistor*** at both physical ends of the cable. Without it, the signal "bounces" off the ends of the wires and corrupts the data.
- **Baud Rate:** Everyone on the bus must use the same speed (usually 250kbps or 500kbps for batteries). If one device is at 250 and another at 500, nothing will work.
- **CAN High and CAN Low:** These are the two wires. CANbus is differential, meaning it sends the signal as the difference between these two wires, making it extremely resistant to electrical noise.

## 5. What you need to know before coding for Batteries:
If you are about to code a driver to talk to a lithium battery, you need these four things:<br>
1.)**The DBC File:** This is the most important document. It is a "dictionary" that tells you: "If you see a message with ID `0x355`, the first two bytes represent voltage, the next two represent current." Without this, you are just looking at raw, meaningless hex numbers.<br>
2.)**CAN ID Structure:** Understand whether the battery uses ***Standard (11-bit)*** or ***Extended (29-bit)*** IDs. Most modern BMS systems use 29-bit IDs.<br>
3.)**Frequency:** Know how often the battery broadcasts. Some batteries send data every 100ms, others every 1s. If your code isn't ready to process 100ms updates, your system will lag.<br>
4.)**Error Handling:** CANbus hardware has built-in error counters. If a device is "bouncing" (sending bad data repeatedly), the hardware will automatically "go silent" to protect the network. Your code must be able to detect this *"Bus-Off"* state and re-initialize the connection.

### The "Noob" Golden Rule:
Don't try to "read" the battery. Instead, set up your code to "listen" to the bus. Let the battery push data to you. If you treat CANbus like Modbus (constantly sending requests), you will likely overwhelm the battery's BMS or cause it to ignore you.

## NOTE:
let’s look at a raw CAN message from a standard Lithium battery. This is exactly what you would see on your logic analyzer or in your code buffer.

### The Raw CAN Message
Imagine your code just received this packet from the Battery Management System (BMS):<br>
`ID: 0x355 | DLC: 8 | Data: 0A 0B 00 32 01 04 00 00`

### Breaking Down the Terms
1.)**ID: `0x355` (The "Address")** <br>
- **What it is:** This is the "Subject Line" of the email.
- **Why it's there:** In a crowded CAN network, the BMS might send voltage data on `0x355` and temperature data on `0x356`. Your code checks this ID first to decide if it should "open" the message or ignore it.

2.)**DLC: `8` (Data Length Code)** <br>
- **What it is:** This tells you how many bytes are following (from 0 to 8).
- **Why it's there:** CANbus messages are fixed-size, but they don't always use all 8 bytes. If the DLC is 8, your code knows to read all 8 bytes into your array.

3.)**Data:** `0A 0B 00 32 01 04 00 00` **(The Payload)** <br>
This is the actual information, broken into individual bytes.

### Decoding the Payload (The "Why")
In the battery manual (the DBC), you would find a table that maps those bytes to actual values. Let’s assume this specific BMS manual says:
- **Bytes 0 & 1**: Total Voltage (Scale: 0.1V)
- **Bytes 2 & 3:** Current (Scale: 0.1A)
- **Bytes 4 & 5:** State of Charge (SoC) (%)

|Byte|Hex Value|Decimal|Decoding Logic|Final Meaning|
|--|--|--|--|--|
|0 & 1|`0A 0B`|2571|$2571 \times 0.1V$| 257.1 Volts|
|2 & 3|`00 32`|50|$50 \times 0.1A$|5.0 Amps|
|4 & 5|`01 04`|260|$260 / 10$|26.0% SoC|

**Question:Why is it structured this way?** <br>
1.)**Efficiency:** CANbus is limited to 8 bytes per message. By packing data into bytes, the BMS can send a huge amount of information in a tiny 8-byte packet.<br>
2.)**Scaling (The `0.1` factor):** Why not just send "257"? Because CANbus can only send integers (whole numbers). To send a decimal like "257.1", the battery manufacturer multiplies it by 10 or 100 to make it an integer, and you "scale" it back when you receive it.<br>
3.)**Endianness (The `0A 0B` mystery):** Notice that `0A 0B` (hex) is 2571 decimal. Sometimes, manufacturers swap the order of the bytes (`0B 0A`), which is called "Little-Endian" vs "Big-Endian." If your data looks like crazy, huge numbers, it’s usually because you need to swap the byte order in your code.<br>

**Question:What one needs to know before coding?** <br>
1.)**Endianness:** Does the battery send the "Most Significant Byte" first or last? <Br>
2.)**Signed vs Unsigned:** Is the current positive (charging) or negative (discharging)? You need to check if the battery uses "Two's Complement" to represent negative numbers.
3.)**The Frequency:** If you don't read the buffer fast enough, the battery will overwrite the old data with new data, and you'll miss the update.
