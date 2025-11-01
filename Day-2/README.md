# Velocity Saturation and CMOS Inverter VTC

Complete guide to short-channel effects, velocity saturation, and CMOS inverter voltage transfer characteristics using Sky130 PDK.

##  Table of Contents

- [SPICE Simulation for Lower Nodes](#spice-simulation-for-lower-nodes)
- [Long Channel vs Short Channel Comparison](#long-channel-vs-short-channel-comparison)
- [Velocity Saturation Effect](#velocity-saturation-effect)
- [Sky130 Labs: ID-VGS Analysis](#sky130-labs-id-vgs-analysis)
- [CMOS Voltage Transfer Characteristics](#cmos-voltage-transfer-characteristics)
- [CMOS Inverter Analysis](#cmos-inverter-analysis)
- [Load Line Construction](#load-line-construction)

---

##  SPICE Simulation for Lower Nodes

### Long Channel NMOS Characteristics

**Device Specifications:**
- W = 1.8μm, L = 1.2μm
- W/L ratio = 1.5
- Channel Length: **Long channel** (L > 250nm)

### Operating Regions

#### Linear Region
- **Condition:** V<sub>DS</sub> < (V<sub>GS</sub> - V<sub>th</sub>)
- **Behavior:** I<sub>D</sub> varies **linearly** with V<sub>DS</sub>
- **Equation:** `I_D = k_n[(V_GS - V_th)V_DS - V_DS²/2]`

#### Saturation Region
- **Condition:** V<sub>DS</sub> ≥ (V<sub>GS</sub> - V<sub>th</sub>)
- **Behavior:** I<sub>D</sub> depends on **channel length modulation** and V<sub>DS</sub>
- **Equation:** `I_D = (k_n/2)(V_GS - V_th)²(1 + λV_DS)`

**The region BEFORE V<sub>DS</sub> = V<sub>GS</sub> - V<sub>th</sub> is LINEAR.**  
**The region AFTER V<sub>DS</sub> = V<sub>GS</sub> - V<sub>th</sub> is SATURATION.**

---

##  Long Channel vs Short Channel Comparison

### Observation 1: I<sub>D</sub>-V<sub>GS</sub> Characteristics

#### Test Setup
Comparing two NMOS devices with **same W/L ratio = 1.5**:

| Device Type | Width (W) | Length (L) | Classification |
|-------------|-----------|------------|----------------|
| **Long Channel** | 1.8μm | 1.2μm | L > 250nm |
| **Short Channel** | 0.375μm | 0.25μm | L < 250nm |

#### Measurement Method
- Apply constant V<sub>DS</sub>
- Sweep V<sub>GS</sub> from 0 to V<sub>DD</sub>
- Observe I<sub>D</sub> vs V<sub>GS</sub> characteristics

---

### Key Differences in Behavior

####  Long Channel Devices (L > 250nm)

**Current-Voltage Relationship:**
```
I_D ∝ (V_GS - V_th)²
```

**Characteristics:**
- 🔸 **Ideal quadratic dependence** on V<sub>GS</sub> throughout entire range
- 🔸 Follows classical MOSFET theory perfectly
- 🔸 No velocity saturation effects
- 🔸 Higher peak drain current

---

####  Short Channel Devices (L < 250nm)

**Current-Voltage Relationship:**
```
At LOW V_GS:  I_D ∝ (V_GS - V_th)²  (Quadratic)
At HIGH V_GS: I_D ∝ (V_GS - V_th)   (Linear)
```

**Characteristics:**
- 🔸 **Quadratic at low V<sub>GS</sub>** (follows classical theory)
- 🔸 **Linear at high V<sub>GS</sub>** (velocity saturation kicks in)
- 🔸 Transition from quadratic to linear behavior
- 🔸 Lower peak drain current compared to long channel

---

##  Velocity Saturation Effect

### Physical Mechanism

#### Low Electric Field (Long Channel)
```
Carrier Velocity ∝ Electric Field
v = μ × E (Linear relationship)
```
- Carriers accelerate freely
- Mobility (μ) remains constant
- Ideal quadratic I<sub>D</sub>-V<sub>GS</sub> relationship

#### High Electric Field (Short Channel)
```
Carrier Velocity → v_sat (Constant)
```
- Electric field reaches critical value
- Carrier velocity **saturates** at maximum limit (v<sub>sat</sub>)
- Velocity no longer increases with field
- I<sub>D</sub>-V<sub>GS</sub> becomes **linear** instead of quadratic

### Velocity-Field Relationship

```
  v
  |     
  |         ┌─────────────  v_sat (Saturation)
  |        ╱
  |       ╱
  |      ╱
  |     ╱ (Linear region)
  |    ╱
  |   ╱
  |──────────────────────────── E
     Low E        High E
```

**At LOW E:** v ∝ E (Linear)  
**At HIGH E:** v = v<sub>sat</sub> (Constant)

---

##  Operating Regions Comparison

### Long Channel Devices (L > 250 nm)

```
┌─────────┐      ┌────────┐      ┌────────────┐
│ Cut-off │ ───► │ Linear │ ───► │ Saturation │
└─────────┘      └────────┘      └────────────┘
```

**Three Modes:**
1. **Cut-off** (V<sub>GS</sub> < V<sub>th</sub>)
2. **Resistive/Linear** (V<sub>DS</sub> < V<sub>GS</sub> - V<sub>th</sub>)
3. **Saturation** (V<sub>DS</sub> ≥ V<sub>GS</sub> - V<sub>th</sub>)

---

### Short Channel Devices (L < 250 nm)

```
┌─────────┐      ┌────────┐      ┌──────────────────┐      ┌────────────┐
│ Cut-off │ ───► │ Linear │ ───► │ Velocity         │ ───► │ Saturation │
│         │      │        │      │ Saturation       │      │            │
└─────────┘      └────────┘      └──────────────────┘      └────────────┘
```

**Four Modes:**
1. **Cut-off** (V<sub>GS</sub> < V<sub>th</sub>)
2. **Resistive/Linear** (V<sub>DS</sub> < V<sub>GS</sub> - V<sub>th</sub>)
3. **Velocity Saturation**  (NEW - High V<sub>GS</sub>, carrier velocity saturates)
4. **Saturation** (V<sub>DS</sub> ≥ V<sub>GS</sub> - V<sub>th</sub>)

**Key Point:** An **additional velocity saturation region** appears due to short channel effects.

---

##  Observation 2: Peak Current Comparison

### Experimental Results

| Parameter | Long Channel | Short Channel |
|-----------|--------------|---------------|
| **Width** | 1.8μm | 0.375μm |
| **Length** | 1.2μm | 0.25μm |
| **W/L Ratio** | 1.5 | 1.5 |
| **Peak I<sub>D</sub>** | **410 μA** | **210 μA** |
| **Reduction** | Reference | **~49% lower** |

### Key Insights

 **Advantages of Short Channel:**
- Smaller device size
- Faster switching speed
- Higher integration density
- Lower capacitance

 **Disadvantage of Short Channel:**
- **~50% reduction in peak drain current**
- Limited by velocity saturation
- Lower drive strength

### Why Lower Current?

**Long Channel:**
- Carriers **accelerate freely** under electric field
- Velocity increases with V<sub>DS</sub>
- Higher I<sub>D</sub> achievable

**Short Channel:**
- **Velocity saturation** limits carrier speed
- Maximum velocity = v<sub>sat</sub> (constant)
- Current cannot increase beyond v<sub>sat</sub> limit
- Results in lower peak I<sub>D</sub>

---

##  Sky130 Labs: I<sub>D</sub>-V<sub>GS</sub> Analysis

### Lab Setup

Clone the repository and navigate to design files:

```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

---

### Lab 1: NMOS I<sub>D</sub>-V<sub>DS</sub> Characteristics

**Device:** Short Channel NMOS (W=0.39μm, L=0.15μm)

**File:** `day2_nfet_idvds_L015_W039.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
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
# Start ngspice with SPICE file
ngspice day2_nfet_idvds_L015_W039.spice

# In ngspice prompt, plot drain current
ngspice 1 -> plot -vdd#branch
```

**Expected Output:** I<sub>DS</sub> vs V<sub>DS</sub> curves for multiple V<sub>GS</sub> values
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-2/Pictures/idvds.png)
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-2/Pictures/idvds_vdd_branch.png)
**What to Observe:**
- Linear region at low V<sub>DS</sub>
- Saturation region at high V<sub>DS</sub>
- Effect of velocity saturation on curve shape

---

### Lab 2: NMOS Transfer Characteristics

**Device:** Short Channel NMOS (W=0.39μm, L=0.15μm)

**File:** `day2_nfet_idvgs_L015_W039.spice`

```spice
*Model Description
.param temp=27

*Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

*Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

*simulation commands
.op
.dc Vin 0 1.8 0.1 

.control
run
display
setplot dc1
.endc
.end
```

#### Run Simulation

```bash
# Start ngspice with SPICE file
ngspice day2_nfet_idvgs_L015_W039.spice

# In ngspice prompt, plot drain current
ngspice 1 -> plot -vdd#branch
```

**Expected Output:** I<sub>DS</sub> vs V<sub>GS</sub> transfer characteristic
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-2/Pictures/idvgs.png)
![image](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/blob/main/Day-2/Pictures/idvgs_vdd_branch.png)
**What to Observe:**
- Quadratic behavior at low V<sub>GS</sub>
- Transition to linear at high V<sub>GS</sub>
- Clear evidence of velocity saturation

---

##  CMOS Voltage Transfer Characteristics

### MOSFET as a Switch

#### OFF State (Open Switch)
```
Condition: |V_GS| < |V_th|
Resistance: R_OFF → ∞ (Infinite)
Current: I_D ≈ 0
```

#### ON State (Closed Switch)
```
Condition: |V_GS| > |V_th|
Resistance: R_ON (Finite, small value)
Current: I_D = f(V_GS, V_DS)
```

### Switch Behavior Summary

| State | Condition | Resistance | Behavior |
|-------|-----------|------------|----------|
| **OFF** | \|V<sub>GS</sub>\| < \|V<sub>th</sub>\| | ∞ (Infinite) | Open circuit |
| **ON** | \|V<sub>GS</sub>\| > \|V<sub>th</sub>\| | R<sub>ON</sub> (Finite) | Conducting path |

---

##  CMOS Inverter Analysis

### Circuit Structure

```
       V_DD
         |
         |
      ┌──┴──┐
      │PMOS │  (P-channel)
   ┌──┤     ├──┐
   │  └──┬──┘  │
Vin├─────┤     ├──── Vout ──── C_L
   │  ┌──┴──┐  │
   └──┤     ├──┘
      │NMOS │  (N-channel)
      └──┬──┘
         |
         |
       V_SS (GND)
```

**Components:**
- **PMOS** - Connected between V<sub>DD</sub> and output
- **NMOS** - Connected between output and GND
- **V<sub>in</sub>** - Applied to both gates (common input)
- **V<sub>out</sub>** - Taken from common drain node
- **C<sub>L</sub>** - Load capacitance

---

### Transistor-Level and Switch-Level Views

#### Case 1: V<sub>in</sub> = V<sub>DD</sub> (Logic HIGH Input)

**Transistor States:**
```
PMOS: V_GS = V_DD - V_DD = 0V < |V_th,p|  → OFF
NMOS: V_GS = V_DD - 0 = V_DD > V_th,n    → ON
```

**Switch Model:**
```
       V_DD
         |
         X  (PMOS OFF - Open Switch)
         |
Vin ─────┤
(HIGH)   |
         |
       ──┴──  R_n (NMOS ON - Resistor)
         |
        GND
```

**Result:** V<sub>out</sub> = **0V** (Logic LOW)

---

#### Case 2: V<sub>in</sub> = 0V (Logic LOW Input)

**Transistor States:**
```
PMOS: V_GS = 0 - V_DD = -V_DD < -|V_th,p|  → ON
NMOS: V_GS = 0 - 0 = 0V < V_th,n           → OFF
```

**Switch Model:**
```
       V_DD
         |
       ──┴──  R_p (PMOS ON - Resistor)
         |
Vin ─────┤
(LOW)    |
         |
         X  (NMOS OFF - Open Switch)
         |
        GND
```

**Result:** V<sub>out</sub> = **V<sub>DD</sub>** (Logic HIGH)

---

### CMOS Inverter Truth Table

| V<sub>in</sub> | PMOS State | NMOS State | V<sub>out</sub> | Function |
|---|---|---|---|---|
| **0V** (LOW) | ON (R<sub>p</sub>) | OFF (Open) | **V<sub>DD</sub>** (HIGH) | Inversion |
| **V<sub>DD</sub>** (HIGH) | OFF (Open) | ON (R<sub>n</sub>) | **0V** (LOW) | Inversion |

### Key Properties

🔸 **Logic Inversion:** Output is complement of input  
🔸 **Low Static Power:** Only one transistor ON at steady state, no DC path  
🔸 **Sharp Transitions:** Fast switching between logic levels  
🔸 **Rail-to-Rail Output:** V<sub>out</sub> swings from 0 to V<sub>DD</sub>  
🔸 **Foundation of CMOS Logic:** Basic building block for all digital circuits

---

##  Load Line Construction for VTC

### Goal
Plot the **Voltage Transfer Characteristic (VTC)** curve showing V<sub>out</sub> vs V<sub>in</sub> for a CMOS inverter by merging NMOS and PMOS characteristics.

---

### Step 1: Convert PMOS Voltages to Circuit Variables

**PMOS Gate-Source Voltage:**
```
V_GS,p = V_G - V_S = V_in - V_DD
```

**Convert to equivalent input voltage:**
```
V_in = V_DD + V_GS,p
```

**Replace all internal voltages with:**
- V<sub>in</sub> (Input)
- V<sub>DD</sub> (Supply)
- V<sub>SS</sub> (Ground = 0V)
- V<sub>out</sub> (Output)

---

### Step 2 & 3: Express Drain-Source Voltages

**For NMOS:**
```
V_DS,n = V_D - V_S = V_out - 0 = V_out
```

**For PMOS:**
```
V_DS,p = V_D - V_S = V_out - V_DD
or equivalently:
V_SD,p = V_DD - V_out
```

**Normalize both to V<sub>out</sub>:**
- NMOS load curve: I<sub>D,n</sub> vs V<sub>out</sub>
- PMOS load curve: I<sub>D,p</sub> vs V<sub>out</sub>

---

### Step 4: Merge Load Curves and Plot VTC

**Current Equality at DC:**
```
I_D,n = I_D,p  (at steady state, no capacitor current)
```

**VTC Construction:**
1. For each V<sub>in</sub> value (0 to V<sub>DD</sub>):
   - Plot NMOS I-V curve as function of V<sub>out</sub>
   - Plot PMOS I-V curve as function of V<sub>out</sub>
   - Find intersection point where I<sub>D,n</sub> = I<sub>D,p</sub>
   - That V<sub>out</sub> is the operating point

2. Repeat for all V<sub>in</sub> values

3. Plot V<sub>out</sub> vs V<sub>in</sub> → **VTC Curve**

**VTC Regions:**
```
         V_out
          |
      V_DD├─┐
          │  ╲
          │   ╲ (Transition)
          │    ╲
      V_M ├─────●────
          │      ╲
          │       ╲
        0 ├────────┘
          └────────────── V_in
          0   V_M    V_DD
```

**Where:**
- **V<sub>M</sub>** = Switching threshold (V<sub>in</sub> = V<sub>out</sub> point)
- **Steep transition** = High gain region
- **Output swing** = 0 to V<sub>DD</sub> (rail-to-rail)

---

##  VTC Characteristics Summary

### Five Regions of Operation

| Region | V<sub>in</sub> Range | NMOS | PMOS | V<sub>out</sub> | Gain |
|--------|---------|------|------|------|------|
| **1** | 0 to V<sub>th,n</sub> | Cut-off | Linear | ≈ V<sub>DD</sub> | Low |
| **2** | V<sub>th,n</sub> to V<sub>M</sub> | Saturation | Linear | Falling | High |
| **3** | ≈ V<sub>M</sub> | Saturation | Saturation | V<sub>M</sub> | **Very High** |
| **4** | V<sub>M</sub> to (V<sub>DD</sub>-\|V<sub>th,p</sub>\|) | Linear | Saturation | Falling | High |
| **5** | (V<sub>DD</sub>-\|V<sub>th,p</sub>\|) to V<sub>DD</sub> | Linear | Cut-off | ≈ 0V | Low |

### Critical Parameters

**Switching Threshold (V<sub>M</sub>):**
```
V_M ≈ (V_DD/2) for symmetric inverter
```

**Noise Margins:**
```
NM_H = V_OH - V_IH  (High-level noise margin)
NM_L = V_IL - V_OL  (Low-level noise margin)
```

**Gain in Transition Region:**
```
Gain = |dV_out / dV_in| >> 1
```

---

##  Quick Reference

### Key Equations

**Long Channel I<sub>D</sub>:**
```
I_D = (k_n/2)(V_GS - V_th)²
```

**Short Channel I<sub>D</sub> (with velocity saturation):**
```
At low V_GS:  I_D ∝ (V_GS - V_th)²
At high V_GS: I_D ∝ (V_GS - V_th)
```

**Velocity Saturation:**
```
v_sat ≈ 10^7 cm/s (for silicon)
```

---

##  Common ngspice Commands

```bash
# Load SPICE file
ngspice filename.spice

# In interactive mode
ngspice 1 -> run                    # Execute simulation
ngspice 2 -> display                # Show available plots
ngspice 3 -> setplot dc1           # Select DC analysis
ngspice 4 -> plot -vdd#branch      # Plot drain current
ngspice 5 -> plot vout vs vin      # Plot VTC
ngspice 6 -> quit                  # Exit
```

---

##  References

- [Sky130 PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [Original Workshop Repository](https://github.com/kunalg123/sky130CircuitDesignWorkshop)
- [ngspice Manual](http://ngspice.sourceforge.net/docs.html)

---


