# Schematic sanity review — 2026-11-22

**Current disposition: connection corrections verified; proceed to footprint assignment.**
The follow-up export contains 188 components and 159 nets. U14 and C18 now use
VBUS; C3 is 470 nF; C30 is removed; codec output assignments and bypass sense are
corrected; C77/C78 restore the omitted bulk/bypass parts; D6/D7 connect to the
headphone ring/tip respectively. The I2C rail and Level value remain correct.
Pin-type metadata is excluded at the user's request. This is a connection-level
signoff for footprint assignment, not a manufacturing or hardware-test release.

The original findings below are retained as history; their circuit corrections
are complete in the latest saved schematics.

The initial review used the actual KiCad 10 hierarchy and exported XML netlist, not just
the connection guide. It covers six circuit sheets, 185 fitted schematic components,
159 nets in the latest export and all 206 IC/protection terminals. References below are the current
KiCad references; resistors, capacitors, switches and connectors have been
reannotated from the guide.

The I2C rail and Level-pot value were corrected in the saved project during the
review. A fresh netlist confirmed both corrections. The remaining findings below
were still present in that intermediate export and are now corrected. Review evidence and source hashes are
in `review/`. No source schematic or board was edited by this review.

## Corrections, grouped by sheet

### powerUSB.kicad_sch

| Priority | Component | Observed | Correction and reason |
| --- | --- | --- | --- |
| High | U14, C18 | The overvoltage supervisor is powered by protected +5V, which disappears when its OUTB shuts off U1. | Move U14 pin 5 and the supply end of its local C18 to raw VBUS. Keep C18 at 100 nF, with a 25-V rating. Keep U14 pin 6 on EFUSE_EN and retain the existing sense dividers. This corrects an oversight in the earlier Rev A.2 plan. |
| Medium | C3 | U1 dVdt capacitor is 100 nF. | Change to the planned 470 nF. At 5 V, the nominal ramp changes from about 11.9 ms to 56.0 ms, reducing downstream capacitor-charging current before USB configuration. |

