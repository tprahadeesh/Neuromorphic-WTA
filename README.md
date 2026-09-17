# GPDK45 Time-Domain Winner-Take-All Network

Cadence Virtuoso implementation of the time-domain winner-take-all integrate-and-fire network from Figure 1 of Abrahamsen, Hafliger, and Lande (ISCAS 2004), retargeted to the GPDK45 1 V device domain.

## Design documents

1. [Calculations](docs/01-calculations.md) - every electrical value, equation, and derived timing result.
2. [Architecture](docs/02-architecture.md) - cell, shared reset, top-level organization, interfaces, and signal flow.
3. [Design decisions](docs/03-design-decisions.md) - rationale, alternatives, assumptions, and limitations.
4. [Final design](docs/04-final-design.md) - transistor-level implementation, Cadence construction sequence, verification, and layout requirements.

## Baseline

- Four signal neurons plus one 5 nA timeout neuron
- 1.0 V `nmos1v` / `pmos1v` device domain
- 1 nA to 100 nA signal-current range
- 235 fF effective soma capacitance
- Two-inverter neuromorphic threshold amplifier
- Approximately 167 ns shared reset pulse

## Change policy

The current revision is **1.0 (2026-09-17)**. When a parameter changes:

1. Change its source value in `docs/01-calculations.md`.
2. Recalculate every dependent quantity listed there.
3. Update the topology or device tables in the relevant document.
4. Rerun the affected tests in `docs/04-final-design.md`.
5. Update the revision notes in all affected files in the same commit.

## Source-paper policy

The IEEE paper is not committed to this repository. Obtain it through an authorized source. This repository contains an original implementation guide and derived engineering calculations, not a copy of the publication.

## Status

The design is calculation-complete. Exact inverter thresholds, current-source biases, MIM-cap dimensions, PVT behavior, mismatch, and extracted parasitics must be measured with the locally installed GPDK45 Spectre models before layout sign-off.
