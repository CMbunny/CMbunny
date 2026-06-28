# OTA Method
***OTA (Over-the-Air)*** updating is the process of wirelessly delivering new firmware, software, or configuration data to an embedded device. Instead of manually connecting a cable to your device, you push the code through the air (Wi-Fi, Bluetooth, or Cellular).<br>

**1. How OTA Works (The Logic)** <br>
For an ESP32 or STM32 to update itself, it must be able to overwrite its own memory while still running. This requires a ***Dual-Partition Strategy.***
- **Partition A (The Active App):** The firmware currently running.
- **Partition B (The OTA Slot):** An empty "waiting room" in the device's Flash memory.

### The Workflow:

**1.)Download:** The device connects to your server, downloads the new binary file, and writes it into ***Partition B.*** <br>
**2.)Verify:** The device checks a checksum (or a digital signature) of the downloaded file to ensure it isn't corrupted.<br>
**3.)The Switch (Bootloader):** The device sets a flag in the Bootloader (a tiny piece of code that runs before your app). It tells the chip: "Next time you reboot, run the code in Partition B instead of A."<br>
**4.)Reboot:** The chip resets, the Bootloader reads the flag, switches the active partition, and your device runs the new firmware.<br>

**2. What you need to do OTA**
- **Flash Space:** You must have enough Flash memory to hold two copies of your firmware (Partition A + Partition B).
- **Partition Table:** Your `partitions.csv` file must explicitly define an `ota_0` and `ota_1` partition, and an `ota_data` partition to track which one is active.
- **Network Connectivity:** A reliable way to reach a server (HTTPS is mandatory for security).
- **Secure Server:** A cloud server (like AWS S3 or your own web server) that hosts the `.bin` firmware file.
- **Rollback Logic:** The "Golden Rule" of OTA. If the new firmware crashes immediately after the update, the device must be able to detect the crash and automatically switch back to the previous working version.<br>

**3. OSI Layer Placement**
OTA updates operate primarily at the ***Application Layer (Layer 7).***

- **Why?** The firmware update is just a payload—a giant file being sent from a server to a device.
- **TCP (Layer 4)** handles the segmentation and reliability of the download.
- **IP (Layer 3)** handles the routing to your specific device.
- **Your Code (Layer 7)** handles the actual "writing" to the flash memory and the "switching" of the partitions.<br>

**4. Advanced Concepts & Security**<br>
**Digital Signatures (Secure Boot)**<br>
How do you stop a hacker from pushing "malicious" firmware to your devices?<br>
- You sign your firmware binary with a ***Private Key*** on your computer.
- The ESP32 holds a ***Public Key*** in its eFuse (hardware).
- During the update, the device verifies the signature. If it doesn't match your key, the device refuses to install it.<br>

**5.Doubt Questions** <br>
**Question:** What if the power cuts off during the download?<br>
**Answer:** Because you are writing to "Partition B" (the empty slot), your current app in "Partition A" remains completely untouched. Once power returns, the device simply starts from Partition A again and tries the download over. The device never bricks.<br>
**Question:** Can I use OTA with the Authentication code I wrote?<br>
**Answer:** Yes! You should use your auth module to ensure that only a user with ROLE_ADMIN can trigger an OTA update.<br>
**Question:** Does it work the same on STM32?<br>
**Answer:** The concept is identical, but the implementation is more manual. On ESP32, the ESP-IDF provides "Native OTA" functions. On STM32, you often have to write the "Bootloader" logic (IAP - In-Application Programming) yourself, which is significantly more complex.<br>

## Delta Updates (Advanced)
If your firmware is 2MB but you only changed one function, you don't need to send the whole 2MB. A ***Delta Update*** only sends the difference (the "diff") between the old version and the new version. This saves massive amounts of bandwidth and battery.<br>

Delta updates (often called "patching" or "diffing") are a game-changer for bandwidth-constrained projects, especially when devices are powered by batteries or rely on expensive cellular data.<br>
Here is an elaboration on how this works and why it is advanced:<br>

**How Delta Updates Work** <br>
The fundamental logic relies on comparing the binary file of the "Old Version" to the "New Version" at the byte level.
- **Binary Comparison:** A server-side tool (the "patch generator") compares two firmware files side-by-side. It identifies exactly which bytes stayed the same and which bytes changed.
- **Creating the "Diff":** The generator creates a small file (the "delta" or "patch") that contains:<br>
i)The changed bytes.<br>
ii)Instructions on where to "patch" these changes into the existing old binary.
- **Transmission:** Only this small patch file is sent over the network to your device.
- **Reconstruction:** The ESP32 receives the patch, reads the currently running firmware from Flash, applies the changes, and writes a full new binary into the inactive Partition slot.

#### Why it is Efficient?
- **Massive Bandwidth Savings:** If you have a 2MB firmware and you change only 50KB of code, the patch might be as small as 60-70KB. You are sending ~3% of the total size, saving 97% of the data transfer cost.
- **Battery Preservation:** The radio (Wi-Fi/LTE) is one of the biggest power consumers on a microcontroller. By keeping the radio active for seconds instead of minutes, you significantly extend the device's battery life.
- **Reduced "Air-time" Risk:** The shorter the time the device spends downloading, the lower the probability of a network glitch, a connection drop, or a power outage interrupting the process.

#### The "Catch" (The Trade-off)
While it sounds perfect, Delta updates add significant complexity to your architecture:
- **Server-Side Logic:** You need a build system that keeps track of every previous version of your firmware. You have to generate a specific patch for "v1.0 to v2.0", "v1.1 to v2.0", etc.
- **Device-Side Reconstruction:** Your device needs enough RAM or a clever buffering system to read the old binary, apply the patch, and write the new binary simultaneously.
- **Versioning Dependency:** If a device is several versions behind (e.g., still on v1.0 but you are at v3.0), the device either needs to download multiple sequential patches or a "master" patch, which can get messy to manage.

### Notes:
|Strategy|Full OTA|Delta OTA|
|--|--|--|
|**Data Sent**|Entire firmware (100%)|Only changed parts (~1-10%)|
|**Complexity**|Simple|High (Server + Device side)|
|**Suitability**|Best for small firmware|Best for large firmware / low bandwidth|
|**Reliability**|Very High|Medium (requires strict version control)|