U14's supply issue is a circuit inference from its specified output behavior:
when its supply falls below UVLO, OUTB becomes high impedance. The input divider
can then enable U1 again even while VBUS remains excessive. The circuit can
therefore repeatedly restart instead of maintaining overvoltage shutdown. Powering
the supervisor upstream keeps the shutdown decision powered. TLV6700 supports
1.8–18 V operation; its startup and input-transient behavior still need the normal
hardware measurements. [TLV6700, sections 7.5 and 8.4](https://www.ti.com/lit/ds/symlink/tlv6700.pdf).

The ramp estimates use the TPS2595 equation `SR_ON = 42000 / C_dVdt[pF]` V/ms.
They are nominal calculations, not measurements. The existing current-limit
resistor gives approximately 0.492 A nominal; firmware must continue to enforce
the configured operating-current budget rather than treating this tolerance-limited
protection setting as a precise USB consumption limit.
[TPS2595, section 8.3.3](https://www.ti.com/lit/ds/symlink/tps2595.pdf).

### mcuStore.kicad_sch

| Priority | Component | Observed | Correction and reason |
| --- | --- | --- | --- |
| Medium | C30 | An added 100-nF capacitor shunts BOOTSEL to ground behind its 1-kΩ resistor. | Remove C30. Retain SW1 and R28, and keep R27 as the flash-select pull-up. BOOTSEL shares the flash-select strap, so an unneeded debounce capacitor adds startup-state history and loading. This is a conservative reference-circuit choice, not a claim that the present circuit is proven to fail every boot. |

C31 on RUN may remain. It is a reset capacitor rather than an audio/UI button
debounce network. The Raspberry Pi reference uses the BOOTSEL switch/header and
series resistor without the added shunt capacitor, and explains the boot-strap
sampling and flash-select power-up requirement.
[RP2040 hardware guide, section 2.2](https://datasheets.raspberrypi.com/rp2040/hardware-design-with-rp2040.pdf).

### mic.kicad_sch

| Priority | Component group | Observed | Correction and reason |
| --- | --- | --- | --- |
| Low; restore planned filtering | MK1 bias supply | C34 provides 10 µF, but the planned parallel high-frequency bypass is omitted. | Add 100 nF from the R34/R35/C34 junction to GND beside this bias network. |

RV1 is correctly wired: terminal 1 follows R37, and terminals 2/3 join U5 pin 2.
Its nominal noninverting gain is `1 + (22k + RV1) / 1k`, or 23–73 V/V.
The U5B reference follower is connected correctly, with the large reference
capacitors on its input divider rather than directly on its output.

### codec.kicad_sch

| Priority | Component group | Observed | Correction and reason |
| --- | --- | --- | --- |
| High | U6 line-output labels | Pin 27 LEFT_LOP is SPK_LINE_P; pin 29 RIGHT_LOP is PLAYBACK_FEED. This reverses the locked output allocation. | Swap those two net assignments: pin 27 → PLAYBACK_FEED; pin 29 → SPK_LINE_P. |
| Medium; restore planned decoupling | U6 analog/driver supply | The three 100-nF analog-rail capacitors are present, but the planned local 10-µF bulk capacitor is missing. | Add 10 µF from +3V3A to GND beside U6's driver-supply group. Keep the individual pin bypasses. |

The codec has programmable cross-routing, so the swapped line outputs are not an
irreparable hardware limitation. They do conflict with the locked firmware and
connection plan. With those planned routes, the analog effect would receive the
processed monitor instead of stored dry playback, creating an unintended feedback
path in Play mode. Swapping the two labels preserves the agreed channel roles.
[AIC3104 output routing and registers 81–93](https://www.ti.com/lit/ds/symlink/tlv320aic3104.pdf).

R7/R8 now both connect to the powered +3V3 net. The initial export had an isolated
+3V3D node containing only these two resistor ends. The latest export confirms
that this fault is corrected; do not add a second pull-up pair at the MCU.
The replacement power symbol uses `power:+3V3` correctly. Remove the old ERC
exclusion and recheck: the latest netlist has no separate +3V3D net, but the
all-severity ERC report still carries the old excluded power-drive message.

Codec supply rails and grounds are connected to the intended physical pins.
The reset pull-down is present. The supervisor/LDO arrangement provides a
readiness threshold, but the codec's AVDD-to-DVDD timing limit and the requirement
that analog supplies remain at least DVDD during shutdown still need waveform
verification. This review does not certify those waveforms.
[AIC3104, section 12 and Table 12-1](https://www.ti.com/lit/ds/symlink/tlv320aic3104.pdf).

### modulo.kicad_sch

| Priority | Component | Observed | Correction and reason |
| --- | --- | --- | --- |
| Medium | SW3B | Ground is on pin 6. That makes BYP_SENSE low when SW3A selects BLEND_OUT, whereas the plan defines low as bypass. | Move the ground connection to pin 4, and put the no-connect marker on pin 6. Keep pin 5 as BYP_SENSE. Then the dry/bypass position grounds the sense input. |

The source-selector contacts, normal/inverted ring-switch inputs, U8 feedback,
tone filter, blend buffer and oscillator/divider topology are connected correctly.
The unused divider half has defined inputs. U10/U11 on +3V3 and the audio stages
on +3V3A are intentional.

R53 and R54 currently use two-terminal variable-resistor symbols. Before choosing
ordinary three-terminal rotary potentiometer footprints, change those symbols to
three-terminal pots and explicitly tie the wiper to an end terminal as described
in the connection guide. A real two-terminal adjustable part could use the
existing symbol, but a three-pad footprint must not silently lose the intended
end-to-wiper tie. RV2 is the panel Blend control; its current trimmer symbol should
be replaced by the appropriate panel-pot symbol when fixing the physical part.

### speakerCont.kicad_sch

| Priority | Component group | Observed | Correction and reason |
| --- | --- | --- | --- |
| Medium; restore planned connector protection | J3 headphone jack | Coupling, series and discharge components are present; the two planned external ESD devices are absent. | Add one PESD5V0S1BA,115 bidirectional suppressor from each jack-side signal, tip and ring, to GND. Put them beside J3. |

RV4 now has its correct 10-kΩ value; the initial export still had the default
symbol name as its value. Its end terminals, ADC series resistor and smoothing
capacitor are connected correctly. Use a panel potentiometer for this user control
when selecting the physical part.

The PAM8403 uses the correct left bridge, supply pins, separate VREF bypass and
input attenuation. The unused right outputs are left open, its input is
AC-grounded, and neither speaker lead is grounded. Both active-low control pins
share AMP_RUN. SW6's two poles agree on the armed position, and the pull-down
keeps the amplifier disabled when disarmed. Headphone capacitor polarity and
separate left/right drivers are correct.
[PAM8403 pin descriptions and application information](https://www.diodes.com/datasheet/download/PAM8403.pdf).

## ERC and symbol quality — informational

Per the user's instruction, pin-type metadata is excluded from the connection-level
disposition. The entries below document ERC limitations, not additional required
wiring changes.

The all-severity ERC export includes exclusions; a clean normal view does not
mean every reported condition was resolved. In the initial export there were
14 nonexcluded library/symbol/footprint warnings, three excluded pin-conflict
warnings and two excluded power-drive errors. One of those excluded errors was
the real isolated +3V3D supply fault described above. The other is affected by
incorrect regulator/eFuse symbol pin types. The latest raw report is
`review/erc.json`. Its latest counts remain 14 nonexcluded warnings, three excluded
warnings and two excluded errors; the netlist check above establishes the current
I2C connectivity independently of that retained excluded message.

The following symbol-definition issues limit what ERC can validate:

| Symbol | Actual issue | Electrical-type correction for reference |
| --- | --- | --- |
| U1 TPS259531DSGR | GND and exposed-pad pins are marked power outputs; OUT is merely a signal output; FLT is a generic output. | Supply inputs/grounds: power inputs; OUT: power output; FLT: open collector/open drain; dVdt/ILM: appropriate passive/analog types. |
| U2 TLV1117LV33DCYR | IN is a signal input and both OUT/TAB are signal outputs. | IN/GND: power inputs; OUT: power output; TAB: passive tied to OUT. Remove the redundant output-rail flag if the corrected power-output pin supplies that rail. |
| U6 TLV320AIC3104IRHBR | Every one of its 33 terminals is passive. | Give supplies/grounds power-input types, control/clock/data pins their input/output/bidirectional types, SDA open-drain/bidirectional semantics, and analog pins their appropriate types. |
| U11 SN74HC74DR | All 14 terminals are passive. | Clock/data/reset/preset: inputs; Q/nQ: outputs; VCC/GND: power inputs. |
| U14 TLV6700DDCR | All six terminals are passive. | Sense pins: inputs; outputs: open collector/open drain; VDD/GND: power inputs. |

These are metadata errors rather than physical shorts, but they explain why ERC
misses real supply or direction mistakes. Fix the library definitions and update
the cached symbols together; simply refreshing symbols from an unchanged library
can restore the bad types. Revisit exclusions individually after cleanup. A
power flag after the passive analog-rail ferrite remains a normal way to tell ERC
that the filtered rail is powered.

Footprint-link warnings for imported parts and blank-library footprint names are
expected work for the upcoming assignment stage. Resolve them using project-local
library tables and exact ordering codes. They do not explain away the circuit
findings above.

## USB naming cleanup

End-to-end USB polarity is currently correct, despite two opposing naming swaps:

| Connector contacts | Exported route | MCU terminal |
| --- | --- | --- |
| J1 A6/B6 D+ | local DN → D1 pins 3/4 → USB_DN → R11 | U3 pin 47 USB_DP |
| J1 A7/B7 D− | local DP → D1 pins 1/6 → USB_DP → R10 | U3 pin 46 USB_DM |

Rename these consistently before routing. For example, make the D+ path DP / USB_DP
throughout and the D− path DM / USB_DM throughout. Update all affected sheet labels
and matching ports together; verify the same physical end-to-end routes afterward.
Do not swap only one endpoint, which would turn a naming issue into a polarity
fault. The two USBLC6 channels are interchangeable.

## Items that passed the schematic sanity check

- USB-C VBUS and ground contacts are joined, CC resistors are independent, shield
  is grounded, and SBU contacts are unused.
- RP2040 has both DVDD pins on its generated 1.1-V rail, TESTEN grounded, its
  exposed pad grounded, correct flash data assignments, and SWD/reset access.
- Crystal ground pads, XOUT series resistor and the two 15-pF load capacitors
  match the selected crystal arrangement.
- Recording-flash SPI pins map to a valid RP2040 SPI1 group; CS, WP and HOLD have
  defined idle states.
- Source-select and codec-reset default pull-downs are present. Record/Play are
  active-low, and Speaker Arm has matching physical enable and sense poles.
- ADC dry/effect input allocation and AC coupling are correct. Op-amp feedback,
  common-mode reference, ring carrier, tone and blend connections are coherent.
- Amplifier bridge wiring, headphone coupling and indicator polarity are correct.

These are connection-level checks, not measured audio, startup-current, thermal,
EMC or acoustic-feedback results. Keep the previously defined bring-up tests.

## Handoff to footprints

After the corrections, re-export the full hierarchy and re-run ERC with exclusions
visible. Confirm the physical knob/switch/jack/debug-connector parts before assigning
their footprints. J5 currently uses STDC14 pin numbering rather than the earlier
Cortex 2x5 connector; either can carry SWD, but the symbol, cable and footprint must
refer to the same connector. This review does not approve a generic 2x5 footprint
for the current J5 symbol.

Record the following part requirements when assigning footprints/BOM fields:
R45/R46 are the matched 0.1% inverter pair; R14/R15/R17/R18 are the 0.1% supervisor
dividers; C9/C10/C40 use C0G; the timing/tone capacitors use stable 5% parts.
Keep the wide-body boot/recording flash packages and verify every exposed-pad
number against the exact IC package drawing. Bulk capacitance must be checked at
its working voltage. Controls and connectors need their exact manufacturer
ordering codes rather than generic symbol names.
