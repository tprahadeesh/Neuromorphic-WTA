# GPDK45 Time-Domain Winner-Take-All Network

Cadence Virtuoso implementation of the time-domain winner-take-all integrate-and-fire network from Figure 1 of Abrahamsen, Hafliger, and Lande (ISCAS 2004), retargeted to the GPDK45 1 V device domain.

## Design documents

1. [Calculations](docs/01-calculations.md) - every electrical value, equation, and derived timing result.
2. [Architecture](docs/02-architecture.md) - cell, shared reset, top-level organization, interfaces, and signal flow.
3. [Design decisions](docs/03-design-decisions.md) - rationale, alternatives, assumptions, and limitations.
4. [Final design](docs/04-final-design.md) - transistor-level implementation, Cadence construction sequence, verification, and layout requirements.
5. [Circuit walkthrough](docs/05-circuit-walkthrough.md) - expanded transistor schematics, naming convention, stage-by-stage operation, and paper-to-GPDK45 mapping.

## Baseline

- Four signal neurons plus one 5 nA timeout neuron
- 1.0 V `nmos1v` / `pmos1v` device domain
- 1 nA to 100 nA signal-current range
- 235 fF effective soma capacitance
- Two-inverter neuromorphic threshold amplifier
- Approximately 167 ns shared reset pulse

## Change policy

The current revision is **1.1 (2026-09-17)**. When a parameter changes:

1. Change its source value in `docs/01-calculations.md`.
2. Recalculate every dependent quantity listed there.
3. Update the topology or device tables in the relevant document.
4. Rerun the affected tests in `docs/04-final-design.md`.
5. Update the revision notes in all affected files in the same commit.

## Source-paper policy

The locally supplied IEEE paper is used as a licensed study reference and is excluded from version control. The design documents distinguish the paper's reported 0.6 um circuit from this repository's original 1 V GPDK45 retarget and do not treat the publication as a drop-in fabrication specification.

## Status

The design is calculation-complete. Exact inverter thresholds, current-source biases, MIM-cap dimensions, PVT behavior, mismatch, and extracted parasitics must be measured with the locally installed GPDK45 Spectre models before layout sign-off.
