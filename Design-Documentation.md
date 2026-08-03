# Isolated USB to UART Hardware Design

## Research

## System Block Diagram Design

## USB (Type-C) Selection for Design

### Design Research

We need to select a USB Type-C connector for a my custom isolated USB to UART Module design, need to take the mechanical and electrical decision right in the design.
Since a USB-UART/RS485/RS232/CAN bridge only operates at USB 2.0 speeds (Full-Speed 12 Mbps or High Speed 480 MbPs), we dont need the complexity of a full Type-C implementation.

We need to select the right connector and design the interface by focusing on:
1. Choosing the Receptacle Type (Pin Count)
2. Mechanical Robustness (Mounting Style)
3. CC1 & CC2 pin configurations
4. Data Pins D+ & D-
5. How to Select the Right Connector (Our Checklist)

#### Choosing the Receptacle Type (Pin Count)
Type C Connectors come in serveral variants and choosing the right one dictates how difficult the board will be to route and solder.
- 24 Pin Receptacles: We will need to avoid this, these are full-featured connectors supporting USB3.2 SuperSpeed, DisplayPort, etc.
- 16 Pin Receptacles: These drop the inner SuperSpeed TX/RX pairs but keep all the standard USB 2.0, power, and CC pins. These are also arranged in a single SMT row or a slightly staggered setup, making them much easier to route and manufacture, therefore making this is a good option for the design.

#### Mechanical Robustness (Mounting Style)
The Module will be plugged in, yanked out, and tossed in a bag constantly, therefore the connector held on only surface-mount pads will eventually rip off the PCB, taking the copper traces with it. The options we have our:
1. SMT only: We will need to avoid this for dongles or frequentlly used tools like our Isolated USB-UART module.
2. Through-Hole-Only: Very Strong Mechanical Integration but routing high-speed USB D+/D- through large plated holes can sometime casue minor signal integrity issues but okay for USB 2.0.
3. Hybrid (The Standard): The electrical pins (VBUS, D+, D-, GND) are Surface Mount (SMT) for easy routing on the top layer, but the four large mechanical mounting shell tabs are heavy Through-Hole (TH) legs and soldering these massive tabs into the board will provide immense mechancial strain relief, therefore making this the ideal choice in the design.

#### CC1 & CC2 Pin Configurations

In a standard USB Type-C connector, the CC1 and CC2 (Configuration Channel) pins are responsible for detecting cable attachment, cable orientation, and power delivery capabilities. Because the Type-C plug is reversible, these pins ensure the host and device can communicate regardless of which way the cable is plugged in.

Since the Isolated USB to UART Module will act as a "device" which is basically drawing power from a host like a laptop, and is classified as an Upstream Facing Port (UFP).

#### The Golden Rule: Two 5.1kΩ Resistors
To properly configure your module as a UFP, you must place a 5.1kΩ pull-down resistor (tied to Ground) on CC1, and a separate 5.1kΩ pull-down resistor on CC2.

Why 5.1kΩ? -> The USB specification mandates the 5.1kΩ value (labeled as Rd) for UFP devices. This specific resistance tells the host computer, "I am a device that consumes power, please turn on VBUS (5V)".

Why two separate resistors? -> Active Type-C cables (like those meant for high-wattage charging or Thunderbolt) contain an e-marker chip that is powered via one of the CC pins.

Therefore, To act as a UFP (Upstream Facing Port / Device):
1. Place a 5.1kOhms pull-down resistor on the CC1 pin.
2. Place a separate 5.1kOhms pull-down resistor on the CC2 pin.

**Critical Rule:** Do not tie CC1 and CC2 together and route them through a single 5.1kΩ resistor. Active Type-C cables (which have e-marker chips inside) will misinterpret this, and the host will refuse to output 5V to VBUS.

#### How the Host Detects our Module

