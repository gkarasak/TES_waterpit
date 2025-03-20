# Thermal Energy Storage (TES) Simulation: Water Pit

This repository contains a Python simulation of a **water pit thermal energy storage (TES)** system. The project models the dynamic thermal behavior of a stratified water pit using:

- A **1D conduction model**
- **Dynamic water properties** from [CoolProp](https://github.com/CoolProp/CoolProp)
- A **truncated-pyramid geometry**
- **Stratified charging/discharging logic**
- An **exergy analysis** to quantify thermodynamic performance

---

## Table of Contents

1. [Introduction](#introduction)
2. [Geometry & Modeling](#geometry--modeling)
    - [Truncated-Pyramid Geometry](#truncated-pyramid-geometry)
    - [Governing Equations and Formulas](#governing-equations-and-formulas)
3. [Simulation Details](#simulation-details)
    - [Dynamic Properties](#dynamic-properties)
    - [Heat Transfer and Conduction](#heat-transfer-and-conduction)
    - [Charging and Discharging Process](#charging-and-discharging-process)
4. [Exergy Analysis](#exergy-analysis)
5. [Results and Visualization](#results-and-visualization)
6. [Flowchart of the Simulation Process](#flowchart-of-the-simulation-process)
7. [Assumptions](#assumptions)
8. [How to Run the Simulation](#how-to-run-the-simulation)
9. [Repository Structure](#repository-structure)
10. [License](#license)

---

## 1. Introduction

This project simulates the thermal behavior of a water pit TES system over one year (8760 hours). It captures:

- **1D conduction** between stratified layers
- **Heat losses** through side, bottom, and top surfaces
- **Charging** (hot inlet at 80 °C) and **discharging** (cold inlet at 40 °C)
- **Exergy** calculation for each layer
- A **3D animation** of the evolving temperature profile

---

## 2. Geometry & Modeling

### Truncated-Pyramid Geometry

The water pit is modeled as a truncated pyramid with:

- **Bottom square side:** \( L_{\mathrm{bot}} \)
- **Top square side:** \( L_{\mathrm{top}} \)
- **Total height:** \( h_{\mathrm{total}} = 16\,\text{m} \)

The pit is subdivided into \( n = 10 \) layers, each with a height

\[
h_{\mathrm{layer}} = \frac{h_{\mathrm{total}}}{n}.
\]

For each layer \( i \):

- The bottom side length \( L_{\mathrm{bot},i} \) and the top side length \( L_{\mathrm{top},i} \) are linearly interpolated between \( L_{\mathrm{bot}} \) and \( L_{\mathrm{top}} \).
- The volume of layer \( i \) is computed by:

![Volume Formula](images/formulas/volume_formula.png)

*Figure: Volume of layer \(i\) calculated as*  
\[
V_i = \frac{h_{\mathrm{layer}}}{3} \Bigl( A_{1,i} + A_{2,i} + \sqrt{A_{1,i} A_{2,i}} \Bigr),
\]  
where \(A_{1,i} = L_{\mathrm{bot},i}^2\) and \(A_{2,i} = L_{\mathrm{top},i}^2\).

### Governing Equations and Formulas

#### Conduction Between Layers

The 1D conduction between adjacent layers is modeled using Fourier's law:

![Fourier's Law](images/formulas/fouriers_law.png)

*Figure: Fourier’s law*  
\[
Q_{\text{cond}} = k \, A_{\text{cond}} \, \frac{T_{i+1} - T_i}{\Delta x},
\]
where  
- \(k\) is the thermal conductivity (dynamically obtained from CoolProp),
- \(A_{\text{cond}}\) is the conduction area (top area of the lower layer),
- \(\Delta x = h_{\mathrm{layer}}\).

#### Heat Loss

Heat loss is computed from the external surfaces:

- **Side Surface Loss:**

![Side Loss Formula](images/formulas/side_loss.png)

\[
Q_{\text{side}} = U_{\text{side}} \, A_{\text{side}} \, (T - T_{\text{soil}}).
\]

- **Bottom Surface Loss:**  
  Only for the bottom layer.

- **Top Surface Loss:**  
  For the top layer, a projected area \(A_{\text{proj}}\) is used:

![Top Loss Formula](images/formulas/top_loss.png)

\[
Q_{\text{top}} = U_{\text{top}} \, A_{\text{proj}} \, (T - T_{\text{amb}}).
\]

#### Thermal Exergy

The thermal exergy of each layer is calculated by:

![Exergy Formula](images/formulas/exergy_formula.png)

\[
\text{Ex} = m\, c_p \,\Bigl[(T_K - T_0) - T_0 \ln\Bigl(\frac{T_K}{T_0}\Bigr)\Bigr],
\]
where  
- \(m\) is the mass of water in the layer,  
- \(c_p\) is the specific heat,  
- \(T_K = T_{\text{layer}} + 273.15\) (Kelvin),  
- \(T_0 = 298.15\,\text{K}\) (25 °C).

Exergy destruction due to heat losses is estimated as:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

The overall exergy efficiency is then defined as:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 3. Simulation Details

### Dynamic Properties

At each time step, water properties such as density (\(\rho\)), specific heat (\(c_p\)), and thermal conductivity (\(k\)) are retrieved dynamically from **CoolProp** based on the average temperature of the TES.

### Heat Transfer and Conduction

- **Conduction:**  
  1D conduction between layers is computed using an explicit finite difference scheme.
- **Heat Losses:**  
  Heat losses are calculated from the side surfaces (all layers), the bottom surface (only for the first layer), and the top surface (only for the last layer, using a projected area).

### Charging and Discharging Process

- **Charging:**  
  Hot water at 80 °C is injected at the top layer and flows downward.
- **Discharging:**  
  Cold water at 40 °C enters at the bottom layer and flows upward.

---

## 4. Exergy Analysis

Exergy for each layer is computed using:

\[
\text{Ex} = m\, c_p \,\Bigl[(T_K - T_0) - T_0 \ln\Bigl(\frac{T_K}{T_0}\Bigr)\Bigr].
\]

Exergy destruction is estimated from the heat losses by:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

The overall exergy efficiency is defined as:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 5. Results and Visualization

The simulation produces:
- **Temperature Evolution Plots:** Time series for each layer.
- **Total Thermal Exergy Plot:** Total exergy stored over time.
- **Exergy Destruction Bar Chart:** Exergy loss per layer.
- **3D Animation:** An animated 3D visualization of the TES showing the truncated-pyramid geometry with layers colored according to their temperature.

---

## 6. Flowchart of the Simulation Process

Below is a flowchart illustrating the simulation workflow:

```mermaid
flowchart TD
    A[Start] --> B[Initialize Geometry & Layers]
    B --> C[Set Physical Parameters & Simulation Setup]
    C --> D[Retrieve Dynamic Water Properties]
    D --> E[Compute 1D Conduction Between Layers]
    E --> F[Calculate Heat Losses per Layer]
    F --> G[Apply Stratified Charging/Discharging]
    G --> H[Update Temperature Storage]
    H --> I[Perform Exergy Analysis]
    I --> J[Generate Plots & 3D Animation]
    J --> K[End]
