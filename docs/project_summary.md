# Project Summary: 4x4 Differential Resistive Crossbar Array

**One-line description:** Designed and SPICE-simulated a 4x4 differential resistive crossbar array for analog in-memory computing (IMC), with hand-verified parasitic and device-variation analysis.

## What this is

A resistive crossbar array is the core computational unit inside analog in-memory computing accelerators, hardware designed to run neural network math directly within a memory array using resistors instead of digital logic, reducing the data movement bottleneck common in traditional processors. This project builds one from scratch in LTspice: a 4x4 grid of resistors representing stored weights, with independent positive and negative arrays combined through a differential amplifier stage to represent signed weights, matching the architecture described in recent IMC research (LIMCA, Vungarala et al. 2025).

## What was built

- A 4x4 resistive crossbar with independent row inputs and column outputs, using net labels rather than physical wiring to guarantee correct, unambiguous connectivity
- Transimpedance amplifiers (TIAs) converting summed column currents into voltages, using op-amp negative feedback
- A second, independently-weighted negative array and a unity-gain differential amplifier stage, producing genuine signed outputs across all four columns
- Parasitic interconnect resistance modeling, quantifying a ~0.49% output deviation from a small, realistic 5 ohm wire resistance
- A three-level Monte Carlo device variation study (+/-5%, +/-10%, +/-20% resistor tolerance, 20 trials each), finding that output standard deviation scales approximately linearly with tolerance

## Why it matters

Every stage of this circuit was independently verified by hand, using Kirchhoff's Current Law and Ohm's law, before trusting the simulator's output, rather than treating LTspice as a black box. This mirrors the rigor required in real analog circuit design, where a simulator can produce a confident, exact-looking answer for a circuit that is wired completely wrong, so independent verification is the only real safeguard.

## Background

This project connects directly to prior research experience through the U.S. CMS PURSUE Program at Baylor University and Fermilab, performing signal and power integrity analysis on CMS Mezzanine boards. Both efforts share the same underlying skill: characterizing how real-world non-idealities (parasitics, variation, noise) degrade an ideal circuit's behavior, and quantifying that degradation rigorously.

## Full repository

Complete schematics, verification logs, and Monte Carlo datasets available in the project repository, including the full README with detailed results tables and methodology.