The host computer (known as the Downstream Facing Port or DFP) has pull-up resistors (labeled as Rp) on its own CC pins. When you plug in a Type-C cable, there is only one physical CC wire inside the cable itself. So when you plug in your module:
1. The single CC wire inside the cable connects the host's pull-up resistor to either CC1 or CC2 on your module's connector (depending on which way you inserted the plug).
2. This creates a voltage divider between the host's Rp and your module's 5.1kΩ Rd.
3. The host measures this voltage and detects your 5.1kΩ resistor. It now knows two things: a UFP device has been attached, and which orientation the cable is in (based on whether the voltage dropped on CC1 or CC2).
4. The host immediately turns on the 5V VBUS power, bringing our module to life.

#### Data Pins D+ & D-
Type-C Cables are reversible, to handle this, the connector has two sets of D+ and D- pins (D1+/D1- & D2+/D2-).
- For a USB 2.0 device, you simple short D1+ to D2+ and D1- to D2- right at the connector footprint.
- From there, route the combined D+ & D- traces to your ESD protection diodes, and then to the USB-to-UART bridge IC.
- Alos we need to ensure these two traces are routed as a differential pair and keep them the same length, keep them close together, and avoid using vias if possible.

#### How to Select the Right Connector
1. USB Standard: We select USB 2.0 for this design.
2. Mounting Type: We go for Hybrid Type which offers us SMT + Through-Hole design.
3. Pin Arrangements: The USB Type C connector must have a single row which are exposed and stick out slightly at the back, which allows us to drag-solder them easily.
4. EDA Footprint Availability: Try to go for a part that has a verified footprint in the CAD (KiCad) or trusted platform such as SnapMagic or UltraLibrarian. else you can design your own footpint if possible.

### Part Details

**Part Selected:** TYPE-C-31-M-12-Hroparts-5A 1 16P Female Type-C SMD USB Connectors ROHS

