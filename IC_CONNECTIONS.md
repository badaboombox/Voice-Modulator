# Voice Modulator — Rev A.2 connections by sheet

The electrical design remains Rev A.2. Draw the six sheets in the order below.

**Passive instructions appear exactly once, beside their associated component.**
IC pin tables list nets/direct wires only. Pull-ups, pull-downs, series resistors,
coupling, filtering and supply bypass parts are specified only in the local
passive table under the component they belong beside. At the other end of a
signal, connect the named net; do not add another passive.

The passive table gives its full wiring, so the pin table does not repeat it.
All tables within one sheet are completed before moving to the next sheet.
Association here identifies the schematic group; retain explicit driver-side
placement notes when laying out clock/data series parts.

Use global labels for supply rails. Use matching hierarchical ports for the
other inter-sheet signals. Names ending in _DRV mark the driver-side node of an
existing series part; coupled/filtered nodes identify its other connections.
These names clarify the same circuit rather than adding components.

The first column of an IC pin table is the physical pin number. NC means leave
the terminal unconnected, never ground. Instantiate all op-amp/logic units,
including power units. Button debounce is implemented in firmware.

## Drawing order

| Sheet | Draw these component groups |
| --- | --- |
| powerUSB.kicad_sch | J1, D101, D1, U1, U2, U14, U7 |
| mcuStore.kicad_sch | U3, Y1, U4, SW1, SW2, U13, J5 |
| mic.kicad_sch | U5, MIC1, P5 |
| codec.kicad_sch | U6 |
| modulo.kicad_sch | U8, U9, U10, U11, P1, P2, P3, P6, SW5 |
| speakerCont.kicad_sch | U12, J3, J4, SW3, SW4, SW6, P4, D801, D802, D803 |

## 1. powerUSB.kicad_sch

**Power:** Generates +5V_PROT, +3V3_D, +3V3_A and +1V8_CODEC; common GND.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| USB_DP_ESD / USB_DM_ESD | Output | mcuStore USB data inputs |
| EFUSE_nFAULT | Output | mcuStore fault input |

### J1 — USB-C connector

GCT USB4105-GF-A. VBUS contacts to VBUS_RAW; grounds and shield to GND. Join A6/B6 as USB_DP and A7/B7 as USB_DM. A5/B5 are CC1/CC2; SBU contacts are NC.

**Passives to draw beside J1 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R101 | 5.1 kΩ / 1% | CC1 → GND |
| R102 | 5.1 kΩ / 1% | CC2 → GND |
| C101 | 1 µF / 25 V | VBUS_RAW → GND |
| C102 | 100 nF / 25 V | VBUS_RAW → GND |

### D101 — VBUS transient protection

SMF6.0A. Cathode to VBUS_RAW, anode to GND.

### D1 — USBLC6-2SC6

**Package:** SOT23-6L. **Function:** USB data/VBUS ESD protection.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | I/O1 | USB_DP, from J1 D+ contacts |
| 2 | GND | GND |
| 3 | I/O2 | USB_DM, from J1 D− contacts |
| 4 | I/O2 | USB_DM_ESD, MCU-sheet port |
| 5 | VBUS | VBUS_RAW |
| 6 | I/O1 | USB_DP_ESD, MCU-sheet port |

Both pins of each data channel are used for flow-through routing. VBUS is the connector-side USB supply; D+ and D− remain separate.

**Passives to draw beside D1 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| C113 | 100 nF / 25 V | VBUS_RAW / pin 5 → GND |

### U1 — TPS259531DSGR

**Package:** DSG WSON-8, 2 x 2 mm, pad 9. **Function:** USB eFuse.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | dVdt | EFUSE_DVDT |
| 2 | EN/UVLO | EFUSE_EN |
| 3 | IN | VBUS_RAW |
| 4 | IN | VBUS_RAW |
| 5 | OUT | +5V_PROT |
| 6 | nFLT | EFUSE_nFAULT |
| 7 | ILM | EFUSE_ILM |
| 8 | GND | GND |
| 9 | Exposed pad | GND; solder exposed pad |

Nominal current limit is approximately 0.492 A and nominal 5-V soft start is about 56 ms. Normal configured operating current remains below 450 mA. USB power negotiation is a separate firmware requirement.

**Passives to draw beside U1 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R103 | 237 kΩ | VBUS_RAW → EFUSE_EN |
| R104 | 100 kΩ | EFUSE_EN → GND |
| R105 | 4.42 kΩ / 1% | EFUSE_ILM / pin 7 → GND |
| R106 | 10 kΩ | EFUSE_nFAULT / pin 6 → +3V3_D |
| C103 | 470 nF | EFUSE_DVDT / pin 1 → GND |
| C104 | 47 µF / 10 V | +5V_PROT / pin 5 → GND |
| C105 | 100 nF | +5V_PROT / pin 5 → GND |

### U2 — TLV1117LV33DCYR

**Package:** DCY SOT-223. **Function:** Main 3.3-V regulator.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | GND | GND |
| 2 | OUT | +3V3_D |
| 3 | IN | +5V_PROT |
| Tab | OUT | +3V3_D; same output as pin 2 |

