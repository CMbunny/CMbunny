# DLMS Protocol
**DLMS/COSEM** and **HDLC** are not competing protocols; rather, they serve different "layers" of the communication stack in smart meters. Think of ***DLMS/COSEM*** as the language and structure of the data, while ***HDLC*** is the transport mechanism that carries that language across the wire.<br>

## 1. The Core Distinction
- **DLMS (Device Language Message Specification):** The application-layer standard. It defines how the data is structured, modeled, and identified so that a system can understand what it is reading (e.g., "This is voltage," "This is billing data").
- **HDLC (High-Level Data Link Control):** The data-link layer standard (ISO/IEC 13239). It defines how the data packets are physically framed, addressed, and error-checked during transit.

## 2. How the Protocol Works (The "Stack" View)
When a smart meter sends data, it travels through these layers:<br>

1.)**COSEM Layer (Application Layer - DLMS):**
- ***Modeling:*** All meter data (energy, voltage, current) is modeled as COSEM Objects.
- ***Identification:*** Every object is assigned an OBIS code. This tells the head-end system exactly what the data is.
- ***Messaging:*** The application takes these objects and turns them into standardized messages (Application Protocol Data Units - APDU).

2.)**Transport/Data Link Layer (HDLC):**
- The APDU is handed down to the HDLC layer.
- ***Framing:*** HDLC wraps the DLMS message into a "frame." This adds start/end flags, address fields (to identify the meter/server), and control fields.
- ***Error Checking:*** It adds a Frame Check Sequence (FCS) at the end, ensuring that if any bit was flipped due to line noise, the receiver detects it.

3.)**Physical Layer (Media):** 
- The bits are converted into electrical signals (RS-485, optical probe, etc.) and sent.

## 3. Understanding OBIS Codes
**OBIS (Object Identification System)** is the "address book" of the meter. Every parameter inside the meter has a unique, hierarchical 6-part ID (A.B.C.D.E.F).
|Group|Meaning|Example|
|--|--|--|
|*A*|Media type (Electricity, Gas, Heat)|`1` (Electricity)|
|*B*|Channel (Internal/External)|`0` (Logical Device)|
|*C*|Measured quantity (Current, Voltage)|`1` (Active Energy)|
|*D*|Processing (Integration, Average)|`8` (Total Import)|
|*E*|Tariff/Rate|`0` (All registers)|
|*F*|Historical/Current values|`0` (Current value)|

*Example:* `1.0.1.8.0.255` is a common code for Total Active Energy Import. Without these codes, a meter from "Manufacturer A" and a meter from "Manufacturer B" would both send raw data, but the head-end system wouldn't know which is voltage and which is current.

## 4. Comparison Summary
|**Feature**|**DLMS/COSEM**|**HDLC**|
|--|--|--|
|**Layer**|Application Layer|Data Link Layer|
|**Purpose**|Data identification and modeling|Framing and error protection|
|**Dependency**|Independent of transport media|Handles media-specific framing|
|**Analogy**|The "Letter" (contents)|The "Envelope" (delivery)|

## 5. Common Doubts
- ***Can DLMS work without HDLC?** Yes. Modern smart meters often use TCP/IP instead of HDLC. DLMS is "communication agnostic"—it just needs some transport layer to move its messages.
- ***What if I use Modbus instead of DLMS?*** Modbus is a simple register-reading protocol. DLMS is much more complex and object-oriented. DLMS is preferred for smart grids because it is standardized, secure (supports encryption), and flexible enough to handle complex tariff schemes that Modbus cannot easily support.

## COMMUNICATION FLOW:
To understand the communication flow, you must visualize the **"Client-Server"** relationship. In the DLMS/COSEM world, the Client is your Data Collection System (or HMI), and the Server is the Smart Meter itself.The communication follows a strict "handshake" sequence before any actual meter data (like energy consumption) is ever sent.

1. **The Connection Sequence (The Handshake)** <br>
Before reading an OBIS code, the client must "log in" and agree on the rules of engagement.<br>
i)***AARQ (Association Request):*** The Client sends an "Association Request" to the Meter. This contains the Client's ID, the password (if required), and a list of "Proposed Services" (e.g., "I want to use AES-128 encryption").
ii)***AARE (Association Response):*** The Meter processes the request. If the credentials are valid, it sends an "Association Response" back, confirming that a secure Association (Session) has been established.
iii)***The "Locked-in" State:*** Once the association is established, the Meter is now "bound" to that specific client session.

