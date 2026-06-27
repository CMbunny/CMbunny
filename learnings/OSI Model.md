# OSI Model
The OSI (Open Systems Interconnection) Model is the conceptual blueprint for how data travels across a network. It breaks the complex process of communication into seven distinct layers, each adding its own "envelope" of information to the data.
## 1. The 7 Layers of the OSI Model
|Layer|Name|What it does|Data Unit|
|--|--|--|
|7|**Application**|Interface for apps (HTTP, MQTT, Modbus)|Data|
|6|**Presentation**|Encryption/Decryption, Compression|Data|
|5|**Session**|Manages start/end of communication sessions|Data|
|4|**Transport**|Reliability, Flow control (TCP/UDP)|Segment|
|3|**Network**|Routing, IP addressing|Packet|
|2|**Data Link**|MAC addressing, Error detection (Ethernet)|Frame|
|1|**Physical**|Voltage, Cables, Signals|Bits|

## 2. How the Packet Grows (Encapsulation)
As data moves down from the Application layer to the Physical layer, each layer adds a Header (and sometimes a Footer) to the data packet.<br>
1.)**Transport (Layer 4):** Adds a Port Number (e.g., Port 502 for Modbus) to tell the receiving device which application needs this data.<br>
2.)**Network (Layer 3):** Adds the Source and Destination IP Address. This is how the packet finds its way across the internet to your specific device.<br>
3.)**Data Link (Layer 2):** Adds the MAC Address and a Frame Check Sequence (FCS). The FCS is the hardware-level "checksum" that ensures the physical wire didn't corrupt the data.<br.

## 3. Ensuring Authenticity & Correctness
You asked how we know the data is from an authentic source and is correct. This is where your code and the OSI layers work together:<br>
- ***Integrity (Correctness):*** - **Layer 2 (FCS):** Detects if the signal was corrupted by cable noise.
- - **Layer 4 (TCP Checksum):** Ensures the data wasn't scrambled during transit across the network.
  - **SHA-256 (Application Layer):** In your code, even if TCP says "the data arrived perfectly," you do one final check: Is the JWT Signature valid? If the signature matches, you know the data hasn't been tampered with at the application level.
  - ***Authenticity:*** <br>
  - **JWT/HMAC:** This is your primary defense. Even if someone manages to "spoof" the IP address (Layer 3) or the MAC address (Layer 2), they cannot create a valid JWT signature because they don't have the `s_jwt_secret` stored in your NVS.

## 4. OSI & Your Code: Important Reminders
- **Where does your code live?** Your entire authentication logic (SHA-256, JWT, OTP) runs strictly at ***Layer 7 (Application Layer)***.
- **Why it matters:** * If you rely solely on network-level security (like IP whitelisting), it is very weak. A hacker can easily bypass Layer 3.
   @)By placing your security at Layer 7, your data remains protected even if the underlying network is insecure (like Wi-Fi or the public internet).
- **Partitioning:** Your use of NVS falls outside the OSI model; it is your device's persistent memory. However, the data you fetch from NVS is what you eventually send out through the Application layer to the network.
-
#### Common Doubt Questions
**Questions:** If Layer 2 and Layer 4 already check for errors, why do I need SHA-256? <br>
**Answer:** Layer 2 and 4 check for accidental errors (like electrical interference). They do not check for malicious modification. A hacker could intercept the packet, change the data, calculate a new TCP checksum, and send it on. SHA-256 (your Layer 7 security) is what detects that someone intentionally changed the data.<br>
**Question:** Do I need to implement all 7 layers in my C code?<br>
**Answer:** No. Your ESP32's internal SDK (ESP-IDF) handles Layers 1 through 4 (the physical signal, Ethernet/Wi-Fi frame, IP, and TCP). You only need to write code for Layer 7—your application logic.
