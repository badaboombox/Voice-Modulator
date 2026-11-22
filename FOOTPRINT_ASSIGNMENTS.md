# Footprint assignment — current schematic references

The saved connection corrections have been verified in the full KiCad netlist:
188 components, 159 nets. Proceed to footprint assignment. Source schematic and
board files were not modified by the initial assignment document.

Update 2026-11-22: the saved schematic now assigns C4/C7/C27/C66/C74/C75
to the through-hole selections in CAPACITOR_CONTROL_FOOTPRINTS.md, and assigns
D6/D7, FB1/FB2/FB3, J1 (USB-C) and J3 (audio jack) to the exact footprints
in the tables below. C7/C27 now use nonpolar ceramic symbols. Other rows
retain their prior assignment/recommendation status.

Layout update: all 188 schematic components are now assigned and transferred
to a first-pass unrouted PCB. R53/R54 terminal mapping and U6 exposed-pad
footprint have been reconciled; see [PCB_LAYOUT.md](PCB_LAYOUT.md) for the
current placement, verification and remaining work.

For the larger capacitor, pot and switch selections, use
[CAPACITOR_CONTROL_FOOTPRINTS.md](CAPACITOR_CONTROL_FOOTPRINTS.md). Its tables
resolve the package/part choices identified below and take precedence for those
components. Small fixed passives may use the subsequently approved 0402 packages.

The standard footprint names below were checked against the installed KiCad 10
libraries. Imported footprints require their library folders to be registered in
the project footprint-library table. The tables are grouped in drawing order.

## 1. powerUSB.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U1 | TPS259531DSGR | `Package_DFN_QFN:DFN-8-1EP_2x2mm_P0.5mm_EP0.9x1.6mm` |
| U2 | TLV1117LV33DCYR | `TLV1117LV33DCYR:VREG_TLV1117LV33DCYR` — imported part-specific footprint |
| U7 | TLV70018DDCR | `Package_TO_SOT_SMD:SOT-23-5` |
| U14 | TLV6700DDCR | `Package_TO_SOT_SMD:SOT-23-6` |
| D1 | USBLC6-2SC6 | `Package_TO_SOT_SMD:SOT-23-6` |
| D2 | SMF6V0A | `Diode_SMD:D_SOD-123F` |
| FB1 | BLM18AG601SN1D | `Inductor_SMD:L_0603_1608Metric` |
| J1 | GCT USB4105-GF-A | `Connector_USB:USB_C_Receptacle_GCT_USB4105-xx-A_16P_TopMnt_Horizontal` |

U1's exposed pad is **9**, with a 0.9 × 1.6-mm land. Check its solder-paste windows
and ground connection when placing it. U2's existing symbol explicitly numbers
the output tab **4**; retain the matching imported footprint. The ordinary
`SOT-223-3_TabPin2` footprint does not match that symbol's pad numbering.

U2's footprint source is
`Libraries/TLV1117LV33DCYR/VREG_TLV1117LV33DCYR.kicad_mod`.

## 2. mcuStore.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U3 | RP2040 | `Package_DFN_QFN:QFN-56-1EP_7x7mm_P0.4mm_EP3.2x3.2mm` |
| U4 | W25Q128JVSIQ | `Package_SO:SOIC-8_5.3x5.3mm_P1.27mm` |
| U13 | W25Q128JVSIQ | `Package_SO:SOIC-8_5.3x5.3mm_P1.27mm` |
| Y1 | ABM8-272-T3 | `Crystal:Crystal_SMD_3225-4Pin_3.2x2.5mm` |

U3's exposed pad is **57**. Both flash parts use the wide 208-mil body; do not use
the 3.9-mm SOIC-8 footprint. Y1's grounded package pads remain 2 and 4.

J5 currently uses **STDC14**, with SWDIO on 4, SWCLK on 6 and reference voltage on
3. Choose the physical STDC14 header/cable before assigning it. A generic Cortex
2×5 footprint would not match this symbol. SW1 and SW2 similarly need an exact
pushbutton model; select the physical switch first.

## 3. mic.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U5 | MCP6002-I/SN | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` |
| MK1 | Adafruit 1064 | Imported `1064.kicad_mod` matching the microphone |

The microphone footprint source is
`Libraries/LIB_1064/1064/KiCad/1064.kicad_mod`; register its containing folder under
a project library name, then select that qualified footprint. Check microphone
case/negative orientation. RV1 is the 50-kΩ setup trimmer; choose its exact trimmer
part before assigning the corresponding three-pad footprint.

## 4. codec.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U6 | TLV320AIC3104IRHBR | `Package_DFN_QFN:Texas_RHB0032E_VQFN-32-1EP_5x5mm_P0.5mm_EP3.45x3.45mm` |