**Passives to draw beside U2 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| C106 | 10 µF / 10 V | +5V_PROT / IN → GND |
| C107 | 22 µF / 10 V | +3V3_D / OUT → GND |
| FB101 | BLM18AG601SN1D | +3V3_D → +3V3_A |
| C108 | 22 µF / 10 V | +3V3_A → GND, analog side of the rail filter |
| C109 | 100 nF | +3V3_A → GND, analog side of the rail filter |

### U14 — TLV6700DDCR

**Package:** DDC thin SOT-23-6. **Function:** Overvoltage shutdown and codec core sequencing.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | OUTA | CORE_ENABLE, to U7 EN |
| 2 | GND | GND |
| 3 | INA+ | CORE_SENSE |
| 4 | INB− | OV_SENSE |
| 5 | VDD | +5V_PROT |
| 6 | OUTB | EFUSE_EN, shared with U1 EN/UVLO |

Supervises input overvoltage and analog-rail readiness. Core enable releases near 2.876 V; input shutdown trips near 5.32 V. Validate startup/shutdown sequencing during bring-up.

**Passives to draw beside U14 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R107 | 1.23 MΩ / 0.1% | VBUS_RAW → OV_SENSE |
| R108 | 100 kΩ / 0.1% | OV_SENSE → GND |
| R109 | 619 kΩ / 0.1% | +3V3_A → CORE_SENSE |
| R110 | 100 kΩ / 0.1% | CORE_SENSE → GND |
| R111 | 10 kΩ | CORE_ENABLE → +3V3_A |
| C112 | 100 nF | +5V_PROT / VDD → GND |

### U7 — TLV70018DDCR

**Package:** DDC SOT-23-5. **Function:** Codec 1.8-V regulator.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | IN | +3V3_A |
| 2 | GND | GND |
| 3 | EN | CORE_ENABLE |
| 4 | NC | NC |
| 5 | OUT | +1V8_CODEC |

**Passives to draw beside U7 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R112 | 10 kΩ | +1V8_CODEC → GND |
| C110 | 1 µF | +3V3_A / IN → GND |
| C111 | 1 µF | +1V8_CODEC / OUT → GND |

## 2. mcuStore.kicad_sch

**Power:** Receives +3V3_D/GND; generates the local +1V1_CORE rail for RP2040 only.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| USB_DP_ESD / USB_DM_ESD | Input | powerUSB data outputs |
| EFUSE_nFAULT | Input | powerUSB |
| CODEC_SDA / CODEC_SCL | Bidirectional | codec |
| CODEC_MCLK / I2S_TO_CODEC | Output | codec |
| I2S_BCLK / I2S_WCLK / I2S_FROM_CODEC | Input | codec |
| CODEC_nRESET | Output | codec |
| SOURCE_PLAY | Output | modulo |
| BYPASS_SENSE | Input | modulo |
| REC_BUTTON / PLAY_BUTTON / SPK_ARM_SENSE / LEVEL_ADC | Input | speakerCont |
| SPK_ENABLE / CLIP_LED / REC_LED / STATUS_LED | Output | speakerCont |

### U3 — RP2040

**Package:** QFN-56 + exposed pad 57, 7 x 7 mm. **Function:** Controller.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | IOVDD | +3V3_D |
| 2 | GPIO0 | CODEC_SDA |
| 3 | GPIO1 | CODEC_SCL |
| 4 | GPIO2 | I2S_BCLK |
| 5 | GPIO3 | I2S_WCLK |
| 6 | GPIO4 | I2S_TX_DRV |
| 7 | GPIO5 | I2S_FROM_CODEC |
| 8 | GPIO6 | CODEC_nRESET |
| 9 | GPIO7 | SOURCE_PLAY |
| 10 | IOVDD | +3V3_D |
| 11 | GPIO8 | REC_FLASH_MISO |
| 12 | GPIO9 | REC_FLASH_nCS |
| 13 | GPIO10 | REC_CLK_DRV |
| 14 | GPIO11 | REC_FLASH_MOSI |
| 15 | GPIO12 | REC_BUTTON |
| 16 | GPIO13 | PLAY_BUTTON |
| 17 | GPIO14 | BYPASS_SENSE |
| 18 | GPIO15 | SPK_ENABLE |
| 19 | TESTEN | GND |
| 20 | XIN | XTAL_IN |
| 21 | XOUT | XTAL_DRV |
| 22 | IOVDD | +3V3_D |
| 23 | DVDD | +1V1_CORE |
| 24 | SWCLK | SWCLK, to J5 |
| 25 | SWDIO | SWDIO, to J5 |
| 26 | RUN | RUN, to reset switch and J5 |
| 27 | GPIO16 | SPK_ARM_SENSE |
| 28 | GPIO17 | NC; firmware input with pull-down |
| 29 | GPIO18 | CLIP_LED |
| 30 | GPIO19 | REC_LED |
| 31 | GPIO20 | STATUS_LED |
| 32 | GPIO21 | MCLK_DRV |
| 33 | IOVDD | +3V3_D |
| 34 | GPIO22 | EFUSE_nFAULT |
| 35 | GPIO23 | NC; firmware input with pull-down |
| 36 | GPIO24 | NC; firmware input with pull-down |
| 37 | GPIO25 | NC; firmware input with pull-down |
| 38 | GPIO26 / ADC0 | LEVEL_ADC |
| 39 | GPIO27 / ADC1 | NC; firmware input with pull-down |
| 40 | GPIO28 / ADC2 | NC; firmware input with pull-down |
| 41 | GPIO29 / ADC3 | NC; firmware input with pull-down |
| 42 | IOVDD | +3V3_D |
| 43 | ADC_AVDD | +3V3_D |
| 44 | VREG_VIN | +3V3_D |
| 45 | VREG_VOUT | +1V1_CORE |
| 46 | USB_DM | USB_DM |
| 47 | USB_DP | USB_DP |
| 48 | USB_VDD | +3V3_D |
| 49 | IOVDD | +3V3_D |
| 50 | DVDD | +1V1_CORE |
| 51 | QSPI_SD3 | QSPI_SD3 |
| 52 | QSPI_SCLK | BOOT_CLK_DRV |
| 53 | QSPI_SD0 | QSPI_SD0 |
| 54 | QSPI_SD2 | QSPI_SD2 |
| 55 | QSPI_SD1 | QSPI_SD1 |
| 56 | QSPI_nSS | QSPI_nSS |
| 57 | GND / exposed pad | GND; solder exposed pad |

