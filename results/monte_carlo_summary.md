# Monte Carlo Device Variation Study — col1 Positive Array

## Setup

Applied to the four grid resistors feeding col1's positive array TIA (R1, R9, R13, R5), which already had parasitic interconnect resistance (Step 2.3) applied. Each resistor value was changed from a fixed nominal to `{mc(nominal, tolerance)}`, and `.step param run 1 20 1` was used to run 20 independent trials per tolerance level, each drawing new random resistor values.

Baseline deterministic result (no variation, parasitics only, from Step 2.3): Vout_col1 = -1.66681V

## Results

| Tolerance | Mean | Std Dev | Min | Max | Range |
|---|---|---|---|---|---|
| +/-5%  | -1.6583V | 0.031V | -1.7389V | -1.5969V | 0.1420V |
| +/-10% | -1.6533V | 0.064V | -1.8174V | -1.5327V | 0.2847V |
| +/-20% | -1.6538V | 0.128V | -1.9973V | -1.4185V | 0.5788V |

Raw per-trial data for each tolerance level is in:
- monte_carlo_5pct_vout_col1.txt
- monte_carlo_10pct_vout_col1.txt
- monte_carlo_20pct_vout_col1.txt

## Findings

1. **Standard deviation scales approximately linearly with tolerance.** Doubling the resistor tolerance (5% to 10%, 10% to 20%) approximately doubles the output standard deviation (0.031 to 0.064 to 0.128), indicating first-order linear sensitivity of the crossbar's output to device-level resistance variation, at least within this tolerance range.

2. **The mean stays stable across tolerance levels.** All three means (-1.6583V, -1.6533V, -1.6538V) cluster within about 2% of the deterministic baseline (-1.66681V), and within the expected sampling variation for n=20 trials (standard error ~ std/sqrt(20)). This confirms the random variation is unbiased, it widens the output spread without introducing a systematic shift.

3. **Verification via extreme-case envelope.** For the +/-10% run, the theoretical worst-case bound (all four resistors simultaneously at their most extreme correlated values) was calculated at approximately -1.862V to -1.507V. All 20 observed trial values fell within this envelope, consistent with correct, independent random variation across the four resistors.

## Context

This mirrors the kind of non-ideality characterization described in the LIMCA paper (Section II-B), where process variation and noise are identified as key factors affecting the robustness of large-scale analog IMC deployments. This study quantifies that sensitivity directly for a small-scale crossbar column, at a smaller scale than the paper's full HSPICE dataset but using the same underlying SPICE-based verification approach.
