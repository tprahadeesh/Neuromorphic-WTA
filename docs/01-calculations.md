# Calculations

**Design:** GPDK45 time-domain winner-take-all network

**Revision:** 1.0 - 2026-09-17

**Related:** [Architecture](02-architecture.md) | [Decisions](03-design-decisions.md) | [Final design](04-final-design.md)

## 1. Source parameters

This is the single source of truth. Change this table before changing a schematic value.

| Symbol | Meaning | Revision 1.0 value |
|---|---|---:|
| `N` | Signal neurons | 4 |
| `NTO` | Timeout neurons | 1 |
| `VDD` | Core supply | 1.00 V |
| `IIN` | Signal-current range | 1 nA to 100 nA |
| `ITO` | Timeout current | 5.00 nA |
| `CSOMA` | Explicit soma capacitor | 200 fF |
| `CFB` | Feedback capacitor | 20 fF |
| `CPAR,SOMA` | Pre-layout soma parasitic budget | 15 fF |
| `CTOT` | Total integration capacitance | 235 fF |
| `VTH,N` | NMOS hand-calculation assumption | 0.32 V |
| `|VTH,P|` | PMOS hand-calculation assumption | 0.34 V |
| `mu_n*Cox` | NMOS hand-calculation assumption | 250 uA/V^2 |
| `mu_p*Cox` | PMOS hand-calculation assumption | 100 uA/V^2 |
| `VTRIP` | Threshold-amplifier trip point | 0.490 V |
| `IDIS` | Soma reset current | 2.00 uA |
| `VSPIKELEN` | Initial reset-current bias | 0.446 V |
| `CREQ,TOT` | Total request-node capacitance | 80 fF |
| `CREQ` | Explicit request capacitor | 70 fF |
| `IRESETLEN` | Weak request-node pull-up | 250 nA |
| `VBP_RESETLEN` | Initial PMOS pull-up gate bias | 0.56 V |
| `TD` | Maximum spike-to-effective-reset delay target | 2.0 ns |
| `VRESET_TARGET` | Soma voltage required before reset release | 50 mV |
| `TEMP` | Nominal temperature | 27 degC |

The voltage thresholds and mobility terms are explanatory estimates. GPDK45 uses BSIM models; Spectre operating-point results replace these estimates during calibration.

## 2. Threshold amplifier calculation

The neuromorphic amplifier is two CMOS inverters. The first senses the analog soma voltage; the second restores a rail-to-rail, non-inverted spike.

For the first inverter:

```text
beta_n = mu_n*Cox*(Wn/Ln)
beta_p = mu_p*Cox*(Wp/Lp)
```

Balanced strength requires:

```text
Wp/Wn = (mu_n*Cox)/(mu_p*Cox) = 250/100 = 2.5
```

Choose `MN1 = 0.36/0.09 um` and `MP1 = 0.90/0.09 um`:

```text
beta_n = 250*(0.36/0.09) = 1000 uA/V^2
beta_p = 100*(0.90/0.09) = 1000 uA/V^2
VTRIP = (VDD + VTH,N - |VTH,P|)/2
      = (1.00 + 0.32 - 0.34)/2
      = 0.490 V
```

The second inverter is stronger: `MN2 = 0.72/0.06 um`, `MP2 = 1.80/0.06 um`. Its width ratio is also 2.5. A Spectre DC sweep must place the first-inverter trip point between 0.45 V and 0.55 V at TT/27 degC.

## 3. Integration capacitance

```text
CTOT = CSOMA + CFB + CPAR,SOMA
     = 200 fF + 20 fF + 15 fF
     = 235 fF
```

After extraction:

```text
CSOMA,new = 235 fF - CFB,extracted - CPAR,SOMA,extracted
```

For the chosen GPDK45 capacitor PCell:

```text
Area_CSOMA = CSOMA/Cdensity
Area_CFB   = CFB/Cdensity
```

Read `Cdensity` from the installed PDK. Do not assume a universal density or PCell name.

## 4. Latency and firing rate

```text
VSOMA(t) = IIN*t/CTOT
QTH = VTRIP*CTOT = 0.49*235 fF = 115.15 fC
TINT = QTH/IIN
```

| `IIN` | `TINT` | Ideal `1/TINT` | Including 166.8 ns reset |
|---:|---:|---:|---:|
| 1 nA | 115.15 us | 8.684 kHz | 8.672 kHz |
| 5 nA | 23.03 us | 43.42 kHz | 43.11 kHz |
| 10 nA | 11.515 us | 86.84 kHz | 85.60 kHz |
| 50 nA | 2.303 us | 434.2 kHz | 404.9 kHz |
| 100 nA | 1.1515 us | 868.4 kHz | 758.5 kHz |

The timeout neuron receives 5 nA. A signal neuron must exceed 5 nA to beat it.