All GPIOs marked unused are configured as inputs with firmware pull-downs. Local names ending in _DRV identify the driver side of an existing series element; the normal sheet-port name is on its far side.

**Passives to draw beside U3 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R201 | 27 Ω | USB_DP_ESD → USB_DP / pin 47; source-required placement near RP2040 |
| R202 | 27 Ω | USB_DM_ESD → USB_DM / pin 46; near RP2040 |
| R203 | 10 kΩ | RUN → +3V3_D |
| R206 | 1 kΩ | XTAL_DRV / XOUT → XTAL_OUT / crystal signal terminal |
| R210 | 33 Ω | REC_CLK_DRV / GP10 → REC_FLASH_SCK; source termination near RP2040 |
| R211 | 33 Ω | BOOT_CLK_DRV / QSPI_SCLK → BOOT_SCLK; source termination near RP2040 |
| R212 | 100 kΩ | SOURCE_PLAY → GND |
| R213 | 100 kΩ | CODEC_nRESET → GND |
| R603 | 33 Ω | MCLK_DRV / GP21 → CODEC_MCLK; source termination near RP2040 |
| R606 | 33 Ω | I2S_TX_DRV / GP4 → I2S_TO_CODEC; source termination near RP2040 |
| C201 | 100 nF | +3V3_D / U3 pin 1 → GND |
| C202 | 100 nF | +3V3_D / U3 pin 10 → GND |
| C203 | 100 nF | +3V3_D / U3 pin 22 → GND |
| C204 | 100 nF | +3V3_D / U3 pin 33 → GND |
| C205 | 100 nF | +3V3_D / U3 pin 42 → GND |
| C206 | 100 nF | +3V3_D / U3 pin 49 → GND |
| C207 | 100 nF | +3V3_D / U3 pin 48 → GND |
| C208 | 100 nF | +1V1_CORE / U3 pin 23 → GND |
| C209 | 100 nF | +1V1_CORE / U3 pin 50 → GND |
| C210 | 100 nF | +3V3_D / ADC_AVDD → GND |
| C211 | 1 µF | +3V3_D / VREG_VIN → GND |
| C212 | 1 µF | +1V1_CORE / VREG_VOUT → GND |
| C213 | 10 µF | +3V3_D → GND |

### Y1 — crystal

ABM8-272-T3, 12 MHz, CL = 10 pF. Pin 1 XTAL_IN; pin 3 XTAL_OUT; pins 2/4 GND. CL is the effective crystal load including parasitics, not the value of each external capacitor.

**Passives to draw beside Y1 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| C214 | 15 pF / C0G | XTAL_IN / Y1 pin 1 → GND |
| C215 | 15 pF / C0G | XTAL_OUT / Y1 pin 3 → GND |

### U4 — W25Q128JVSIQ

**Package:** 208-mil SOP-8, 5.3-mm body. **Function:** QSPI boot flash.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | nCS | QSPI_nSS |
| 2 | DO / IO1 | QSPI_SD1 |
| 3 | nWP / IO2 | QSPI_SD2 |
| 4 | GND | GND |
| 5 | DI / IO0 | QSPI_SD0 |
| 6 | CLK | BOOT_SCLK |
| 7 | nHOLD / IO3 | QSPI_SD3 |
| 8 | VCC | +3V3_D |

QSPI boot device. Its additional data pins remain data connections; the BOOTSEL switch uses the dedicated BOOT_BUTTON node.

