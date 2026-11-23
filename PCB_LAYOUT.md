# Routed layout for review

Open `Voice Modulator.kicad_pro` in the project root. The board is based on the approved `layout/codec-speaker` layout at `697dd4e`. All seven schematic sheets match that revision byte for byte.

J2 faces the board's top edge at (74.7, 52.8) mm. MK1 moved 1 mm left. With the project STEP models loaded, the J2 and MK1 body envelopes have 2.26 mm of lateral separation. The connector insertion axis points off the board. Both filtered speaker traces are 0.6 mm wide. In1 remains the GND reference, and In2 distributes +3V3, +3V3A and +5V.

Top silkscreen identifies Record, Play, Bootsel, Reset, Bypass, speaker arm, microphone gain, carrier trim, speaker connector and the three status LEDs. The six restored local STEP references are J2, MK1, Y1, U1, U2 and U14. The headphone jack J3 and the four PTV09A potentiometers still lack matching 3D files in the project and installed KiCad libraries.

KiCad returned zero DRC errors, zero unconnected items, zero schematic parity findings and zero ERC errors. There are 16 footprint-library mismatch warnings and six symbol-library mismatch warnings. USB impedance depends on the fabrication stackup. Enclosure openings and the physical mating plug still need a fit check.

[PCB DRC report](review/preproduction-drc.json) · [Schematic ERC report](review/preproduction-erc.json)

## Historical first-pass placement

The notes below describe the original placement study. Its plots, checks and routing instructions refer to that earlier board.


The saved `Voice Modulator.kicad_pcb` contains all 188 schematic components,
four tentative M3 mounting holes and a 95 x 65-mm board outline. Four copper
layers are enabled and the board thickness is 1.6 mm. This is an unrouted
placement study, dated 2026-11-22.

![Top-view placement](review/first-pass-placement.png)

## Placement arrangement

| Area | Components / intent |
| --- | --- |
| Upper left | MK1 microphone, U5 preamp/reference and RV1 gain trim |
| Upper centre | Record/Play buttons, indicator LEDs, J3 headphone jack and its protection/coupling parts |
| Upper right | J1 USB-C, input protection, eFuse/main regulator and U12 speaker amplifier/J2 connector |
| Centre | U6 codec and its bypass/input parts; U7 core supply and analog-rail filtering nearby |
| Right centre | U3 RP2040, adjacent U4 boot flash, U13 recording flash, Y1 crystal and SWD header |
| Lower left/centre | U8/U9 analog effect and U10/U11 carrier generation; RV3 carrier trim |
| Lower edge | Frequency, Tone, Blend and Level, with shaft centres at x = 16/37/58/79 mm, y = 56 mm |
| Side edges | SW3 Bypass at left and SW6 Speaker Arm at right; angled actuators intentionally project outside the PCB |

Coordinates are measured from the upper-left outline corner. The board corner is
at (50, 50) mm in KiCad. Pot shafts are spaced 21 mm apart; actual knob diameters,
panel support and enclosure openings are still to be chosen. Mounting holes are
3.2-mm NPTH, 3.5 mm inward from each corner. These positions remain adjustable.

Small parts are grouped with their associated circuit. Supply bypasses target
specific IC supply pads, and source resistors target their driving pins. The
next placement pass must refine pad-level access and routing escape, particularly
at U3/U6, and account for regulator heat spreading and exposed-pad ground vias.
All SMT components are on the front side. The preview uses reference overlays;
small-component references are on F.Fab rather than crowding the silkscreen.

## Schematic/footprint corrections included

- R53/R54 now use three-terminal `Device:R_Potentiometer` symbols. Terminal 2
  (wiper) is tied to terminal 1. The former second rheostat endpoint is terminal
  3. Frequency and Tone retain their original two circuit endpoints. A fresh
  exported netlist verifies that these are the only electrical changes.
- U6 now uses the package-specific
  `Package_DFN_QFN:Texas_RHB0032E_VQFN-32-1EP_5x5mm_P0.5mm_EP3.45x3.45mm`
  footprint identified in the assignment notes.
- Custom footprint libraries are registered in the project `fp-lib-table` using
  project-relative paths. All schematic fields and instance paths are carried
  into the PCB, so Update PCB from Schematic can match the existing footprints.
- J3's two silkscreen segments crossing the board edge were moved to F.Fab.
  Its copper, drill geometry and connector position are unchanged. This causes
  the known library-copy warning below.

Original changed source files are preserved in `review/layout-source-snapshot`.

## Checks on the saved placement

| Check | Result |
| --- | --- |
| Schematic components transferred | 188/188, plus 4 board-only mounting holes |
| Source netlist | 159 nets, including named no-connect nets |
| Pad connectivity compared with source netlist | Every source pin matches every corresponding PCB pad |
| KiCad schematic parity | 0 issues |
| Conservative courtyard-envelope intersections | 0 |
| KiCad placement/clearance errors | 0 |
| Unrouted connections reported by KiCad | 446; no tracks/vias have been added |
| Other DRC findings | 1 library-copy warning for the documented J3 silkscreen edit |

Full DRC results are in `review/placement-drc.json`. The SVG preview is in
`review/first-pass-placement.svg`, with positions in
`review/first-pass-placement.json`. The builder script is reproducible but will
overwrite manual PCB edits if rerun; do not rerun it once manual refinement starts.

## Routing setup and next pass

| Layer | Intended use |
| --- | --- |
| F.Cu | Components and primary signal routing |
| In1.Cu | Continuous GND reference |
| In2.Cu | Power distribution and limited secondary routing |
| B.Cu | Secondary signals and GND fill |

Starter rules use 0.15-mm minimum track/clearance, 0.6/0.3-mm through vias and
0.5-mm copper-to-edge clearance. Default tracks are 0.2 mm; Power, Audio and Clock
classes have nominal widths of 0.6, 0.25 and 0.2 mm respectively. Fine-pitch escape
can use the minimum width where needed. These rules have not been matched to a
selected fabricator. The dielectric stack and USB pair width/gap are not fixed;
derive the 90-ohm differential geometry from the selected manufacturing stackup
before routing USB. Four layers enabled is not an impedance specification.

Refine connector/knob positions first, then critical local placement. Route the
crystal and boot QSPI, USB through its ESD protection, local bypass/ground returns,
codec clocks and the microphone/effect paths before general controls. Keep the
class-D output/return loops local to U12/J2, preserve the continuous L2 reference,
and provide ground stitching and thermal paths during routing. Ground/power pours,
test points, exposed-pad vias and thermal copper remain to be implemented.

The placement is ready for inspection and adjustment. It is not a routed board
or a fabrication release.
