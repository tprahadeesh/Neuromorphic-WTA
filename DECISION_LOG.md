# Decision Log

## 2026-09-17 18:39:49 IST - Diagram readability redesign

### Work performed

- Replaced the crowded combined schematic with a focused one-neuron diagram.
- Added a separate shared-reset diagram so global circuitry no longer competes with neuron detail.
- Rebuilt the array diagram as five aligned rows with three straight, color-coded shared buses.
- Removed dense sizing annotations from the drawings; exact sizes remain in the adjacent device tables where they are easier to compare.
- Added the licensed source PDF to `.gitignore` so the implementation can be pushed without redistributing the local paper.

### Design choices and reasoning

1. **One visual question per diagram.** The neuron image answers “what is inside one cell,” the shared image answers “how is reset generated,” and the array image answers “how are cells connected.”
2. **Functional blocks with explicit MOSFET names.** Grouping complementary devices inside named inverter blocks is more legible than miniature transistor symbols while preserving unambiguous device references.
3. **Straight buses and no wire crossings.** Request, reset, and bias use separate vertical buses in the array overview.
4. **Keep exact terminal tuples and dimensions in tables.** This prevents the diagram from becoming a netlist rendered as an image while retaining implementation precision in the walkthrough and final-design document.

### Verification performed

- Validated all three SVGs as well-formed XML.
- Rendered and visually inspected all three at full resolution.
- Confirmed all ten functional MOSFET names remain present across the neuron and shared-reset diagrams.

## 2026-09-17 18:31:13 IST - Expanded TD-WTA circuit documentation

### Work performed

- Read the complete four-page ISCAS 2004 paper, including Figures 1-5, equations (1)-(4), measured behavior, head-start failure, timeout neuron, and optional extensions.
- Reconciled the compact paper schematic with the repository's proposed 1 V GPDK45 implementation.
- Added an editable transistor-level SVG and a full-array interconnect SVG.
- Added a stage-by-stage walkthrough, complete device dictionary, active-polarity table, event sequence, equations, paper-to-retarget distinctions, and verification order.
- Replaced ambiguous numeric transistor names with functional names throughout the design documentation.

### Design choices and reasoning

1. **Use hierarchical functional references.** `XNEUR2/MN_REQ` identifies both ownership and function, whereas `MN1` changes meaning when a schematic is copied or flattened. This improves schematic review, waveform probing, LVS debugging, and layout cross-probing.
2. **Expand the paper's amplifier triangle into two CMOS inverters.** The paper text explicitly defines two inverters in series. Showing all four amplifier MOSFETs removes the main ambiguity in Figure 1.
3. **Separate reset switching from reset-current limiting.** `MN_RST_SW` is controlled by global `VRESET`; `MN_RST_LIM` is controlled by `VSPIKELEN`. The names expose the distinct digital and analog responsibilities already present in the topology.
4. **Keep the active-low request distinct from active-high reset.** Color and naming make `VRESET_REQ` and `VRESET` difficult to confuse, which is important because the shared inverter reverses polarity.
5. **Represent the paper's `IresetLen` as a function, not a transistor name.** In the retarget, `MP_REQ_PU` realizes the weak pull-up current and `VBP_RESETLEN` controls it. This preserves correspondence with the paper without hiding the implementation device.
6. **State what is original to the retarget.** The explicit `C_REQ`, 1 V device sizes, 235 fF soma target, biases, and reset-width target are repository design choices pending GPDK45 calibration; they are not values reported by the paper.
7. **Keep the timeout neuron physically identical.** Only its 5 nA input differs. This avoids making a second cell type and reduces systematic comparison error.

### Verification performed

- Validated both SVG files as well-formed XML.
- Rasterized and visually inspected both diagrams for connectivity, labels, clipping, and legibility.
- Checked all Markdown links and all local image references.
- Searched the documentation for legacy ambiguous MOSFET references and inconsistent source-paper statements.
- Recomputed the documented equations independently with scripted numeric checks.

### Open calibration items

The topology and documentation are complete, but exact inverter thresholds, `VSPIKELEN`, `VBP_RESETLEN`, MIM dimensions, PVT behavior, mismatch, and extracted parasitics still require the installed GPDK45 Spectre models.
