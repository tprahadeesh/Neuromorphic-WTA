# Codebase Guide

This repository currently contains design documentation and source schematics as editable SVG; it does not yet contain HDL, a Spectre netlist, or a Cadence library database.

| File | Purpose and design rationale |
|---|---|
| `README.md` | Entry point, baseline design summary, document order, and project status. |
| `DECISION_LOG.md` | Timestamped record of completed work, choices, reasoning, verification, and remaining calibration work. |
| `ARCHITECTURE.md` | Repository-level hierarchy, dataflow, and port-interface summary required by the project directives. |
| `CODEBASE_GUIDE.md` | File-by-file index and maintenance map. |
| `docs/A_time_domain_winner-take-all_network_of_integrate-and-fire_neurons.pdf` | Local-only licensed source paper used to understand the original topology, equations, results, and extensions. It is intentionally excluded from version control and is not a GPDK45 implementation specification. |
| `docs/01-calculations.md` | Single source of truth for nominal voltages, currents, capacitances, dimensions, timing, discrimination, and first-order power. Change this file first when an electrical parameter changes. |
| `docs/02-architecture.md` | Functional architecture of the neuron, shared request/reset path, top-level array, interfaces, startup, and scaling. |
| `docs/03-design-decisions.md` | Rationale for the 1 V retarget, two-inverter amplifier, timeout neuron, reset strategy, sizing style, matching, and tie behavior. |
| `docs/04-final-design.md` | Device connection tables, nominal sizes, Cadence construction order, testbench cases, pass criteria, layout guidance, and sign-off boundary. |
| `docs/05-circuit-walkthrough.md` | Main explanatory document. Maps Figure 1 to named MOSFETs, explains every stage and node, distinguishes paper values from retarget values, and gives the full event sequence. |
| `docs/diagrams/tdwta_transistor_schematic.svg` | Clean editable schematic of one repeated neuron. Separates integration, two-inverter threshold detection, feedback, request, and soma discharge while naming all seven local MOSFETs. |
| `docs/diagrams/tdwta_shared_reset.svg` | Clean editable schematic of the shared weak pull-up, request capacitor, reset inverter, and reset fanout. Names all three shared MOSFETs. |
| `docs/diagrams/tdwta_array_interconnect.svg` | Uncluttered system diagram showing four signal neurons, one matched timeout neuron, three shared buses, and winner outputs. |

## Naming authority

The authoritative local transistor names are:

```text
Neuron: MP_AMP1, MN_AMP1, MP_AMP2, MN_AMP2,
        MN_REQ, MN_RST_SW, MN_RST_LIM

Shared: MP_REQ_PU, MP_RST_INV, MN_RST_INV
```

Use hierarchical references such as `XNEUR0/MN_RST_SW` in schematic and simulation documentation. Use `_TO` for the timeout instance only at the hierarchy suffix, not as a different device design.

## Maintenance rules

1. Change source electrical values in `docs/01-calculations.md` and propagate every dependency.
2. If hierarchy, ports, or dataflow change, update both `ARCHITECTURE.md` and `docs/02-architecture.md`.
3. If a device or net is renamed, update the three SVGs, walkthrough, final device tables, and this guide together.
4. Record design choices and verification evidence in `DECISION_LOG.md`.
5. Do not mark calculated bias values as final until Spectre PVT, Monte Carlo, DRC/LVS, and extracted-view criteria pass.
