# ESP32-S3
The ESP32-S3 is a highly integrated System-on-Chip (SoC) designed specifically for AIoT (AI + IoT) applications, featuring robust processing power and advanced connectivity.
### Core Hardware Architecture 
- **Processor:** Dual-core 32-bit Xtensa® LX7 CPU, capable of running up to 240 MHz. It includes ***vector instructions*** that provide hardware acceleration for neural networks and signal processing.
- **Memory:** 512 KB of internal SRAM, with support for high-speed external Octal SPI flash and PSRAM (up to 16 MB Flash/8 MB PSRAM).
- **Wireless:** Integrated 2.4 GHz Wi-Fi (802.11 b/g/n, up to 150 Mbps) and Bluetooth 5 (LE) with support for long-range communication.
- **Security:** Hardware-accelerated cryptographic engine supporting AES, SHA, RSA, and ECC, plus secure boot and flash encryption.
- **Coprocessors:** Includes an Ultra-Low-Power (ULP) RISC-V coprocessor and a ULP-FSM state machine for background monitoring while the main cores are in deep sleep.
 ### Peripherals & Connectivity
- **GPIOs:** 45 programmable General-Purpose Input/Output pins.
- **Digital Interfaces:** 4x SPI, 3x UART, 2x I2C, 2x I2S, 1x full-speed USB OTG, 1x USB Serial/JTAG, and 1x TWAI® (CAN bus compatible).
- **Analog Interfaces:** 2x 12-bit SAR ADCs (up to 20 channels) and 14x capacitive touch-sensing GPIOs.
- **Specialized:** Dedicated LCD interface (parallel RGB/I8080) and DVP 8-bit to 16-bit camera interface for vision tasks.
### ESP32-S3 Block Diagram
The following diagram illustrates how the core components are interconnected within the SoC:<br>
```
graph TD
    subgraph "Core Processing"
        CPU1[Xtensa LX7 Core 1] <--> Bus[System Bus / Crossbar]
        CPU2[Xtensa LX7 Core 2] <--> Bus
        AI[Vector/AI Accelerator] <--> Bus
    end

    subgraph "Memory & Storage"
        Bus <--> SRAM[512 KB SRAM]
        Bus <--> ROM[384 KB ROM]
        Bus <--> Flash[Ext. Flash/PSRAM Interface]
    end

    subgraph "Connectivity & Peripherals"
        Bus <--> Wifi[Wi-Fi Baseband]
        Bus <--> BLE[Bluetooth 5 LE]
        Bus <--> USB[USB OTG / Serial JTAG]
        Bus <--> GPIO[45x Programmable GPIOs]
        Bus <--> ADC[ADC / Touch Sensors]
    end

    subgraph "Power & Security"
        PMU[Power Management Unit]
        ULP[ULP Coprocessor]
        SEC[Crypto Accelerators / Secure Boot]
    end

    PMU --> CPU1
    PMU --> CPU2
    SEC -.-> Bus
```
### Reference Table  
|**Feature**|**Specification**|
|--|--|
|**CPU**|Dual-Core Xtensa LX7 (up to 240 MHz)|
|**GPIO**|45 Programmable|
|**SRAM**|512 KB internal|
|**Connectivity**|Wi-Fi 4 + Bluetooth 5 (LE)|
|**USB**|1x Full-speed USB OTG|
|**ADC**|2x 12-bit (up to 20 channels)|
|**Touch**|14 Capacitive touch pins|


*Let's dive into these "hidden" layers of the ESP32-S3* <br>

**1. Memory Mapping & Flash Constraints** <br>
The ESP32-S3 uses a Harvard architecture, where instruction and data memory are separate.<br>
- ***Internal SRAM (512 KB):*** This is the high-speed "playground" for your active tasks. It is split into blocks, and some parts are reserved for Wi-Fi/Bluetooth stacks.
- ***External Flash/PSRAM:*** Since your code resides in external SPI Flash, the S3 uses an MMU (Memory Management Unit) to map this external storage into the address space of the CPU.
- ***The Constraint:*** Accessing external memory is slower than internal SRAM. If your Modbus polling or JSON serialization code in PolyCap_finaV is bottlenecked, we usually move performance-critical functions into IRAM (Instruction RAM) using the IRAM_ATTR macro. This forces the code to run from internal memory, bypassing the SPI Flash speed limit.<br>

**2. Power Management (The ULP Secrets)** <br>
The ULP (Ultra-Low-Power) Coprocessor is a separate, tiny processor inside the S3 that runs even when the main Xtensa cores are in "Deep Sleep." <br>
- ***The Workflow:*** You can write a small assembly or C program that runs on the ULP. While your main Curator-H system is sleeping to save power, the ULP can monitor your RS485/Modbus or analog sensors.
- ***The Trigger:*** The ULP can be programmed to wake the main CPU only when a specific condition is met (e.g., an energy threshold is reached). This allows your device to stay in a near-zero power state until it is absolutely necessary to wake up the main system. <br>

