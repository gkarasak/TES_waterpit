# Thermal Energy Storage (TES) Simulation: Water Pit

This repository contains a Python simulation of a **water pit thermal energy storage** (TES) system. The project models the dynamic thermal behavior of a stratified water pit using:

- A 1D conduction model  
- Dynamic water properties from **CoolProp**  
- A **truncated-pyramid** geometry  
- Stratified charging/discharging logic  
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

This project simulates the thermal behavior of a **water pit TES** system over a one-year period (8760 hours). It captures:

- **1D conduction** between stratified layers  
- **Heat losses** through side, bottom, and top surfaces  
- **Charging** (hot inlet at 80 °C) and **discharging** (cold inlet at 40 °C)  
- **Exergy** calculation for each layer  
- A **3D animation** of the evolving temperature profile  

---

## Geometry & Modeling

### Truncated-Pyramid Geometry

The water pit is modeled as a **truncated pyramid** with:

- **Bottom square side** \(L_{\mathrm{bot}}\)  
- **Top square side** \(L_{\mathrm{top}}\)  
- **Height** \(h_{\mathrm{total}} = 16\,\text{m}\)

It is subdivided into \(n = 10\) layers (each of height \(\displaystyle h_{\mathrm{layer}} = \frac{h_{\mathrm{total}}}{n}\)). For layer \(i\):

- Bottom side length \(L_{\mathrm{bot},i}\) and top side length \(L_{\mathrm{top},i}\) are linearly interpolated.
- The **volume** of layer \(i\) is computed by

\[
V_i \;=\; \frac{h_{\mathrm{layer}}}{3}\,\Bigl(A_{1,i} + A_{2,i} + \sqrt{\,A_{1,i}\,A_{2,i}\,}\Bigr),
\]

where \(A_{1,i}\) and \(A_{2,i}\) are the squares of the bottom and top side lengths, respectively.

### Governing Equations and Formulas

1. **Conduction Between Layers**  
   The 1D conduction between adjacent layers is modeled by Fourier’s law:

   \[
   Q_{\text{cond}} \;=\; k \; A_{\text{cond}} \;\frac{\,\bigl(T_{i+1} - T_i\bigr)\,}{\Delta x},
   \]

   - \(k\) is the thermal conductivity (dynamically obtained from CoolProp),  
   - \(A_{\text{cond}}\) is the conduction area (top area of the lower layer),  
   - \(\Delta x = h_{\mathrm{layer}}\).

2. **Heat Loss**  
   - **Side surface** loss:

     \[
     Q_{\text{side}} \;=\; U_{\text{side}} \; A_{\text{side}} \;\bigl(T - T_{\text{soil}}\bigr).
     \]

   - **Bottom surface** loss: only for the bottom layer.  
   - **Top surface** loss: only for the top layer, using a projected area to avoid excessive loss:

     \[
     Q_{\text{top}} \;=\; U_{\text{top}} \; A_{\text{proj}} \;\bigl(T - T_{\text{amb}}\bigr).
     \]

3. **Thermal Exergy**  
   The exergy of each layer is calculated by:

   \[
   \text{Ex} \;=\; m\,c_p\,\Bigl[\,(T_K - T_0)\;-\;T_0 \,\ln\!\Bigl(\frac{T_K}{T_0}\Bigr)\Bigr],
   \]

   where:
   - \(m\) is the mass of water in the layer,  
   - \(c_p\) is the specific heat,  
   - \(T_K\) is the layer temperature in Kelvin,  
   - \(T_0\) is the reference temperature (25 °C in Kelvin, i.e., 298.15 K).

---

## Simulation Details

### Dynamic Properties

At each time step, the code retrieves water properties \(\rho, c_p, k\) from **CoolProp** at the **average tank temperature**.

### Heat Transfer and Conduction

**1D conduction** between layers is handled with an **explicit** scheme. Heat losses are computed separately for each layer’s external surfaces (side, bottom if \(i=0\), top if \(i=n-1\)).

### Charging and Discharging Process

- **Charging**: Hot water (80 °C) enters the top layer and flows downward.  
- **Discharging**: Cold water (40 °C) enters the bottom layer and flows upward.

---

## Exergy Analysis

Exergy is computed per layer using:

\[
\text{Ex} = m\,c_p \,\bigl[(T_K - T_0) - T_0 \ln\!\bigl(\tfrac{T_K}{T_0}\bigr)\bigr].
\]

Exergy destruction is estimated by:

\[
\text{Ex}_{\text{destroyed}} \;\approx\; Q_{\text{loss}}\;\Bigl(1 \;-\;\frac{T_0}{T_{\text{layer}}}\Bigr),
\]

where \(Q_{\text{loss}}\) is the heat lost by the layer at temperature \(T_{\text{layer}} > T_0\).

The overall **exergy efficiency** is:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## Results and Visualization

The simulation produces:

- A **temperature vs. time** plot for each layer,  
- A **total exergy vs. time** plot,  
- A **bar chart** of exergy destruction per layer,  
- A **3D animation** showing the truncated-pyramid geometry, colored by temperature.

---

## Flowchart of the Simulation Process

```mermaid
flowchart TD
    A[Start] --> B[Initialize Geometry & Layers]
    B --> C[Set Physical Parameters & Setup]
    C --> D[Retrieve Dynamic Water Props from CoolProp]
    D --> E[Compute 1D Conduction]
    E --> F[Calculate Heat Losses per Layer]
    F --> G[Stratified Charging/Discharging]
    G --> H[Update Temperature Storage]
    H --> I[Exergy Analysis]
    I --> J[Generate Plots & 3D Animation]
    J --> K[End]
