
# Day 8: Circuit Design and SPICE Simulation

A comprehensive guide to understanding SPICE simulation and NMOS transistor operation for circuit design.

---

## Table of Contents

- [SPICE Simulation Overview](#spice-simulation-overview)
- [NMOS Transistor Fundamentals](#nmos-transistor-fundamentals)
- [Regions of Operation](#regions-of-operation)
- [Hands-on SPICE Setup](#hands-on-spice-setup)

---

## SPICE Simulation Overview

![SPICE Elements](images/spice_elements.png)
![SPICE Units](images/units.png)

### What is SPICE?

**SPICE** (Simulation Program with Integrated Circuit Emphasis) is a powerful electronic circuit simulator developed at UC Berkeley in the 1970s. It allows designers to model and analyze circuit behavior before physical fabrication.

**Key Benefits:**
- ✅ **Verify Correctness** - Catch design errors early with realistic device models
- ✅ **Predict Performance** - Extract timing parameters, bandwidth, and power consumption
- ✅ **Assess Power** - Quantify dynamic and leakage power across conditions
- ✅ **Explore Variability** - Test component tolerances and manufacturing variations

### SPICE Deck Structure

A SPICE input file (deck) contains:
- **Netlist** - Components and nodes detailing circuit connectivity
- **Device Models** - Component behavior and model parameters
- **Initial Conditions** - Starting state of the circuit
- **Stimulus** - Input signals to the circuit
- **Simulation Commands** - Type of analysis to perform

### Common SPICE Elements

| Element | SPICE Syntax | Example |
|---------|-------------|---------|
| Resistor | `R<name> <node1> <node2> <value>` | `R1 in out 1k` |
| Capacitor | `C<name> <node1> <node2> <value>` | `C1 out 0 10p` |
| Voltage Source | `V<name> <node+> <node-> <value>` | `Vdd vdd 0 1.8` |
| MOSFET | `M<name> <d> <g> <s> <b> <model> W=<> L=<>` | `M1 out in 0 0 nmos W=1u L=0.5u` |

### SPICE Units

| Prefix | Symbol | Multiplier |
|--------|--------|-----------|
| Tera | T | 10¹² |
| Giga | G | 10⁹ |
| Mega | Meg | 10⁶ |
| Kilo | k | 10³ |
| Milli | m | 10⁻³ |
| Micro | u | 10⁻⁶ |
| Nano | n | 10⁻⁹ |
| Pico | p | 10⁻¹² |
| Femto | f | 10⁻¹⁵ |

---

## NMOS Transistor Fundamentals

### Basic Structure

![NMOS Basic Structure](images/nmos_basic_structure.png)

An NMOS transistor consists of:
- **P-substrate (Body)** - Bulk p-type silicon, typically tied to ground
- **N⁺ Source/Drain** - Heavily doped n-type regions
- **Gate Oxide** - Thin SiO₂ insulating layer
- **Gate Electrode** - Conductive poly-silicon or metal layer

### Threshold Voltage (V<sub>th</sub>)

The **threshold voltage** is the minimum gate-to-source voltage required to form a conductive channel between source and drain.

![NMOS Threshold - No Bias](images/nmos_threshold1_SD_gnd.png)
*Initial state: V<sub>gs</sub> = 0, transistor OFF*

![NMOS Threshold - Increasing Vgs](images/nmos_threshold3.png)
*Increasing V<sub>gs</sub>: Holes repelled, electrons attracted*

![NMOS Threshold - Channel Formation](images/nmos_threshold4.png)
*V<sub>gs</sub> ≥ V<sub>th</sub>: Strong inversion, channel forms*

**Channel Formation Process:**
1. **V<sub>gs</sub> = 0** → No channel, transistor OFF
2. **V<sub>gs</sub> increases** → Holes repelled, electrons attracted
3. **V<sub>gs</sub> = V<sub>th</sub>** → Strong inversion, channel forms
4. **V<sub>gs</sub> > V<sub>th</sub>** → Transistor ON, current flows

### Body Effect

![Body Effect - Normal Operation](images/body_effect_1.png)
![Body Effect - With Vsb](images/body_effect_2.png)
![Body Effect - Depletion Region](images/body_effect_3.png)

When there's a voltage difference between source and substrate (V<sub>sb</sub> > 0):
- Threshold voltage **increases**
- Depletion region widens
- More V<sub>gs</sub> required to turn ON the transistor

![Body Effect Comparison](images/body_effect_4.png)

**Modified Threshold Voltage:**

![Body Effect Equation](images/body_effect_5.png)

```
V_th = V_th0 + γ(√(2φ_f + V_sb) - √(2φ_f))
```
where γ is the body effect coefficient.

---

## Regions of Operation

### 1. Linear (Triode) Region

**Conditions:** V<sub>gs</sub> ≥ V<sub>th</sub> and V<sub>ds</sub> < (V<sub>gs</sub> - V<sub>th</sub>)

![Resistive Region Operation](images/resistive_region_1.png)
![Resistive Region Voltage Distribution](images/resistive_region_2.png)

The transistor acts as a **voltage-controlled resistor**.

#### Drift Current Theory

![Drift Current 1](images/drift_current_1.png)
![Drift Current 2](images/drift_current_2.png)
![Drift Current 3](images/drift_current_3.png)
![Drift Current 4](images/drift_current_4.png)

**Drain Current Equation:**

![ID Equation Derivation 1](images/id_1.jpg)
![ID Equation Derivation 2](images/id_2.jpg)
![ID Equation Derivation 3](images/id_3.jpg)

```
I_D = k_n[(V_gs - V_th)V_ds - V_ds²/2]
```

**For small V<sub>ds</sub>:**
```
I_D ≈ k_n(V_gs - V_th)V_ds
```

where `k_n = μ_n × C_ox × (W/L)` is the gain factor.

**SPICE Simulation Results:**

![SPICE Linear Region 1](images/spice_1.jpg)
![SPICE Linear Region 2](images/spice_2.jpg)

**Key Characteristics:**
- Current varies with both V<sub>gs</sub> and V<sub>ds</sub>
- Used in analog switches and resistive loads
- Channel is continuous from source to drain

### 2. Saturation (Pinch-off) Region

**Conditions:** V<sub>gs</sub> ≥ V<sub>th</sub> and V<sub>ds</sub> ≥ (V<sub>gs</sub> - V<sub>th</sub>)

![Saturation Region](images/saturation_region.png)
![Pinch-off Condition](images/pinch_off_2.png)
![Pinch-off Detail](images/pinch_off_3.png)

The channel **pinches off** at the drain end.

**Drain Current Model:**

![Drain Current Model 1](images/drain_current_model_1.png)
![Drain Current Model 2](images/drain_current_model_2.png)

**Drain Current Equation:**
```
I_D = (k_n/2)(V_gs - V_th)²(1 + λV_ds)
```

where λ is the channel-length modulation parameter.

**Key Characteristics:**
- Current relatively independent of V<sub>ds</sub>
- Used in amplifiers and current sources
- Channel thickness goes to zero at drain

---

## Hands-on SPICE Setup

### SPICE Deck Structure

![SPICE Setup 1](images/sp1.png)
![SPICE Setup 2](images/sp2.png)

### Writing SPICE Netlist

![SPICE Basic Setup](images/bs1.png)

**Component Syntax:**

![SPICE Nodes](images/spice_nodes.jpg)
![SPICE Netlist Structure 1](images/spice_netlist_1.png)
![SPICE Netlist Structure 2](images/spice_netlist_2.png)
![SPICE Netlist Structure 3](images/spice_netlist_3.png)
![SPICE Netlist Structure 4](images/spice_netlist_4.png)
![SPICE Netlist Structure 5](images/spice_netlist_5.png)

**Width and Length Parameters:**

![Width Parameter](images/spice_netlist_W6.png)
![Length Parameter](images/spice_netlist_L7.png)

**Resistor Definition:**

![Resistor 1](images/spice_netlist_R1.png)
![Resistor 2](images/spice_netlist_R2.png)
![Resistor 3](images/spice_netlist_R3.png)
![Resistor 4](images/spice_netlist_R4.png)

**Voltage Source - VDD:**

![VDD 1](images/spice_netlist_Vdd1.png)
![VDD 2](images/spice_netlist_Vdd2.png)
![VDD 3](images/spice_netlist_Vdd3.png)

**Voltage Source - Vin:**

![Vin 1](images/spice_netlist_Vin1.png)
![Vin 2](images/spice_netlist_Vin2.png)
![Vin 3](images/spice_netlist_Vin3.png)
![Vin 4](images/spice_netlist_Vin4.png)

**Technology File Integration:**

![Tech File](images/spice_netlist_tech_file.png)
![Tech File Details](images/spice_netlist_tech_file1.png)

### Example: NMOS I<sub>D</sub> vs V<sub>DS</sub> Characteristics

![SPICE NMOS Setup 1](images/spn1.png)
![SPICE NMOS Setup 2](images/spn2.png)
![SPICE NMOS Setup 3](images/spn3.png)

**Circuit Description:**
```spice
*** NMOS ID vs VDS Characteristics ***

*** Model Files ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Circuit Netlist ***
XM1 vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8
Vin in 0 1.8

*** Simulation Commands ***
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
plot -vdd#branch
.endc

.end
```

### Running the Simulation

**Clone the repository:**
```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

**Run ngspice:**
```bash
ngspice day1_nfet_idvds_L2_W5.spice
```

**Plot the output:**
```
plot -vdd#branch
```

**Simulation Results:**

![NGSpice Lab Results](images/ngspice_lab1.png)

This generates I<sub>D</sub> vs V<sub>DS</sub> curves for different V<sub>GS</sub> values, showing both linear and saturation regions.

---

## CMOS Inverter Analysis

### Circuit Configuration

![Inverter Schematic](images/inverter_schematic.png)

**Structure:**
- PMOS connected between V<sub>DD</sub> and output
- NMOS connected between output and ground
- Both gates tied to input
- Load capacitance at output

**Operation:**
- **V<sub>in</sub> = HIGH** → NMOS ON, PMOS OFF → V<sub>out</sub> = LOW
- **V<sub>in</sub> = LOW** → NMOS OFF, PMOS ON → V<sub>out</sub> = HIGH

### Inverter Characteristics

![Inverter I-V Characteristics](images/inverter_characteristics.png)

**Left Graph:** NMOS drain current (I<sub>D</sub>) vs output voltage (V<sub>out</sub>) for different gate biases  
**Right Graph:** Voltage transfer characteristics (V<sub>out</sub> vs V<sub>in</sub>)

### Delay Tables (LUT)

![2D Lookup Table](images/2D_LUT.png)

In digital timing analysis, cell delay is stored in 2D or 3D lookup tables:
- **2D LUT:** Input Slew × Output Load → Delay
- **Rows:** Input transition times (20ps, 40ps, 60ps, 80ps)
- **Columns:** Output load capacitances (10fF, 30fF, 50fF, 70fF, 90fF, 110fF)
- **Values:** Corresponding delay values

### SPICE Analysis Types

| Analysis | Purpose |
|----------|---------|
| **DC Analysis** | Find operating point (all voltages/currents) |
| **AC Analysis** | Small-signal frequency response |
| **Transient** | Time-domain behavior |
| **Noise** | Device-generated noise analysis |

---

## Key Takeaways

1. **SPICE is essential** for verifying circuit designs before fabrication
2. **Threshold voltage** determines when a transistor turns ON
3. **Body effect** increases V<sub>th</sub> when V<sub>sb</sub> > 0
4. **Linear region** → Transistor acts as a resistor
5. **Saturation region** → Current relatively constant, used in amplifiers
6. **Always simulate** to catch design issues early and optimize performance

---

## Additional Resources

- [NGSpice Documentation](http://ngspice.sourceforge.net/docs.html)
- [SkyWater 130nm PDK](https://github.com/google/skywater-pdk)
- [SPICE Simulation Tutorial](https://www.seas.upenn.edu/~jan/spice/spice.overview.html)

---

**Note:** This guide uses the open-source SkyWater 130nm PDK for all examples. Simulation results may vary with different technology nodes and process corners.