**Passives to draw beside U4 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R204 | 10 kΩ | QSPI_nSS → +3V3_D |
| R205 | 1 kΩ | QSPI_nSS → BOOT_BUTTON; switch closes BOOT_BUTTON to GND |
| C216 | 100 nF | +3V3_D / VCC → GND |

### SW1 — BOOTSEL

Momentary normally-open switch: BOOT_BUTTON to GND.

### SW2 — RESET

Momentary normally-open switch: RUN to GND.

### U13 — W25Q128JVSIQ

**Package:** 208-mil SOP-8, 5.3-mm body. **Function:** Dedicated SPI recording flash.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | nCS | REC_FLASH_nCS |
| 2 | DO / IO1 | REC_FLASH_MISO |
| 3 | nWP / IO2 | REC_nWP |
| 4 | GND | GND |
| 5 | DI / IO0 | REC_FLASH_MOSI |
| 6 | CLK | REC_FLASH_SCK |
| 7 | nHOLD / IO3 | REC_nHOLD |
| 8 | VCC | +3V3_D |

Recording device uses ordinary SPI with Quad Enable clear. It is separate from the firmware XIP flash.

**Passives to draw beside U13 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R207 | 10 kΩ | REC_FLASH_nCS → +3V3_D |
| R208 | 10 kΩ | REC_nWP / pin 3 → +3V3_D |
| R209 | 10 kΩ | REC_nHOLD / pin 7 → +3V3_D |
| C217 | 100 nF | +3V3_D / VCC → GND |
| C218 | 1 µF | +3V3_D / VCC → GND |

### J5 — SWD connector

Cortex 2×5: pin 1 +3V3_D sense, 2 SWDIO, 3/5/9 GND, 4 SWCLK, 10 RUN; 6/7/8 NC. This is not a board-power input.

## 3. mic.kicad_sch

**Power:** Receives +3V3_A/GND; generates VREF1V65 as a buffered analog signal.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| MIC_PRE | Output | codec and modulo, as separate branches |
| VREF1V65 | Output | modulo analog reference |

### U5 — MCP6002-I/SN

**Package:** SOIC-8, 3.9-mm body. **Function:** Microphone preamp and midpoint buffer.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | OUT A | MIC_PRE |
| 2 | IN− A | MIC_FB |
| 3 | IN+ A | MIC_INPUT |
| 4 | VSS | GND |
| 5 | IN+ B | VREF_RAW |
| 6 | IN− B | VREF1V65; directly connected to pin 7 |
| 7 | OUT B | VREF1V65 |
| 8 | VDD | +3V3_A |

The reference amplifier is a follower. Do not add a large capacitor directly on its output. Initial microphone gain is set to minimum.

**Passives to draw beside U5 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R301 | 10 kΩ | +3V3_A → VREF_RAW |
| R302 | 10 kΩ | VREF_RAW → GND |
| R305 | 22 kΩ | MIC_INPUT / IN+ A → VREF1V65 |
| R306 | 1 kΩ | MIC_FB / IN− A → VREF1V65 |
| R307 | 22 kΩ | MIC_PRE / OUT A → GAIN_TRIM_IN |
| C301 | 100 nF | +3V3_A / VDD → GND |
| C302 | 1 µF | +3V3_A / VDD → GND |
| C303 | 10 µF | VREF_RAW → GND |
| C304 | 100 nF | VREF_RAW → GND |
| C307 | 100 nF | MIC_POS → MIC_INPUT / IN+ A |
| C308 | 22 pF / C0G | MIC_PRE / OUT A → MIC_FB / IN− A, across the complete feedback path |

### MIC1 — electret microphone

Adafruit 1064. Positive MIC_POS; negative/case GND.

**Passives to draw beside MIC1 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R303 | 1 kΩ | +3V3_A → MIC_BIAS_SUPPLY |
| R304 | 2.2 kΩ | MIC_BIAS_SUPPLY → MIC_POS |
| C305 | 10 µF | MIC_BIAS_SUPPLY → GND |
| C306 | 100 nF | MIC_BIAS_SUPPLY → GND |

### P5 — microphone gain trimmer

50 kΩ. Terminal 1 GAIN_TRIM_IN; tie terminals 2/3 to MIC_FB. Start at minimum gain.

## 4. codec.kicad_sch

**Power:** Receives +3V3_D, +3V3_A, +1V8_CODEC and GND; no VREF1V65 connection.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| CODEC_SDA / CODEC_SCL | Bidirectional | mcuStore |
| CODEC_MCLK / CODEC_nRESET / I2S_TO_CODEC | Input | mcuStore |
| I2S_BCLK / I2S_WCLK / I2S_FROM_CODEC | Output | mcuStore |
| MIC_PRE | Input | mic |
| FX_RETURN | Input | modulo |
| PLAYBACK_FEED | Output | modulo |
| SPK_LINE_P / HP_L_RAW / HP_R_RAW | Output | speakerCont |

### U6 — TLV320AIC3104IRHBR

