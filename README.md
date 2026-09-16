# 4x4 Differential Resistive Crossbar Array — SPICE Simulation and Non-Ideality Analysis

## Overview

This project designs and simulates a small analog In-Memory Computing (IMC) resistive crossbar array in LTspice, modeled after the positive/negative differential architecture described in Vungarala et al., "LIMCA: LLM for Automating Analog In-Memory Computing Architecture Design Exploration" (2025). Rather than using an automated framework, this project builds and hand-verifies the underlying circuit manually, to demonstrate the SPICE-level circuit fundamentals that tools like IMAC-Sim and LIMCA automate at scale.

The crossbar performs an analog matrix-vector multiply: row voltages representing input activations are multiplied by column resistor conductances representing stored weights, with the resulting currents summed and converted to signed output voltages via transimpedance amplifiers (TIAs) and a differential subtraction stage.

## Circuit Architecture

**Positive array:** a 4x4 grid of resistors (R1-R16), each row fed by an independent voltage source (V1-V4, representing DAC outputs at 0.2V, 0.4V, 0.6V, 0.8V), each column summing current into a TIA held at a shared reference voltage (Vref = 0.1V).

**Negative array:** a second, independently-valued 4x4 grid (R21-R36), sharing the same row inputs but with distinct resistor values, representing the negative half of each signed weight.

**Difference stage:** four unity-gain differential amplifiers (1k resistors) subtract each column's positive TIA output from its negative TIA output, producing a final signed output per column.

**Feedback resistors:** all TIAs and difference amps use 3k (TIA) and 1k (difference amp) feedback resistors. Supply rails: Vcc = +5V, Vdd = -5V.

Resistor values were assigned in a distinct, hand-traceable pattern per row and array (see `/schematics` for the full `.asc` file and value tables), specifically so every intermediate result could be checked by hand.

## Results

### Full array output (ideal, no non-idealities)

| Column | Vout (positive array) | Vout (negative array) | Vout (differential) |
|---|---|---|---|
| col1 | -1.67499V | -3.12498V | -1.44999V |
| col2 | -2.02498V | -2.77498V | -0.749993V |
| col3 | -2.77498V | -2.02498V | +0.749993V |
| col4 | -3.12498V | -1.67499V | +1.44999V |

The differential outputs form a clean, symmetric spread (roughly -1.45, -0.75, +0.75, +1.45V), confirming the array correctly represents both positive and negative net weights across all four columns.

### Parasitic interconnect resistance (col1 positive path)

5 ohm parasitic resistors were inserted in series along both the row-side and column-side interconnects feeding col1's positive array.

| Condition | Vout_col1 | Deviation from ideal |
|---|---|---|
| Ideal | -1.67499V | - |
| With parasitics | -1.66681V | ~0.49% |

### Monte Carlo device variation (col1 positive array, R1/R9/R13/R5)

Resistor values were randomized using LTspice's `mc()` function across three tolerance levels, 20 trials each (`.step param run 1 20 1`).

| Tolerance | Mean | Std Dev | Min | Max |
|---|---|---|---|---|
| +/-5%  | -1.6583V | 0.031V | -1.7389V | -1.5969V |
| +/-10% | -1.6533V | 0.064V | -1.8174V | -1.5327V |
| +/-20% | -1.6538V | 0.128V | -1.9973V | -1.4185V |

**Key finding:** output standard deviation scales approximately linearly with resistor tolerance (0.031 -> 0.064 -> 0.128V as tolerance doubles from 5% to 10% to 20%), indicating first-order linear sensitivity of the crossbar's output to device-level resistance variation within this range. All three means stay within about 2% of the deterministic baseline, confirming the variation is unbiased and only widens the output spread rather than shifting it systematically.

## Verification Approach

Every stage of this circuit was verified against independent hand calculations before moving to the next stage, rather than trusting SPICE output directly:

- **Kirchhoff's Current Law (KCL):** at every column node, the sum of the four row currents was checked against the current through the corresponding feedback resistor, confirming correct current summing at each stage.
- **Ohm's law:** used throughout to independently predict node voltages and branch currents given known source voltages and resistor values.
- **TIA output equation** (Vout = Vref - I*Rf): used to predict every transimpedance amplifier's output before comparing against the simulated value.
- **Difference amplifier equation** (Vout = V_neg - V_pos, for unity-gain resistor matching): verified both by the standard formula and by back-calculating actual resistor values from simulated node currents, confirming true unity gain rather than assuming it from the schematic alone.
- **Extreme-case envelope check:** for the Monte Carlo study, a theoretical worst-case output bound was calculated by hand and confirmed that all 20 trials per tolerance level fell within that physically possible range.

## Connection to Prior Work

This project builds on research experience gained through the U.S. CMS PURSUE Program at Baylor University and Fermilab, where signal and power integrity analysis was performed on CMS Mezzanine boards. The non-ideality analysis in this project (parasitic interconnect resistance, device-to-device variation) directly parallels that experience: both involve characterizing how real-world electrical imperfections (parasitics, variation, noise) degrade an ideal circuit's behavior, and quantifying that degradation through hand-verified analysis rather than simulation alone.

## Tools Used

- **LTspice** (schematic capture and SPICE simulation)
- **Monte Carlo analysis** via LTspice's `mc()` function and `.step` directive
- Manual nodal analysis and KCL/Ohm's law verification throughout

## Repository Structure

```
/schematics   - LTspice .asc schematic file(s)
/results      - Exported .op logs and Monte Carlo data (.txt), organized by verification checkpoint
/docs         - One-page project summary
/scripts      - (reserved for future data analysis scripts)
```

## Reference

D. Vungarala, M. H. Amin, P. Mercati, A. Ghosh, A. Roohi, R. Zand, S. Angizi, "LIMCA: LLM for Automating Analog In-Memory Computing Architecture Design Exploration," arXiv:2503.13301, 2025.
