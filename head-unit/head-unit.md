# Head Unit

I have decided to build my own head unit as I want good audio and I also want to collect telmetry while I am driving a car. Notice this project assume you will be collecting telemtry from OB2 compatilbe unit.
This project will also give possablity to use android auto and hands-free system for pickingup callas and making calls.

## Contents

1. [Parts](#parts)
    - [Computer](#computer)
    - [Audio](#audio)
    - [Power section Amplifier](#power-section-amplifier)
    - [Power section Raspberry Pi](#power-section-raspberry-pi)
    - [Software](#software)
    - [Additional parts](#additional-parts)
2. [Whole system high level schema](#whole-system-high-level-schema)
    - [Meramid schema](#meramid-schema)
    - [Picture](#picture)
3. [How to connect Amplifier](#how-to-connect-amplifier)
    - [Amplifier - Instruction step by step](#amplifier---instruction-step-by-step)
    - [Amplifier - High level schema (Mermaid)](#amplifier---high-level-schema-mermaid)
    - [Amplifier - High level schema (Plain)](#amplifier---high-level-schema-plain)
    - [Amplifier - High level schema (Picture)](#amplifier----high-level-schema-picture)
4. [How to connect Computer](#how-to-connect-computer)
    - [RPI5 - Instruction step by step](#rpi5---instruction-step-by-step)
    - [RPI5 - High level schema (Mermaid)](#rpi5---high-level-schema-mermaid)
    - [RPI5 - High level schema (Plain)](#rpi5---high-level-schema-plain)
    - [RPI5 - High level schema (Picture)](#rpi5---high-level-schema-picture)

## Parts

### Computer

1. Raspberry Pi 5 8GB RAM
2. Raspberry Pi Active Cooler
3. RTC JST Battery
4. Waveshare 2-CH CAN FD HAT (Waveshare 17075)
5. Waveshare PCIe to M.2 adapter (Waveshare 26583)
6. Waveshare LCD IPS 7" Screen (Waveshare 11199)
7. Geekworm X1200 Raspberry Pi 5 UPS HAT (Geekworm X1200)
8. SSD M.2 2230 PCIe NVMe 512GB (WESTERN DIGITAL SN530 SDBPTPZ-512G-1012)

### Audio

1. Audio Amplifier (PIONEER GM-D8704)
2. 2x fron speaker + 2x column woofer (Audio System R165 EVO3)
3. 1x subwoofer 20cm 200/280W RMS (Audio System R08 EVO)
4. Microphone usb adapter UGREEN CM477 USB-A - Jack 3.5 mm
5. Behringer U-CONTROL UCA202
6. 2x RCA(male) - RCA(male) wire
7. Microphone (3.5mm audio jack) or cusotm project
   - Microphone 2,2kΩ;Ø9,7x5mm;0,5mA;1÷10V;-24±3dB (PUI AUDIO AOM-5024L-HD-R)
   - Jack 3.5mm male connector with 3 pols (AMPHENOL KS3PC-AU)
   - 3M of wire with 2 vines and good shield (TASKER C114)

### Power section Amplifier

1. M8 Ring Terminal 16mm²
2. Fuse holder Mini-ANL 16mm² + Fuse 80A
3. Wire H01N2-D 16mm² (3m)
4. Wire OFC 2x2.5mm² to connect subwoffer (2m)
5. Wire FLRY-B 1mm² (1m)

### Power section Raspberry Pi

1. M8 Ring Terminal 1.5mm²
2. Sealed fuse holder 1.5mm^2 + Fuse 5A
3. 12V/24V to 5V/5A DC-DC Converter
4. USB-C male connctor
5. UPS Hat with power section for Raspberry Pi 5 (Geekworm X1200)

### Software

1. Raspberry PI OS
2. LIVI Android Auto emulator
3. Python script to control safe shotdown

### Additional parts

1. LiYCY 2x0.34 control cable (to read data from OB2 and send to CAN Hat to Rpi 5)
2. ISO adapter connector for Toyota MR2
3. [OPTIONAL] USB - CAN adapter TJA1051T/3 (needed for testing CAN hat signals)

## Whole system high level schema

### Meramid schema

```mermaid

flowchart TD

    %% ==========================================
    %% 1. CAR BATTERY AND POWER DISTRIBUTION
    %% ==========================================

    subgraph BATTERY["1. CAR BATTERY AND POWER DISTRIBUTION"]
        BAT["Car Battery 12V"]
        DIST(("12V Power Distribution"))
        CHASSIS["Car Chassis / Ground Point"]
        IGN["Ignition Switch ACC / IGN"]

        BAT --> DIST
    end

    %% ==========================================
    %% 2. DEDICATED AMPLIFIER POWER SUPPLY
    %% ==========================================

    subgraph AMP_POWER["2. AMPLIFIER POWER SUPPLY — SEPARATE CIRCUIT"]
        AFUSE["Mini-ANL Fuse 60–80A<br/>Maximum 30 cm from battery"]
        AMP_BATT["Pioneer GM-D8704<br/>BATT +12V Terminal"]
        AMP_GND["Pioneer GM-D8704<br/>GND Terminal"]
        AMP_REM["Pioneer GM-D8704<br/>REM Terminal"]

        DIST -->|"16 mm² H01N2-D Power Cable"| AFUSE
        AFUSE --> AMP_BATT

        CHASSIS -->|"16 mm² Ground Cable"| AMP_GND
        IGN -->|"ACC / IGN Signal"| AMP_REM
    end

    %% ==========================================
    %% 3. RASPBERRY PI POWER SUPPLY
    %% ==========================================

    subgraph PI_POWER["3. RASPBERRY PI 5 POWER SUPPLY"]
        ISOLATOR["Main Battery Isolator Switch"]
        FUSE5["5A Fuse"]
        RELAY30["JD 1914 — Pin 30<br/>12V Power Input"]
        RELAY87["JD 1914 — Pin 87<br/>Switched 12V Output"]
        RELAY86["JD 1914 — Pin 86<br/>IGN / Ignition Control"]
        RELAY85["JD 1914 — Pin 85<br/>Coil Ground"]
        CONVERTER["DC-DC Converter<br/>12V / 24V → 5V / 5A"]
        USB_C["USB-C Male Connector"]
        UPS["Geekworm X1200 UPS HAT<br/>+ 18650 Battery Cells"]
        PI["Raspberry Pi 5<br/>8GB RAM"]

        DIST -->|"1.0–1.5 mm² Wire"| ISOLATOR
        ISOLATOR --> FUSE5
        FUSE5 --> RELAY30

        RELAY30 -.->|"Internal Working Contact"| RELAY87

        IGN --> RELAY86
        CHASSIS --> RELAY85
        RELAY87 --> CONVERTER
        CHASSIS -->|"DC-DC Input Ground"| CONVERTER

        CONVERTER -->|"5V / 5A Output"| USB_C
        USB_C --> UPS
        UPS -->|"Power via Pogo Pins"| PI
    end

    %% ==========================================
    %% 4. RASPBERRY PI MODULES
    %% ==========================================

    subgraph PI_MODULES["4. RASPBERRY PI 5 MODULES"]
        RTC["RTC JST Battery"]
        COOLER["Raspberry Pi Active Cooler"]
        PCIE["Waveshare PCIe to M.2 Adapter<br/>26583"]
        SSD["WD SN530 NVMe SSD<br/>512GB M.2 2230"]
        CAN["Waveshare 2-CH CAN FD HAT<br/>17075"]

        PI --> RTC
        PI --> COOLER
        PI --> PCIE
        PCIE --> SSD
        PI --> CAN
    end

    %% ==========================================
    %% 5. DISPLAY
    %% ==========================================

    subgraph DISPLAY["5. DISPLAY"]
        HDMI["Micro-HDMI Port"]
        USB_TOUCH["USB-A Port<br/>Touch / Data"]
        SCREEN["Waveshare 7-inch IPS LCD<br/>11199"]

        PI --> HDMI
        HDMI -->|"HDMI Cable"| SCREEN

        PI --> USB_TOUCH
        USB_TOUCH -->|"Touch Data"| SCREEN
    end

    %% ==========================================
    %% 6. AUDIO SOURCES AND MICROPHONE
    %% ==========================================

    subgraph AUDIO_SOURCE["6. AUDIO SOURCES AND MICROPHONE"]
        UGREEN["UGREEN CM477<br/>USB-A to 3.5mm Jack Adapter"]
        MIC["3.5mm TRS Microphone<br/>or PUI AOM-5024L-HD-R"]
        BEHRINGER["Behringer U-CONTROL UCA202"]
        RCA["RCA / Cinch Cables<br/>Left + Right Channels"]

        PI -->|"USB Audio"| UGREEN
        UGREEN --> MIC

        PI -->|"USB Audio"| BEHRINGER
        BEHRINGER -->|"LINE OUT L / R"| RCA
    end

    %% ==========================================
    %% 7. PIONEER AMPLIFIER AND CONFIGURATION
    %% ==========================================

    subgraph AMPLIFIER["7. PIONEER GM-D8704 AMPLIFIER"]
        AMP["Pioneer GM-D8704<br/>4-Channel Amplifier"]
        INPUT["INPUT CH A — L / R"]
        SELECT["INPUT SELECT: 2CH"]
        CH_A["CHANNEL A<br/>HPF @ 80Hz"]
        CH_B["CHANNEL B<br/>LPF @ 80Hz"]
        TERM_A_L["CH A: L+ / L-"]
        TERM_A_R["CH A: R+ / R-"]
        TERM_B_LPLUS["CH B: L+ (POSITIVE)"]
        TERM_B_RMINUS["CH B: R- (NEGATIVE)"]
        UNUSED["CH B: L- and R+<br/>Not Used"]

        RCA --> INPUT
        INPUT --> AMP
        AMP --> SELECT
        SELECT --> CH_A
        SELECT --> CH_B

        CH_A --> TERM_A_L
        CH_A --> TERM_A_R

        CH_B --> TERM_B_LPLUS
        CH_B --> TERM_B_RMINUS
        CH_B --> UNUSED

        AMP_BATT --> AMP
        AMP_GND --> AMP
        AMP_REM --> AMP
    end

    %% ==========================================
    %% 8. FRONT SPEAKERS
    %% ==========================================

    subgraph FRONT["8. FRONT SPEAKERS"]
        FRONT_L["Front Left Speaker<br/>Audio System R165 EVO3"]
        FRONT_R["Front Right Speaker<br/>Audio System R165 EVO3"]

        TERM_A_L --> FRONT_L
        TERM_A_R --> FRONT_R
    end

    %% ==========================================
    %% 9. SUBWOOFER — SERIES CONNECTED VOICE COILS
    %% ==========================================

    subgraph SUB["9. SUBWOOFER — SERIES VOICE COIL CONNECTION"]
        SUB_C1_P["Voice Coil 1: (+)"]
        SUB_C1_N["Voice Coil 1: (-)"]
        SUB_C2_P["Voice Coil 2: (+)"]
        SUB_C2_N["Voice Coil 2: (-)"]
        SUBWOOFER["Audio System R08 EVO<br/>Dual Voice Coil Subwoofer"]

        TERM_B_LPLUS --> SUB_C1_P
        SUB_C1_N -->|"Series Jumper"| SUB_C2_P
        TERM_B_RMINUS --> SUB_C2_N

        SUB_C1_P --- SUBWOOFER
        SUB_C1_N --- SUBWOOFER
        SUB_C2_P --- SUBWOOFER
        SUB_C2_N --- SUBWOOFER
    end

    %% ==========================================
    %% COLOR STYLING
    %% ==========================================

    classDef power fill:#ffcccc,stroke:#b30000,stroke-width:2px,color:#222;
    classDef relay fill:#fff0c2,stroke:#b87900,stroke-width:2px,color:#222;
    classDef core fill:#d7f2d4,stroke:#238636,stroke-width:2px,color:#222;
    classDef display fill:#e9defa,stroke:#7040a0,stroke-width:2px,color:#222;
    classDef audio fill:#d9efff,stroke:#1674b8,stroke-width:2px,color:#222;

    class BAT,DIST,AFUSE,AMP_BATT,AMP_GND,AMP_REM power;
    class ISOLATOR,FUSE5,RELAY30,RELAY87,RELAY86,RELAY85,CONVERTER,USB_C relay;
    class UPS,PI,RTC,COOLER,PCIE,SSD,CAN core;
    class HDMI,USB_TOUCH,SCREEN display;
    class UGREEN,MIC,BEHRINGER,RCA,AMP,INPUT,SELECT,CH_A,CH_B,TERM_A_L,TERM_A_R,TERM_B_LPLUS,TERM_B_RMINUS,UNUSED,FRONT_L,FRONT_R,SUB_C1_P,SUB_C1_N,SUB_C2_P,SUB_C2_N,SUBWOOFER audio;

```

### Picture

![whole system connection connection](../pictures/rpi-amplifier-whole-connection.png)
>This picture was generated by AI.

## How to connect Amplifier

### Amplifier - Instruction step by step

#### **Step 1: Amplifier Power & Control Terminals**

All power cables must be of appropriate gauge to ensure stable operation and safety inside the vehicle.

- **`+12V (BATT)`**: Run a heavy-gauge power wire directly from the car battery. **You must install a Mini ANL fuse (60A–80A)** on this power wire within a maximum distance of 30 cm (12 inches) from the battery terminal.
- **`GND` (Ground)**: Connect a short, heavy-gauge ground wire to a clean, unpainted, solid metal point on the car chassis.
- **`REM` (Remote / Turn-On Signal)**: Connect the turn-on control wire (e.g., to the vehicle's ACC / ignition switch line or system turn-on control output).

---

#### **Step 2: Audio Signal Connection (USB DAC U-CONTROL UCA202 $\rightarrow$ Amplifier)**

Use a single high-quality, dual-shielded RCA (Cinch) cable to connect the audio source to the amplifier:

- **DAC Output**: Connect the RCA cable to the `LINE OUT` jacks (labeled **L** and **R**) on the **Behringer U-CONTROL UCA202** audio interface.
- **Amplifier Input**: Plug the RCA connectors directly into the **`INPUT CH A` (`L` and `R`)** jacks on the Pioneer amplifier panel.
- **`INPUT CH B` Jacks (Left Unused)**: The UCA202 interface only provides one stereo output pair. The audio signal is passed internally to Channel B inside the amplifier by setting the `INPUT SELECT: 2CH` switch in Step 3.
- **`RCA OUTPUT` Jacks (Left Unused)**: These pass-through outputs are only used to chain signal to a second external amplifier, which is not required in this setup.

---

#### **Step 3: Switch & Crossover Configuration**

Set the switches on the amplifier body according to the following specifications for a front component set + subwoofer system:

- **`INPUT SELECT`**: Set to the **`2CH`** position (this is crucial as it internally splits the signal from `CH A` input to all 4 output channels, eliminating the need for external RCA Y-splitters).
- **`CHANNEL A` (Front Speakers)**: Set the crossover switch to **`HPF`** (High-Pass Filter), and set the frequency dial (`FREQ`) to approximately **80 Hz** (protects the front door speakers from reproducing low bass frequencies).
- **`CHANNEL B` (Subwoofer)**: Set the crossover switch to **`LPF`** (Low-Pass Filter), and set the frequency dial (`FREQ`) to approximately **80 Hz** (ensures only low frequencies are sent to the subwoofer).

---

#### **Step 4: Connecting Front Speakers – Section A (Stereo)**

The front component set (Audio System Radion R165 EVO3) is powered in standard stereo mode via Channel A:

- **Left Channel**: Connect the speaker wires from the left crossover network to the **`CH A (L+)`** and **`CH A (L-)`** terminals on the amplifier.
- **Right Channel**: Connect the speaker wires from the right crossover network to the **`CH A (R+)`** and **`CH A (R-)`** terminals on the amplifier.

---

#### **Step 5: Connecting & Bridging the Subwoofer – Section B (Bridged)**

The dual-voice-coil (DVC) subwoofer (Audio System R08 EVO) requires a series voice-coil connection and bridged amplifier wiring to present a safe $4\,\Omega$ load to the amplifier:

1. **Subwoofer Voice Coil Wiring (Series Connection):**

   - Connect a short jumper wire between **Coil 1 (-)** terminal and **Coil 2 (+)** terminal directly on the subwoofer terminal block.

2. **Amplifier-to-Subwoofer Wiring:**

   - Connect the wire from the **`CH B (L+)`** amplifier terminal to the open **Coil 1 (+)** terminal on the subwoofer.
   - Connect the wire from the **`CH B (R-)`** amplifier terminal to the open **Coil 2 (-)** terminal on the subwoofer.

3. **`CH B (L-)` and `CH B (R+)` Terminals (Left Unused)**: In bridged mode, the amplifier draws power exclusively from the outer terminals (`L+` and `R-`), leaving the inner Channel B terminals unconnected.

### Amplifier - High level schema (Mermaid)

```mermaid
graph TD
    %% Zasilanie główne wzmacniacza i systemu
    Battery["Car Battery 12V"] --> Fuse["Mini ANL Fuse 60-80A"]
    Fuse --> AmpBATT["Terminal: BATT (+12V)"]
    
    %% Zasilanie Raspberry Pi z akumulatora
    Pi5["Sound Source (Raspberry Pi 5)"]
    
    Chassis["Chassis Ground"] --> AmpGND["Terminal: GND"]
    Ignition["ACC Ignition Signal"] --> AmpREM["Terminal: REM"]

    %% Źródło dźwięku (Raspberry Pi 5)
    Pi5 --> DAC["USB DAC (U-CONTROL UCA202)"]
    DAC --> AmpInput["INPUT CH A"]
    
    %% Jednolity Blok Wzmacniacza
    subgraph Pioneer_Amp [Pioneer GM-D8704 Amplifier]
        AmpBATT
        AmpGND
        AmpREM
        
        AmpInput --> Switch["INPUT SELECT 2CH"]
        
        Switch --> ChannelA["CH A HPF 80Hz"]
        Switch --> ChannelB["CH B LPF 80Hz"]
        
        ChannelA --> TermA_L["CH A Front Left Terminals"]
        ChannelA --> TermA_R["CH A Front Right Terminals"]
        
        subgraph Bridged_Section [CH B Bridged Output Terminals]
            ChannelB --> TermB_LPlus["CH B L+ Terminal POSITIVE"]
            ChannelB --> TermB_RMinus["CH B R- Terminal NEGATIVE"]
            ChannelB --> TermB_Unused["CH B L- and R+ Terminals LEFT OPEN"]
        end
    end

    %% Szczegółowe terminale cewek subwoofera
    subgraph Subwoofer_Terminals [Subwoofer Audio System - 4 Terminals DVC]
        Sub_C1_Plus["Coil 1: Terminal (+)"]
        Sub_C1_Minus["Coil 1: Terminal (-)"]
        Sub_C2_Plus["Coil 2: Terminal (+)"]
        Sub_C2_Minus["Coil 2: Terminal (-)"]
        
        Sub_C1_Minus -->|Series Jumper| Sub_C2_Plus
    end

    %% Wyjścia głośnikowe przednie
    TermA_L --> FrontLeft["Front Left Speaker Radion"]
    TermA_R --> FrontRight["Front Right Speaker Radion"]
    
    %% Połączenia ze wzmacniacza do terminali subwoofera
    TermB_LPlus --> Sub_C1_Plus
    TermB_RMinus --> Sub_C2_Minus
```

### Amplifier - High level schema (Plain)

```plaintext
[ CAR BATTERY 12V ]
       │
       ├──( 1. Main power cable + Fuse 60-80A )──────────────────────────────┐
       │                                                                      ▼
       └──( 2. 12V power supply to the system )                         [ PIONEER GM-D8704 AMPLIFIER ]
              │                                                        • [ +12V ] (BATT)
              ▼                                                        • [ GND ]  (Chassis Ground)
        [ BUCK CONVERTER 12V -> 5V/5A ]                                        • [ REM ]  (ACC / Ignition signal)
               │                                                                      │
               └──( Stable 5V / 5A output )                                           │
                      │                                                               │
                      ▼                                                               │
        [ GEEKWORM X1200 UPS HAT + SANYO BATTERIES ]                                  │
               │                                                                      │
               └──( Vertical power via Pogo-pins )                                    │
                      │                                                               │
                      ▼                                                               │
        [ RASPBERRY PI 5 + NVMe SSD + CAN HAT ]                                       │
               │                                                                      │
               └──( USB audio data transfer )                                         │
                      │                                                               │
                      ▼                                                               │
        [ USB DAC (PCM5102A) ]                                                        │
               │                                                                      │
               └──( RCA Cable: L and R Channels ) ────────────────────────────────────► [ INPUT CH A (L / R) ]
                                                                                      [ INPUT CH B ] (Left empty)
                                                                                              │
                                                            ┌─────────────────────────────────┴─────────────────────────────────┐
                                                            ▼                                                                   ▼
                                                [ CH A OUTPUT SECTION ]                                             [ CH B OUTPUT SECTION ]
                                                • Setting: HPF @ 80 Hz                                              • Setting: LPF @ 80 Hz
                                                • CH A (L+ / L-) ──► Front Left Speaker                             • CH B (L+)  ──► Subwoofer (+)
                                                • CH A (R+ / R-) ──► Front Right Speaker                            • CH B (R-)  ──► Subwoofer (-)
                                                                                                                        (CH B: L- and R+ left open)
```

### Amplifier -  High level schema (Picture)

![amplifier connection](../pictures/amplifier-connection.png)
>This picture was generated by AI.

## How to connect Computer

### RPI5 - Instruction step by step

#### **Step 1: Main Power Section Setup (12V)**

1. **Main Battery Isolator Switch:** Connect to the positive terminal of the 12V car battery. Use a wire with a cross-section of $1.0\text{--}1.5\text{ mm}^2$.
2. **Circuit Protection:** Run the output from the isolator switch directly to the **5A fuse** holder.
3. **Connecting to the JD 1914 Automotive Relay:**
   - Crimp a terminal onto the output wire from the 5A fuse and connect it to **Pin 30** of the relay (12V power input).
   - Connect the **IGN (Ignition)** signal wire from the ignition switch to **Pin 86** (relay coil control).
   - Connect **Pin 85** to a factory chassis ground point.
   - Fully isolate **Pin 87a** using heat-shrink tubing to prevent short circuits.
   - Run the switched 12V output wire from **Pin 87**.

---

#### **Step 2: Voltage Conversion and UPS Wiring**

1. **DC-DC Step-Down Converter (12V $\rightarrow$ 5V):**
    - Connect the wire from relay **Pin 87** to the positive input ($IN+$) of the converter.
    - Connect the input ground ($IN-$) to the factory chassis ground point.
2. **Geekworm UPS Module:**
    - Connect the 5V output from the DC-DC converter to the power input port of the Geekworm UPS module via a USB-C cable (secure the connector with thermal glue to resist vibration).
    - Insert the 18650 cells into the UPS battery holder.

---

#### **Step 3: Installing HATs and Modules on the Raspberry Pi 5**

1. **RTC Battery Backup:** Plug the RTC battery (CR1220) into the dedicated RTC connector on the Raspberry Pi 5 board.
2. **NVMe SSD HAT:** Install the M.2 2230 SSD onto the NVMe HAT, then connect the HAT to the Raspberry Pi 5 PCIe port using the FFC ribbon cable.
3. **CAN Bus HAT:** Mount the CAN Bus HAT onto the 40-pin GPIO header of the Raspberry Pi 5.
4. **Pi Power Supply:** Connect the Geekworm UPS module to the Raspberry Pi 5 stack (via GPIO power pins or the dedicated pogo-pin/USB-C connection).

---

#### **Step 4: Touchscreen Display Setup (Waveshare 7" IPS)**

1. **Video Signal:** Connect the micro-HDMI port on the Raspberry Pi 5 to the HDMI input port on the Waveshare screen using a micro-HDMI $\rightarrow$ HDMI cable.
2. **Touch & Data:** Connect one of the USB-A ports on the Raspberry Pi 5 to the USB (Touch/Data) port on the screen using a USB cable.

---

#### **Step 5: Independent Audio Setup (USB Dual-Card)**

1. **Unitek Y-247A Sound Card (Audio Input / Microphone):**
    - Plug the Unitek Y-247A sound card directly into a free USB-A port on the Raspberry Pi 5.
    - Connect the Amphenol TRS 3.5mm microphone to the 3.5mm mic jack on the Unitek card.
2. **Behringer UCA202 Audio Interface (Audio Output / DAC):**
    - Plug the Behringer UCA202 interface directly into another free USB-A port on the Raspberry Pi 5.
    - Connect the RCA (Cinch) outputs from the Behringer via shielded RCA cables to the input terminals of the car audio amplifier.
3. **Speakers:** Connect the speaker output terminals from the amplifier to the car speakers.

### RPI5 - High level schema (Mermaid)

```mermaid
graph TD
    %% Main Power Section
    Batt["Car Battery"] --> Isolator["Main Battery Isolator Switch"]
    Isolator -->|1.0-1.5 mm² Wire| Fuse["5A Fuse"]
    
    %% Compact relay group as a single component
    subgraph Relay ["JD 1914 Automotive Relay"]
        Pin30["Pin 30: 12V Power Input"]
        Pin86["Pin 86: IGN / Ignition Control"]
        Pin85["Pin 85: Coil Ground (GND)"]
        Pin87["Pin 87: Switched 12V Power Output"]
        
        Pin30 -.->|"Internal working contact"| Pin87
    end

    Fuse --> Pin30
    Ignition["Car Ignition Switch (IGN)"] --> Pin86
    CarBody["Car Body / Ground Point"] --> Pin85
    
    Pin87 -->|"When contacts close"| Converter["DC-DC Converter 12V -> 5V"]
    CarBody -.->|"Input Ground"| Converter
    
    Converter -->|"USB-C Plug + Thermal Glue"| UPS["Geekworm UPS Module + 18650 Cells"]
    UPS --> RPi["Raspberry Pi 5"]
    
    %% RPi 5 HATs and Peripherals Section
    RPi --> RTC["RTC Battery"]
    RPi --> NVMe["NVMe HAT + SSD"]
    RPi --> Can["CAN Bus HAT"]
    
    %% Display Section (Waveshare 11199)
    RPi --> HDMI["Micro-HDMI Port"]
    HDMI -->|"HDMI Cable (Video)"| Screen["Waveshare 7 inch IPS LCD (11199)"]
    RPi --> USBDisp["USB-A Port (Touch)"]
    USBDisp -->|"USB Cable (Touch/Data)"| Screen
    
    %% Audio Section (Dual-Card)
    RPi --> USB1["USB-A Port 1"]
    RPi --> USB2["USB-A Port 2"]
    
    USB1 --> Unitek["Unitek Y-247A Sound Card"]
    Unitek --> Mic["Amphenol TRS 3.5mm Microphone"]
    
    USB2 --> Behringer["Behringer UCA202 Audio Interface"]
    Behringer -->|"RCA / Cinch Cables"| Amp["Car Audio Amplifier"]
    Amp --> Speakers["Car Speakers"]

    %% Color Styling
    classDef power fill:#ffcccc,stroke:#333,stroke-width:2px;
    classDef relayBox fill:#fff3cd,stroke:#333,stroke-width:2px;
    classDef core fill:#d4edda,stroke:#333,stroke-width:2px;
    classDef audio fill:#d1ecf1,stroke:#333,stroke-width:2px;
    classDef display fill:#e2d9f3,stroke:#333,stroke-width:2px;
    
    class Batt,Isolator,Fuse,Converter power;
    class Pin30,Pin86,Pin85,Pin87 relayBox;
    class UPS,RPi,NVMe,Can,RTC core;
    class Unitek,Behringer,Mic,Amp audio;
    class Screen display;
```

### RPI5 - High level schema (Plain)

```
[CAR BATTERY]
      │
      ▼
[MAIN ISOLATOR SWITCH]
      │
      ├──> [5A FUSE]
      │         │
      │         ▼
      │    [RELAY PIN 30][cite: 1, 2]
      │         │ (IGN On)
      │         ▼
      │    [RELAY PIN 87][cite: 1, 2]
      │         │
      │         ▼
      │    [DC-DC 12V->5V]
      │         │ (USB-C)
      │         ▼
      │    [GEEKWORM UPS]
      │         │
      │         ▼
      │    [RASPBERRY PI 5]
      │    ├── [NVMe HAT + SSD]
      │    ├── [CAN Bus HAT]
      │    ├── [RTC Battery]
      │    └── [USB Ports]
      │          ├── [Unitek Y-247A] ──> [Amphenol Mic (TRS)]
      │          └── [Behringer UCA202] ──> [RCA] ──> [Car Amp]
      │
      ├──> [RELAY PIN 86 (IGN)][cite: 1, 2]
      │
      └──> [RELAY PIN 85 (GND)][cite: 1, 2] ──> [CAR BODY]
```

### RPI5 - High level schema (Picture)

![rpi5 connection](../pictures/rpi-5-connection.png)
>This picture was generated by AI.