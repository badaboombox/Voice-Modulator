# Voice Modulator — Rev A.2 schematic plan

The electrical design remains Rev A.2. Draw the circuit from
[IC_CONNECTIONS.md](IC_CONNECTIONS.md), grouped by sheet and component.

**That document is the sole drawing source for passive values and wiring.**
Each passive appears once in the table beside its associated component.
IC pins and sheet ports use net names, so the same pull, series part, filter or
bypass capacitor is not specified again at a remote endpoint.

This file records architecture, operating modes, firmware settings and acceptance
requirements. Schematic capture, simulation, ERC, routing and hardware measurements
are implementation work still to be completed.

## 1. Fixed specification

| Item | Rev A decision |
| --- | --- |
| Tier 1 | Record a dry sentence, replay it through the adjustable hardware effect |
| Tier 2 | Continuous microphone processing with headphones |
| Tier 3 | Analog switching ring modulator, adjustable analog oscillator, tone and blend |
| Audio | 32 ksample/s, signed 16-bit mono recordings; stereo codec channels have separate assigned roles |
| Recording | One 10-second dry recording slot; additional flash capacity remains unused in Rev A |
| PCB | 95 x 65 mm, four layers, 1.6-mm FR-4, ordinary plated through vias |
| Stack | L1 components/signals; L2 continuous GND; L3 power/limited signals; L4 secondary signals/GND |
| Assembly | Single-sided SMT; controls and connectors hand installed |
| Controls | Frequency, Tone, Blend, Level; Record, Play/Stop, Bypass, Speaker Arm |
| Power | USB 2.0 host-powered, 500-mA configuration; no USB PD |
| Monitoring | Headphones live; internal speaker for recorded playback only |
| Budget | $185 allocation; total landed order must remain below $200 |

The screenshot's RP2040, MCP6002, speaker, connector, regulator, eFuse and crystal
are retained. Resolve its abbreviated flash label to **W25Q128JVSIQ** for U4/U13.
Use the user-specified **PAM8403DR in SOP-16** for the speaker amplifier. These are the fixed ordering
codes, not alternative packages. Preserve the screenshot labels in a CoursePart
field for traceability. D1 is the user-selected ST USBLC6-2SC6, matching the current schematic.

## 2. Sheets and drawing order

| Order | Existing sheet | Component groups |
| --- | --- | --- |
| 1 | powerUSB.kicad_sch | J1, D101, D1, U1, U2, U14, U7 |
| 2 | mcuStore.kicad_sch | U3, Y1, U4, SW1, SW2, U13, J5 |
| 3 | mic.kicad_sch | U5, MIC1, P5 |
| 4 | codec.kicad_sch | U6 |
| 5 | modulo.kicad_sch | U8, U9, U10, U11, P1, P2, P3, P6, SW5 |
| 6 | speakerCont.kicad_sch | U12, J3, J4, SW3, SW4, SW6, P4, D801, D802, D803 |

Use global labels for supplies and matching hierarchical ports for other signals.
The sheet-by-sheet connection document assigns each component to one schematic
group. Follow that assignment even when a signal runs to another sheet.

## 3. Locked IC roster

| Ref | Exact part | Function |
| --- | --- | --- |
| U1 | TPS259531DSGR | Input eFuse |
| U2 | TLV1117LV33DCYR | Main 3.3-V regulator |
| U3 | RP2040 | Audio streaming, recording, playback and UI |
| U4 | W25Q128JVSIQ | QSPI firmware flash |
| U5 | MCP6002-I/SN | Microphone preamp and 1.65-V reference buffer |
| U6 | TLV320AIC3104IRHBR | Stereo codec and headphone drivers |
| U7 | TLV70018DDCR | Codec 1.8-V core supply |
| U8 | TLV9064IPWR | Analog effect buffers and inverter |
| U9 | TS5A23157DGSR | Source select and ring-modulation switch |
| U10 | TLC555CDR | Analog carrier oscillator |
| U11 | SN74HC74DR | Carrier duty-cycle divider |
| U12 | PAM8403DR | Stereo speaker amplifier, SOP-16; left channel used |
| U13 | W25Q128JVSIQ | Dedicated recording flash |
| U14 | TLV6700DDCR | Overvoltage shutdown and codec core-enable supervision |
| D1 | USBLC6-2SC6 | USB data/VBUS ESD array, SOT23-6L |

