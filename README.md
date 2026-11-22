# Voice Modulator

[PCB_LAYOUT.md](PCB_LAYOUT.md): first-pass 95 x 65-mm placement, preview,
checks and routing setup (2026-11-22). The PCB is placed and unrouted.

**Rev A.2 electrical plan locked for schematic capture — 2026-11-21.**

- [SCHEMATIC_GUIDE.md](SCHEMATIC_GUIDE.md): fixed architecture, six
  sheets, operating modes, power sequencing, layout constraints and acceptance work.
- [IC_CONNECTIONS.md](IC_CONNECTIONS.md): draw one sheet at a time. Pin tables
  list nets; each passive value and connection appears once beside its associated
  component. All 15 IC/protection devices and inter-sheet ports are covered.

The screenshot components remain the foundation. The abbreviated flash ordering
code is resolved to W25Q128JVSIQ; the required speaker amp is PAM8403DR (SOP-16). The added codec,
analog-switch ring modulator, oscillator, flash and protection implement Tiers 2/3
and replay of recorded speech through the analog effect.

| Constraint | Locked decision |
| --- | --- |
| PCB | 95 x 65 mm, four layers, 1.6-mm FR-4 |
| Effect | Analog metallic ring modulation, tone and dry/wet blend |
| Audio | 32 ksample/s, 16-bit; one 10-second dry recording slot |
| Monitoring | Headphones live; armed internal speaker for recorded playback |
| Power | USB 2.0 host-powered, 500-mA configuration |
| Budget | $185 allocation, under-$200 landed-order requirement |

The plan is frozen. Schematic capture, ERC, simulation, firmware, PCB layout,
hardware validation and the manufacturing quote are not yet completed.

Rev A.2 retains U6 TLV320AIC3104IRHBR after comparison with AIC3204 and adopts
D1 USBLC6-2SC6 for USB ESD protection, matching the part already in the schematic.

Rev A.2 corrects U12 to the user-specified PAM8403DR already present in KiCad.
Its 16-pin table includes one-channel speaker wiring, input attenuation, VREF
bypass and mute/shutdown control.