**Link:** [Type C 16P 5A USB Connector](https://robu.in/product/type-c-31-m-12-hroparts-5a-1-16p-female-type-c-smd-usb-connectors-rohs/)

**KiCad EDA Support:**
- Schematic Symbol: `USB_C_Receptacle_USB2.0_16P`
- PCB Footprint: `USB_C_Receptable_HRO_TYPE-C-31-M-12`

### Going through USB type Connector Datasheet

The pin layout for the 16-pin USB 2.0 Type-C receptacle [HRO TYPE-C-31-M-12] explains:
**Power & Ground Pins**
- GND Pins (A1, B12, A13, B1):
    - Function: System power and signal reference ground for the host side.
    - Physical Layout: On this connector, A1 & B12 are tied together into a single left pad, and A12 & B1 are tied into a single right pad.
    - How to Wire: Connect both outer ground pads to your USB Host Ground plane (GND_USB). Do not connect this directly to your target board's isolated ground plane.
- vbus pINS (A4, B9, A9, B4):
    - Function: +5V DC power provided by the host computer (up to 500mA by default for USB 2.0, or 1.5A/3A if advertised).
    - How to Wire: Connect these pads together to form your primary VBUS_5V rail. This rail will power:
        - Your USB-to-UART bridge IC (CP2102N / CH343).
        - Side 1 (Host side) of your digital isolator IC.
        - The primary input side of your isolated DC-DC converter.
    - Design Note: Place a 10µF bulk capacitor in parallel with a 0.1µF ceramic decoupling capacitor right next to these pins to filter incoming noise.

**Configuration Channel Pins**
- CC1 (A5) & CC2 (B5) Pins:
    - Function: Used by the host (laptop/PC) to detect cable insertion, determine plug orientation, and allocate power.
    - How to Wire:
        - Connect CC1 (A5) through a 5.1 kΩ resistor (1% tolerance) to `GND_USB`.
        - Connect CC2 (B5) through a separate 5.1 kΩ resistor (1% tolerance) to `GND_USB`.
    - Warning: Do not short CC1 and CC2 together to share a single 5.1 kΩ resistor! Doing so causes electronically marked (E-Marked) Type-C cables to misidentify your board, and the host will refuse to send 5V on VBUS.

**USB 2.0 Data Differentail Pair**
- DP1 (A6) & DP2 (B6) Pins are the USB Data Plus (D+) and DN1 (A7) & DN2 (B7) Pins are the USB Data Minus (D-).
- Function: The high-speed differential signal pair that carries USB 2.0 data packets.
- Why are there two DP and two DN pins? To allow the Type-C cable to be reversible. Depending on which way you plug the cable in, the host connects its single `D+`/`D-` pair to either set (A6/A7 or B6/B7).
- How to Wire:
    - On your PCB layout, short DP1 (A6) directly to DP2 (B6) right at the connector pads to create a single `USB_D+` trace.
    - Short DN1 (A7) directly to DN2 (B7) to create a single `USB_D-` trace.
    - Route these two traces as a differential pair through a TVS ESD protection diode array (like a USBLC6-2SC6) directly to the `D+` and `D-` pins of your bridge IC.

**Sideband Use Pins**
- SBU1 (A8) & SBU2 (B8) Pins are the Sideband Use Pins.
- Function: Reserved for USB Type-C "Alternate Modes" (e.g., carrying DisplayPort audio/video signals or Thunderbolt sideband data).
- How to Wire: Leave these pins floating / Not Connected (NC). USB-to-UART, RS485, and CAN devices do not use Sideband signals.

**Shield Pins**
- These pins are at the Outer Shell of the connector.
- Connect to `GND_USB` (optionally via 1MΩ + 4.7nF RC filter for ESD)
- How to Wire the Shield Pins:
    - Option A: Direct to USB Ground (The Simple Way)
        - How: Connect all four shield pins directly to your `GND_USB` plane.
        - Pros: Requires zero extra components. It provides a massive, immediate path for ESD to disperse into the ground plane.
        - Cons: It can create a "DC Ground Loop." If the host PC's chassis ground sits at a slightly different voltage potential than your board's local ground, a continuous DC current will flow through the USB cable's shield, which can actually introduce low-frequency noise.
    - Option B: The RC Filter (The Professional/Industrial Way)
        - How: Connect the shield pins to a dedicated SHIELD net in your schematic. Then, connect that SHIELD net to your GND_USB plane through a parallel Resistor-Capacitor (RC) network.
        - The Components: Use a high-value resistor (typically 1MΩ) in parallel with a high-voltage ceramic capacitor (typically 4.7nF or 10nF rated for 2kV or 3kV).
        - Why it works: The 1MΩ resistor blocks continuous DC ground loops from forming between the laptop chassis and your module. However, when a high-frequency event occurs (like an ESD strike or RF noise), the capacitor acts as a short circuit, instantly shunting the dangerous high-frequency energy directly into the `GND_USB` plane.

**Summary Checklist for Connector Connections**

| **Pin / Pad** | **Name** | **Schematic Connection** |
| --- | --- | --- |
| **A1, B12, A12, B1** | GND | Connect to `GND_USB` plane |
| **A4, B9, A9, B4** | VBUS | Connect to `VBUS_5V` (with 10µF + 0.1µF caps) |
| **A5** | CC1 | 5.1 kΩ resistor $\rightarrow$ `GND_USB` |
| **B5** | CC2 | 5.1 kΩ resistor $\rightarrow$ `GND_USB` |
| **A6, B6** | DP1, DP2 | Tie together $\rightarrow$ ESD Diode $\rightarrow$ Bridge IC $D+$ |
| **A7, B7** | DN1, DN2 | Tie together $\rightarrow$ ESD Diode $\rightarrow$ Bridge IC $D-$ |
| **A8, B8** | SBU1, SBU2 | Leave Unconnected (NC) |
| **Shield Pins** | Outer Shell | Connect to `GND_USB` (optionally via 1MΩ + 4.7nF RC filter for ESD) |

### USB C TVS Diode Selection - USBLC6-2SC6
The USBLC6 (specifically the USBLC6-2SC6 by STMicroelectronics) is the gold standard for USB 2.0 ESD protection in custom embedded hardware.
The USBLC6-2SC6 offers:
1. Ultra-Low Capacitance: It boasts a parasitic capacitance of typically 3.5 pF. This is low enough to ensure pristine signal integrity even at USB 2.0 High-Speed (480 Mbps) rates.
2. Rail-to-Rail Topology: Inside the SOT23-6L package, there are four "steering diodes" and one large central TVS clamping diode. If a massive positive ESD spike hits the `D+` line, the steering diode instantly dumps that energy into the `VBUS` line, where the central TVS diode clamps it safely to `GND`. If a negative spike hits, it steers it directly to `GND`.
3. VBUS Protection Included: Because it connects to the VBUS rail, it also provides a layer of transient protection for the 5V power line itself.

### USB C Overcurrent Protection
- Part 1 selected: WT1206-050
- Part 1 Link: [WT1206-050](https://sharvielectronics.com/product/wt1206-050-pptc-resettable-fuse-fuse-6v-500ma-1206-package/?srsltid=AfmBOooZ-QqrnW0wS0_e6LutVeNZSuHTBTzuZISC6iU85kQOpFlGASu-)

- Part 2: 1206L050YR
- Part 2 Link: [1206L050YR](https://www.lioncircuits.com/parts/1206L050YR?srsltid=AfmBOor-UaOBYRXg38iGdY1AvVDoV_3IbOvpE2enfzJ_iLIpDmzofCsz)

### USB C Power Line TVS Diode (EXTRA)
- Part Selected: ESD5Z5.0T1G
- Part Link: [ESD5Z5.0T1G](https://www.lioncircuits.com/parts/ESD5Z5.0T1G?srsltid=AfmBOop8Hnb_ssgq6vq52z1SgommCiPQPrxm-6Xce4t_LaQ80rv-LdHY)


#### Schematic Wiring (SOT23-6L Package)
The USBLC6-2SC6 has 6 pins. It acts as a "pass-through" filter sitting between the Type-C connector and your USB-to-UART Bridge IC.
- Pin 1 (I/O 1 - Input): Connect to DP1/DP2 (D+) coming directly from the Type-C connector.
- Pin 2 (GND): Connect to your `GND_USB` plane.
- Pin 3 (I/O 2 - Input): Connect to DN1/DN2 (D-) coming directly from the Type-C connector.
- Pin 4 (I/O 2 - Output): Connect this to the D- input on your USB-to-UART Bridge IC.
- Pin 5 (VBUS): Connect to your `VBUS_5V` rail.
- Pin 6 (I/O 1 - Output): Connect this to the D+ input on your USB-to-UART Bridge IC.

Note: The traces must physically flow into Pins 1 & 3 of the USBLC6, and the traces to the bridge IC must originate from Pins 6 & 4. This forces the physical energy to pass directly over the internal silicon of the protection diodes.


## USB-UART IC Selection for Design

### Design Research
A USB-to-UART Bridge IC is a chip that acts as a translator between two completely different communication protocols.

On one side, it speaks USB (Universla Serail Bus), which handles the complex enumeration process with your laptop, negotiates power, and receives packets of data. On the other side, it speaks UART which strips away the USB packaging and outputs raw, serial high/low pulses via TX & RX pins that microcontrollers understand.

To your laptop, the IC appears as a virtual COM Port (`COMx` on windows and `/dev/ttyUSBx` on Linux). When you type PuTTY or hit Upload in your IDE, the OS Sends data to that COM Port, and the IC receives it over USB, converts it, and sends it out the TX pin.

There are three primary families of USB-to-UART chips that is widely available:
1. **The FTDI Family (FT232RL, FT230X, FT234XD):** Future Technology Devices Internaltional (FTDI) is the old guard. They are famous for having bulletproof drivers that work on every OS without tweaking.
    1. **Pros:** Rock-solid driver stability. The FT230X and FT234XD are modern, low-cost versions with excellent features and very small footprints.
    2. **Cons:** FTDI chips are heavily counterfeited. In the past, FTDI released a Windows driver update that intentionally "bricked" counterfeit chips (changing their PID to 0000), which caused chaos in the hardware community. Genuine FTDI chips are also more expensive.
2. **The Silicon Labs Family (CP2102, CP2102N, CP2104):** Silicon Labs is widely considered the modern gold standard for high quality, embedded USB-to-UART Conversion. The CP2102N is thier current flagship for our application.
    1. **Pros:** Excellent driver support on Windows, Mac, and Linux (they "just work"). Very compact QFN packages. They feature an internal voltage regulator that can output 3.3V to power external components. The CP2102N specifically handles high baud rates (up to 3 Mbps) perfectly.
    2. **Cons:** The QFN (quad flat no-leads) packages can be slightly intimidating to hand-solder for absolute beginners, though a hot-air gun makes it trivial.
3. **The WCH Family (CH340G, CH340C, CH340E):** WCH (Nanjing Qinheng Microelectronics) makes ultra-cheap chips that dominate the Arduino clone market.
    1. **Pros:** Extremely cheap. The "C" and "E" variants have built-in crystal oscillators, saving BOM cost and PCB space. They are easy to solder (SOP-16 or MSOP-10 packages).
    2. **Cons:** Windows and Mac users often have to manually download and install the CH340 driver. They occasionally have quirky behavior with very high baud rates or specific DTR/RTS timing (which can sometimes cause issues with ESP32 auto-reset circuits).

### How to Select one for Your Design:
When designing an isolated tool that will interface with an STM32, a Raspberry Pi 5, or future RS485/CAN networks, you are building a professional-grade instrument. You need reliability over absolute bottom-dollar cost.

The Criteria for selecting a USB-to-UART Bridge IC:
1. **Driver Stability:** You do not want to fight drivers on your development machine. The IC must be recognized instantly.
2. **Integrated Components:** Look for an IC that has an internal oscillator and internal EEPROM (for custom device names). This saves you from having to place external crystals and memory chips on your PCB.
3. **Baud Rate Support:** Standard UART is 115200 bps, but flashing firmware to modern MCUs (or eventually interfacing with high-speed CAN) might require 1 Mbps to 3 Mbps.
4. **Full Handshaking Pins:** To implement the ESP32 auto-reset circuit and the RS485 Data Enable (DE) feature, you absolutely need an IC that exposes RTS and DTR. Make sure the specific package you select breaks these pins out
5. **3.3V vs 5V Logic:** The bridge IC will sit on the "USB side" of the isolation barrier, running off the 5V USB VBUS. However, most digital isolators (like the TI ISO77xx) perform best when both sides operate at 3.3V. You want an IC with an internal 3.3V regulator so you can power the USB-side of the isolator without adding an external LDO.

Possible options for USB-to-UART Bridge IC Selection which is widely available and proven are:
1. Silicon Labs CP2102N
2. WCH CH343
3. FTDI FT232RL
4. FTDI FT230X

#### Silicon Labs CP2102N
The CP2102N is the modern flagship from Silicon Labs and is arguably the best all-around choice for custom hardware design.
- **Baud Rate:** Up to 3 Mbps, which is more than enough for flashing an ESP32 or fast data logging.
- **Logic Levels:** Integrated 3.3V regulator; handles 3.3V and 5V logic easily.
- **Hardware Features:** Breaks out RTS, CTS, DTR, and other full modem signals, which is critical for ESP32 auto-reset circuits and RS485 direction control.
- **Integration:** Requires no external crystal oscillator, resistors on the USB lines, or external EEPROM, drastically simplifying the BOM.
- **Drawbacks:** Available primarily in QFN-20, QFN-24, or QFN-28 packages, which are small, leadless, and slightly harder to hand-solder without hot air.

#### WCH CH343
Nanjing Qinheng Microelectronics (WCH) is famous for the ultra-cheap CH340 found in Arduino clones. The CH343 is their much newer, vastly improved high-speed successor.
- **Baud Rate:** An impressive 6 Mbps maximum, doubling the CP2102N.
-  **Logic Levels:** It has an independent I/O voltage pin (VIO) that specifically supports 1.8V, 2.5V, 3.3V, and 5V directly. This makes it incredibly versatile if you want to interface with low-voltage FPGAs or modern processors.
- **Hardware Features:** Supports full hardware flow control and all modem signals (RTS, CTS, DTR). It also features a dedicated TNOW pin designed explicitly for hardware half-duplex RS485 switching, simplifying your future daughterboard.
- **Integration:** Like the CP2102N, it has a built-in crystal oscillator and power-on reset.
- **Drawbacks:** While Linux support is built-in (kernel 5.x+), Windows and Mac users may occasionally need to manually install the WCH driver, though it's less of an issue now than with the older CH340.

#### FTDI FT232RL
The FT232RL was the industry standard for a decade. While older, it is still heavily used due to its bulletproof driver ecosystem.
- **Baud Rate:** Up to 3 Mbps.
- **Logic Levels:** Built-in 3.3V regulator, supports 1.8V to 5V I/O via a VCCIO pin.
- **Hardware Features:** Full modem control (RTS, CTS, DTR) and a dedicated TXDEN pin specifically for RS485 direction control.
- **Driver Support:** Unmatched. Every OS recognizes genuine FTDI chips instantly.
- **Drawbacks:** It is expensive (often 3x to 5x the cost of a CP2102N or CH343). The market is also flooded with counterfeits, which FTDI drivers have historically bricked.

#### FTDI FT230X
If you want FTDI's driver stability without the high cost and large footprint of the FT232RL, the FT230X is the answer.
- **Baud Rate:** Up to 3 Mbps.
- **Integration:** Very compact, requires no external crystal, and features integrated battery charging detection.
- **Drawbacks:** It is a "basic" UART bridge. It breaks out RTS and CTS, but usually drops DTR. This makes it unsuitable for the ESP32 auto-reset circuit, though it can still handle RS485.

### Recommendation for Our Modular Design
If soldering QFN packages is possible, the Silicon Labs CP2102N remains the most robust choice with the least driver friction.
However, if an easier soldering experience (SOP-16 package) is preferred and value the dedicated RS485 `TNOW` pin and native 1.8V IO support, the WCH CH343 is a phenomenal, modern alternative that gives you all the control pins you need for future expansion boards like RS485, CAN, RS232 and so on.

### Part Details

**Part Selected:** CP2102N-A02-GQFN24R - USB Interface IC USBXpress - USB to UART Bridge QFN24

**Link:** [CP2102N-GQFN24R IC Link](https://evelta.com/cp2102n-a02-gqfn24r-usb-interface-ic-usbxpress-usb-to-uart-bridge-qfn24/?sku=029-CP2102N-A02-GQFN24R&utm_source=google&utm_campaign=19631771445&utm_medium=cpc&utm_content=&utm_term=&gad_source=1&gad_campaignid=19639270469&gbraid=0AAAAADwtsXl8Op3-6GXY-IERDRNTgOBbL&gclid=CjwKCAjwj7HTBhBiEiwA8s35Om3-aVp3vB-N9s5K7jmhGVIeMftIjTMpv306FMF6I0V0ixiyjQZWeBoClLUQAvD_BwE)

**KiCad EDA Support:**
- Schematic Symbol: `CP2102N-Axx-xQFN24`
- PCB Footprint: `Package_DFN_QFN:QFN-24-1EP_4x4mm_P0.5mm_EP2.6x2.6mm`

### Going through CP2102N Datasheet
WIP


## Digital Isolator IC Selection for Design

### Design Research
A Digital Isolator is essentially a silicon firewall. It allows data (1s and 0s) to pass between your USB bridge and your target hardware, but physically blocks voltage spikes, griund loops, and electrical noise.

Older designs used optocouplers (LEDs and phototransistors), which are slow, consume a lot of power, and degrade over time. Modern digital isolators use microscopic internal capacitors or transformers built directly into the silicon to jump the data across a microscopic gap (silicon dioxide).

#### How Digital Isolators will fit into our application
A digital isolator sits exactly between your USB-to-UART bridge (CP2102N) and your target interface (the pin headers, or future RS485/CAN transceivers).

To function, an isolator requires power on both sides of the barrier:
1. **Side 1 (Host/USB):** Powered by the 3.3V output from your CP2102N. The ground pin here connects to your USB Ground.
2. **Side 2 (Target/Isolated):** Powered by your isolated DC-DC converter (or external power from the target board). The ground pin here connects to your Isolated Ground.
3. **The Golden Rule:** The entire point of the isolator is ruined if the copper planes for USB Ground and Isolated Ground ever touch. Your PCB layout must have a physical gap (at least 2mm to 8mm, depending on voltage ratings) underneath the isolator IC where absolutely no copper, traces, or planes exist on any layer.

#### How to Select the Isolator
We need to send 5 signals across the barrier i.e. TXD, RXD, RTS, CTS, DTR. This dictates our selection criteria.

So the four parameters, we must check in the datasheet to select the right isolator:
1. **Channel Directionality:** Isolators are sold with specific forward/reverse channel layouts.
    1. Forward (Host to target): TX, RTS, DTR
    2. Reverse (Target to Host): RX, CTS
    3. Requirement: We will need a 3-Forward/2-Reverse setup, because 5-Channel Isolators are rare and designers typically buy a 6-Channel isolator (like 4-Forward/2-Reverse) and leave one pin floating, or use a 4-Channel IC and Drop the CTS Pin.
2. **Speed (Data Rate):** The CP2102N maxes out at 3 Mbps. Almost all modern capacitive digital isolators support 100 Mbps (like the TI ISO77xx series), so this is rarely a bottleneck.
3. **Default Output State (Failsafe):** If your laptop goes to sleep, Side 1 of the isolator loses power, but Side 2 might still be powered by the target device. You must select an isolator with a Default High (often denoted by an "F" suffix, like ISO7762F). If it defaults low, it will send a continuous "0" (Break state) to the target's UART RX pin, causing errors.
4. **Package Size (Creepage):** For high-voltage protection (5kVrms), you must select the "Wide-Body SOIC-16" package (DW suffix). The physical width of the chip provides the 8mm of surface distance required to stop high-voltage arcs.

#### Recommended ICs for this Design
Baed on our requirement to support RS485 & Standard UART with flow control"
1. If you stick to 4 Channels (Drop DTR, keep RTS for RS485): Use the Texas Instruments ISO7741F (3 Forward, 1 Reverse) or ISO7742F (2 Forward, 2 Reverse).
2. If you upgrade to 6 Channels (Full ESP32 Reset + Flow Control): Use the Texas Instruments ISO7762F (4 Forward, 2 Reverse).
3. Alternative (Magnetic): The Analog Devices ADuM162N provides 6 channels (4 Forward, 2 Reverse) using micro-transformers instead of capacitors, offering similar excellent performance.

### Part Details
**Part Selected:** ISO6762DWR

**Link:** [ISO6762DWR IC Link from Lion Circuits](https://www.lioncircuits.com/parts/ISO6762DWR)

**KiCad EDA Support:**
- Schematic Symbol: `XX`
- PCB Footprint: `XX`

#### Part Information
The ISO6762DWR is indeed a 6-channel (4 Forward, 2 Reverse) digital isolator from Texas Instruments. It perfectly matches the required channel configuration for your modular tool.
**DBQ Package:** This is a 16-pin SSOP (Shrink Small Outline Package). It is very compact.
**Isolation Rating:** Because the chip is physically narrow, the creepage distance is smaller. It is rated for 5000 Vrms.

A 5000 Vrms (5 kV) rating is more than enough to protect your laptop and handle massive ground loops in industrial environments, robotics, drones, or standard RS485/CAN networks. It will easily protect against accidental shorts to 12V or 24V rails.

You only strictly need the wide-body DW package (5000 Vrms) if your hardware is interacting directly with mains AC voltage (e.g., 220V/110V grid applications) or medical equipment attached to patients, where strict safety regulations mandate 8mm of physical creepage.

#### Part Number Breakdown Confirmation:
Here is exactly what the part number `ISO6762DWR` means:
- ISO6762DWR: The base IC (6 Channels: 4 Forward, 2 Reverse).
ISO67: The silicon family (Cost-optimized, 50 Mbps general-purpose isolation).
- 62: The channel configuration (6 total channels: 4 Forward, 2 Reverse).
- DW: The package type (Wide-body SOIC-16). This dictates your physical PCB footprint and provides the 8mm clearance and creepage needed for 5000 Vrms isolation.
- R: The packaging format (Tape and Reel). This tells the distributor how the chips are physically shipped to you for pick-and-place assembly.
- (No Suffix): The absence of an "F" before the package code confirms the default output state is logic HIGH when unpowered.

---