## 4. Power behavior

USB-C supplies the input protection and main regulator. The protected bus powers
the speaker amplifier. The regulated logic rail supplies RP2040 and storage;
the filtered analog rail supplies the microphone/effect and codec analog stages.
The codec core has its separate supervised regulator. RP2040 generates its own
core rail.

The input supervisor handles excessive input voltage, and its other comparator
enables the codec core only after the analog rail reaches its threshold.
Keep codec reset asserted while supplies settle. Fault shutdown, ramp ordering,
USB startup current and suspend behavior are verified during bring-up.

The firmware requests a USB 2.0 500-mA configuration and keeps audio drivers muted
until configuration. Normal configured operating current must remain below
450 mA. Maintain the established current-limit/soft-start design; its actual
parts are specified once beside the associated devices in IC_CONNECTIONS.md.

## 5. Audio signal allocation and operating modes

| Codec channel | Fixed assignment |
| --- | --- |
| ADC left | MIC_PRE: dry microphone capture |
| ADC right | FX_RETURN: processed analog capture |
| DAC left | Stored dry recording to PLAYBACK_FEED |
| DAC right | Processed monitoring to speaker/headphones |

| Mode | SOURCE_PLAY | DAC left | DAC right | Storage |
| --- | ---: | --- | --- | --- |
| Live | 0 | Digital silence | ADC-right samples | Idle |
| Record | 0 | Digital silence | ADC-right samples | Write ADC-left samples |
| Play | 1 | Stored dry samples | ADC-right samples | Read stored recording |
| Stop | 0 | Digital silence | Return to live monitoring | Idle |

Record starts a new 10-second dry recording. Play toggles playback/stop. Playback
runs once, then returns to live mode. There is no processed-recording mode in
Rev A. Pre-erase the recording slot before declaring Record Ready.

Never route ADC right into DAC left. All unintended codec analog bypass/mixing
paths are disabled. DAC left feeds only the analog playback source. DAC right
feeds only the monitor outputs. The Level knob changes monitor volume, leaving
the playback-feed gain fixed.

## 6. Analog effect behavior

The microphone preamp and buffered midpoint reference feed the effect source
selector. The selector chooses live microphone or stored playback. A buffered
normal path and a polarity-inverted path feed the switching ring modulator.
An analog oscillator and divide-by-two stage supply its carrier. The tone stage
filters the modulated signal; Blend crossfades dry and wet; Bypass selects the
normal path.

| Control | Function |
| --- | --- |
| Frequency | Analog carrier, approximately 73 Hz–1.68 kHz |
| Tone | Low-pass corner, approximately 717 Hz–7.23 kHz |
| Blend | Dry through fully modulated audio |
| Level | Smoothed monitor volume |
| Gain trim | Microphone setup gain, initially minimum |
| Carrier-null trim | Minimize residual carrier with zero audio |

Place the controls and their component-level networks using the connection
document. It carries their terminal wiring and fitted values once, beside the
appropriate device. Button debounce is performed in firmware.

## 7. Codec clocking, supply and output setup

| Item | Fixed setting |
| --- | --- |
| MCU audio interface | Full-duplex PIO slave, DMA ping-pong buffers, 64 frames per buffer |
| MCLK | GP21 CLK_GPOUT0 sourced from 12-MHz XOSC |
| Codec PLL | P=1, R=1, J=8, D=1920; 48-kHz reference |
| ADC/DAC sample-rate divisors | 1.5, producing 32 ksample/s |
| Serial format | I2S, 16-bit samples, 32 BCLKs per stereo frame |
| Codec clock role | BCLK and WCLK master |
| Expected clocks | WCLK 32 kHz; BCLK 1.024 MHz |
| ADC mode | Single-ended; preamp/PGA gain initially 0 dB; AGC off |
| DAC-left level | Fixed 0-dB digital gain for dry playback feed |
| DAC-right level | Level knob maps mute to −6 dB maximum, with smoothed ramps |
| Speaker output routes | DAC right only to RIGHT_LOP single-ended feed, output gain 0 dB; RIGHT_LOM left unconnected |
| Headphone routes | DAC right only to HPLOUT and HPROUT; additional −12-dB route attenuation |
| Unused inputs | AC-ground through their individually listed 100-nF capacitors; routes disabled |
| Unused outputs | NC; corresponding driver/function disabled |

