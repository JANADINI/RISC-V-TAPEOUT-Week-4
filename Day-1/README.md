# DAY-1: Circuit Design and SPICE Simulation Workshop

A comprehensive guide to CMOS circuit design and SPICE simulation using Sky130 PDK.

## Table of Contents

- [Introduction](#introduction)
- [SPICE Simulation Overview](#spice-simulation-overview)
- [NMOS Transistor Fundamentals](#nmos-transistor-fundamentals)
- [SPICE Setup and Syntax](#spice-setup-and-syntax)
- [Lab Exercises](#lab-exercises)
- [References](#references)

---

##  Introduction

This repository contains materials for learning circuit design and SPICE simulation, focusing on CMOS transistor behavior and circuit analysis using the Sky130 process design kit (PDK).

### What is SPICE?

**SPICE** (Simulation Program with Integrated Circuit Emphasis) is a powerful circuit simulation tool developed at UC Berkeley in the 1970s for modeling and analyzing electronic circuits before physical fabrication.

### Why Use SPICE?

-  **Verify Correctness** - Catch design errors early with realistic device models
-  **Predict Performance** - Extract timing, bandwidth, and power metrics
-  **Assess Power** - Quantify dynamic and leakage power consumption
-  **Explore Variability** - Study manufacturing variations and tolerances

---

##  SPICE Simulation Overview

### SPICE Deck Structure

A SPICE input file (deck) contains:

1. **Netlist** - Circuit components and connectivity
2. **Device Models** - Transistor behavior models and parameters
3. **Initial Conditions** - Starting state of the circuit
4. **Stimulus** - Input signals to the circuit
5. **Analysis Commands** - Type of simulation to run

### Common SPICE Elements

| Element | Symbol | Description |
|---------|--------|-------------|
| Resistor | R | `R<name> <node1> <node2> <value>` |
| Capacitor | C | `C<name> <node1> <node2> <value>` |
| Voltage Source | V | `V<name> <node+> <node-> <DC value>` |
| Current Source | I | `I<name> <node+> <node-> <DC value>` |
| MOSFET | M | `M<name> <drain> <gate> <source> <bulk> <model> W=<> L=<>` |

### SPICE Units

| Unit | Suffix | Example |
|------|--------|---------|
| Tera | T | 1T = 10¹² |
| Giga | G | 1G = 10⁹ |
| Mega | Meg | 1Meg = 10⁶ |
| Kilo | K | 1K = 10³ |
| Milli | m | 1m = 10⁻³ |
| Micro | u | 1u = 10⁻⁶ |
| Nano | n | 1n = 10⁻⁹ |
| Pico | p | 1p = 10⁻¹² |
| Femto | f | 1f = 10⁻¹⁵ |

### Analysis Types

| Type | Description | Command |
|------|-------------|---------|
| **DC Analysis** | Find DC operating point | `.op` or `.dc` |
| **AC Analysis** | Small-signal frequency response | `.ac` |
| **Transient** | Time-domain response | `.tran` |
| **Noise** | Device noise analysis | `.noise` |

---

##  NMOS Transistor Fundamentals

### NMOS Structure

```
        Gate (G)
           |
    ----[Gate Oxide]----
    |                 |
 Source(S)         Drain(D)
   [n+]              [n+]
    |                 |
    ---[P-Substrate]---
           |
        Body (B)
```

**Key Components:**
- **P-substrate (Body)** - Bulk p-type silicon (typically grounded)
- **n⁺ Source/Drain** - Heavily doped n-type regions
- **Gate Oxide** - Thin SiO₂ insulating layer
- **Gate** - Conductive poly-silicon or metal layer

### Threshold Voltage (V<sub>th</sub>)

**Definition:** Minimum gate-to-source voltage required to create a conducting channel between source and drain.

**With Body Effect:**

```
V_th = V_th0 + γ(√(|2φ_F + V_SB|) - √(|2φ_F|))
```

Where:
- V_th0 = Threshold voltage at V_SB = 0
- γ = Body effect coefficient
- φ_F = Fermi potential
- V_SB = Source-to-body voltage

### Regions of Operation

#### 1. **Cut-off Region**
- **Condition:** V_GS < V_th
- **Current:** I_D ≈ 0
- **Behavior:** Transistor is OFF

#### 2. **Linear/Triode Region**
- **Condition:** V_GS > V_th and V_DS < (V_GS - V_th)
- **Current Equation:**
  ```
  I_D = k_n[(V_GS - V_th)V_DS - V_DS²/2]
  ```
- **Simplified (small V_DS):**
  ```
  I_D = k_n(V_GS - V_th)V_DS
  ```
- **Behavior:** Acts like a voltage-controlled resistor

Where: `k_n = μ_n × C_ox × (W/L)` (Gain factor)

#### 3. **Saturation/Pinch-off Region**
- **Condition:** V_GS > V_th and V_DS ≥ (V_GS - V_th)
- **Current Equation:**
  ```
  I_D = (k_n/2)(V_GS - V_th)²(1 + λV_DS)
  ```
- **Without channel-length modulation (λ=0):**
  ```
  I_D = (k_n/2)(V_GS - V_th)²
  ```
- **Behavior:** Current saturates, weakly dependent on V_DS

---

##  SPICE Setup and Syntax

### Basic SPICE Netlist Template

```spice
*** Circuit Title ***
*** Model Description ***
.param temp=27

*** Include Technology Library ***
.lib "path/to/models.lib" tt

*** Netlist Description ***
* Components go here

*** Voltage/Current Sources ***
* Sources go here

*** Simulation Commands ***
.op
.dc / .tran / .ac

.control
run
display
setplot
plot <signal>
.endc

.end
```

### Component Syntax

#### MOSFET
```spice
M<name> <drain> <gate> <source> <bulk> <model_name> W=<width> L=<length>

Example:
M1 vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5u l=2u
```

#### Resistor
```spice
R<name> <node1> <node2> <value>

Example:
R1 n1 in 55
```

#### DC Voltage Source
```spice
V<name> <node+> <node-> <DC_value>

Example:
Vdd vdd 0 1.8
```

#### Pulsed Voltage Source
```spice
V<name> <node+> <node-> PULSE(<V1> <V2> <Tdelay> <Trise> <Tfall> <Ton> <Tperiod>)

Example:
Vin in 0 PULSE(0 1.8 0 10p 10p 1n 2n)
```

### Simulation Commands

```spice
*** DC Operating Point ***
.op

*** DC Sweep ***
.dc <source> <start> <stop> <step>
.dc Vdd 0 1.8 0.1

*** Nested DC Sweep ***
.dc <source1> <start1> <stop1> <step1> <source2> <start2> <stop2> <step2>
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

*** Transient Analysis ***
.tran <tstep> <tstop> <tstart> <tmax>
.tran 10p 4n

*** AC Analysis ***
.ac dec <points_per_decade> <fstart> <fstop>
.ac dec 10 1 1G
```

### Control Block Commands

```spice
.control
run                    # Run simulation
display                # Show available plots
setplot dc1           # Select plot
plot -vdd#branch      # Plot current through Vdd
plot vout             # Plot voltage at node vout
quit                  # Exit ngspice
.endc
```

---

##  Lab Exercises

### Setup Instructions

#### 1. Clone Repository

```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

#### 2. Directory Structure

```
design/
├── day1_nfet_idvds_L2_W5.spice
├── day2_nfet_idvds_L015_W039.spice
├── day2_nfet_idvgs_L015_W039.spice
├── day3_inv_tran_Wp084_Wn036.spice
├── day3_inv_vtc_Wp084_Wn036.spice
├── day4_inv_noisemargin_wp1_wn036.spice
├── day5_inv_devicevariation_wp7_wn042.spice
├── day5_inv_supplyvariation_Wp1_Wn036.spice
└── sky130_fd_pr/
    └── models/
        └── sky130.lib.spice
```

### Lab 1: NMOS I-V Characteristics

**File:** `day1_nfet_idvds_L2_W5.spice`

```spice
*** Model Description ***
.param temp=27

*** Including sky130 library files ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Netlist Description ***
XM1 vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=5 l=2
R1 n1 in 55
Vdd vdd 0 1.8
Vin in 0 1.8

*** Simulation Commands ***
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
display
setplot dc1
.endc

.end
```

#### Run Simulation

```bash
# Start ngspice
ngspice day1_nfet_idvds_L2_W5.spice

# In ngspice prompt, plot drain current
ngspice 1 -> plot -vdd#branch
```

**Expected Output:** I_D vs V_DS curves for different V_GS values

![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-1/Pictures/model%20description)
---

### Lab 2: NMOS Transfer Characteristics

**File:** `day2_nfet_idvgs_L015_W039.spice`

```spice
*** Model Description ***
.param temp=27

*** Including sky130 library files ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Netlist Description ***
XM1 vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8
Vin in 0 1.8

*** Simulation Commands ***
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
plot -vdd#branch
.endc

.end
```

#### Run Simulation

```bash
ngspice day2_nfet_idvgs_L015_W039.spice
```

**Expected Output:** I_D vs V_GS transfer curve

---

### Lab 3: CMOS Inverter VTC

**File:** `day3_inv_vtc_Wp084_Wn036.spice`

```spice
*** Model Description ***
.param temp=27

*** Including sky130 library files ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Netlist Description ***
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8
Vin in 0 1.8

*** Simulation Commands ***
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
plot out vs in
.endc

.end
```

#### Run Simulation

```bash
ngspice day3_inv_vtc_Wp084_Wn036.spice
```

**Expected Output:** Voltage Transfer Characteristic (VTC) curve

---

### Lab 4: CMOS Inverter Transient Response

**File:** `day3_inv_tran_Wp084_Wn036.spice`

```spice
*** Model Description ***
.param temp=27

*** Including sky130 library files ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Netlist Description ***
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8
Vin in 0 PULSE(0 1.8 0 10p 10p 1n 2n)

*** Simulation Commands ***
.tran 10p 4n

.control
run
setplot tran1
plot out vs time in
.endc

.end
```

#### Run Simulation

```bash
ngspice day3_inv_tran_Wp084_Wn036.spice
```

**Expected Output:** Input and output waveforms showing propagation delay

---

##  Understanding Delay Tables (Liberty Format)

Delay tables (LUTs) store cell delay as a function of:
- **Input Slew (Transition Time)**
- **Output Load (Capacitance)**

### 2D LUT Example

```
Index_1: Input Slew (20ps, 40ps, 60ps, 80ps)
Index_2: Output Load (10fF, 30fF, 50fF, 70fF, 90fF, 110fF)

CBUF1 Delay Table:
        10fF  30fF  50fF  70fF  90fF  110fF
20ps  │ x1    x2    x3    x4    x5    x6
40ps  │ x7    x8    x9    x10   x11   x12
60ps  │ x13   x14   x15   x16   x17   x18
80ps  │ x19   x20   x21   x22   x23   x24
```

---

##  Common ngspice Commands

### Interactive Mode

```bash
# Launch ngspice
ngspice

# Load circuit file
ngspice 1 -> source circuit.spice

# Run simulation
ngspice 2 -> run

# List available plots
ngspice 3 -> display

# Select plot
ngspice 4 -> setplot dc1

# Plot signals
ngspice 5 -> plot vout
ngspice 6 -> plot -vdd#branch

# Save plot to file
ngspice 7 -> set hcopydevtype=postscript color
ngspice 8 -> hardcopy output.ps vout

# Exit
ngspice 9 -> quit
```

### Batch Mode

```bash
# Run simulation and exit
ngspice -b circuit.spice -o output.log

# Run with specific commands
ngspice -b circuit.spice << EOF
run
print all
quit
EOF
```

---

##  References

### Key Parameters

| Parameter | Description | Typical Value (Sky130) |
|-----------|-------------|------------------------|
| V_DD | Supply Voltage | 1.8V |
| V_th (NMOS) | Threshold Voltage | ~0.4V |
| V_th (PMOS) | Threshold Voltage | ~-0.4V |
| t_ox | Gate Oxide Thickness | ~4nm |
| μ_n | Electron Mobility | ~350 cm²/V·s |
| μ_p | Hole Mobility | ~100 cm²/V·s |
| L_min | Minimum Length | 150nm |

### CMOS Inverter Design

**Symmetric Inverter (V_M = V_DD/2):**
```
W_p / W_n = μ_n / μ_p ≈ 3.5

For Sky130:
If W_n = 0.36μm, then W_p ≈ 0.84μm - 1.0μm
```

---

##  Useful Links

- [Sky130 PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [ngspice Official Site](http://ngspice.sourceforge.net/)
- [Original Workshop Repository](https://github.com/kunalg123/sky130CircuitDesignWorkshop)

---

##  License

This material is provided for educational purposes. Please refer to the original repository for licensing information.

---

##  Support

For questions or issues:
1. Check the documentation in this README
2. Review SPICE syntax examples
3. Verify Sky130 model files are correctly loaded
4. Check ngspice version compatibility

---