**3. Physical Layout & Pin Multiplexing** <br>
The ESP32-S3 features an IO_MUX and a GPIO Matrix. This is what gives you flexibility in your PCB designs.
- ***IO_MUX:*** Most pins have a default function. If you use a pin for its default (like UART0 for programming), it is fast and direct.
- ***GPIO Matrix:*** This allows you to route almost any peripheral (like your second UART for Modbus) to almost any GPIO pin.
- ***The Trap:*** Because the signal has to pass through the matrix, there is a slight propagation delay compared to the direct IO_MUX path. For high-speed applications, we prefer to keep signals on their "native" pins.
- ***Internal Resistors:*** Every pin has software-configurable pull-up/pull-down resistors. However, they are weak (~45kΩ). In industrial environments (like your inverter projects), these are often insufficient to overcome noise, which is why you likely see external pull-ups on your boards. <br>

**4. Secure Boot & Flash Encryption** <br>
This is how you protect your intellectual property in the Curator-H project.
- ***Flash Encryption:*** The chip uses a hardware-based AES-256 engine. Your firmware is stored in the external flash in an encrypted state. The key is stored in the eFuse—a "write-once" memory on the chip that cannot be read back by any external tool.
- ***Secure Boot:*** The chip verifies the digital signature of the firmware against a public key burned into the eFuse. If you try to flash "cracked" or modified code, the bootloader will detect the signature mismatch and refuse to execute.
- ***Result:*** Even if someone desolders your Flash chip and reads the raw data, they will only see ciphertext. <br>

**5.Wi-Fi Capabilities** 
- **Inbuilt Wi-Fi:** Yes, the ESP32-S3 features an integrated 2.4 GHz Wi-Fi controller that supports the 802.11 b/g/n standard.
- **Performance:** It supports data rates of up to 150 Mbps.
- **Protocol Support:** It is designed to handle standard networking protocols, which is why it is effective for the WebSockets and HTTP communication used in your Curator-H project.
**6.Ethernet Capabilities**
- **Built-in MAC:** The ESP32-S3 does not have an integrated Ethernet PHY (Physical Layer) inside the chip, but it does have an integrated MAC (Media Access Control) layer.
- **External Support:** To use Ethernet with the ESP32-S3, you must connect an external Ethernet PHY chip (such as the LAN8720A or similar) via the RMII (Reduced Media Independent Interface) protocol.
- **Integration:** Since the ESP32-S3 supports RMII, you can achieve wired Ethernet connectivity by routing the necessary signals to the external PHY; the internal MAC will then handle the data processing. <br>

***Key Connectivity Summary***
|Feature|Support|Note|
|--|--|--|
|**Wi-Fi**|Integrated (2.4 GHz)|Native support up to 150 Mbps.|
|**Bluetooth**|Integrated (v5.0 LE)|Supports long-range mode.|
|**Ethernet**|External PHY Required|Uses internal MAC via RMII interface.|
|**USB**|Integrated|1x Full-speed USB OTG and 1x USB Serial/JTAG.|  

## IMPORTANT
**1. NVS (Non-Volatile Storage)** <br>
The ESP32-S3 uses the NVS library as a key-value storage system in your flash memory. 
- ***Purpose:*** It is designed to store small amounts of data (like your JWT_SECRET_NVS_KEY or user credentials) that must persist across power cycles.
- ***Internal Mechanics:*** It works by organizing data into pages and sectors in the flash; it includes a wear-leveling mechanism to prevent specific flash cells from dying due to frequent writes. <br>

**2. RTC Clock & Power Management** <br>
The ESP32-S3 features a sophisticated Real-Time Clock (RTC) subsystem that stays powered even when the rest of the chip is in sleep modes.
- ***Function:*** It tracks absolute time (if synced via SNTP, as you do in final_task_0) and provides the logic for wake-up timers.
- ***RTC Memory:*** There is a dedicated, tiny "RTC Fast/Slow Memory" block that persists during Deep Sleep, allowing you to pass variables (like state flags) to the ULP coprocessor or back to the main CPU after a wake-up. <br>

**3. SD Card (SDMMC/SPI)** <br>
The ESP32-S3 has a dedicated SDMMC Host peripheral that supports high-speed data transfer.
- ***Connectivity:*** You can interface with SD cards using either the SDMMC protocol (which is much faster and uses more pins) or the SPI protocol (which is slower but requires fewer pins and is easier to route on a PCB).
- ***Usage:*** In your PolyCap_finaV context, this is ideal for local logging of energy data if the Wi-Fi connection to your server drops.<br>

**4. JTAG (On-Chip Debugging)** <br>
The ESP32-S3 has a built-in USB Serial/JTAG controller, which is a massive upgrade over older ESP32 versions.
- ***Function:*** It allows you to debug your code (step through lines, set breakpoints, inspect variables) directly over the same USB-C cable you use for flashing.
- ***Advantage:*** You no longer need an external JTAG adapter (like a J-Link) to perform hardware-level debugging; you can use esp-prog or the built-in USB capability to diagnose complex crashes in inverter_task.c.

### Summary Table
|**Feature**|Primary Use Case|Critical Note|
|--|--|--|
|NVS|Config & Auth Secrets|Use sparingly to minimize flash wear.|
|RTC|Timekeeping & Deep Sleep|RTC memory is volatile if main power is lost.|
|SD Card|Data Logging|Use SDMMC for performance; SPI for simplicity.|
|JTAG|Real-time Debugging|Built-in USB JTAG simplifies troubleshooting.|  