Codec supply ordering is I/O/analog before the 1.8-V core. U14 supervises the analog
rail before enabling U7. Keep RESET low until rails are stable, then start codec
configuration, clocking and DMA while outputs remain muted. Wait 500 ms for analog
settling before the first volume ramp. On source/bypass changes, ramp monitor down,
switch, allow settling, then ramp back up.

Recording format is 64,000 byte/s; a 10-second slot uses 640,000 bytes plus metadata.
U13 is standard SPI with Quad Enable clear. Use a 16-KiB recording queue and
256-byte page writes. Do not erase sectors while recording. Store valid length
and CRC only after a completed recording; reject an incomplete record after reset.
Do not erase/program U4 during streaming.

## 8. Outputs and monitoring

U12 is the user-specified PAM8403DR already instantiated in KiCad as PAM8403D.
Its left input and bridge output drive the prescribed Adafruit speaker through
the specified Molex connector. Its unused right input is AC-grounded and right
outputs are unconnected. The input attenuator, supply/reference bypass and
mute/shutdown network are all specified once beside U12.

Use headphones for live/record modes. The internal speaker is enabled only for
recorded playback when physically armed and USB-configured. The arm switch
interrupts AMP_RUN as well as reporting its state to firmware.

Headphone outputs remain independently AC-coupled to the jack and carry the same
mono monitor through the codec's two drivers. Their coupling, series, discharge
and ESD parts are specified at the jack group. Neither speaker bridge output
is a headphone output or GND.

## 9. Layout and acceptance work

Place the microphone front end opposite USB and the speaker amplifier. Keep
oscillator timing, I2S clocks, flash clocks and class-D returns away from MIC1/U5.
Use four mounting points, panel-supported controls and restrained white labels on
a dark enclosure/faceplate. Mechanically fit the selected parts before routing.
The schematic freeze does not assert that enclosure fit or routing is completed.

Include test points for rails, VREF1V65, MIC_PRE, normal/inverted paths, CARRIER_50,
RING_RAW, TONE_NODE, FX_RETURN, MCLK and I2S clocks. Do not enlarge crystal-node
copper with oversized test pads. Use SWD and physical BOOTSEL/reset access.

Acceptance work remains:

1. Capture the fixed schematic and check all symbol/pad numbers; run ERC.
2. Simulate and prototype gain/headroom, carrier null, tone response and bypass.
3. Prove codec-master PIO audio, sample order, dry record and replay through effect.
4. Measure startup/suspend/configured current, rail sequencing, thermal rise and
   fault shutdown. Verify the core-supply delay against the codec datasheet.
5. Verify flash writes against maximum program time, SPI overhead and power-loss
   behavior. Inspect recordings for dropped frames.
6. Measure headphone pop/DC behavior, live latency (requirement: under 10 ms),
   and speaker output. Require at most 2.0 Vrms across an 8-ohm load in a
   full-scale sine test; check worst-case nonsinusoidal power below 1 W.
7. Complete layout/DRC, mechanical checks and the full landed manufacturing quote.

The $185 allocation remains $55 components, $50 fabrication/assembly service,
$25 mechanics, $25 shipping/tax and $30 contingency. These are spending allocations,
not supplier prices. Cost and measured performance are acceptance results still
to be established, rather than alternative design choices.

## 10. Drawing-source policy and part decisions

[IC_CONNECTIONS.md](IC_CONNECTIONS.md) is authoritative for pin assignments and
all component-level wiring/values. Each passive is specified under exactly one
associated component; net names identify its connections elsewhere. No hardware
debounce network is added by this formatting update.

The selected codec remains TLV320AIC3104IRHBR, the USB protector is USBLC6-2SC6,
and the speaker amplifier is PAM8403DR. The codec supports the existing independent
playback feed and processed monitor/headphone routing. AIC3204 remains a possible
future redesign rather than a required quality upgrade.

All manufacturer sources remain linked at the end of IC_CONNECTIONS.md.
