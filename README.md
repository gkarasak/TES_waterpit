# Thermal Energy Storage (TES) Truncated-Pyramid Model

## Description
This simulation models a truncated-pyramid shaped thermal energy storage system with water as the storage medium. It performs exergy analysis and visualizes temperature evolution over time.

## Running the Model in Google Colab

### Prerequisites
- A Google account (for accessing Google Colab)
- Internet connection
- No software installation required

### Step-by-Step Instructions

1. **Open Google Colab**  
   - Go to [Google Colab](https://colab.research.google.com/)  
   - Sign in with your Google account if prompted

2. **Create a New Notebook**  
   - Click on **File → New notebook**  
   - A blank notebook will open with an empty code cell

3. **Install Required Packages**  
   - Copy and paste the following code into the cell and run it (click the play button or press Shift+Enter):
   
   ```python
   !pip install matplotlib-venn
   !pip install CoolProp
4. **Add the Simulation Code**

   - Click **+ Code** to create a new code cell.
   - Copy the entire TES model code into this cell.
   - Run the cell (click the play button or press Shift+Enter).

5. **View the Results**

      Scroll down to see:
   - Temperature evolution graphs
   - Exergy plots
   - Performance metrics table
   - 3D animation of the thermal storage system

6. **Modifying Parameters**

      To change the simulation conditions, modify values in these sections:
   - Geometry parameters (Section 1)
   - Physical parameters (Section 2)
   - Charging/discharging settings (Section 2)

## System Architecture & Workflow

<img src="flowchart_.png" alt="Flowchart Diagram" width="33%">



## How the Code Works

The simulation models a **truncated-pyramid water pit thermal energy storage system** with stratified temperature layers, analyzing both thermal and exergy performance over an annual cycle.

### Key Components

#### 1. Geometry & Layer Definition
```python
h_total = 16.0               # Total height (m)
theta_rad = 0.464257581      # Slope angle in radians
L_bot = 65.66216595          # Bottom square side (m)
L_top = 123.5646893          # Top square side (m)
num_layers = 10              # Number of thermal layers
```

- Each layer is modeled as a truncated pyramid section
- For layer i, the volume is calculated as:

$$V_i = \frac{h_{layer}}{3} \cdot (A_{bottom,i} + A_{top,i} + \sqrt{A_{bottom,i} \cdot A_{top,i}})$$

Where:
- $h_{layer}$ is the height of each layer
- $A_{bottom,i}$ and $A_{top,i}$ are the bottom and top cross-sectional areas

#### 2. Heat Transfer Mechanisms

##### 2.1. Conduction Between Layers
The conduction between adjacent layers follows Fourier's law:

$$Q_{cond,i,i+1} = k_{water} \cdot A_{cond,i} \cdot \frac{T_{i+1} - T_i}{h_{layer}}$$

Where:
- $k_{water}$ is the thermal conductivity of water
- $A_{cond,i}$ is the conduction area between layers
- $T_i$ and $T_{i+1}$ are the temperatures of adjacent layers

##### 2.2. Heat Losses
Heat losses occur through the sides, bottom, and top surfaces:

$$Q_{loss,side,i} = U_{side} \cdot A_{side,i} \cdot (T_i - T_{soil})$$
$$Q_{loss,bottom} = U_{bottom} \cdot A_{bottom} \cdot (T_0 - T_{soil})$$
$$Q_{loss,top} = U_{top} \cdot A_{top} \cdot (T_{n-1} - T_{ambient})$$

Where:
- $U$ values are heat transfer coefficients
- $A$ values are the corresponding surface areas
- $T_{soil}$ and $T_{ambient}$ are environmental temperatures

##### 2.3. Charging and Discharging
The charging process introduces hot water from the top:

**ṁ_in = Q_in / [c_p · (T_hot - T_out)]**

The discharging process extracts heat from the bottom:

**ṁ_out = Q_out / [c_p · (T_in - T_cold)]**




#### 3. Temperature Update
For each time step and each layer, the temperature is updated:

$$T_{i,new} = T_{i,old} + \frac{Q_{net,i} \cdot \Delta t}{m_i \cdot c_{p,i}}$$

Where:
- $Q_{net,i}$ is the net heat flow to/from layer i
- $\Delta t$ is the time step
- $m_i$ is the mass of water in layer i
- $c_{p,i}$ is the specific heat capacity

#### 4. Exergy Analysis
The exergy stored in each layer is calculated as:

$$E_{x,i} = m_i \cdot c_p \cdot \left[ (T_i - T_{ref}) - T_{ref} \cdot \ln\left(\frac{T_i}{T_{ref}}\right) \right]$$

The exergy destruction due to heat losses:

$$E_{x,dest,i} = Q_{loss,i} \cdot \left(1 - \frac{T_{ref}}{T_i}\right)$$

The overall exergy efficiency:

$$\eta_{ex} = \frac{E_{x,in} - E_{x,dest}}{E_{x,in}} \times 100\%$$

## Simulation Results

### 1. Temperature Profiles

The model generates temperature profiles for each layer over time, showing the charging phase (first half of the year) and discharging phase (second half). The profiles demonstrate thermal stratification and the effects of heat losses.

### 2. Exergy Analysis

Total exergy stored and destroyed is tracked throughout the simulation period. The exergy destruction per layer shows where the greatest irreversibilities occur, helping identify areas for potential improvement.

### 3. 3D Visualization

The temperature distribution is visualized in 3D, with a color gradient representing temperatures from 40°C (cold) to 80°C (hot) across the truncated pyramid structure.

## Performance Metrics

The simulation calculates key performance metrics:

| Parameter | Description |
|-----------|-------------|
| Total Heat Input (J) | Energy input during charging phase |
| Total Heat Output (J) | Energy extracted during discharging phase |
| Total Heat Losses (J) | Accumulated heat losses through all surfaces |
| Total Exergy Destruction (J) | Irreversible exergy losses during operation |
| Exergy Efficiency (%) | Ratio of useful exergy to input exergy |

## Code Implementation Details

### 1D Conduction (Explicit Method)
```python
def conduction_1D_explicit(T_old, k_water, A_cond, thickness, masses, cp_water, dt):
    T_new = T_old.copy()
    for i in range(num_layers - 1):
        Q_cond = k_water * A_cond[i] * (T_old[i+1] - T_old[i]) / thickness
        Q_cond_total = Q_cond * dt
        T_new[i] += Q_cond_total / (masses[i] * cp_water)
        T_new[i+1] -= Q_cond_total / (masses[i+1] * cp_water)
    return T_new
```

### Water Properties (using CoolProp)
```python
def water_props(T_C):
    T_K = T_C + 273.15
    cp = CP.PropsSI('C','T',T_K,'P',101325,'Water')   # J/(kg·K)
    rho = CP.PropsSI('D','T',T_K,'P',101325,'Water')   # kg/m³
    k = CP.PropsSI('L','T',T_K,'P',101325,'Water')     # W/(m·K)
    return cp, rho, k
```

### Stratified Charging
```python
def stratified_charge(T_old, m_dot, cp_water, T_in, masses, dt):
    T_new = T_old.copy()
    T_fluid = T_in
    for i in reversed(range(num_layers)):
        Q_dot = m_dot * cp_water * (T_fluid - T_old[i])
        Q_total = Q_dot * dt
        T_new[i] += Q_total / (masses[i] * cp_water)
        T_fluid = T_new[i]
    return T_new
```

### Stratified Discharging
```python
def stratified_discharge(T_old, m_dot, cp_water, T_in, masses, dt):
    T_new = T_old.copy()
    T_fluid = T_in
    for i in range(num_layers):
        Q_dot = m_dot * cp_water * (T_old[i] - T_fluid)
        Q_total = Q_dot * dt
        T_new[i] -= Q_total / (masses[i] * cp_water)
        T_fluid = T_new[i]
    return T_new
```

## Key Features

### Temperature-Dependent Properties
The simulation accounts for temperature-dependent water properties using CoolProp, providing more accurate results than constant-property models.

### Stratified Storage Modeling
The multi-layer approach models thermal stratification, which is crucial for accurately representing thermal energy storage performance.

### Comprehensive Heat Loss Calculation
Heat losses are calculated separately for each layer and surface type (side, bottom, top), providing detailed insights into system performance.

### Exergy Analysis
Beyond simple energy accounting, the model performs exergy analysis to evaluate the quality of energy and identify sources of thermodynamic irreversibilities.

### 3D Visualization
The animated 3D visualization helps understand the spatial temperature distribution and its evolution over time.

## Advanced Applications

This model can be used for:
- Optimizing the design of water pit thermal energy storage systems
- Evaluating the impact of different insulation strategies
- Analyzing seasonal thermal energy storage performance
- Assessing the integration of thermal storage with renewable energy sources
- Studying the effects of different charging/discharging strategies

## References
- CoolProp for fluid properties: http://www.coolprop.org/

