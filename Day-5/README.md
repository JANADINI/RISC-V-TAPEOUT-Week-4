# CMOS Power Supply and Device Variation Robustness

Guide to evaluating CMOS inverter robustness under supply and device variations using Sky130 PDK.

##  Table of Contents

- [Power Supply Variation](#power-supply-variation)
- [Sky130 Labs: Supply Variation](#sky130-labs-supply-variation)
- [Device Variation](#device-variation)
- [Sky130 Labs: Device Variation](#sky130-labs-device-variation)

---

##  Power Supply Variation

### Impact on CMOS Inverter

Power supply scaling affects:
- **Switching threshold (V<sub>m</sub>)** - moves toward V<sub>DD</sub>/2
- **Noise margins** - shrink proportionally with V<sub>DD</sub>
- **Robustness** - decreases at lower supply voltages

### Key Observations

```
As V_DD decreases:
  → V_m → V_DD/2 (more centered)
  → Noise margins shrink (~30-40% of V_DD)
  → Circuit becomes more noise-sensitive
```

**Example:**
| V<sub>DD</sub> | V<sub>m</sub> | NM<sub>L</sub> | NM<sub>H</sub> |
|---|---|---|---|
| 2.5V | ~1.25V | ~0.90V | ~0.90V |
| 1.8V | ~0.90V | ~0.60V | ~0.60V |
| 1.0V | ~0.50V | ~0.30V | ~0.30V |

---

##  Advantages of Low Supply Voltage

### 1. Energy Efficiency (Primary Benefit)
```
E_dynamic = C_L × V_DD²

V_DD: 2.5V → 0.5V (5× reduction)
Energy: 96% savings per switch!
```

### 2. Increased Gain
- At very low V<sub>DD</sub> (~0.5V): ~50% gain improvement
- Sharper transitions due to subthreshold operation

### 3. Reliability
- Lower electric field stress
- Better device lifetime
- Reduced hot carrier effects

---

##  Disadvantages of Low Supply Voltage

### 1. Reduced Noise Margins
```
V_DD = 0.5V → NM ≈ 0.12V (only 120mV margin!)
```

### 2. Performance Degradation
```
V_DD = 1.8V: t_p ≈ 50ps
V_DD = 0.5V: t_p ≈ 500ps (10× slower!)
```

### 3. Increased PVT Sensitivity
- Process variations more critical
- Temperature effects amplified

---

##  Sky130 Labs: Supply Variation

### Lab: Supply Voltage Sweep

**File:** `day5_inv_supplyvariation_Wp1_Wn036.spice`

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

.control
let powersupply = 1.8
alter Vdd = powersupply
let voltagesupplyvariation = 0
dowhile voltagesupplyvariation < 6
    dc Vin 0 1.8 0.01
    let powersupply = powersupply - 0.2
    alter Vdd = powersupply
    let voltagesupplyvariation = voltagesupplyvariation + 1
end

plot dc1.out vs in dc2.out vs in dc3.out vs in dc4.out vs in dc5.out vs in dc6.out vs in xlabel "input voltage(V)" ylabel "output voltage(V)" title "Inverter dc characteristics as a function of supply voltage"
.endc

.end
```

### Run Simulation

```bash
ngspice day5_inv_supplyvariation_Wp1_Wn036.spice
```

**Output:** 6 VTC curves for V<sub>DD</sub> = 1.8V, 1.6V, 1.4V, 1.2V, 1.0V, 0.8V

![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-5/Pictures/inv_devicevariation.png)
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-5/Pictures/inv_devicevariation_out-vs-in.png)
---

##  Device Variation

### Why Study Device Variations?

- **Design:** Uses nominal device parameters
- **Reality:** Fabrication introduces variations
- **Good News:** CMOS inverter relatively insensitive to variations

---

##  Sources of Device Variation

### 1. Etching Process Variation

**Over-Etching:**
```
L_actual < L_target
→ Shorter channel
→ Higher I_D
→ Faster switching
→ Lower V_th
```

**Under-Etching:**
```
L_actual > L_target
→ Longer channel
→ Lower I_D
→ Slower switching
→ Higher V_th
```

---

### 2. Oxide Thickness Variation

**Thinner Oxide (t<sub>ox</sub> ↓):**
```
C_ox ↑ → k_n ↑ → I_D ↑
→ Faster switching
→ Higher leakage
```

**Thicker Oxide (t<sub>ox</sub> ↑):**
```
C_ox ↓ → k_n ↓ → I_D ↓
→ Slower switching
→ Lower leakage
```

---

##  Transistor Strength Definitions

**Strong PMOS:**
- Lower R<sub>ON</sub>
- Achieved by: Wider W<sub>p</sub>, Shorter L<sub>p</sub>, Thinner oxide

**Weak NMOS:**
- Higher R<sub>ON</sub>
- Achieved by: Narrower W<sub>n</sub>, Longer L<sub>n</sub>, Thicker oxide

**Weak PMOS:**
- Higher R<sub>ON</sub>
- Achieved by: Narrower W<sub>p</sub>, Longer L<sub>p</sub>, Thicker oxide

**Strong NMOS:**
- Lower R<sub>ON</sub>
- Achieved by: Wider W<sub>n</sub>, Shorter L<sub>n</sub>, Thinner oxide

---

##  Sky130 Labs: Device Variation

### Lab: Extreme Device Skew Study

**Scenario:** Very strong PMOS, weak NMOS

**Device Specs:**
- PMOS: W = 7.0μm (very wide!)
- NMOS: W = 0.42μm
- W<sub>p</sub>/W<sub>n</sub> ≈ 16.7 (extreme!)

**File:** `day5_inv_devicevariation_wp7_wn042.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=7 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.42 l=0.15

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

### Run Simulation

```bash
ngspice day5_inv_devicevariation_wp7_wn042.spice
ngspice 1 -> plot out vs in
```

---

### Expected Results

**Switching Threshold:**
```
V_m ≈ 1.5-1.6V (very high!)
→ Strong PMOS pulls balance point toward V_DD
```

**Noise Margins:**
```
NM_L ≈ 1.4V (Excellent!)
NM_H ≈ 0.15V (Poor!)
→ Highly imbalanced
```

**Timing:**
```
t_pLH ≈ 20ps (very fast rise)
t_pHL ≈ 150ps (slow fall)
→ Imbalanced delays
```

---

##  Comparison Table

| Parameter | Balanced Design | Skewed Design (Lab) |
|-----------|----------------|---------------------|
| W<sub>p</sub>/W<sub>n</sub> | ~2.2 | ~16.7 |
| V<sub>m</sub> | ~1.15V | ~1.55V |
| NM<sub>L</sub> | ~0.70V | ~1.40V |
| NM<sub>H</sub> | ~0.65V | ~0.15V |
| t<sub>pLH</sub> | ~75ps | ~20ps |
| t<sub>pHL</sub> | ~75ps | ~150ps |
| **Balanced?** |  Yes |  No |

**Conclusion:** Extreme device variations shift V<sub>m</sub> and create imbalanced performance - not recommended for production designs.

---

##  Design Guidelines

### For Robust Operation:

**Supply Voltage:**
- Maintain V<sub>DD</sub> ≥ 0.8V for good noise margins
- V<sub>DD</sub> ≥ 1.0V for reliable digital operation

**Device Sizing:**
- Keep W<sub>p</sub>/W<sub>n</sub> ≈ 2-3 for balanced operation
- Verify across all process corners (TT, FF, SS, FS, SF)

**Noise Margins:**
- Target NM ≥ 30% of V<sub>DD</sub>
- Balance NM<sub>L</sub> ≈ NM<sub>H</sub>

---

##  Quick ngspice Commands

```bash
# Run simulation
ngspice filename.spice

# Plot VTC
ngspice 1 -> plot out vs in

# Measure V_m
ngspice 2 -> let diff = v(out) - v(in)
ngspice 3 -> meas dc Vm WHEN diff=0

# Calculate gain
ngspice 4 -> let gain = deriv(out)
ngspice 5 -> plot gain
```

---

##  References

- [Sky130 PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [Original Workshop Repository](https://github.com/kunalg123/sky130CircuitDesignWorkshop)

---


