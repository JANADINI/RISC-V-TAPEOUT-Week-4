# CMOS Noise Margin Robustness Evaluation

Complete guide to understanding and evaluating noise margins in CMOS inverters for robust digital circuit design using Sky130 PDK.

##  Table of Contents

- [Introduction to Noise Margin](#introduction-to-noise-margin)
- [Noise Margin Voltage Parameters](#noise-margin-voltage-parameters)
- [Noise Margin Equations and Summary](#noise-margin-equations-and-summary)
- [Noise Margin vs PMOS Width Variation](#noise-margin-vs-pmos-width-variation)
- [Sky130 Labs: Noise Margin Analysis](#sky130-labs-noise-margin-analysis)

---

##  Introduction to Noise Margin

### Definition

**Noise Margin** is the maximum noise voltage a CMOS circuit can tolerate without causing logic errors.

```
Noise Margin = Amount of noise a circuit can withstand
               without compromising correct operation
```

### Purpose

Noise margin ensures that:

 **Logic '1' Integrity:**
- Any signal representing logic '1' with added noise
- Is still recognized as logic '1'
- NOT misinterpreted as logic '0'

 **Logic '0' Integrity:**
- Any signal representing logic '0' with added noise
- Is still recognized as logic '0'
- NOT misinterpreted as logic '1'

---

##  Ideal vs Real Inverter Characteristics

### Ideal Inverter VTC

```
V_out
  |
  |    ┌────────────────
  |    │
V_DD  │
  |    │
  |    │ (Infinite slope)
  |    │
  |    │
  0  ──┴────────────────
      0   V_DD/2   V_DD    V_in
```

**Characteristics:**
- Abrupt switching at V<sub>DD</sub>/2
- Infinite slope (gain → ∞)
- Zero transition region
- **Theoretical only** - not physically realizable

---

### Real Inverter VTC

```
V_out
  |
  |  ┌─────┐
  |  │     │
V_DD │      ╲
  |  │       ╲ (Finite slope)
  |  │        ╲
  |  │         ╲
  |  │          ╲
  0  │           └─────
      0    V_m    V_DD    V_in
      ↑          ↑
     V_IL       V_IH
```

**Characteristics:**
- Gradual transition region (V<sub>IL</sub> to V<sub>IH</sub>)
- Finite slope (gain is large but finite)
- Defined noise margin regions
- **Physically realizable**

---

##  Noise Margin Voltage Parameters

### Critical Voltage Points

The Voltage Transfer Characteristic (VTC) defines four critical voltages:

#### 1. **V<sub>IL</sub>** - Input Low Threshold Voltage

```
Definition: Maximum input voltage still recognized as logic '0'

Characteristic: Point where dV_out/dV_in = -1 (unity gain)

Location: V_IL ≈ 10% to 30% of V_DD
```

**Physical Meaning:**
- Below V<sub>IL</sub>: Input definitely logic '0'
- Above V<sub>IL</sub>: Input starts entering undefined region

---

#### 2. **V<sub>IH</sub>** - Input High Threshold Voltage

```
Definition: Minimum input voltage recognized as logic '1'

Characteristic: Point where dV_out/dV_in = -1 (unity gain)

Location: V_IH ≈ 70% to 90% of V_DD
```

**Physical Meaning:**
- Above V<sub>IH</sub>: Input definitely logic '1'
- Below V<sub>IH</sub>: Input starts entering undefined region

---

#### 3. **V<sub>OL</sub>** - Output Low Voltage

```
Definition: Maximum output voltage in logic '0' state

Typical: V_OL ≈ 0V to 10% of V_DD

Result: When V_in > V_IH, V_out = V_OL
```

**Physical Meaning:**
- NMOS fully ON, PMOS OFF
- Output pulled down to near ground

---

#### 4. **V<sub>OH</sub>** - Output High Voltage

```
Definition: Minimum output voltage in logic '1' state

Typical: V_OH ≈ 90% to 100% of V_DD

Result: When V_in < V_IL, V_out = V_OH
```

**Physical Meaning:**
- PMOS fully ON, NMOS OFF
- Output pulled up to near V<sub>DD</sub>

---

### Slope = -1 Points

The points V<sub>IL</sub> and V<sub>IH</sub> are defined where:

```
dV_out / dV_in = -1
```

**Why slope = -1?**
- Maximum tolerance to noise
- Beyond these points, small input changes cause large output changes
- Defines the boundary of the undefined region

---

### VTC with Critical Points Marked

```
V_out
  │
  │  ●────────────────── V_OH (Output High)
  │  │
  │  │  ╲
  │  │   ╲ Slope = -1 at V_IL
  │  │    ╲
  │  │     ●─── Undefined
  │  │      ╲   Region
  │  │       ╲
  │  │        ● Slope = -1 at V_IH
  │  │         ╲
  │  │          ╲
  │  └───────────●────── V_OL (Output Low)
  │
  └──────────────────────────── V_in
     V_IL    V_m    V_IH
     
     ←─NM_L─→   ←─NM_H─→
```

---

##  Noise Margin Equations and Summary

### Noise Margin Definitions

#### High-Level Noise Margin (NM<sub>H</sub>)

```
┌────────────────────────────────┐
│  NM_H = V_OH - V_IH            │
│                                 │
│  Tolerance for noise on         │
│  logic '1' signal               │
└────────────────────────────────┘
```

**Interpretation:**
- Maximum noise that can be added to logic '1' input
- Signal still recognized as logic '1'
- Larger NM<sub>H</sub> = Better noise immunity for HIGH signals

---

#### Low-Level Noise Margin (NM<sub>L</sub>)

```
┌────────────────────────────────┐
│  NM_L = V_IL - V_OL            │
│                                 │
│  Tolerance for noise on         │
│  logic '0' signal               │
└────────────────────────────────┘
```

**Interpretation:**
- Maximum noise that can be added to logic '0' input
- Signal still recognized as logic '0'
- Larger NM<sub>L</sub> = Better noise immunity for LOW signals

---

### Overall Noise Margin

```
NM = min(NM_H, NM_L)
```

**Design Goal:** Maximize both NM<sub>H</sub> and NM<sub>L</sub> for robust operation

---

##  Noise Bump Scenarios

### Three Critical Cases

#### **Case (a): Noise on Logic '0'**

```
Input Signal: Logic '0' + Noise

Voltage Range: V_OL < V_in < V_IL

         ╱╲
        ╱  ╲  ← Noise bump
  ─────╱    ╲─────────
  V_OL        V_IL

Result: Still interpreted as logic '0'
Status:  SAFE (within NM_L)
```

**Explanation:**
- Noise adds positive voltage to logic '0'
- As long as total stays below V<sub>IL</sub>
- Signal remains in valid logic '0' region

---

#### **Case (b): Noise in Undefined Region**

```
Input Signal: Logic + Large Noise

Voltage Range: V_IL < V_in < V_IH

              ╱╲
             ╱  ╲  ← Large noise bump
  ──────────╱    ╲──────────
         V_IL    V_IH

Result: Output becomes UNDEFINED
Status:  UNSAFE (logic unstable)
```

**Explanation:**
- Noise pushes signal into transition region
- Inverter neither fully ON nor OFF
- Output unpredictable - can be metastable
- **Must be avoided in design**

---

#### **Case (c): Noise on Logic '1'**

```
Input Signal: Logic '1' + Noise (negative)

Voltage Range: V_IH < V_in < V_OH

             ╲  ╱
              ╲╱  ← Noise dip
  ────────────────────────
       V_IH        V_OH

Result: Still interpreted as logic '1'
Status:  SAFE (within NM_H)
```

**Explanation:**
- Noise reduces voltage from logic '1'
- As long as total stays above V<sub>IH</sub>
- Signal remains in valid logic '1' region

---

### Visual Summary of Valid Regions

```
     Valid        Undefined       Valid
    Logic '0'      Region        Logic '1'
  ┌──────────┬──────────────┬──────────┐
  │          │              │          │
  │   NM_L   │              │   NM_H   │
  │          │              │          │
  └──────────┴──────────────┴──────────┘
  0      V_IL  V_m    V_IH        V_DD
```

**Design Rule:**
```
For reliable operation:
  Signal must stay within:
    [0, V_IL] for logic '0'
  OR
    [V_IH, V_DD] for logic '1'
    
  Avoid:
    [V_IL, V_IH] - undefined region
```

---

##  Noise Margin vs PMOS Width Variation

### Experimental Setup

**Objective:** Study how W<sub>p</sub> affects noise margins

**Fixed Parameters:**
- V<sub>DD</sub> = 1.8V
- L<sub>n</sub> = L<sub>p</sub> = 0.15μm
- W<sub>n</sub> = 0.36μm (fixed)
- C<sub>L</sub> = 50fF

**Variable Parameter:**
- W<sub>p</sub> (PMOS width)

---

### Impact of W<sub>p</sub> on Voltage Parameters

#### Effect on V<sub>m</sub> (Switching Threshold)

```
As W_p increases:
  → V_m shifts HIGHER (toward V_DD)
  → PMOS becomes stronger
  → Balance point moves up
```

#### Effect on V<sub>IL</sub> and V<sub>IH</sub>

```
As W_p increases:
  → V_IL increases slightly
  → V_IH increases more significantly
  → Transition region widens and shifts right
```

#### Effect on V<sub>OL</sub> and V<sub>OH</sub>

```
V_OL ≈ constant (≈ 0V)
  → Determined by NMOS pull-down strength
  → W_n is fixed
  
V_OH ≈ constant (≈ V_DD)
  → Already at rail voltage
  → Cannot go higher than V_DD
```

---

### Noise Margin Variation Results

| W<sub>p</sub> (μm) | W<sub>p</sub>/W<sub>n</sub> | V<sub>m</sub> (V) | V<sub>IL</sub> (V) | V<sub>IH</sub> (V) | NM<sub>L</sub> (V) | NM<sub>H</sub> (V) | Remarks |
|-------|-----------|-------|---------|---------|---------|---------|---------|
| 0.42 | 1.17 | ~0.90 | ~0.65 | ~1.10 | **0.65** | **0.70** | Smaller W<sub>p</sub> |
| 0.84 | 2.33 | ~1.20 | ~0.70 | ~1.25 | **0.70** | **0.55** | Balanced |
| 1.00 | 2.78 | ~1.30 | ~0.72 | ~1.35 | **0.72** | **0.45** | Larger W<sub>p</sub> |
| 1.50 | 4.17 | ~1.45 | ~0.75 | ~1.50 | **0.75** | **0.30** | Very large W<sub>p</sub> |

**Assumptions for table:** V<sub>OL</sub> ≈ 0V, V<sub>OH</sub> ≈ 1.8V

---

### Key Observations

#### 1. **NM<sub>L</sub> Behavior**

```
NM_L = V_IL - V_OL

As W_p increases:
  → V_IL increases slightly
  → V_OL stays ≈ 0V
  → NM_L increases slightly

Conclusion: Larger PMOS → Better low-level noise margin
```

---

#### 2. **NM<sub>H</sub> Behavior**

```
NM_H = V_OH - V_IH

As W_p increases:
  → V_IH increases significantly
  → V_OH stays ≈ V_DD
  → NM_H decreases

Conclusion: Larger PMOS → Worse high-level noise margin
```

---

#### 3. **Trade-off**

```
┌─────────────────┬────────────┬────────────┐
│   W_p/W_n       │    NM_L    │    NM_H    │
├─────────────────┼────────────┼────────────┤
│   Small (~1)    │    Low     │    High    │
│   Medium (~2-3) │   Medium   │   Medium   │
│   Large (>4)    │    High    │    Low     │
└─────────────────┴────────────┴────────────┘
```

**Sweet Spot:** W<sub>p</sub>/W<sub>n</sub> ≈ 2-3 for balanced noise margins

---

#### 4. **Overall Noise Margin**

```
NM = min(NM_L, NM_H)

Optimal: When NM_L ≈ NM_H
  → W_p/W_n ≈ 2 to 2.5
  → Both margins ≈ 0.55-0.70V
  → Best overall robustness
```

---

### Design Implications

#### For W<sub>p</sub>/W<sub>n</sub> = 1 (Equal Sizing)
```
 Advantages:
  - Good NM_H (high-level noise immunity)
  - Smaller area

 Disadvantages:
  - Poor NM_L (low-level noise immunity)
  - Imbalanced delays
  - V_m < V_DD/2
```

---

#### For W<sub>p</sub>/W<sub>n</sub> = 2-3 (Recommended)
```
 Advantages:
  - Balanced NM_L and NM_H
  - Good overall noise margin
  - Balanced rise/fall delays
  - V_m ≈ V_DD/2

 Disadvantages:
  - Moderate area overhead
```

---

#### For W<sub>p</sub>/W<sub>n</sub> > 4 (Large PMOS)
```
 Advantages:
  - Excellent NM_L
  - Fast rise time

 Disadvantages:
  - Poor NM_H
  - Large area
  - V_m > V_DD/2
  - Imbalanced delays
```

---

##  Sky130 Labs: Noise Margin Analysis

### Lab Setup

Clone repository and navigate to design files:

```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

---

### Lab: CMOS Inverter Noise Margin Characterization

**Objective:** Extract V<sub>IL</sub>, V<sub>IH</sub>, V<sub>OL</sub>, V<sub>OH</sub> and calculate noise margins

**Device Specifications:**
- PMOS: W = 1.0μm, L = 0.15μm
- NMOS: W = 0.36μm, L = 0.15μm
- W<sub>p</sub>/W<sub>n</sub> = 2.78
- Load: C<sub>L</sub> = 50fF

**File:** `day4_inv_noisemargin_wp1_wn036.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=1 l=0.15
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

---

### Run Simulation

```bash
# Start ngspice and plot VTC
ngspice day4_inv_noisemargin_wp1_wn036.spice

# In ngspice prompt
ngspice 1 -> plot out vs in
```

**Expected Output:** VTC curve with clear transition region

![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-4/Pictures/inv_noisemargin.png)
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-4/Pictures/inv_noisemargin_out-vs-in.png)
---

### Extracting Noise Margin Parameters

#### Method 1: Graphical Method

**Step 1:** Plot VTC
```bash
ngspice 1 -> plot out vs in
```

**Step 2:** Identify V<sub>OH</sub> and V<sub>OL</sub>
```
V_OH: Output voltage when V_in = 0V (left side of curve)
V_OL: Output voltage when V_in = V_DD (right side of curve)
```

**Step 3:** Find slope = -1 points (V<sub>IL</sub> and V<sub>IH</sub>)
- Look for steepest part of transition
- V<sub>IL</sub>: Lower slope = -1 point
- V<sub>IH</sub>: Upper slope = -1 point

**Step 4:** Calculate noise margins
```
NM_L = V_IL - V_OL
NM_H = V_OH - V_IH
```

---

#### Method 2: Analytical Method (Derivative)

Calculate gain (dV<sub>out</sub>/dV<sub>in</sub>) and find where gain = -1:

```bash
# In ngspice
ngspice 2 -> let gain = deriv(out)
ngspice 3 -> plot gain vs in

# Find where gain = -1
ngspice 4 -> print out when gain=-1
```

This gives exact V<sub>IL</sub> and V<sub>IH</sub> values.

---

#### Method 3: SPICE Measurement Commands

Add measurement commands to SPICE file:

```spice
.control
run
setplot dc1

* Measure V_OH (output when input = 0)
meas dc VOH FIND v(out) WHEN v(in)=0

* Measure V_OL (output when input = V_DD)
meas dc VOL FIND v(out) WHEN v(in)=1.8

* Find V_IL (where gain = -1 on left side)
let gain = deriv(out)
meas dc VIL WHEN gain=-1 CROSS=1

* Find V_IH (where gain = -1 on right side)
meas dc VIH WHEN gain=-1 CROSS=2

* Calculate noise margins
let NML = VIL - VOL
let NMH = VOH - VIH

print VOH VOL VIL VIH NML NMH
.endc
```

---

### Expected Results (Typical for W<sub>p</sub>=1.0μm, W<sub>n</sub>=0.36μm)

```
V_OH  ≈ 1.78V to 1.80V
V_OL  ≈ 0.00V to 0.05V
V_IL  ≈ 0.70V to 0.75V
V_IH  ≈ 1.30V to 1.35V

NM_L  ≈ 0.70V (Good)
NM_H  ≈ 0.45V (Acceptable)

V_m   ≈ 1.20V to 1.30V
```

---

### Analysis Questions

After running the simulation, consider:

**Q1:** Is NM<sub>L</sub> = NM<sub>H</sub>?
```
No, NM_L > NM_H for this design
Reason: W_p/W_n = 2.78 (PMOS stronger than symmetric)
```

**Q2:** What would improve NM<sub>H</sub>?
```
Reduce W_p or increase W_n
Trade-off: NM_L would decrease
```

**Q3:** Is this design robust?
```
Yes, both NM_L and NM_H > 0.4V
Acceptable for 1.8V operation
```

**Q4:** What is the undefined region width?
```
Undefined Region = V_IH - V_IL
≈ 1.30V - 0.72V = 0.58V
≈ 32% of V_DD (acceptable)
```

---

##  Comparison Study: Different W<sub>p</sub> Values

### Complete Comparison Table

| Design | W<sub>p</sub> (μm) | W<sub>n</sub> (μm) | W<sub>p</sub>/W<sub>n</sub> | V<sub>IL</sub> (V) | V<sub>IH</sub> (V) | NM<sub>L</sub> (V) | NM<sub>H</sub> (V) | NM (V) | Best For |
|--------|-------|-------|-----------|---------|---------|---------|---------|-----|----------|
| 1 | 0.36 | 0.36 | 1.0 | 0.65 | 1.10 | 0.65 | **0.70** | 0.65 | High NM<sub>H</sub> |
| 2 | 0.72 | 0.36 | 2.0 | 0.70 | 1.25 | **0.70** | 0.55 | **0.55** | **Balanced** |
| 3 | 1.00 | 0.36 | 2.78 | 0.72 | 1.35 | **0.72** | 0.45 | 0.45 | High NM<sub>L</sub> |
| 4 | 1.50 | 0.36 | 4.17 | 0.75 | 1.50 | **0.75** | 0.30 | 0.30 | Area-heavy |

**Recommendation:** Design 2 (W<sub>p</sub> = 0.72μm) offers best overall noise immunity.

---

##  Design Guidelines for Robust Inverters

### Target Specifications

```
For 1.8V CMOS:
  NM_L ≥ 0.5V (≥ 28% of V_DD)
  NM_H ≥ 0.5V (≥ 28% of V_DD)
  NM_L ≈ NM_H (balanced)
```

### Sizing Recommendations

**For symmetric noise margins:**
```
W_p/W_n ≈ 2 to 2.5

Typical:
  W_n = 0.36μm (minimum)
  W_p = 0.72 to 0.90μm
  L_n = L_p = 0.15μm (minimum)
```

### Trade-off Summary

```
┌──────────────┬────────────────┬───────────────┐
│  Optimize    │   Approach     │   Trade-off   │
├──────────────┼────────────────┼───────────────┤
│  NM_L        │  Increase W_p  │  NM_H drops   │
│  NM_H        │  Decrease W_p  │  NM_L drops   │
│  Balanced    │  W_p ≈ 2W_n    │  Best overall │
│  Area        │  Min W_p, W_n  │  Worse NM     │
│  Speed       │  Max W_p, W_n  │  More area    │
└──────────────┴────────────────┴───────────────┘
```

---

##  Quick Reference

### Key Equations

```
NM_L = V_IL - V_OL
NM_H = V_OH - V_IH
NM = min(NM_L, NM_H)
```

### Critical Points Definition

```
V_IL: Point where dV_out/dV_in = -1 (lower)
V_IH: Point where dV_out/dV_in = -1 (upper)
V_OL: Output when V_in = V_DD
V_OH: Output when V_in = 0
```

### Typical Values (Sky130, 1.8V)

```
V_OH ≈ 1.75-1.80V
V_OL ≈ 0.00-0.05V
V_IL ≈ 0.65-0.75V (depends on W_p)
V_IH ≈ 1.10-1.40V (depends on W_p)
```

---

##  ngspice Commands for Noise Margin

```bash
# Basic VTC plot
ngspice> plot out vs in

# Calculate gain
ngspice> let gain = deriv(out)
ngspice> plot gain vs in

# Find slope = -1 points
ngspice> print in when gain=-1

# Measure specific voltages
ngspice> meas dc VOH FIND v(out) WHEN v(in)=0
ngspice> meas dc VOL FIND v(out) WHEN v(in)=1.8

# Export data for analysis
ngspice> print all > vtc_data.txt
```

---

##  References

- [Sky130 PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [ngspice Manual](http://ngspice.sourceforge.net/docs.html)
- [Original Workshop Repository](https://github.com/kunalg123/sky130CircuitDesignWorkshop)

---

