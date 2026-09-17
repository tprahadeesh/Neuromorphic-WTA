# Final Design

**Design:** GPDK45 time-domain winner-take-all network

**Revision:** 1.0 - 2026-09-17

**Related:** [Calculations](01-calculations.md) | [Architecture](02-architecture.md) | [Decisions](03-design-decisions.md)

## 1. Cadence cell hierarchy

Create these schematic cells:

```text
tdwta_neuron       - repeated integrate-and-fire WTA cell
tdwta_reset_shared - weak pull-up, request capacitor, strong reset inverter
tdwta4_top          - four signal cells, one timeout cell, shared block
tdwta4_tb           - verification testbench
```

Use GPDK45 1 V standard-Vt devices. Confirm symbol and capacitor PCell names in the locally installed PDK reference manual.

## 2. `tdwta_neuron` pins

| Pin | Type | Description |
|---|---|---|
| `VDD` | power | 1.0 V supply |
| `VSS` | power | Ground |
| `IIN` | analog input | Current injected into `VSOMA` |
| `VRESET` | digital input | Shared active-high reset |
| `VSPIKELEN` | analog input | Reset-current bias |
| `VRESET_REQ` | analog inout | Shared active-low wired-NOR node |
| `VSPIKE` | digital output | Winner spike |

## 3. Final neuron device table

Connections are listed as `(D, G, S, B)`.

| Ref. | Device | Connections | Size/value | Function |
|---|---|---|---:|---|
| `MN1` | `nmos1v` | `(A1_OUT, VSOMA, VSS, VSS)` | 0.36/0.09 um | First-inverter pull-down |
| `MP1` | `pmos1v` | `(A1_OUT, VSOMA, VDD, VDD)` | 0.90/0.09 um | First-inverter pull-up |
| `MN2` | `nmos1v` | `(VSPIKE, A1_OUT, VSS, VSS)` | 0.72/0.06 um | Output-inverter pull-down |
| `MP2` | `pmos1v` | `(VSPIKE, A1_OUT, VDD, VDD)` | 1.80/0.06 um | Output-inverter pull-up |
| `M_REQ` | `nmos1v` | `(VRESET_REQ, VSPIKE, VSS, VSS)` | 0.72/0.06 um | Pull request low on spike |
| `M_RST` | `nmos1v` | `(VSOMA, VRESET, RST_MID, VSS)` | 0.60/0.06 um | Reset switch |
| `M_LEN` | `nmos1v` | `(RST_MID, VSPIKELEN, VSS, VSS)` | 0.18/0.18 um | Approximately 2 uA discharge limiter |
| `C_SOMA` | PDK MIM cap | `(VSOMA, VSS)` | 200 fF | Integrator |
| `C_FB` | PDK MIM cap | `(VSPIKE, VSOMA)` | 20 fF | Regenerative feedback |

Use the listed orientation consistently even where MOS source and drain can exchange electrically.

## 4. `tdwta_reset_shared` pins and devices

Pins: `VDD`, `VSS`, `VBP_RESETLEN`, `VRESET_REQ`, and `VRESET`.

| Ref. | Device | Connections | Size/value | Function |
|---|---|---|---:|---|
| `MP_RL` | `pmos1v` | `(VRESET_REQ, VBP_RESETLEN, VDD, VDD)` | 0.18/0.36 um | Approximately 250 nA weak pull-up |
| `MN_RI` | `nmos1v` | `(VRESET, VRESET_REQ, VSS, VSS)` | 1.44/0.06 um | Reset-inverter pull-down |
| `MP_RI` | `pmos1v` | `(VRESET, VRESET_REQ, VDD, VDD)` | 3.60/0.06 um | Reset-inverter pull-up |
| `C_REQ` | PDK MIM cap | `(VRESET_REQ, VSS)` | 70 fF | Reset extension; target 80 fF with parasitics |

During first functional simulation, replace `MP_RL` with an ideal 250 nA current source from `VDD` into `VRESET_REQ`. Restore `MP_RL` before PVT, mismatch, layout, or extraction.

## 5. `tdwta4_top` wiring

1. Instantiate five identical `tdwta_neuron` cells.
2. Use four cells for `IIN0` through `IIN3`.
3. Drive the fifth cell with the matched 5.00 nA timeout current.
4. Tie all `VRESET_REQ` pins to the shared block.
5. Tie all `VRESET` pins to the shared inverter output.
6. Tie all `VSPIKELEN` pins to the calibrated bias, initially 0.446 V.
7. Export `VSPIKE0` through `VSPIKE3`, `TIMEOUT_SPIKE`, `VRESET`, and `VRESET_REQ`.
8. Buffer outputs only when external load exceeds 5 fF. Use identical buffers on all five outputs.

## 6. Schematic construction order