**Package:** RHB0032E VQFN-32, 5 x 5 mm, exposed pad 33. **Function:** Stereo ADC/DAC and headphone driver.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | MCLK | CODEC_MCLK |
| 2 | BCLK | CODEC_BCLK_DRV |
| 3 | WCLK | CODEC_WCLK_DRV |
| 4 | DIN | I2S_TO_CODEC |
| 5 | DOUT | CODEC_DOUT_DRV |
| 6 | DVSS | GND |
| 7 | IOVDD | +3V3_D |
| 8 | SCL | CODEC_SCL |
| 9 | SDA | CODEC_SDA |
| 10 | MIC1LP / LINE1LP | ADC_L_IN |
| 11 | MIC1LM / LINE1LM | Unused analog input; AC-ground connection listed below |
| 12 | MIC1RP / LINE1RP | ADC_R_IN |
| 13 | MIC1RM / LINE1RM | Unused analog input; AC-ground connection listed below |
| 14 | MIC2L / LINE2L / MICDET | Unused analog input; routing/detection disabled |
| 15 | MICBIAS | NC; microphone-bias output disabled |
| 16 | MIC2R / LINE2R | Unused analog input; routing disabled |
| 17 | AVSS1 | GND |
| 18 | DRVDD | +3V3_A |
| 19 | HPLOUT | HP_L_RAW, output-sheet port |
| 20 | HPLCOM | NC; common-output driver disabled |
| 21 | DRVSS | GND |
| 22 | HPRCOM | NC; common-output driver disabled |
| 23 | HPROUT | HP_R_RAW, output-sheet port |
| 24 | DRVDD | +3V3_A |
| 25 | AVDD | +3V3_A |
| 26 | AVSS2 | GND |
| 27 | LEFT_LOP | PLAYBACK_FEED, effect-sheet port |
| 28 | LEFT_LOM | NC |
| 29 | RIGHT_LOP | SPK_LINE_P, output-sheet port |
| 30 | RIGHT_LOM | NC |
| 31 | nRESET | CODEC_nRESET |
| 32 | DVDD | +1V8_CODEC |
| 33 | Exposed pad | GND; solder exposed pad |

ADC left records dry microphone; ADC right captures effect return. DAC left feeds playback into the effect; DAC right feeds speaker/headphone monitoring. Disable unintended bypass routes and unused output functions.

**Passives to draw beside U6 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R601 | 2.2 kΩ | CODEC_SCL → +3V3_D |
| R602 | 2.2 kΩ | CODEC_SDA → +3V3_D |
| R604 | 33 Ω | CODEC_BCLK_DRV / pin 2 → I2S_BCLK; source termination near codec |
| R605 | 33 Ω | CODEC_WCLK_DRV / pin 3 → I2S_WCLK; source termination near codec |
| R607 | 33 Ω | CODEC_DOUT_DRV / pin 5 → I2S_FROM_CODEC; source termination near codec |
| R608 | 220 Ω | ADC_L_COUPLED → ADC_L_IN / pin 10 |
| R609 | 220 Ω | ADC_R_COUPLED → ADC_R_IN / pin 12 |
| C601 | 100 nF | pin 7 / +3V3_D → GND |
| C602 | 100 nF | pin 32 / +1V8_CODEC → GND |
| C603 | 100 nF | pin 25 / +3V3_A → GND |
| C604 | 100 nF | pin 18 / +3V3_A → GND |
| C605 | 100 nF | pin 24 / +3V3_A → GND |
| C606 | 10 µF | +3V3_A → GND, output-driver supply bulk |
| C607 | 1 µF | +1V8_CODEC → GND |
| C608 | 1 µF / nonpolar | MIC_PRE → ADC_L_COUPLED |
| C609 | 1 µF / nonpolar | FX_RETURN → ADC_R_COUPLED |
| C610 | 1 nF | ADC_L_IN / pin 10 → GND |
| C611 | 1 nF | ADC_R_IN / pin 12 → GND |
| C612 | 100 nF | U6 unused input pin 11 → GND |
| C613 | 100 nF | U6 unused input pin 13 → GND |
| C614 | 100 nF | U6 unused input pin 14 → GND |
| C615 | 100 nF | U6 unused input pin 16 → GND |

## 5. modulo.kicad_sch

**Power:** Receives +3V3_A, +3V3_D, GND and the VREF1V65 signal from mic.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| MIC_PRE / VREF1V65 | Input | mic |
| PLAYBACK_FEED | Input | codec |
| SOURCE_PLAY | Input | mcuStore |
| FX_RETURN | Output | codec |
| BYPASS_SENSE | Output | mcuStore |

### U8 — TLV9064IPWR

**Package:** PW TSSOP-14, 4.4 x 5 mm. **Function:** Effect buffers and inverter.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | OUT A | AUDIO_NORMAL |
| 2 | IN− A | AUDIO_NORMAL; directly connected to pin 1 |
| 3 | IN+ A | EFFECT_SOURCE |
| 4 | V+ | +3V3_A |
| 5 | IN+ B | INV_REF |
| 6 | IN− B | INV_SUM |
| 7 | OUT B | AUDIO_INVERTED |
| 8 | OUT C | WET_BUFFERED |
| 9 | IN− C | WET_BUFFERED; directly connected to pin 8 |
| 10 | IN+ C | TONE_NODE |
| 11 | V− | GND |
| 12 | IN+ D | BLEND_WIPER |
| 13 | IN− D | BLEND_OUT; directly connected to pin 14 |
| 14 | OUT D | BLEND_OUT |

