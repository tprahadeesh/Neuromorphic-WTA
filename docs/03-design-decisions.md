# Design Decisions

**Design:** GPDK45 time-domain winner-take-all network

**Revision:** 1.0 - 2026-09-17

**Related:** [Calculations](01-calculations.md) | [Architecture](02-architecture.md) | [Final design](04-final-design.md)

## D1. Retarget the paper to 1 V

The paper describes a 0.6 um implementation with about 5 V signal plots, a 1.56 pF integrating capacitance, and a threshold near 1.9 V. Direct reuse would violate the GPDK45 1 V core-device domain. The topology is retained, but all voltages, capacitances, timing, and biases are recalculated for 1.0 V.

## D2. Use standard-Vt 1 V MOS devices

Standard `nmos1v` and `pmos1v` devices are the starting point. They give a practical mid-supply inverter threshold with less leakage than LVT devices. HVT devices may have insufficient margin at 1 V. The installed PDK reference manual controls exact names and voltage limits.

## D3. Use two inverters as the neuromorphic amplifier

Two CMOS inverters faithfully reproduce the simple amplifier in Figure 1. The first behaves as a threshold element; the second restores polarity and drives the feedback capacitor and request transistor. This is smaller and lower-power than an op-amp or conventional continuous-time comparator.

A Schmitt trigger was not selected because it introduces a second threshold and changes the original operating principle.

## D4. Choose 235 fF total soma capacitance

This value provides microsecond-scale latency for 1-100 nA inputs while avoiding the area of the original 1.56 pF design. It keeps the highest nominal event rate below approximately 0.8 MHz after reset overhead. Fifteen femtofarads are reserved for pre-layout parasitics.

## D5. Keep feedback at 20 fF

The feedback capacitor produces an approximately 85 mV upward step when a spike begins. It makes the threshold crossing regenerative without dominating the current-to-time conversion. If chatter appears, raise it in 5 fF increments and recalculate every capacitance-dependent value.

## D6. Use a timeout neuron rather than per-cell leakage

A fifth matched neuron with a constant 5 nA input establishes the minimum winning current. This avoids adding a mismatched leakage transistor to every signal cell and creates a useful idle/health event.

## D7. Make reset longer than the ideal discharge time

The calculated ideal discharge is 61.7 ns. The design requires at least 154 ns and targets about 167 ns. This 2.5x margin addresses the paper's head-start failure: if reset ends early, a losing neuron can retain charge and incorrectly win a later cycle.

## D8. Separate reset switching and reset-current control

`M_RST` is a wide switch controlled by shared reset. `M_LEN` is a longer, smaller device controlled by `VSPIKELEN`. This ensures the bias device, rather than a voltage-dependent switch resistance, primarily sets discharge current.

## D9. Use longer channels for analog current-setting devices

`M_LEN` uses 0.18 um channel length and the weak PMOS pull-up uses 0.36 um. Longer channels improve output resistance and matching. Digital inverters and switches use 0.06-0.09 um to maintain speed.

## D10. Treat calculated bias voltages as starting values

The initial 0.446 V `VSPIKELEN` and 0.56 V `VBP_RESETLEN` come from square-law estimates. Short-channel BSIM effects make them unsuitable as final sign-off values. Spectre sweeps over PVT must replace them with measured settings.

## D11. Use a shared active-low request and active-high reset

NMOS request devices naturally implement a wired-NOR on `VRESET_REQ`. A strong inverter then generates active-high `VRESET`. This uses one request transistor per neuron and one shared reset driver.

## D12. Add explicit request-node capacitance

The reset pulse must remain controlled when layout parasitics change. A 70 fF explicit capacitor plus a 10 fF parasitic budget gives an 80 fF total target. After extraction, reduce the explicit capacitor if necessary to keep the total near 80 fF.

## D13. Preserve identical neuron layout

Signal and timeout neurons use the same circuit, orientation, capacitor geometry, and routing. Edge dummies, symmetric reset routing, identical output loading, and common-centroid bias structures reduce systematic mismatch.

## D14. Do not promise deterministic resolution of an exact tie

Perfectly equal inputs have equal ideal latency. A single winner, noise-selected winner, alternating winners, or simultaneous spikes are physically possible. Requiring strict one-hot output for an exact tie needs a separate asynchronous arbiter, which is outside Figure 1.

## D15. Keep the paper outside the repository

The source PDF is licensed material and is not required to build the design. The repository contains citations and an original implementation, not a redistributed copy.

## D16. Define “final” through measurable criteria

The topology and nominal device table are final for Revision 1.0. Process-dependent values are final only after the Spectre, corner, Monte Carlo, DRC/LVS, and extracted-view criteria in the final-design document pass.

## Revision notes

- **1.0 - 2026-09-17:** Initial decisions for the 1 V, four-input plus timeout implementation.
