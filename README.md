# Thermal Energy Storage (TES) Simulation: Water Pit

This repository contains a Python simulation of a water pit thermal energy storage (TES) system. The project models the dynamic thermal behavior of a stratified water pit using a one-dimensional (1D) conduction model, dynamic water properties from CoolProp, and a truncated-pyramid geometry. In addition, an exergy analysis is performed to quantify the system’s thermodynamic performance.

## Table of Contents

- [Introduction](#introduction)
- [Geometry & Modeling](#geometry--modeling)
  - [Truncated-Pyramid Geometry](#truncated-pyramid-geometry)
  - [Governing Equations and Formulas](#governing-equations-and-formulas)
- [Simulation Details](#simulation-details)
  - [Dynamic Properties](#dynamic-properties)
  - [Heat Transfer and Conduction](#heat-transfer-and-conduction)
  - [Charging and Discharging Process](#charging-and-discharging-process)
- [Exergy Analysis](#exergy-analysis)
- [Results and Visualization](#results-and-visualization)
- [Assumptions](#assumptions)
- [How to Run the Simulation](#how-to-run-the-simulation)
- [Repository Structure](#repository-structure)
- [License](#license)

## Introduction

This project simulates the thermal behavior of a water pit TES system over one year (8760 hours). The simulation uses:
- **Dynamic water properties** (density, specific heat, thermal conductivity) from CoolProp.
- A **1D conduction model** to simulate inter-layer heat transfer.
- **Stratified charging/discharging** to simulate the operation of the TES.
- A **truncated-pyramid geometry** to represent the changing cross-sectional area and external surfaces of the water pit.
- An **exergy analysis** to quantify the thermodynamic losses and overall performance.

## Geometry & Modeling

### Truncated-Pyramid Geometry

The water pit is modeled as a truncated pyramid. This geometry is defined by:
- A **bottom square side** length, \( L_{bot} \), and a **top square side** length, \( L_{top} \).
- The pit is divided into \( n = 10 \) layers, each of height \( h_{\text{layer}} = \frac{h_{\text{total}}}{n} \).

For each layer \( i \):
- **Bottom and top side lengths** are linearly interpolated between the overall bottom and top dimensions.
- The **volume** of layer \( i \) is computed using the truncated pyramid formula:
  
  \[
  V_i = \frac{h_{\text{layer}}}{3} \left( A_{1,i} + A_{2,i} + \sqrt{A_{1,i} A_{2,i}} \right)
  \]
  
  where \( A_{1,i} = \text{(bottom side length)}^2 \) and \( A_{2,i} = \text{(top side length)}^2 \).

- The **external surfaces** (bottom, top, and side areas) are computed to determine the heat losses from the system.

### Governing Equations and Formulas

1. **Conduction Between Layers:**  
   The 1D conduction between adjacent layers is modeled by Fourier's law:
   
   \[
   Q_{\text{cond}} = k \, A_{\text{cond}} \frac{(T_{i+1} - T_i)}{\Delta x}
   \]
   
   where \( k \) is the thermal conductivity (dynamically obtained from CoolProp), \( A_{\text{cond}} \) is the conduction area (taken as the top face of layer \( i \)), and \( \Delta x = h_{\text{layer}} \).

2. **Heat Loss:**  
   Heat losses occur from:
   - **Side surfaces** using the corresponding side area and the difference between the layer temperature and soil temperature.
   - **Bottom surface** (for the bottom layer).
   - **Top surface** (for the top layer) using a projected area (to avoid excessive loss).
   
3. **Thermal Exergy:**  
   The exergy of each layer is calculated by:
   
   \[
   Ex = m \, c_p \, \left[ (T_K - T_0) - T_0 \ln\left(\frac{T_K}{T_0}\right) \right]
   \]
   
   where:
   - \( m \) is the mass of water in the layer,
   - \( c_p \) is the specific heat,
   - \( T_K \) is the layer temperature in Kelvin,
   - \( T_0 \) is the reference temperature in Kelvin (with \( T_{\text{ref}} = 25^\circ\text{C} \) used in this project).

## Simulation Details

### Dynamic Properties

Water properties such as density (\( \rho \)), specific heat (\( c_p \)), and thermal conductivity (\( k \)) are dynamically retrieved from CoolProp for the average tank temperature at each time step.

### Heat Transfer and Conduction

- **Inter-layer Conduction:**  
  Modeled as a 1D explicit scheme using the dynamically updated properties.
  
- **Heat Losses:**  
  Calculated separately for side, bottom (if applicable), and top surfaces.
  
### Charging and Discharging Process

- **Stratified Charging:**  
  Hot fluid (80 °C) enters at the top and flows downward.
  
- **Stratified Discharging:**  
  Cold fluid (40 °C) enters at the bottom and flows upward.

## Exergy Analysis

The simulation computes:
- **Total thermal exergy stored** over time.
- **Exergy destruction** based on the heat losses and the local temperature.
- **Exergy efficiency**, defined as:
  
  \[
  \eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%
  \]
  
  where the exergy input during charging is estimated by multiplying the total heat input by a Carnot factor.

## Results and Visualization

The repository generates several plots:
- **Temperature Evolution:** A graph showing the temperature in each layer over time.
- **Exergy Over Time:** A graph showing the total thermal exergy stored in the TES.
- **Exergy Destruction per Layer:** A bar chart summarizing the exergy destruction for each layer.
- **3D Animation:** An animated 3D visualization of the TES temperature profile evolving over time. The animation uses a persistent colorbar with a fixed range of 40–80 °C.

Below is a sample 3D snapshot (or video) of the TES:
  
![3D TES Snapshot](images/3d_snapshot.png)  
*Figure: 3D Visualization of TES Layers (each layer colored by temperature)*

## Assumptions

- **Geometry:**  
  The water pit is modeled as a truncated pyramid.
- **Dynamic Properties:**  
  Water properties are obtained from CoolProp at 1 atm.
- **Heat Transfer:**  
  A 1D conduction model is used; lateral conduction effects are neglected.
- **Stratification:**  
  The system is assumed to have well-defined layers with minimal mixing.
- **Steady Pressure:**  
  The simulation assumes a constant pressure of 101325 Pa.

## How to Run the Simulation

1. Ensure you have [Python](https://www.python.org/) and [Conda](https://docs.conda.io/en/latest/) installed.
2. Install required packages:
   ```bash
   conda install numpy pandas matplotlib
   pip install CoolProp matplotlib-venn

