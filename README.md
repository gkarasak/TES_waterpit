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
11. [Generate Formula Images Locally (Optional)](#generate-formula-images-locally-optional)

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

The water pit is modeled as a **truncated pyramid** with:

- **Bottom square side:** \( L_{\mathrm{bot}} \)
- **Top square side:** \( L_{\mathrm{top}} \)
- **Total height:** \( h_{\mathrm{total}} = 16\,\text{m} \)

It is subdivided into \( n = 10 \) layers, each with height

$$
h_{\mathrm{layer}} = \frac{h_{\mathrm{total}}}{n}.
$$

For each layer \( i \):
- The **bottom side length** \( L_{\mathrm{bot},i} \) and **top side length** \( L_{\mathrm{top},i} \) are linearly interpolated between \( L_{\mathrm{bot}} \) and \( L_{\mathrm{top}} \).
- The **volume** of layer \( i \) is computed as:

![Volume Formula](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MDAiIGhlaWdodD0iNTAiPjx0ZXh0IHg9IjAiIHk9IjM1IiBmb250LXNpemU9IjI0IiBmaWxsPSJibGFjayI+VihpID0gKGhlX3Nhbmdlcl9cLzMpIC8gKEFfMSwgQF8yLCBcdHNxdXJlKFxfMSBpIEFfMikpPC90ZXh0Pjwvc3ZnPg==)

*Figure: Volume of layer \( i \) calculated as*  
\[
V_i = \frac{h_{\mathrm{layer}}}{3} \left( A_{1,i} + A_{2,i} + \sqrt{A_{1,i} A_{2,i}} \right),
\]
where \(A_{1,i} = L_{\mathrm{bot},i}^2\) and \(A_{2,i} = L_{\mathrm{top},i}^2\).

### Governing Equations and Formulas

#### Conduction Between Layers

The 1D conduction between adjacent layers is modeled by Fourier’s law:

![Fourier's Law](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MDAiIGhlaWdodD0iNTAiPjx0ZXh0IHg9IjAiIHk9IjM1IiBmb250LXNpemU9IjI0IiBmaWxsPSJibGFjayI+UXVfY29uZCA9IGsgbF97QV9jb25kfSAoVGlfKzEpIC8gXGZyYWN7IF90eH0gPC90ZXh0Pjwvc3ZnPg==)

*Figure: Fourier’s law*  
\[
Q_{\text{cond}} = k \, A_{\text{cond}} \, \frac{T_{i+1} - T_i}{\Delta x},
\]
where:
- \(k\) is the thermal conductivity (dynamically obtained from CoolProp),
- \(A_{\text{cond}}\) is the conduction area (the top area of the lower layer),
- \(\Delta x = h_{\mathrm{layer}}\).

#### Heat Loss

Heat losses are computed from the external surfaces:

- **Side Surface Loss:**

![Side Loss Formula](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MDAiIGhlaWdodD0iNTAiPjx0ZXh0IHg9IjAiIHk9IjM1IiBmb250LXNpemU9IjI0IiBmaWxsPSJibGFjayI+UV9zaWRlID0gVV9zaWRlICogQV9zaWRlICogKE0gLSBUX3NpZGwpPC90ZXh0Pjwvc3ZnPg==)

\[
Q_{\text{side}} = U_{\text{side}} \, A_{\text{side}} \, (T - T_{\text{soil}}).
\]

- **Bottom Surface Loss:** (Only for the bottom layer)

- **Top Surface Loss:** (For the top layer, using a projected area)

![Top Loss Formula](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MDAiIGhlaWdodD0iNTAiPjx0ZXh0IHg9IjAiIHk9IjM1IiBmb250LXNpemU9IjI0IiBmaWxsPSJibGFjayI+UV90b3AgPSBVX3RvcCkgKiBBX3RvcCAoVCAtIFRfYW1iKTx0ZXh0Pjwvc3ZnPg==)

\[
Q_{\text{top}} = U_{\text{top}} \, A_{\text{proj}} \, (T - T_{\text{amb}}).
\]

#### Thermal Exergy

Thermal exergy for each layer is calculated by:

![Exergy Formula](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0MDAiIGhlaWdodD0iNjAiPjx0ZXh0IHg9IjAiIHk9IjM1IiBmb250LXNpemU9IjI0IiBmaWxsPSJibGFjayI+RXhlcmd5ID0gbVx0YXBcYXBwIFtcdGFoIC0gVF97fV0gXG5cdGFoPC90ZXh0Pjwvc3ZnPg==)

*Figure: Thermal Exergy Formula*  
\[
\text{Ex} = m\, c_p \,\left[(T_K - T_0) - T_0 \ln\!\left(\frac{T_K}{T_0}\right)\right],
\]
where:
- \(m\) is the mass of water in the layer,
- \(c_p\) is the specific heat,
- \(T_K = T_{\text{layer}} + 273.15\) (in Kelvin),
- \(T_0 = 298.15\,\text{K}\) (25 °C).

Exergy destruction is estimated as:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

Overall exergy efficiency is:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 3. Simulation Details

### Dynamic Properties

At every time step, water properties (\(\rho\), \(c_p\), \(k\)) are dynamically retrieved from **CoolProp** using the average TES temperature.

### Heat Transfer and Conduction

- **Conduction:**  
  The 1D conduction between layers is computed using an explicit finite difference scheme based on Fourier’s law.

- **Heat Losses:**  
  Computed from:
  - Side surfaces (all layers)
  - Bottom surface (only for the bottom layer)
  - Top surface (only for the top layer, using a projected area)

### Charging and Discharging Process

- **Charging:**  
  Hot water (80 °C) is injected at the top and flows downward.
  
- **Discharging:**  
  Cold water (40 °C) enters at the bottom and flows upward.

---

## 4. Exergy Analysis

Exergy for each layer is computed by:

\[
\text{Ex} = m\, c_p \,\left[(T_K - T_0) - T_0 \ln\!\left(\frac{T_K}{T_0}\right)\right],
\]

with \(T_K = T_{\text{layer}} + 273.15\) and \(T_0 = 298.15\,\text{K}\).

Exergy destruction is estimated by:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

Overall exergy efficiency is:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 5. Results and Visualization

The simulation produces:
- **Temperature Evolution Plots** for each layer over time.
- A **Total Thermal Exergy Plot**.
- An **Exergy Destruction Bar Chart**.
- A **3D Animation** of the TES (truncated-pyramid geometry) with layers colored by temperature.

---

## 6. Flowchart of the Simulation Process

Below is the flowchart summarizing the simulation process:

![Simulation Flowchart](data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0c
