# Capacitor, pot and switch footprint selections

Use these selections with the current schematic references. The footprint names
were checked against the installed KiCad 10 library. These are footprint/part
selections, not an order or an assembled-cost quote. The 22-uF-and-larger
capacitor footprints below are assigned in the saved schematic (2026-11-22).
The PCB has a first-pass unrouted placement; see [PCB_LAYOUT.md](PCB_LAYOUT.md).
Pot and switch footprint assignments are included in the current schematic.

## Larger capacitors

22 uF and larger use through-hole packages, as requested. The 10-uF parts retain
1206 SMD footprints.

| Value | Current references | Selected construction / part | KiCad footprint |
| --- | --- | --- | --- |
| 10 uF | C6, C17, C34, C36, C77 | X7R ceramic, 16 V; Murata GRM31CR71C106KA12L | `Capacitor_SMD:C_1206_3216Metric` |
| 22 uF | C7, C27 | THT X7R ceramic, 16 V; TDK FG20X7R1C226MRT00; 5.5 x 4.0-mm maximum body plan, 5-mm lead pitch | `Capacitor_THT:C_Rect_L7.0mm_W4.5mm_P5.00mm` |
| 47 uF | C4, C66 | THT polarized aluminum electrolytic, 25 V; Panasonic EEUFR1E470; 5-mm diameter, 11-mm body height, 2-mm lead pitch | `Capacitor_THT:CP_Radial_D5.0mm_P2.00mm` |
| 220 uF | C74, C75 | THT polarized aluminum electrolytic, 10 V; Panasonic EEUFR1A221; 6.3-mm diameter, 11.2-mm body height, 2.5-mm lead pitch | `Capacitor_THT:CP_Radial_D6.3mm_P2.50mm` |

C7 must remain ceramic: TI's TLV1117LV best-practice section specifies a ceramic
output capacitor and says not to use an electrolytic output capacitor. THT does
not imply electrolytic. C7 and C27 now use nonpolar `Device:C` symbols; their
pin positions and electrical connections are unchanged. The selected TDK part
is a production FG-series dipped radial MLCC. Its maximum seated body height is
7 mm; allow additional assembly clearance for the lead forming.

The stock rectangular footprint is a conservative body envelope for the TDK
part, with 0.8-mm drills for its 0.5-mm nominal leads (0.6-mm maximum). Mount C7
close to U2 OUT/GND, with short leads and short return traces. Keep the existing
small local bypass capacitors. MLCC nominal capacitance still requires checks
for DC bias, temperature, tolerance and aging during BOM validation.

C4/C66 remain polarized, and C74/C75 retain positive terminals toward the codec.
The electrolytic footprints match the straight-lead bulk part numbers above;
formed/taped suffixes can require a different pitch. Allow enclosure clearance
above the cans. These are footprint selections, not completed layout or hardware
validation.

Manufacturer sources:

- [TDK FG20X7R1C226MRT00 dimensions and ratings](https://product.tdk.com/en/search/capacitor/ceramic/lead-mlcc/info?part_no=FG20X7R1C226MRT00)
- [TLV1117LV, section 8.3](https://www.ti.com/lit/ds/symlink/tlv1117lv.pdf)
- [Panasonic EEUFR1E470 dimensions and ratings](https://na.industrial.panasonic.com/products/capacitors/aluminum-electrolytic-capacitors/lineup/aluminum-electrolytic-capacitors-radial-lead-type/series/83367/model/83711)
- [Panasonic EEUFR1A221 dimensions and ratings](https://industrial.panasonic.com/ww/products/pt/aluminum-cap-lead/models/EEUFR1A221)
- [Murata 10-uF part listing](https://www.murata.com/-/media/webrenewal/tool/library/common-pdf/dynamic-model/component-list-d-mlcc-2608.ashx?la=en)

## Knobs and setup trimmers

| Reference | Function / value | Part selection | KiCad footprint |
| --- | --- | --- | --- |
| R53 | Frequency, 100 kΩ linear | Bourns PTV09A-1 mounting family, no detent; 100-kΩ linear variant | `Potentiometer_THT:Potentiometer_Bourns_PTV09A-1_Single_Vertical` |
| R54 | Tone, 20 kΩ linear | Bourns PTV09A-1 mounting family, no detent; 20-kΩ linear variant | `Potentiometer_THT:Potentiometer_Bourns_PTV09A-1_Single_Vertical` |
| RV2 | Blend, 10 kΩ linear | Bourns PTV09A-1 mounting family, no detent; 10-kΩ linear variant | `Potentiometer_THT:Potentiometer_Bourns_PTV09A-1_Single_Vertical` |
| RV4 | Level, 10 kΩ linear | Same 10-kΩ panel-pot variant as Blend | `Potentiometer_THT:Potentiometer_Bourns_PTV09A-1_Single_Vertical` |
| RV1 | Microphone gain, 50 kΩ | Bourns 3296W-1-503LF, multiturn top-adjust trimmer | `Potentiometer_THT:Potentiometer_Bourns_3296W_Vertical` |
| RV3 | Carrier null, 10 kΩ | Bourns 3296W-1-103LF, multiturn top-adjust trimmer | `Potentiometer_THT:Potentiometer_Bourns_3296W_Vertical` |

The four front-panel controls use one 9-mm mounting family, with the knob shaft
perpendicular to the PCB. The selected family is **PTV09A-1**; a PTV09A-2 or another
mounting configuration has different geometry. The PTV09 datasheet includes
10-kΩ, 20-kΩ and 100-kΩ values in its standard resistance range. A full shaft-style,
shaft-length and taper ordering code, plus availability, still needs to be fixed
for procurement. Match all four shaft styles and knob diameters before finalizing
the enclosure. Do not infer physical compatibility from resistance value alone.

Use normal `Device:R_Potentiometer` symbols for the four knobs. R53/R54 are
three-terminal potentiometers **wired as rheostats**, so replace their current
two-terminal symbols before assigning the three-pad footprint. Preserve the two
current circuit endpoints with these terminal assignments:

| Control | Terminal 1 | Terminal 2, wiper | Terminal 3 |
| --- | --- | --- | --- |
| R53 Frequency | Timing-capacitor node, U10 pins 2/6 | Tie to terminal 1 | TMR_RB |
| R54 Tone | TONE_NODE | Tie to terminal 1 | TONE_POT_HI |

This gives clockwise-increasing frequency and tone cutoff. RV1's existing wiper
and terminal 3 tie remains as drawn. The divider/Blend/Level three-terminal
connections remain as drawn. Use the marked terminal numbering, not the visual
left-to-right ordering of a mirrored or rotated symbol.

The two 3296W trimmers are internal setup adjustments, not front-panel knobs.
Their multiturn adjustment is useful for setting gain and minimizing carrier
leakage. Leave screwdriver access above them.

Manufacturer sources:

- [Bourns PTV09 dimensions, ordering options and resistance table](https://www.bourns.com/docs/product-datasheets/ptv09.pdf)
- [Bourns 3296W dimensions and resistance codes](https://www.bourns.com/docs/product-datasheets/3296.pdf)

## Switches

| References | Function | Selected part | KiCad footprint |
| --- | --- | --- | --- |
| SW1, SW2 | BOOTSEL, RESET | C&K PTS645SL43-2LFS; normally-open tactile pushbutton, through-hole, 4.3-mm actuator height | `Button_Switch_THT:SW_PUSH_6mm_H4.3mm` |
| SW4, SW5 | Record, Play/Stop | Same C&K PTS645SL43-2LFS | `Button_Switch_THT:SW_PUSH_6mm_H4.3mm` |
| SW3, SW6 | Bypass, Speaker Arm | E-Switch EG2219; maintained DPDT ON–ON slide switch, right-angle through-hole | `Button_Switch_THT:SW_E-Switch_EG2219_DPDT_Angled` |

The tactile footprint has four physical pads with repeated logical pad numbers
1/1 and 2/2, so it matches the two-terminal pushbutton symbol. Do not use a
four-number footprint without mapping its internally common pairs.

The EG2219 footprint keeps commons **2 and 5** and puts throws **1/4** at one
physical position and **3/6** at the other. That agrees with the intended same-state
coupling of the current generic DPDT symbol. Keep the current SW3 and SW6 wiring.
The slide actuator projects toward the board edge; its enclosure opening needs
to follow the actual mounted part. The Record/Play buttons may need enclosure
plungers, depending on the selected faceplate height.

Manufacturer sources:

- [C&K PTS645 drawings and ordering options](https://www.ckswitches.com/media/1471/pts645.pdf)
- [E-Switch EG2219 specifications](https://www.e-switch.com/product/eg-series-subminiature-slide-switch/?part-number=EG2219)
- [E-Switch EG2219 dimensioned drawing](https://configured-product-images.s3.amazonaws.com/2D/specs/EG2219.pdf)

## Assignment scope

The package choices above cover every 10/22/47/220-µF capacitor, every adjustable
resistor/pot and all six switches in the saved schematic. Small passives can
remain in the 0402/0603 packages already selected. The recommendations do not
change resistor or capacitor nominal values. Selecting ceramic parts for C7/C27
and showing the third pot terminals are symbol/physical-part matching work, not
a change to the intended audio architecture.