Follower feedback is shown by equal net names on output/inverting-input pairs. The inverter summing node and its reference node are separate.

**Passives to draw beside U8 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R403 | 10 kΩ / 0.1% | AUDIO_NORMAL → INV_SUM / IN− B |
| R404 | 10 kΩ / 0.1% | AUDIO_INVERTED / OUT B → INV_SUM / IN− B |
| R405 | 1 MΩ | NULL_WIPER → INV_REF / IN+ B |
| R406 | 10 kΩ | INV_REF → VREF1V65 |
| R407 | 2.2 kΩ | RING_RAW → TONE_POT_HI |
| C403 | 100 nF | INV_REF → GND |
| C404 | 10 nF / stable 5% | TONE_NODE / IN+ C → GND |
| C405 | 100 nF | +3V3_A / V+ → GND |
| C406 | 1 µF | +3V3_A / V+ → GND |

### U9 — TS5A23157DGSR

**Package:** DGS VSSOP-10, 3 x 3 mm, 0.5-mm pitch. **Function:** Source selection and analog ring switch.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | IN1 | SOURCE_PLAY |
| 2 | NO1 | PLAY_AC |
| 3 | GND | GND |
| 4 | NO2 | AUDIO_INVERTED |
| 5 | IN2 | CARRIER_50 |
| 6 | COM2 | RING_RAW |
| 7 | NC2 | AUDIO_NORMAL |
| 8 | V+ | +3V3_A |
| 9 | NC1 | MIC_AC |
| 10 | COM1 | EFFECT_SOURCE |

NC1/NC2 mean normally closed, not an unused-pin instruction. LOW selects NC; HIGH selects NO.

**Passives to draw beside U9 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R401 | 100 kΩ | MIC_AC → VREF1V65 |
| R402 | 100 kΩ | PLAY_AC → VREF1V65 |
| C401 | 1 µF / nonpolar | MIC_PRE → MIC_AC |
| C402 | 1 µF / nonpolar | PLAYBACK_FEED → PLAY_AC |
| C407 | 100 nF | +3V3_A / V+ → GND |

### U10 — TLC555CDR

**Package:** SOIC-8, 3.9-mm body. **Function:** Analog oscillator.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | GND | GND |
| 2 | TRIG | TIMER_C; directly connected to pin 6 |
| 3 | OUT | OSC_RAW |
| 4 | nRESET | +3V3_D |
| 5 | CONT | TIMER_CONT |
| 6 | THRES | TIMER_C |
| 7 | DISCH | TIMER_DISCH |
| 8 | VDD | +3V3_D |

**Passives to draw beside U10 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R501 | 4.7 kΩ | +3V3_D → TIMER_DISCH / pin 7 |
| R502 | 2.2 kΩ | TIMER_DISCH → TIMER_RB |
| C501 | 100 nF | +3V3_D / VDD → GND |
| C502 | 10 nF | CONT / pin 5 → GND |
| C503 | 47 nF / stable 5% | TIMER_C / pins 2/6 → GND |

### U11 — SN74HC74DR

**Package:** SOIC-14, 3.9-mm body. **Function:** Carrier divide-by-two.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | nCLR1 | +3V3_D |
| 2 | D1 | DIV_FEEDBACK; directly connected to pin 6 |
| 3 | CLK1 | OSC_RAW |
| 4 | nPRE1 | +3V3_D |
| 5 | Q1 | CARRIER_DRV |
| 6 | nQ1 | DIV_FEEDBACK |
| 7 | GND | GND |
| 8 | nQ2 | NC |
| 9 | Q2 | NC |
| 10 | nPRE2 | +3V3_D |
| 11 | CLK2 | GND |
| 12 | D2 | GND |
| 13 | nCLR2 | +3V3_D |
| 14 | VCC | +3V3_D |

Both flip-flops and the power unit must be instantiated. The unused half's inputs are strapped; its outputs are NC.

**Passives to draw beside U11 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R503 | 100 Ω | CARRIER_DRV / Q1 → CARRIER_50; near divider output |
| C504 | 100 nF | +3V3_D / VCC → GND |

### P1 — Frequency

100 kΩ linear rheostat. Terminal 3 TIMER_RB; tie terminals 1/2 to TIMER_C. Clockwise rotation reduces timing resistance and raises frequency.

### P2 — Tone

20 kΩ linear rheostat. Terminal 3 TONE_POT_HI; tie terminals 1/2 to TONE_NODE. Clockwise raises cutoff.

### P3 — Blend

10 kΩ linear. Terminal 1 AUDIO_NORMAL, terminal 3 WET_BUFFERED, terminal 2 BLEND_WIPER.

