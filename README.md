# Thermal Energy Storage (TES) Simulation: Water Pit

This repository contains a Python simulation of a **water pit thermal energy storage (TES)** system. The project models the dynamic thermal behavior of a stratified water pit using:

- A **1D conduction model**
- **Dynamic water properties** from [CoolProp](https://github.com/CoolProp/CoolProp)
- A **truncated-pyramid geometry**
- **Stratified charging/discharging logic**
- An **exergy analysis** to quantify thermodynamic performance

---

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
- [Flowchart of the Simulation Process](#flowchart-of-the-simulation-process)
- [Assumptions](#assumptions)
- [How to Run the Simulation](#how-to-run-the-simulation)
- [Repository Structure](#repository-structure)
- [License](#license)

---

## Introduction

This project simulates the thermal behavior of a water pit TES system over a one-year period (8760 hours). It captures:

- **1D conduction** between stratified layers
- **Heat losses** through side, bottom, and top surfaces
- **Charging** (hot inlet at 80 °C) and **discharging** (cold inlet at 40 °C)
- **Exergy** calculation for each layer
- A **3D animation** of the evolving temperature profile

---

## Geometry & Modeling

### Truncated-Pyramid Geometry

The water pit is modeled as a truncated pyramid with:
- **Bottom square side:** \(L_{\mathrm{bot}}\)
- **Top square side:** \(L_{\mathrm{top}}\)
- **Total Height:** \(h_{\mathrm{total}} = 16\,\text{m}\)

The pit is subdivided into \(n = 10\) layers, each with a height

\[
h_{\mathrm{layer}} = \frac{h_{\mathrm{total}}}{n}.
\]

For each layer \(i\):
- The bottom side length \(L_{\mathrm{bot},i}\) and the top side length \(L_{\mathrm{top},i}\) are linearly interpolated between \(L_{\mathrm{bot}}\) and \(L_{\mathrm{top}}\).
- The **volume** of layer \(i\) is computed using the truncated square pyramid formula:

\[
V_i \;=\; \frac{h_{\mathrm{layer}}}{3}\,\Bigl( A_{1,i} + A_{2,i} + \sqrt{A_{1,i} \, A_{2,i}} \Bigr),
\]

where \(A_{1,i} = L_{\mathrm{bot},i}^2\) and \(A_{2,i} = L_{\mathrm{top},i}^2\).

The external surfaces (bottom, top, and side areas) of each layer are also computed for use in the heat loss calculations.

### Governing Equations and Formulas

1. **Conduction Between Layers**

   The 1D conduction between adjacent layers is modeled by Fourier’s law:

   \[
   Q_{\text{cond}} \;=\; k \, A_{\text{cond}} \, \frac{T_{i+1} - T_i}{\Delta x},
   \]

   where:
   - \(k\) is the thermal conductivity (dynamically obtained from CoolProp),
   - \(A_{\text{cond}}\) is the conduction area (taken as the top area of the lower layer),
   - \(\Delta x = h_{\mathrm{layer}}\).

2. **Heat Loss**

   - **Side Surface Loss:**

     \[
     Q_{\text{side}} \;=\; U_{\text{side}} \, A_{\text{side}} \, \bigl(T - T_{\text{soil}}\bigr)
     \]

   - **Bottom Surface Loss:**  
     Applies only to the bottom layer.

   - **Top Surface Loss:**  
     For the top layer, a **projected area** \(A_{\text{proj}}\) is used to avoid overestimating the loss:

     \[
     Q_{\text{top}} \;=\; U_{\text{top}} \, A_{\text{proj}} \, \bigl(T - T_{\text{amb}}\bigr)
     \]

3. **Thermal Exergy**

   The thermal exergy of each layer is given by:

   \[
   \text{Ex} \;=\; m\, c_p\,\left[\,(T_K - T_0) - T_0\, \ln\!\left(\frac{T_K}{T_0}\right)\right],
   \]

   where:
   - \(m\) is the mass of water in the layer,
   - \(c_p\) is the specific heat,
   - \(T_K = T_{\text{layer}} + 273.15\) (temperature in Kelvin),
   - \(T_0\) is the reference temperature (25 °C in Kelvin, i.e., 298.15 K).

   Exergy destruction due to heat losses is estimated as:

   \[
   \text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \, \left(1 - \frac{T_0}{T_{\text{layer}}}\right),
   \]

   valid for \(T_{\text{layer}} > T_0\).

   Overall exergy efficiency is then:

   \[
   \eta_{\text{exergy}} \;=\; \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
   \]

---

## Simulation Details

### Dynamic Properties

At each time step the code retrieves water properties (\(\rho\), \(c_p\), \(k\)) from CoolProp based on the average temperature of the TES.

### Heat Transfer and Conduction

- **1D Conduction:**  
  Heat is conducted between layers using an explicit finite difference scheme based on Fourier’s law.

- **Heat Losses:**  
  Heat losses are computed separately for the side surfaces (all layers), the bottom surface (only for the bottom layer), and the top surface (only for the top layer using a projected area).

### Charging and Discharging Process

- **Charging:**  
  Hot water at 80 °C is injected into the top layer and flows downward.

- **Discharging:**  
  Cold water at 40 °C is introduced at the bottom layer and flows upward.

---

## Exergy Analysis

Exergy for each layer is computed using:

\[
\text{Ex} \;=\; m\,c_p\,\left[\,(T_K - T_0) - T_0\, \ln\!\left(\frac{T_K}{T_0}\right)\right],
\]

and exergy destruction is estimated from the heat losses by:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \,\left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

The overall exergy efficiency is calculated as:

\[
\eta_{\text{exergy}} \;=\; \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## Results and Visualization

The simulation generates:
- **Temperature Evolution Plots** for each layer over time.
- A **Total Thermal Exergy Plot**.
- A **Bar Chart** showing exergy destruction per layer.
- A **3D Animation** of the TES showing the truncated-pyramid geometry, with layers colored according to their temperature.

---

## Flowchart of the Simulation Process

```mermaid
flowchart TD
    A[Start] --> B[Initialize Geometry & Layers]
    B --> C[Set Physical Parameters & Simulation Setup]
    C --> D[Retrieve Dynamic Water Properties]
    D --> E[Compute 1D Conduction Between Layers]
    E --> F[Calculate Heat Losses per Layer]
    F --> G[Apply Stratified Charging/Discharging]
    G --> H[Update Temperature Storage]
    H --> I[Run Exergy Analysis]
    I --> J[Generate Plots & 3D Animation]
    J --> K[End]