2. **Requesting Data (The "GET" Operation)** <br>
Once the association is active, you don't just "ask" for data; you perform a **GET Request** on a specific COSEM Object.
- **The Request (GET.Request):** The Client sends a packet containing the OBIS Code it wants to read (e.g., `1.0.1.8.0.255`).
- **The Response (GET.Response):** The Meter looks up that specific object in its memory, reads the value (e.g., `1250.5 kWh`), and sends it back in a structured APDU (Application Protocol Data Unit).

3. **Data Transmission (The "Push" Operation)** <br>
Sometimes, the Meter doesn't wait for a request. This is called ***Push Communication.***
- **Data Notification:** The Meter is configured to "push" its data at set intervals (e.g., every 15 minutes).
- **No Association Needed:** In some setups, the Meter simply formats the data into a DLMS packet and fires it at the HMI without an Association Request. This is very common in cellular/GPRS meters where the meter is the one initiating the connection to save power.

4. **Summary Table of the Communication Flow**
|Step|Action|OSI Layer|1|**Association (AARQ/AARE)**|Layer 7 (DLMS)|
|2|**GET/READ Request**|Layer 7 (DLMS)|
|3|**Data Response**|Layer 7 (DLMS)|
|4|**Framing (HDLC Header/FCS)**|Layer 2 (HDLC)|
|5|**Physical Transmission**|Layer 1|

5. **Why this sequence is used**
- **Authentication:** AARQ ensures that only authorized systems can talk to the meter.
- **Flow Control:** The association prevents the meter from being overwhelmed by multiple clients trying to talk at the same time.
- **Error Resilience:** Because every request requires an explicit confirmation, the system knows exactly when a read has failed, allowing it to retry automatically.

 **Question:** What if the connection drops while I am reading the OBIS codes?<br>
 **Answer:** The Association will time out. You will have to restart the Association (AARQ) sequence from step 1. This is why DLMS clients are designed with "retry logic"—they assume the network is unstable and will automatically attempt to re-associate if the heartbeat is lost.

 ### NOTES:
 ***DLMS/COSEM & HDLC: The Smart Meter Communication Standard***
 1. **The Architectural "Stack"** <br>
 Communication is split into two distinct responsibilities:
- **DLMS (The Language):** Defines what the data is and how to model it (Application Layer).
- **HDLC (The Delivery Truck):** Defines how the data is framed, addressed, and protected during transport (Data Link Layer).
2. **Key Terminology** <br>
- **COSEM:** Companion Specification for Energy Metering. It models all meter data as "Objects.
- **"OBIS Codes:** The universal "Object Identification System." A 6-part ID (A.B.C.D.E.F) that tells the system exactly what parameter is being measured (e.g., Active Energy, Voltage).
- **Association:** The mandatory "handshake" process (AARQ/AARE) that establishes a secure session between the Client and the Meter.
3. **Communication Sequence** <br.
The interaction is strictly Client-Server, following this logical flow: <br>
- **AARQ (Association Request):** Client asks to connect, provides credentials, and proposes security settings (e.g., encryption).
- **AARE (Association Response):** Meter validates the request and opens the session.
- **GET/READ Request:** Client sends the OBIS Code of the specific data it needs.
- **GET/READ Response:** Meter retrieves the value from its COSEM objects and returns it.
- **Termination:** The session is closed once the data exchange is complete.
4. **The "Encapsulation" Flow** <br>
When you send a request, the data travels through the OSI layers, getting wrapped at each stage:
- **Application Layer:** DLMS creates the structured message (APDU).
- **Data Link Layer (HDLC):** Wraps the APDU in a frame, adds the Meter Address, and appends the ***FCS (Frame Check Sequence)*** for error detection.
- **Physical Layer:** Converts the frame into raw bits for the wire.
5. **Comparison Summary**
|Aspect|DLMS/COSEM|HDLC|
|--|--|--|
|**Layer**|Application (Layer 7)|Data Link (Layer 2)|
|**Main Job**|Data Modeling & Identification|Framing & Error Protection|
|**Analogy**|The "Letter" (Content)|The "Envelope" (Delivery)|
6. **Pro-Tips for Implementation**
- **Security:** Always use the Association phase to enable encryption. Never transmit raw OBIS data over an insecure connection if sensitive billing or control data is involved.
- **Error Handling:** If the association drops, you must restart from the AARQ handshake. Always implement a "Retry" mechanism in your application code.
- **Versatility:** DLMS is "Transport Agnostic"—it does not care if it's running over HDLC, TCP/IP, or Cellular. This makes it the most flexible protocol for global utility grids.