### P6 — carrier-null trimmer

10 kΩ. Terminal 1 GND, terminal 3 +3V3_A, wiper NULL_WIPER. Adjust with zero audio for minimum carrier leakage.

### SW5 — Bypass

DPDT. Audio common FX_RETURN, throws AUDIO_NORMAL/BLEND_OUT. Sense common BYPASS_SENSE, bypass throw GND, effect throw NC.

**Passives to draw beside SW5 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R803 | 10 kΩ | BYPASS_SENSE → +3V3_D |

## 6. speakerCont.kicad_sch

**Power:** Receives +5V_PROT, +3V3_D and GND. AMP_VREF is separate from VREF1V65.

### Sheet ports

| Net / port | Direction | Other sheet |
| --- | --- | --- |
| SPK_LINE_P / HP_L_RAW / HP_R_RAW | Input | codec |
| SPK_ENABLE / CLIP_LED / REC_LED / STATUS_LED | Input | mcuStore |
| REC_BUTTON / PLAY_BUTTON / SPK_ARM_SENSE / LEVEL_ADC | Output | mcuStore |

### U12 — PAM8403DR

**Package:** SOP-16, 3.9 x 9.9 mm, 1.27-mm pitch; KiCad
`Amplifier_Audio:PAM8403D`, footprint
`Package_SO:SOIC-16_3.9x9.9mm_P1.27mm`.
**Function:** User-specified stereo class-D amplifier; left channel drives SPK1.

| Pin | Name | Net / direct connection |
| --- | --- | --- |
| 1 | +OUT_L / LOUT+ | AMP_L_P |
| 2 | PGND | GND |
| 3 | −OUT_L / LOUT− | AMP_L_M |
| 4 | PVDD | +5V_PROT |
| 5 | nMUTE | AMP_RUN; shared with pin 12 |
| 6 | VDD | +5V_PROT |
| 7 | INL | AMP_IN_L |
| 8 | VREF | AMP_VREF; do not join to VREF1V65 |
| 9 | NC | NC |
| 10 | INR | AMP_IN_R |
| 11 | GND | GND |
| 12 | nSHDN | AMP_RUN |
| 13 | PVDD | +5V_PROT |
| 14 | −OUT_R / ROUT− | NC; unused right bridge output |
| 15 | PGND | GND |
| 16 | +OUT_R / ROUT+ | NC; unused right bridge output |

Use only the left speaker bridge. The right input is AC-grounded; the right outputs remain open. Both active-low control pins share AMP_RUN. Maintain the monitor's −6-dB maximum and verify speaker power.

**Passives to draw beside U12 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R701 | 10 kΩ | SPK_LINE_P → SPK_PAD |
| R702 | 2.2 kΩ | SPK_PAD → GND |
| R703 | 100 Ω | SPK_ENABLE → ARM_DRIVE |
| R704 | 1 kΩ | AMP_RUN / pins 5/12 → GND |
| C701 | 1 µF / nonpolar | SPK_PAD → AMP_IN_L / pin 7 |
| C702 | 1 µF / nonpolar | AMP_IN_R / pin 10 → GND |
| C703 | 100 nF | +5V_PROT / PVDD pin 4 → GND |
| C704 | 47 µF / 10 V | +5V_PROT → GND, local amplifier bulk |
| C705 | 100 nF | +5V_PROT / VDD pin 6 → GND |
| C706 | 1 µF | +5V_PROT / VDD pin 6 → GND |
| C709 | 1 µF / ceramic | AMP_VREF / pin 8 → GND; separate from VREF1V65 |
| C710 | 100 nF | +5V_PROT / PVDD pin 13 → GND |
| FB701 | BLM21PG221SN1D | AMP_L_P / pin 1 → SPK_P |
| FB702 | BLM21PG221SN1D | AMP_L_M / pin 3 → SPK_M |

### J3 / SPK1 — speaker connector and speaker

Molex 53261-0271: pin 1 SPK_P, pin 2 SPK_M. Adafruit 3923, 8 Ω / 1 W, plugs in using its factory PicoBlade harness. Neither speaker lead is GND.

### J4 — headphone jack

CUI SJ-3523-SMT-TR: tip HP_L_OUT, ring HP_R_OUT, sleeve GND. Match these contact names to the selected symbol and footprint. D701/D702 are PESD5V0S1BA,115 bidirectional ESD devices from the respective jack outputs to GND.

**Passives to draw beside J4 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R705 | 10 Ω | HP_L_COUPLED → HP_L_OUT |
| R706 | 10 Ω | HP_R_COUPLED → HP_R_OUT |
| R707 | 100 kΩ | HP_L_OUT → GND |
| R708 | 100 kΩ | HP_R_OUT → GND |
| C707 | 220 µF / 10 V | HP_L_RAW → HP_L_COUPLED; positive terminal at HP_L_RAW |
| C708 | 220 µF / 10 V | HP_R_RAW → HP_R_COUPLED; positive terminal at HP_R_RAW |

### SW3 — Record

Momentary normally-open: REC_BUTTON to GND. Debounce is implemented in firmware.

