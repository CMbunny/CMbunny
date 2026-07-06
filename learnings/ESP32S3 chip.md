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