## 5. Feedback step

For a 1 V output transition:

```text
Delta_VFB = CFB/CTOT*Delta_VSPIKE
          = 20/235*1.00
          = 85.1 mV

VSOMA,PEAK = VTRIP + Delta_VFB
           = 0.490 + 0.085
           = 0.575 V
```

If `CFB` changes, recalculate `CTOT`, latency, reset charge, rate, and energy.

## 6. Reset current and bias

The required removed charge is:

```text
QDIS = CTOT*(VSOMA,PEAK - VRESET_TARGET)
     = 235 fF*(0.575 - 0.050)
     = 123.4 fC

TDIS = QDIS/IDIS = 123.4 fC/2.00 uA = 61.7 ns
```

With a 2.5x PVT/mismatch margin:

```text
TRESET,MIN = 2.5*61.7 ns = 154 ns
```

For current-limiter `M_LEN = 0.18/0.18 um`:

```text
VOV,N = sqrt(2*IDIS/(mu_n*Cox*(W/L)))
      = sqrt(4 uA/(250 uA/V^2*1))
      = 0.126 V

VSPIKELEN = VTH,N + VOV,N = 0.32 + 0.126 = 0.446 V
```

Sweep `VSPIKELEN` from 0.30 V to 0.60 V and replace 0.446 V with the bias that produces 2 uA at `VSOMA = 0.30 V`, `VRESET = 1 V`, TT/27 degC.

## 7. Shared reset duration

```text
TREQ,HOLD = CREQ,TOT*VTRIP/IRESETLEN
          = 80 fF*0.49 V/250 nA
          = 156.8 ns

TSPIKE = CTOT*(0.575 - 0.490)/IDIS
       = 235 fF*0.085/2 uA
       = 10.0 ns

TRESET = TSPIKE + TREQ,HOLD
       = 166.8 ns
```

This exceeds the required 154 ns. Use 70 fF explicitly and budget 10 fF for request-node parasitics.

For `MP_RL = 0.18/0.36 um`, `W/L = 0.5`:

```text
VOV,P = sqrt(2*0.25 uA/(100 uA/V^2*0.5)) = 0.100 V
VSG,P = |VTH,P| + VOV,P = 0.440 V
VBP_RESETLEN = VDD - VSG,P = 0.560 V
```

Start with an ideal 250 nA source, then use `MP_RL` and calibrate its gate bias over PVT.

## 8. WTA discrimination

From the paper:

```text
k = IWIN*TD/(VTRIP*CTOT)
x_min = k/(1+k), where x_min = 1 - IRUNNERUP/IWIN
```

For `TD = 2 ns`:

| Winner current | `k` | Timing-limited `x_min` |
|---:|---:|---:|
| 5 nA | 0.00008684 | 0.00868% |
| 10 nA | 0.00017369 | 0.01737% |
| 50 nA | 0.00086843 | 0.08677% |
| 100 nA | 0.0017369 | 0.17339% |

Mismatch will dominate this ideal limit. The practical target is at least 99% correct selection in 200 Monte Carlo runs for a 1% current difference at 10 nA and above.

## 9. Reset driver and fanout

Use `MN_RI = 1.44/0.06 um` and `MP_RI = 3.60/0.06 um`. For five reset gates:

```text
Agate,total = 5*(0.60 um*0.06 um) = 0.180 um^2
```

Extraction determines actual capacitance. Require request-to-reset delay below 1 ns. If the array grows beyond 16 neurons, use a tapered buffer and recalculate `TD` and discrimination.

## 10. Energy and first-order power

```text
ESOMA <= CTOT*VDD^2 = 235 fF*(1 V)^2 = 235 fJ/cycle
PSOMA at 10 nA <= 235 fJ*85.60 kHz = 20.1 nW
PINPUT for a 10 nA source <= VDD*IIN = 10 nW
```

Measure final power over at least 20 cycles with all neurons, biases, reset logic, and loads included.

## 11. Dependency checklist

| Changed parameter | Recalculate |
|---|---|
| Supply or device flavor | Inverter trip, biases, feedback swing, PVT and voltage limits |
| Any soma capacitance | `CTOT`, latency, rate, feedback, reset, discrimination, energy |
| Input-current range | Latency, rate, discrimination, output bandwidth |
| Timeout current | Decision threshold and timeout latency |
| Reset current/bias | Discharge time, spike width, reset margin |
| Request capacitance/current | Reset hold time and maximum event rate |
| Number of neurons | Reset load, delay, discrimination, shared-node capacitance, power |
| Amplifier dimensions | Trip point, latency, delay, mismatch, power |
| Extracted parasitics | Every timing equation and extracted verification test |

## Revision notes

- **1.0 - 2026-09-17:** Initial calculation set. Spectre and extracted-view calibration pending.