1. Read the installed `gpdk045_pdk_referenceManual.pdf` and confirm 1 V devices, MIM capacitor, voltage limits, and layout rules.
2. Build the first inverter and DC-sweep `VSOMA`. Tune `MP1/MN1` ratio for a nominal 0.49 V trip point.
3. Add the second inverter and verify a clean non-inverting rail-to-rail transition.
4. Add `CSOMA`, `CFB`, an ideal input current, and ideal reset controls.
5. Verify integration, one spike, the feedback step, and complete discharge.
6. Build the shared reset block with an ideal 250 nA pull-up.
7. Verify reset remains high for at least 154 ns.
8. Restore the PMOS pull-up and sweep `VBP_RESETLEN`.
9. Instantiate the complete four-plus-one array.
10. Run functional, PVT, Monte Carlo, layout, DRC/LVS, extraction, and post-layout tests.

## 7. Testbench setup

Use `VDD = 1.00 V`, nominal temperature 27 degC, `VSPIKELEN = 0.446 V` initially, and `VBP_RESETLEN = 0.56 V` initially. Apply a 500 ns startup reset before allowing integration.

Test current sets:

| Test | `[IIN0, IIN1, IIN2, IIN3]` |
|---|---|
| Clear winner | `[6, 7, 8, 10] nA` |
| Timeout | `[1, 2, 3, 4] nA` |
| Near tie | `[10.00, 9.90, 6, 5.5] nA` |
| Exact tie | `[10, 10, 6, 5.5] nA` |
| High-current | `[50, 60, 80, 100] nA` |
| Dynamic handover | Cross `IIN0` and `IIN1` slowly through 10 nA |

## 8. Pass criteria

| Test | Required result |
|---|---|
| Amplifier DC sweep | First-inverter trip from 0.45 V to 0.55 V nominal |
| Latency at 1, 5, 10, 50, 100 nA | Within +/-10% of `VTRIP*CTOT/IIN` pre-layout; +/-15% extracted |
| Feedback | One 70-100 mV positive soma step; no chatter |
| Reset depth | Every soma below 50 mV before reset falls |
| Reset width | At least 154 ns at all required corners |
| Clear winner | Only the 10 nA signal neuron wins for 100 cycles |
| Timeout | Only timeout wins while all signal currents are below 5 nA |
| Near tie | 10.00 nA wins over 9.90 nA nominally |
| Exact tie | One winner or simultaneous spikes are valid; document result |
| Dynamic handover | Winner changes without stuck reset or lost competitions |
| Delay | Spike-to-effective-reset no more than 2.0 ns target |
| Monte Carlo | At least 99% correct for 1% difference at 10 nA+, minimum 200 runs |
| Electrical safety | No terminal exceeds PDK limits |
| Physical verification | Clean DRC and LVS; extracted tests pass |

Recommended process corners are TT, FF, SS, FS, and SF. Use `VDD = 0.90, 1.00, 1.10 V` only if the installed PDK permits that range. If no application temperature range exists, initially test -20 degC, 27 degC, and 85 degC.

## 9. Layout requirements

- Place the five neuron cells as one matched array with edge dummies.
- Use identical capacitor orientation and unit-cap structures.
- Shield every `VSOMA` from spike, reset, and clock lines except for intentional `CFB`.
- Route `VRESET` as a balanced spine or tree, not a daisy chain.
- Keep `VRESET_REQ` compact because its capacitance intentionally controls timing.
- Use identical output loading on every winner output.
- Use common-centroid/interdigitated layout for shared current mirrors or bias replicas.
- Add guard rings and dense local supply contacts.
- Run DRC, LVS, antenna checks, RC extraction, and every extracted-view test.
- Adjust explicit `CSOMA` and `C_REQ` after extraction to preserve 235 fF and 80 fF totals.

## 10. Final versus calibrated values

Final Revision 1.0 topology and nominal values:

- Four signal cells plus one identical timeout cell
- Two-inverter threshold amplifier
- 200 fF soma capacitor and 20 fF feedback capacitor
- Series reset switch/current-limiter topology
- Shared wired-NOR request and strong active-high reset
- Device dimensions in Sections 3 and 4

Values that must be updated from the installed GPDK45 models:

- exact inverter trip points;
- exact `VSPIKELEN` for 2 uA;
- exact `VBP_RESETLEN` for 250 nA;
- physical MIM-cap dimensions;
- extracted total capacitances, reset delay, reset width, mismatch, and power.

Do not treat the design as tapeout-ready until every test in Section 8 passes with the extracted view.

## 11. References

- J. P. Abrahamsen, P. Hafliger, and T. S. Lande, “A Time Domain Winner-Take-All Network of Integrate-and-Fire Neurons,” ISCAS 2004. Figure 1 and equations (1), (3), and (4).
- Cadence Community, [Meaning of 1v in pmos_1v in gpdk045 library](https://community.cadence.com/cadence_technology_forums/f/custom-ic-design/59465/meaning-of-1v-in-pmos-1v-in-gpdk045-library). The response points to the reference manual supplied with the installed PDK.

## Revision notes

- **1.0 - 2026-09-17:** Initial transistor-level design and verification specification.