**Passives to draw beside SW3 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R801 | 10 kΩ | REC_BUTTON → +3V3_D |

### SW4 — Play/Stop

Momentary normally-open: PLAY_BUTTON to GND. Debounce is implemented in firmware.

**Passives to draw beside SW4 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R802 | 10 kΩ | PLAY_BUTTON → +3V3_D |

### SW6 — Speaker Arm

DPDT. Enable common AMP_RUN, armed throw ARM_DRIVE, disarmed throw NC. Sense common SPK_ARM_SENSE, armed throw GND, disarmed throw NC.

**Passives to draw beside SW6 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R804 | 10 kΩ | SPK_ARM_SENSE → +3V3_D |

### P4 — Level

10 kΩ linear. Terminal 1 GND, terminal 3 +3V3_D, wiper LEVEL_WIPER.

**Passives to draw beside P4 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R808 | 1 kΩ | LEVEL_WIPER → LEVEL_ADC |
| C801 | 100 nF | LEVEL_ADC → GND |

### D801 — Clip indicator

Red LED: anode CLIP_ANODE, cathode GND.

**Passives to draw beside D801 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R805 | 1 kΩ | CLIP_LED → CLIP_ANODE |

### D802 — Record indicator

Red LED: anode REC_ANODE, cathode GND.

**Passives to draw beside D802 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R806 | 1 kΩ | REC_LED → REC_ANODE |

### D803 — Status indicator

Green LED: anode STATUS_ANODE, cathode GND.

**Passives to draw beside D803 — specify them here only:**

| Reference | Fitted value | Connection |
| --- | --- | --- |
| R807 | 1 kΩ | STATUS_LED → STATUS_ANODE |

## Completion checks

- Every named passive has one entry in this document. A remote connection uses
  the listed net or sheet port, without another pull, series resistor or capacitor.
- The GPIO, codec, amplifier and protection pin counts are retained, including
  pads and unused pins.
- Source, feedback, coupled and filtered node names remain distinct. Do not put
  the same net label on both sides of a series or coupling part.
- The Bypass switch is drawn on modulo; the Speaker Arm switch is drawn on
  speakerCont. Headphone coupling and jack parts are on speakerCont.
- Connect all six top-level sheet symbols, then run ERC on the full hierarchy.

## Package and capture rules

- U4/U13 use the wide 208-mil SOP-8 package, not a narrow 150-mil SOIC.
- U12 is PAM8403DR in SOP-16, using Package_SO:SOIC-16_3.9x9.9mm_P1.27mm. Its left output pair drives the single speaker; there is no exposed pad.
- U1 uses TI DSG0008A: 2 x 2-mm body, 0.5-mm pitch, exposed pad approximately 0.9 x 1.6 mm. The exposed pad is pin 9 and goes to GND.
- U6 uses TI RHB0032E: 5 x 5-mm body, 0.5-mm pitch, exposed pad 3.45 x 3.45 mm. The exposed pad is pin 33 and goes to GND.
- U2's output tab is +3V3_D, not GND.
- Instantiate every op-amp and logic power unit in KiCad; power pins are not assumed connected by a graphic symbol.
- General resistors are 1%, except the inverter pair and overvoltage divider marked 0.1%. General bypass capacitors are X7R; 15-pF/22-pF crystal/feedback parts are C0G. Use 5% stable timing/tone capacitors. Bulk capacitor effective capacitance must meet the stated value at DC bias.
- IC order and wiring changes are revisions to this document; do not pick between alternative devices during capture.

## Manufacturer references

- [RP2040 reference design](https://datasheets.raspberrypi.com/rp2040/hardware-design-with-rp2040.pdf)
- [Winbond W25Q128JVSIQ](https://www.winbond.com/hq/product/code-storage-flash/qspi-nor/w25q-jv/?__locale=en&partNo=W25Q128JVSIQ)
- [MCP6002](https://www.microchip.com/en-us/product/MCP6002)
- [TPS2595](https://www.ti.com/lit/ds/symlink/tps2595.pdf)
- [TLV1117LV](https://www.ti.com/lit/ds/symlink/tlv1117lv.pdf)
- [TLV320AIC3104](https://www.ti.com/lit/ds/symlink/tlv320aic3104.pdf)
- [TLV700](https://www.ti.com/lit/ds/symlink/tlv700.pdf)
- [TLV9064](https://www.ti.com/lit/ds/symlink/tlv9064.pdf)
- [TS5A23157](https://www.ti.com/lit/ds/symlink/ts5a23157.pdf)
- [TLC555](https://www.ti.com/lit/ds/symlink/tlc555.pdf)
- [SN74HC74](https://www.ti.com/lit/ds/symlink/sn74hc74.pdf)
- [PAM8403](https://www.diodes.com/datasheet/download/PAM8403.pdf)
- [TLV6700](https://www.ti.com/lit/ds/symlink/tlv6700.pdf)
- [ST USBLC6-2SC6](https://www.st.com/resource/en/datasheet/usblc6-2.pdf)

