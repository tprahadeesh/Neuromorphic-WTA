# Repository Architecture

## System structure

```text
tdwta4_top
|-- XNEUR0: tdwta_neuron(IIN0 -> VSPIKE0)
|-- XNEUR1: tdwta_neuron(IIN1 -> VSPIKE1)
|-- XNEUR2: tdwta_neuron(IIN2 -> VSPIKE2)
|-- XNEUR3: tdwta_neuron(IIN3 -> VSPIKE3)
|-- XNEUR_TO: tdwta_neuron(ITO=5 nA -> TIMEOUT_SPIKE)
`-- XRESET: tdwta_reset_shared
```

Every neuron contains:

```text
IIN -> VSOMA integration -> AMP1 threshold inverter -> AMP2 restoring inverter -> VSPIKE
          ^                                                          |
          `------------------------ C_FB -----------------------------'

VSOMA -> MN_RST_SW(VRESET) -> RST_MID -> MN_RST_LIM(VSPIKELEN) -> VSS
VSPIKE -> MN_REQ -> shared VRESET_REQ
```

The shared reset block contains:

```text
VDD -> MP_REQ_PU(VBP_RESETLEN) -> VRESET_REQ -> MP_RST_INV/MN_RST_INV -> VRESET
                                      |
                                     C_REQ
                                      |
                                     VSS
```

## Data and control flow

1. The five input currents charge matched soma capacitances in parallel.
2. Current magnitude is encoded as inverse spike latency.
3. The first `VSPIKE` asserts shared active-low `VRESET_REQ` through its `MN_REQ`.
4. `XRESET` converts that request into active-high `VRESET`.
5. `VRESET` enables every `MN_RST_SW`; `MN_RST_LIM` controls discharge current.
6. The weak `MP_REQ_PU` and request capacitance delay reset release until every soma is empty.

## Port interfaces

### `tdwta_neuron`

| Port | Direction/type | Polarity or range | Purpose |
|---|---|---|---|
| `VDD`, `VSS` | power | 1.0 V, 0 V | Core rails |
| `IIN` | analog input | 1-100 nA baseline | Integrated signal current |
| `VSPIKELEN` | analog input | about 0.446 V initial | Soma discharge-current bias |
| `VRESET_REQ` | analog inout | active low | Wired-NOR request node |
| `VRESET` | digital input | active high | Enables soma discharge |
| `VSPIKE` | digital output | active high | Winner pulse |

### `tdwta_reset_shared`

| Port | Direction/type | Polarity or range | Purpose |
|---|---|---|---|
| `VDD`, `VSS` | power | 1.0 V, 0 V | Core rails |
| `VBP_RESETLEN` | analog input | about 0.56 V initial | Weak PMOS pull-up bias |
| `VRESET_REQ` | analog inout | active low | Receives all neuron requests |
| `VRESET` | digital output | active high | Drives every neuron reset switch |

## Documentation architecture

- `docs/01-calculations.md` is the numerical source of truth.
- `docs/02-architecture.md` describes hierarchy, interfaces, and signal flow.
- `docs/03-design-decisions.md` records design rationale and limitations.
- `docs/04-final-design.md` is the Cadence construction and verification specification.
- `docs/05-circuit-walkthrough.md` is the visual circuit explanation and naming authority.
- `docs/diagrams/` contains editable vector schematics used by the walkthrough.

The implementation remains documentation-complete but model-calibration-pending. No Cadence schematic database, Spectre netlist, or measured GPDK45 results are currently present in the repository.
