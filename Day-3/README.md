# CMOS Switching Threshold and Dynamic Simulations

Complete guide to CMOS inverter VTC analysis, switching threshold derivation, and dynamic characterization using Sky130 PDK.

##  Table of Contents

- [Voltage Transfer Characteristics (VTC) Simulations](#voltage-transfer-characteristics-vtc-simulations)
- [Sky130 Labs: VTC and Transient Analysis](#sky130-labs-vtc-and-transient-analysis)
- [Static Behavior Evaluation](#static-behavior-evaluation)
- [Switching Threshold (Vm) Analysis](#switching-threshold-vm-analysis)
- [Analytical Derivations](#analytical-derivations)
- [Static & Dynamic Simulation Studies](#static--dynamic-simulation-studies)
- [Applications in Clock Networks](#applications-in-clock-networks)

---

##  Voltage Transfer Characteristics (VTC) Simulations

### SPICE Deck Creation for CMOS Inverter

A complete SPICE deck for CMOS inverter simulation must include:

#### 1. **Component Connectivity**
- Define how devices interconnect node-to-node
- Every transistor terminal (drain/gate/source/bulk) must reference a named node
- Specify all component connections clearly

#### 2. **Component Values**
- **Transistor dimensions:** W/L ratios for NMOS and PMOS
- **Load capacitance:** Output load (typically 10-100 fF)
- **Supply voltages:** V<sub>DD</sub> and V<sub>SS</sub> levels
- **Input stimulus:** DC sweep or transient pulse

#### 3. **Identify Nodes**
- Assign each electrical junction a unique node name
- Common nodes: `in`, `out`, `vdd`, `0` (ground)
- Ground is typically node `0`

#### 4. **Name Nodes Consistently**
- Use human-readable, descriptive names
- Examples: `Vin`, `Vout`, `Vdd`, `Vss`
- Simplifies debugging and post-processing

---

### CMOS Inverter SPICE Template

```spice
*** CMOS Inverter VTC Analysis ***

*** Model Description ***
.param temp=27

*** Technology Library ***
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*** Netlist Description ***
* PMOS transistor
XM1 out in vdd vdd <pmos_model> w=<Wp> l=<Lp>

* NMOS transistor
XM2 out in 0 0 <nmos_model> w=<Wn> l=<Ln>

* Load capacitance
Cload out 0 <C_value>

*** Supply and Input ***
Vdd vdd 0 1.8V
Vin in 0 1.8V

*** Simulation Commands ***
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
plot out vs in
.endc

.end
```

---

##  Sky130 Labs: VTC and Transient Analysis

### Lab Setup

Clone repository and navigate to design files:

```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

---

### Lab 1: CMOS Inverter VTC (DC Analysis)

**Objective:** Generate Voltage Transfer Characteristic curve (V<sub>out</sub> vs V<sub>in</sub>)

**Device Specifications:**
- PMOS: W = 0.84μm, L = 0.15μm
- NMOS: W = 0.36μm, L = 0.15μm
- Load: C<sub>L</sub> = 50fF

**File:** `day3_inv_vtc_Wp084_Wn036.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.01

.control
run
setplot dc1
display
.endc

.end
```

#### Run Simulation

```bash
# Start ngspice and plot VTC
ngspice day3_inv_vtc_Wp084_Wn036.spice

# In ngspice prompt
ngspice 1 -> plot out vs in
```

**Expected Output:** VTC curve showing:
- Logic HIGH output (≈1.8V) at low input
- Sharp transition region
- Logic LOW output (≈0V) at high input

**Key Parameters to Extract:**
- **V<sub>IL</sub>** - Maximum input LOW voltage
- **V<sub>IH</sub>** - Minimum input HIGH voltage
- **V<sub>OL</sub>** - Maximum output LOW voltage
- **V<sub>OH</sub>** - Minimum output HIGH voltage
- **V<sub>m</sub>** - Switching threshold (V<sub>in</sub> = V<sub>out</sub>)

---

### Lab 2: CMOS Inverter Transient Analysis

**Objective:** Measure propagation delays and rise/fall times

**Input Stimulus:**
- Pulse waveform: 0V → 1.8V
- Rise/Fall time: 0.1ns
- Pulse width: 2ns
- Period: 4ns

**File:** `day3_inv_tran_Wp084_Wn036.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15

Cload out 0 50fF

Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

*simulation commands
.tran 1n 10n

.control
run
.endc

.end
```

#### Run Simulation

```bash
# Start ngspice and plot transient response
ngspice day3_inv_tran_Wp084_Wn036.spice

# In ngspice prompt
ngspice 1 -> plot out vs time in
```

**Expected Output:** Waveforms showing input and output with propagation delays

---

### Delay Calculations

#### Rise Time (t<sub>r</sub>)
```
t_r = Time taken for output to go from 20% to 80% of V_DD
```

#### Fall Time (t<sub>f</sub>)
```
t_f = Time taken for output to go from 80% to 20% of V_DD
```

#### Propagation Delay (Rising Edge)
```
t_pLH = Time at 50% of output rising edge - Time at 50% of input falling edge
```

#### Propagation Delay (Falling Edge)
```
t_pHL = Time at 50% of output falling edge - Time at 50% of input rising edge
```

#### Average Propagation Delay
```
t_p = (t_pLH + t_pHL) / 2
```

---

##  Static Behavior Evaluation

### Key Parameters Defining CMOS Inverter Robustness

| Parameter | Description | Impact |
|-----------|-------------|--------|
| **Switching Threshold (V<sub>m</sub>)** | V<sub>in</sub> = V<sub>out</sub> point | Determines noise margins |
| **Noise Margin** | Tolerance to input noise | Circuit reliability |
| **Power Supply Variation** | Sensitivity to V<sub>DD</sub> changes | Process robustness |
| **Device Variations** | Impact of W/L mismatch | Manufacturing yield |

---

##  Switching Threshold (V<sub>m</sub>) Analysis

### Definition

**Switching Threshold Voltage (V<sub>m</sub>):**
- The input voltage at which V<sub>in</sub> = V<sub>out</sub>
- At V<sub>m</sub>, both NMOS and PMOS are in **saturation region**
- Both transistors are **ON**, providing **maximum gain**

### Physical Significance

At V<sub>m</sub>:
```
V_in = V_out = V_m
Both transistors in saturation
I_D,n = I_D,p
dV_out/dV_in is maximum (high gain)
```

---

### Effect of Device Sizing on V<sub>m</sub>

#### Case 1: Equal Sizing (W<sub>p</sub>/L<sub>p</sub> = W<sub>n</sub>/L<sub>n</sub>)

**Device Dimensions:**
- W<sub>n</sub>/L<sub>n</sub> = 0.375μm / 0.25μm
- W<sub>p</sub>/L<sub>p</sub> = 0.375μm / 0.25μm
- Ratio: W<sub>p</sub>/L<sub>p</sub> = 1 × W<sub>n</sub>/L<sub>n</sub>

**Result:**
- V<sub>m</sub> ≈ **0.98V**
- Below V<sub>DD</sub>/2 (0.9V) due to mobility difference
- NMOS stronger than PMOS (higher μ<sub>n</sub>)

**Why V<sub>m</sub> < V<sub>DD</sub>/2?**
```
μ_n ≈ 3.5 × μ_p
For equal W/L, NMOS is stronger
Need higher input voltage to balance
V_m shifts below V_DD/2
```

---

#### Case 2: Increased PMOS Width

**Device Dimensions:**
- W<sub>n</sub>/L<sub>n</sub> = 0.375μm / 0.25μm
- W<sub>p</sub>/L<sub>p</sub> = 0.9375μm / 0.25μm
- Ratio: W<sub>p</sub>/L<sub>p</sub> = **2.5** × W<sub>n</sub>/L<sub>n</sub>

**Result:**
- V<sub>m</sub> ≈ **1.20V**
- Above V<sub>DD</sub>/2 (0.9V)
- PMOS pull-up strengthened

**Why V<sub>m</sub> > V<sub>DD</sub>/2?**
```
Wider PMOS → Stronger pull-up
Balance point shifts higher
Need higher V_in to turn ON NMOS strongly
V_m shifts above V_DD/2
```

---

### VTC Regions and Transistor Operation

```
V_out
  |
  |   Region 1: PMOS Linear, NMOS OFF
  |   ●─────────
  |            ╲
  |             ╲ Region 2: PMOS Linear, NMOS Sat
  |              ╲
  |               ● V_m: BOTH in Saturation
  |                ╲
  |                 ╲ Region 4: PMOS Sat, NMOS Linear
  |                  ╲
  |                   ●─────── Region 5: PMOS OFF, NMOS Linear
  └────────────────────────────── V_in
```

**Five Operating Regions:**

| Region | V<sub>in</sub> Range | PMOS State | NMOS State | V<sub>out</sub> |
|--------|---------|------------|------------|------|
| **1** | 0 to V<sub>th,n</sub> | Linear | OFF | ≈ V<sub>DD</sub> |
| **2** | V<sub>th,n</sub> to V<sub>m</sub> | Linear | Saturation | Falling |
| **3** | ≈ V<sub>m</sub> | **Saturation** | **Saturation** | = V<sub>m</sub> |
| **4** | V<sub>m</sub> to (V<sub>DD</sub>-\|V<sub>th,p</sub>\|) | Saturation | Linear | Falling |
| **5** | (V<sub>DD</sub>-\|V<sub>th,p</sub>\|) to V<sub>DD</sub> | OFF | Linear | ≈ 0V |

**Region 3 (V<sub>m</sub>)** is the most critical for:
- Maximum gain
- Noise margin calculations
- Design optimization

---

##  Analytical Derivations

### Deriving V<sub>m</sub> as a Function of (W/L) Ratios

#### Step 1: Current Equality Condition

At V<sub>m</sub>, both transistors are in saturation and carry equal current:

```
I_D,n = I_D,p
```

#### Step 2: Saturation Current Equations

**NMOS in Saturation:**
```
I_D,n = (k_n/2)(V_GS,n - V_th,n)²
      = (k_n/2)(V_m - V_th,n)²
```

**PMOS in Saturation:**
```
I_D,p = (k_p/2)(V_GS,p - V_th,p)²
      = (k_p/2)(V_m - V_DD - V_th,p)²
      = (k_p/2)(V_DD - V_m + V_th,p)²
```

Where:
```
k_n = μ_n × C_ox × (W/L)_n
k_p = μ_p × C_ox × (W/L)_p
```

#### Step 3: Equate Currents

```
(k_n/2)(V_m - V_th,n)² = (k_p/2)(V_DD - V_m + V_th,p)²
```

#### Step 4: Simplify

```
k_n(V_m - V_th,n)² = k_p(V_DD - V_m + V_th,p)²
```

Let **r = √(k_p/k_n)**:

```
(V_m - V_th,n) = r(V_DD - V_m + V_th,p)
```

#### Step 5: Solve for V<sub>m</sub>

```
V_m - V_th,n = r × V_DD - r × V_m + r × V_th,p

V_m + r × V_m = r × V_DD + V_th,n + r × V_th,p

V_m(1 + r) = r × V_DD + V_th,n + r × V_th,p
```

**Final Expression:**

```
┌─────────────────────────────────────────────┐
│  V_m = (r × V_DD + V_th,n + r × V_th,p)    │
│        ─────────────────────────────────     │
│               (1 + r)                        │
│                                              │
│  where: r = √(k_p/k_n)                      │
│           = √[(W/L)_p × μ_p / (W/L)_n × μ_n]│
└─────────────────────────────────────────────┘
```

---

### Deriving (W/L) Ratios as a Function of V<sub>m</sub>

Starting from the V<sub>m</sub> equation, we can solve for the required W/L ratios to achieve a desired V<sub>m</sub>.

#### Step 1: Start with V<sub>m</sub> Expression

```
V_m = (r × V_DD + V_th,n + r × V_th,p) / (1 + r)

where: r = √(k_p/k_n)
```

#### Step 2: Rearrange for r

```
V_m(1 + r) = r × V_DD + V_th,n + r × V_th,p

V_m + V_m × r = r × V_DD + V_th,n + r × V_th,p

V_m × r - r × V_DD - r × V_th,p = V_th,n - V_m

r(V_m - V_DD - V_th,p) = V_th,n - V_m
```

#### Step 3: Solve for r

```
r = (V_th,n - V_m) / (V_m - V_DD - V_th,p)

  = (V_m - V_th,n) / (V_DD - V_m + V_th,p)
```

#### Step 4: Express k<sub>p</sub>/k<sub>n</sub>

```
r² = k_p/k_n = [(W/L)_p × μ_p] / [(W/L)_n × μ_n]
```

#### Step 5: Solve for (W/L)<sub>p</sub>

**Final Expression:**

```
┌─────────────────────────────────────────────────┐
│  (W/L)_p     (V_m - V_th,n)²      μ_n          │
│  ────────  = ────────────────  ×  ────          │
│  (W/L)_n     (V_DD - V_m + V_th,p)²   μ_p      │
└─────────────────────────────────────────────────┘
```

**For a given V<sub>m</sub>, this equation determines the required PMOS-to-NMOS sizing ratio.**

---

### Special Cases

#### Case 1: Symmetric Inverter (V<sub>m</sub> = V<sub>DD</sub>/2)

For V<sub>m</sub> = V<sub>DD</sub>/2 and assuming \|V<sub>th,n</sub>\| = \|V<sub>th,p</sub>\|:

```
(W/L)_p = (μ_n / μ_p) × (W/L)_n

For typical CMOS:
μ_n / μ_p ≈ 3 to 3.5

Therefore:
(W/L)_p ≈ (3 to 3.5) × (W/L)_n
```

#### Case 2: V<sub>m</sub> > V<sub>DD</sub>/2

Requires **stronger PMOS** (larger W<sub>p</sub>):
```
(W/L)_p > (μ_n/μ_p) × (W/L)_n
```

#### Case 3: V<sub>m</sub> < V<sub>DD</sub>/2

Requires **stronger NMOS** (larger W<sub>n</sub>):
```
(W/L)_p < (μ_n/μ_p) × (W/L)_n
```

---

##  Static & Dynamic Simulation Studies

### Experiment Overview

**Objective:** Study the effect of W<sub>p</sub>/W<sub>n</sub> ratio on:
- Rise delay (t<sub>pLH</sub>)
- Fall delay (t<sub>pHL</sub>)
- Switching threshold (V<sub>m</sub>)

**Fixed Parameters:**
- V<sub>DD</sub> = 1.8V
- L<sub>n</sub> = L<sub>p</sub> = 0.15μm
- C<sub>L</sub> = 50fF
- Temperature = 27°C

**Variable Parameter:**
- W<sub>p</sub>/W<sub>n</sub> ratio

---

### Experimental Results

| W<sub>n</sub> (μm) | W<sub>p</sub> (μm) | W<sub>p</sub>/W<sub>n</sub> | V<sub>m</sub> (V) | t<sub>pHL</sub> (ps) | t<sub>pLH</sub> (ps) | Remarks |
|-------|-------|-----------|-------|-----------|-----------|---------|
| 0.36 | 0.36 | 1.0 | ~0.95 | ~60 | ~130 | **NMOS strong** |
| 0.36 | 0.72 | 2.0 | ~1.15 | ~70 | ~80 | **Balanced delays** |
| 0.36 | 0.84 | 2.33 | ~1.20 | ~75 | ~75 | **Nearly balanced** |
| 0.36 | 1.08 | 3.0 | ~1.30 | ~90 | ~60 | **PMOS strong** |

---

### Key Observations

#### 1. **Impact on Switching Threshold (V<sub>m</sub>)**

```
As W_p/W_n increases → V_m increases

W_p/W_n = 1.0  → V_m ≈ 0.95V  (below V_DD/2)
W_p/W_n = 2.0  → V_m ≈ 1.15V  (above V_DD/2)
W_p/W_n = 3.0  → V_m ≈ 1.30V  (well above V_DD/2)
```

**Reason:** Stronger PMOS shifts balance point higher

---

#### 2. **Impact on Fall Delay (t<sub>pHL</sub>)**

```
As W_p/W_n increases → t_pHL increases

Controlled by NMOS strength
W_n fixed → t_pHL relatively stable but increases slightly
```

**Why?** PMOS acts as load during falling transition

---

#### 3. **Impact on Rise Delay (t<sub>pLH</sub>)**

```
As W_p/W_n increases → t_pLH decreases

Controlled by PMOS strength
Larger W_p → Faster pull-up → Lower t_pLH
```

**Why?** Stronger PMOS charges C<sub>L</sub> faster

---

#### 4. **Balanced Delays**

**Optimal Point:** W<sub>p</sub>/W<sub>n</sub> ≈ 2.0 to 2.5

```
W_p ≈ 0.72 to 0.84 μm
W_n = 0.36 μm
W_p/W_n ≈ 2.0 to 2.33

Result:
t_pHL ≈ t_pLH ≈ 75-80 ps
V_m ≈ 1.15-1.20V
```

**This configuration provides:**
- 🔸 Symmetric switching
- 🔸 No duty cycle distortion
- 🔸 Predictable timing

---

### Design Trade-offs

```
┌────────────────┬───────────────┬─────────────────┐
│   Parameter    │  Small W_p/W_n│  Large W_p/W_n  │
├────────────────┼───────────────┼─────────────────┤
│ V_m            │ Low (<V_DD/2) │ High (>V_DD/2)  │
│ t_pHL          │ Fast          │ Slow            │
│ t_pLH          │ Slow          │ Fast            │
│ Symmetry       │ Poor          │ Poor            │
│ Area           │ Small         │ Large           │
└────────────────┴───────────────┴─────────────────┘
```

**Sweet Spot:** W<sub>p</sub>/W<sub>n</sub> ≈ 2-2.5 for balanced performance

---

##  Applications in Clock Networks

### Clock Tree Synthesis (CTS) Requirements

Clock buffers must maintain:
1. **Balanced rise/fall times** → Prevent duty cycle distortion
2. **Low skew** → Synchronize flip-flops
3. **Sufficient drive strength** → Drive large capacitive loads

---

### Duty Cycle Distortion

#### Cause
```
If t_pLH ≠ t_pHL:
  → Output pulse width ≠ Input pulse width
  → Duty cycle distortion accumulates through clock tree
```

#### Example

**Input:** 50% duty cycle (2ns HIGH, 2ns LOW)

**With Imbalanced Delays:**
```
If t_pLH = 100ps, t_pHL = 50ps:

Rising edge delayed by 100ps
Falling edge delayed by 50ps
→ Output HIGH for 2.05ns, LOW for 1.95ns
→ Duty cycle = 51.25% (distorted!)
```

**After multiple stages:** Distortion compounds!

---

### Solution: Balanced Inverter Design

#### Target Specifications
```
t_pLH ≈ t_pHL  (within 5-10%)
V_m ≈ V_DD/2   (symmetric switching)
```

#### Implementation

**For Sky130 (1.8V):**
```
W_n = 0.36 μm
W_p = 0.72 to 0.84 μm
W_p/W_n ≈ 2.0 to 2.33

Result:
t_pLH ≈ 75-80 ps
t_pHL ≈ 75-80 ps
Duty cycle preserved!
```

---

### Duty Cycle Correction Circuits

If delays cannot be matched through sizing, use **DCC circuits**:

#### 1. **Analog DCC**
- Adjusts buffer strength dynamically
- Feedback loop monitors duty cycle
- Corrects in real-time

#### 2. **Digital DCC**
- Delay line compensation
- Phase interpolation
- Used in high-speed interfaces

#### 3. **Placement in Clock Tree**

```
PLL → DCC → Clock Tree → Buffers → Flip-Flops

DCC ensures 50% duty cycle before distribution
```

---

### Static Timing Analysis (STA) Considerations

#### Setup Time Check
```
T_clk ≥ t_pHL(buffer) + t_setup(FF) + t_skew
```

#### Hold Time Check
```
t_pLH(buffer) ≥ t_hold(FF) + t_skew
```

**If t<sub>pLH</sub> ≠ t<sub>pHL</sub>:**
- Setup and hold margins become asymmetric
- Timing closure more difficult
- May need different constraints for rising/falling edges

---

##  Quick Reference

### Key Equations

#### Switching Threshold
```
V_m = (r × V_DD + V_th,n + r × V_th,p) / (1 + r)

where: r = √[(W/L)_p × μ_p / (W/L)_n × μ_n]
```

#### Sizing for Desired V<sub>m</sub>
```
(W/L)_p / (W/L)_n = [(V_m - V_th,n) / (V_DD - V_m + V_th,p)]² × (μ_n / μ_p)
```

#### Propagation Delays
```
t_pHL = f(R_ON,n, C_L)  → Depends on NMOS strength
t_pLH = f(R_ON,p, C_L)  → Depends on PMOS strength
```

---

### Design Guidelines

| Goal | Recommendation |
|------|----------------|
| **Symmetric V<sub>m</sub>** | W<sub>p</sub>/W<sub>n</sub> ≈ μ<sub>n</sub>/μ<sub>p</sub> ≈ 3-3.5 |
| **Balanced Delays** | W<sub>p</sub>/W<sub>n</sub> ≈ 2-2.5 |
| **Low Area** | Minimize W<sub>p</sub>, accept asymmetry |
| **Low Power** | Reduce V<sub>DD</sub>, optimize W/L |
| **High Speed** | Increase W/L, reduce C<sub>L</sub> |
| **Clock Buffers** | Balance delays, use DCC if needed |

---

##  Common ngspice Commands

### VTC Analysis
```bash
# Load and run DC sweep
ngspice filename.spice
ngspice 1 -> run
ngspice 2 -> plot out vs in

# Measure V_m (where V_in = V_out)
ngspice 3 -> print all > results.txt
```

### Transient Analysis
```bash
# Load and run transient simulation
ngspice filename.spice
ngspice 1 -> run
ngspice 2 -> plot out vs time in

# Measure delays
ngspice 3 -> meas tran tphl TRIG v(in) VAL=0.9 FALL=1 TARG v(out) VAL=0.9 FALL=1
ngspice 4 -> meas tran tplh TRIG v(in) VAL=0.9 RISE=1 TARG v(out) VAL=0.9 RISE=1
```

---

##  References

- [Sky130 PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [ngspice Manual](http://ngspice.sourceforge.net/docs.html)
- [Original Workshop Repository](https://github.com/kunalg123/sky130CircuitDesignWorkshop)

---


