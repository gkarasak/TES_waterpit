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

The water pit is modeled as a **truncated pyramid** with the following key dimensions:

- **Bottom square side:** \( L_{\mathrm{bot}} \)
- **Top square side:** \( L_{\mathrm{top}} \)
- **Total height:** \( h_{\mathrm{total}} = 16\,\text{m} \)

The pit is subdivided into \( n = 10 \) layers, each with a height

$$
h_{\mathrm{layer}} = \frac{h_{\mathrm{total}}}{n}.
$$

For each layer \( i \):

- The **bottom side length** \( L_{\mathrm{bot},i} \) and **top side length** \( L_{\mathrm{top},i} \) are linearly interpolated between \( L_{\mathrm{bot}} \) and \( L_{\mathrm{top}} \).
- The **volume** of layer \( i \) is computed as:

![Volume Formula](images/formulas/volume_formula.png)

*Figure: Volume of layer \(i\) calculated as*  
\[
V_i = \frac{h_{\mathrm{layer}}}{3} \left( A_{1,i} + A_{2,i} + \sqrt{A_{1,i} A_{2,i}} \right),
\]
where \(A_{1,i} = L_{\mathrm{bot},i}^2\) and \(A_{2,i} = L_{\mathrm{top},i}^2\).

The external surfaces (bottom, top, and side areas) are computed from these dimensions for use in the heat loss calculations.

### Governing Equations and Formulas

#### Conduction Between Layers

The 1D conduction between adjacent layers is modeled using Fourier’s law:

![Fourier's Law](images/formulas/fouriers_law.png)

*Figure: Fourier’s law*  
\[
Q_{\text{cond}} = k \, A_{\text{cond}} \, \frac{T_{i+1} - T_i}{\Delta x},
\]
where:
- \(k\) is the thermal conductivity (obtained dynamically from CoolProp),
- \(A_{\text{cond}}\) is the conduction area (taken as the top area of the lower layer),
- \(\Delta x = h_{\mathrm{layer}}\).

#### Heat Loss

Heat loss is computed from the external surfaces of each layer as follows:

- **Side Surface Loss:**

![Side Loss Formula](images/formulas/side_loss.png)

\[
Q_{\text{side}} = U_{\text{side}} \, A_{\text{side}} \, (T - T_{\text{soil}}).
\]

- **Bottom Surface Loss:**  
  Applied only for the bottom layer.

- **Top Surface Loss:**  
  For the top layer, a projected area \(A_{\text{proj}}\) is used to avoid excessive losses:

![Top Loss Formula](images/formulas/top_loss.png)

\[
Q_{\text{top}} = U_{\text{top}} \, A_{\text{proj}} \, (T - T_{\text{amb}}).
\]

#### Thermal Exergy

The thermal exergy of each layer is calculated by:

![Exergy Formula](images/formulas/exergy_formula.png)

\[
\text{Ex} = m\, c_p \,\left[(T_K - T_0) - T_0 \ln\!\left(\frac{T_K}{T_0}\right)\right],
\]
where:
- \(m\) is the mass of water in the layer,
- \(c_p\) is the specific heat,
- \(T_K = T_{\text{layer}} + 273.15\) (in Kelvin),
- \(T_0 = 298.15\,\text{K}\) (i.e., 25 °C).

Exergy destruction is estimated by:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

The overall exergy efficiency is:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 3. Simulation Details

### Dynamic Properties

At every time step, water properties such as density (\(\rho\)), specific heat (\(c_p\)), and thermal conductivity (\(k\)) are dynamically retrieved from **CoolProp** based on the average temperature of the TES.

### Heat Transfer and Conduction

- **1D Conduction:**  
  Conduction between layers is computed with an explicit finite difference scheme based on Fourier's law.
  
- **Heat Losses:**  
  Heat losses are calculated for each layer:
  - **Side losses:** For all layers.
  - **Bottom loss:** Only for the bottom layer.
  - **Top loss:** Only for the top layer, using a projected area.

### Charging and Discharging Process

- **Charging:**  
  Hot water (80 °C) enters the top layer and flows downward.
  
- **Discharging:**  
  Cold water (40 °C) enters the bottom layer and flows upward.

---

## 4. Exergy Analysis

Exergy is computed for each layer using the formula:

\[
\text{Ex} = m\, c_p \,\left[(T_K - T_0) - T_0 \ln\!\left(\frac{T_K}{T_0}\right)\right],
\]

and exergy destruction due to heat losses is estimated by:

\[
\text{Ex}_{\text{destroyed}} \approx Q_{\text{loss}} \left(1 - \frac{T_0}{T_{\text{layer}}}\right).
\]

The overall exergy efficiency is then given by:

\[
\eta_{\text{exergy}} = \frac{\text{Exergy In} - \text{Exergy Destruction}}{\text{Exergy In}} \times 100\%.
\]

---

## 5. Results and Visualization

The simulation produces:
- **Temperature Evolution Plots:** Time series for each layer.
- **Total Thermal Exergy Plot:** Total exergy stored over time.
- **Exergy Destruction Bar Chart:** Exergy loss per layer.
- **3D Animation:** An animated 3D visualization of the TES showing the truncated-pyramid geometry with layers colored by temperature.

---

## 6. Flowchart of the Simulation Process

Below is a flowchart summarizing the simulation workflow:

![Simulation Flowchart](images/simulation_flowchart.png)

*Figure: Flowchart of the simulation process.*  
If the diagram does not render, please refer to the attached `simulation_flowchart.png` in the `images/` folder.

---

## 7. Assumptions

| **Assumption**              | **Details**                                                                                           |
|-----------------------------|-------------------------------------------------------------------------------------------------------|
| **Geometry**                | Modeled as a truncated pyramid subdivided into 10 layers                                              |
| **Dynamic Properties**      | Water properties (\(\rho\), \(c_p\), \(k\)) are obtained from CoolProp at 1 atm                        |
| **Conduction Model**        | 1D explicit finite difference; lateral conduction is neglected                                        |
| **Heat Loss**               | Calculated from side surfaces (all layers), bottom surface (only first layer), and top surface (only last layer using a projected area) |
| **Charging/Discharging**      | Stratified: hot water (80 °C) enters at the top; cold water (40 °C) enters at the bottom                  |
| **Exergy Calculation**      | Uses the standard thermal exergy formula with \(T_0 = 25^\circ\text{C}\) (298.15 K)                     |
| **Pressure**                | Assumed constant at 101325 Pa (1 atm)                                                                 |

---

## 8. How to Run the Simulation

1. **Clone** this repository.
2. **Install** the required packages:
   ```bash
   conda install numpy pandas matplotlib
   pip install CoolProp matplotlib-venn