Use this **RHB0032E** footprint, with exposed pad **33**, rather than another
5×5-mm QFN having a different exposed-pad size. The package-specific drawing
specifies a 3.45-mm square exposed pad.

## 5. modulo.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U8 | TLV9064IPWR | `Package_SO:TSSOP-14_4.4x5mm_P0.65mm` |
| U9 | TS5A23157DGSR | `Package_SO:TSSOP-10_3x3mm_P0.5mm` |
| U10 | TLC555CDR | `Package_SO:SOIC-8_3.9x4.9mm_P1.27mm` |
| U11 | SN74HC74DR | `Package_SO:SOIC-14_3.9x8.7mm_P1.27mm` |

R53 Frequency and R54 Tone currently use two-terminal variable-resistor symbols.
If the physical controls are three-terminal rotary potentiometers, capture their
third terminal and end-to-wiper tie before using their three-pad footprints.
RV2 is the Blend panel control; RV3 is the carrier-null setup trimmer. SW3 requires
an actual DPDT part with a verified contact-number mapping. These mechanical
parts do not share interchangeable footprints merely because their functions match.

## 6. speakerCont.kicad_sch

| Reference | Exact part | Footprint |
| --- | --- | --- |
| U12 | PAM8403DR | `Package_SO:SOIC-16_3.9x9.9mm_P1.27mm` |
| FB2, FB3 | BLM21PG221SN1D | `Inductor_SMD:L_0805_2012Metric` |
| D6, D7 | PESD5V0S1BA,115 | `Diode_SMD:D_SOD-323` |
| J2 | Molex 53261-0271 | Imported `532610271.kicad_mod` retaining pads 1, 2, MP1 and MP2 |
| J3 | CUI/Same Sky SJ-3523-SMT-TR | `Connector_Audio:Jack_3.5mm_CUI_SJ-3523-SMT_Horizontal` |

J2's footprint source is
`Libraries/LIB_53261-0271/53261-0271/KiCad/532610271.kicad_mod`.
The stock KiCad Molex footprint combines both hold-downs as pad **MP**, whereas
the imported symbol uses **MP1/MP2**, both grounded. Retain the imported footprint
or explicitly reconcile those pad names; do not lose the hold-down ground nets.
J3's stock footprint uses T/R/S pad names, matching the current symbol.

RV4 is the Level panel control. SW4/SW5 need exact pushbutton parts; SW6 needs an
exact DPDT part. Select the three indicator LED parts and desired panel treatment
before fixing their package sizes.

## Passive package policy

| Group | Assignment basis |
| --- | --- |
| Fixed resistors, excluding R53/R54 controls | `Resistor_SMD:R_0603_1608Metric`; 1% general parts, with the specified precision exceptions |
| Small bypass and pF capacitors | `Capacitor_SMD:C_0603_1608Metric` once the selected capacitor's voltage/dielectric specifications fit this case |
| 1-µF coupling/bypass capacitors | Prefer 0805 for procurement and voltage-bias margin; match the actual selected part |
| Timing/tone capacitors | Stable dielectric and tolerance first; select the corresponding case size, rather than forcing a common package |
| 10/22/47-µF bulk capacitors | 10 uF remains 1206 SMD; 22 uF uses THT ceramic, 47 uF uses radial electrolytic; see the selected parts in CAPACITOR_CONTROL_FOOTPRINTS.md |
| C74/C75, 220-µF headphone coupling | Panasonic EEUFR1A221 radial electrolytic; `Capacitor_THT:CP_Radial_D6.3mm_P2.50mm`; positive toward codec |

The general policy is 0603 resistors and small capacitors, with larger footprints
where the actual capacitor requires them. It does not lock all bulk capacitors to
one package without a corresponding orderable part. Retain 25-V ratings for the
raw-VBUS bypass parts. Use C0G for the crystal/feedback pF capacitors and the
specified precision/timing parts from the design guide.

## Sources and remaining assignment work

- [TPS2595 DSG0008A package drawing](https://www.ti.com/lit/ds/symlink/tps2595.pdf)
- [AIC3104 RHB0032E package drawing](https://www.ti.com/lit/ds/symlink/tlv320aic3104.pdf)
- [TLV700 DDC0005A package drawing](https://www.ti.com/lit/ds/symlink/tlv700.pdf)
- [PESD5V0S1BA SOD323 package](https://assets.nexperia.com/documents/data-sheet/PESD5V0S1BA.pdf)

Start by assigning the fixed IC/package rows and fixed resistors. Complete exact
bulk-capacitor and mechanical-part selection before placing them or committing
the enclosure openings. Footprint assignment is the next phase; PCB routing and
hardware performance have not yet been validated.
