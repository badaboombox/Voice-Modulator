# Routed codec and speaker layout

Open `Voice Modulator.kicad_pro` in this checkout so KiCad loads the matching project rules and footprint libraries. The alternate layout starts from the in-progress board at commit `a683e65`. It uses the same seven schematic files and 95 × 65 mm outline.

## Placement and routing

The speaker amplifier U12 sits between the codec area and J2. The two class-D outputs run 2.2 and 2.3 mm on front copper from U12 to their ferrite beads. Both traces are 0.6 mm wide. The filtered pair goes on to J2 at 0.4–0.6 mm. The through-hole 47 µF supply capacitor C66 connects to the local 5 V pour. C75 moves toward the codec to make space for C66. The Record/Play switches occupy the area the amp vacated. Existing routed sections, including USB, are part of this layout.

| Footprint-origin distance | Starting board | Alternate |
| --- | ---: | ---: |
| U6 codec to U12 amp | 39.6 mm | 19.7 mm |
| U12 amp to J2 speaker connector | 12.6 mm | 10.6 mm |
| U12 amp to C66 supply capacitor | 11.1 mm | 8.2 mm |

In1.Cu carries the GND plane with no routed traces. In2.Cu distributes +3V3, filtered +3V3A and +5V. A 5 V pour reaches from the power-entry area to the amp and C66. In2.Cu also carries signal tracks. The outer layers carry the remaining routing.

## Verification

KiCad 10.0.3 refilled the zones and ran DRC with schematic parity enabled using the existing project rules.

| Check | Starting board | Alternate |
| --- | ---: | ---: |
| Unconnected items | 98 | 0 |
| Schematic parity findings | 9 | 0 |
| Dangling tracks or vias | 8 | 0 |
| Library-copy warnings | 17 | 17 |

All 17 remaining findings are warnings on the same footprint references as the starting board. The codec library entry in this checkout maps the existing schematic footprint name to the TI RHB0032E land pattern used on the PCB. R53 and R54 follow the two connected terminals in the source schematic. The alternate has 316 vias. USB differential impedance and enclosure fit still need verification against the chosen stackup and hardware.

[Layer plots](review/routed-layers.pdf) · [KiCad DRC report](review/routed-drc.json)
