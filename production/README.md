# JLCPCB fabrication and assembly estimate

Upload these three files in the JLCPCB estimator.

| Estimator field | File |
| --- | --- |
| PCB fabrication | [Gerber and drill ZIP](jlcpcb/Voice-Modulator-Gerbers.zip) |
| Assembly BOM | [BOM.csv](jlcpcb/BOM.csv) |
| Component placement | [CPL.csv](jlcpcb/CPL.csv) |

Use the **95 × 65 mm single board**, **four copper layers**, **1.6 mm FR-4**, and **top-side assembly**. Set the PCB and assembly quantities to the number you want priced. Include through-hole assembly in the quote. There are 167 surface-mount placements, one USB-C receptacle with surface-mount signal pins and plated mounting tabs, and 20 through-hole placements per board. Knobs, the external speaker, its cable, and the enclosure are outside the PCB BOM.

The ZIP contains top and bottom copper, both inner copper layers, the board profile, solder mask, silkscreen, paste, and separate plated and non-plated Excellon drills. All use the same absolute millimetre origin. The CPL uses counterclockwise rotations. Its negative Y coordinates align with the Gerbers. For packages whose footprint anchor sits away from the body, the CPL uses the housing, shaft or actuator centre. Check the JLC placement viewer for rotations and polarities, especially J1–J3, the LEDs, polarized capacitors, MK1, and pin 1 on the ICs.

## Assembly coverage

The BOM and CPL each cover the same **188** populated designators. **183** have catalog numbers. The four mounting holes are board features. The vendor calculates availability, minimum purchase quantities, feeder charges and hand-solder charges after matching. An unmatched or deselected line can drop out of the displayed total. [Part matches](part-matches.csv) · [JLCPCB assembly quote workflow](https://jlcpcb.com/help/article/pcb-assembly-faqs)

| Designators | Open selection |
| --- | --- |
| R17 | 1.23 MΩ, 0.1%, 0402 supervisor divider resistor. An exact catalog match still needs sourcing. |
| R53 / R54 / RV2 / RV4 | Bourns PTV09A-1 linear, no detent, respectively 100 kΩ / 20 kΩ / 10 kΩ / 10 kΩ. Select a common shaft style and length. |

Keep those five placements in the quote review. If JLC cannot source them, add the price of separately sourced parts and their installation when comparing against the project budget.

The quote uses catalog selections for generic passives, LEDs and the SWD header. The two inverter resistors and four supervisor-divider positions specify 0.1%. C57 uses C0G. C64 uses a 25 V X7R, ±5% part as a quote candidate. Confirm effective timing capacitance over the operating voltage and temperature before purchase. The microphone selection is the CMA-4544PF-W identified in Adafruit's documentation for product 1064. [Adafruit microphone datasheet](https://cdn-shop.adafruit.com/datasheets/CMA-4544PF-W.pdf)

KiCad's export check returned zero DRC errors, zero open connections and zero schematic parity findings. Sixteen footprint-library warnings remain. Select the final manufacturing stackup and verify USB differential impedance before placing an order. The estimate excludes shipping and tax until JLC calculates them for the selected destination.
