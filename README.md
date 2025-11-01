# RISC-V-TAPEOUT-Week-4
# CMOS Circuit Design Workshop


Complete hands-on workshop for CMOS circuit design, SPICE simulation, and performance characterization using Sky130 PDK.

---

##  Workshop Contents

### [Day 1: SPICE Fundamentals & NMOS Basics](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/tree/main/Day-1)
Learn SPICE simulation basics and NMOS transistor fundamentals
- SPICE deck structure and syntax
- NMOS operating regions (Cut-off, Linear, Saturation)
- Threshold voltage and body effect
- Drain current equations and derivations
- **Lab:** I<sub>D</sub>-V<sub>DS</sub> characteristics

### [Day 2: Velocity Saturation & CMOS Inverter VTC]([./day2_velocity_saturation.md](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/tree/main/Day-2)
Understand short-channel effects and voltage transfer characteristics
- Long channel vs short channel comparison
- Velocity saturation physics
- CMOS inverter switching behavior
- Load line construction for VTC
- **Labs:** I<sub>D</sub>-V<sub>GS</sub> analysis, VTC plotting

### [Day 3: Switching Threshold & Dynamic Analysis]([./day3_switching_threshold.md](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/tree/main/Day-3)
Analyze switching threshold and propagation delays
- Switching threshold (V<sub>m</sub>) derivation
- Analytical W/L ratio calculations
- Rise and fall time measurements
- Delay balancing techniques
- **Labs:** VTC extraction, transient analysis

### [Day 4: Noise Margin Analysis]([./day4_noise_margin.md](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/tree/main/Day-4)
Evaluate circuit robustness through noise margin characterization
- Noise margin definitions (NM<sub>H</sub>, NM<sub>L</sub>)
- Critical voltage parameters (V<sub>IL</sub>, V<sub>IH</sub>, V<sub>OL</sub>, V<sub>OH</sub>)
- Effect of device sizing on noise margins
- **Lab:** Noise margin extraction and analysis

### [Day 5: PVT Variations](https://github.com/JANADINI/RISC-V-TAPEOUT-Week-4/tree/main/Day-5)
Study circuit behavior under process, voltage, and temperature variations
- Power supply variation impact
- Device variation sources (etching, oxide thickness)
- Corner analysis (TT, FF, SS, FS, SF)
- **Labs:** Supply sweep, device skew analysis

---

##  Quick Start

### Prerequisites
```bash
# Install ngspice
sudo apt-get update
sudo apt-get install ngspice

# Verify installation
ngspice --version
```

### Clone Repository
```bash
git clone https://github.com/kunalg123/sky130CircuitDesignWorkshop.git
cd sky130CircuitDesignWorkshop/design
```

### Run Your First Simulation
```bash
# NMOS I-V characteristics
ngspice day1_nfet_idvds_L2_W5.spice
ngspice 1 -> plot -vdd#branch

# CMOS Inverter VTC
ngspice day3_inv_vtc_Wp084_Wn036.spice
ngspice 1 -> plot out vs in
```

---

##  Key Concepts Covered

### Circuit Analysis
-  DC operating point analysis
-  Voltage transfer characteristics (VTC)
-  Transient response and delays
-  Noise margin calculations
-  PVT corner analysis

### Design Parameters
-  Switching threshold (V<sub>m</sub>)
-  Noise margins (NM<sub>H</sub>, NM<sub>L</sub>)
-  Propagation delays (t<sub>pLH</sub>, t<sub>pHL</sub>)
-  Drive strength and W/L ratios
-  Power consumption

### Physics & Models
-  MOSFET operating regions
-  Velocity saturation effects
-  Body effect and threshold voltage
-  Channel length modulation
-  Short-channel effects

---

##  Lab Exercises

| Lab | Description | Key Learning |
|-----|-------------|--------------|
| **Lab 1.1** | NMOS I<sub>D</sub>-V<sub>DS</sub> | Operating regions identification |
| **Lab 2.1** | NMOS I<sub>D</sub>-V<sub>GS</sub> | Velocity saturation observation |
| **Lab 2.2** | Short vs Long channel | Compare device behavior |
| **Lab 3.1** | Inverter VTC | Extract V<sub>m</sub>, gain |
| **Lab 3.2** | Transient response | Measure t<sub>pLH</sub>, t<sub>pHL</sub> |
| **Lab 4.1** | Noise margins | Calculate NM<sub>H</sub>, NM<sub>L</sub> |
| **Lab 5.1** | Supply variation | V<sub>DD</sub> sweep analysis |
| **Lab 5.2** | Device variation | Extreme W/L ratio study |

---

##  Design Guidelines Summary

### CMOS Inverter Sizing
```
For balanced operation:
  W_p/W_n ≈ 2 to 2.5

For Sky130 (typical):
  W_n = 0.36μm (minimum)
  W_p = 0.72-0.90μm
  L = 0.15μm (minimum)
```

### Target Specifications
```
Switching Threshold:  V_m ≈ V_DD/2
Noise Margins:        NM ≥ 30% of V_DD
Delay Balance:        |t_pLH - t_pHL| < 10%
Supply Range:         0.8V ≤ V_DD ≤ 1.8V
```

---

##  Tools & Technologies

- **SPICE Simulator:** ngspice
- **PDK:** SkyWater Sky130 (130nm Open-Source PDK)
- **Models:** BSIM4 for NMOS/PMOS
- **Analysis:** DC, AC, Transient, Corner

---

##  Resources

### Documentation
- [Sky130 PDK Docs](https://skywater-pdk.readthedocs.io/)
- [ngspice Manual](http://ngspice.sourceforge.net/docs.html)
- [SPICE Syntax Reference](http://ngspice.sourceforge.net/ngspice-manual.pdf)

### Workshop Materials
- Original Repository: [kunalg123/sky130CircuitDesignWorkshop](https://github.com/kunalg123/sky130CircuitDesignWorkshop)
- Sky130 Models: `sky130_fd_pr/models/sky130.lib.spice`

---

##  Learning Outcomes

By completing this workshop, you will:

 Understand MOSFET physics and operating principles  
 Master SPICE simulation for circuit analysis  
 Design and optimize CMOS inverters  
 Characterize circuit performance (speed, power, noise)  
 Analyze PVT variations and design for robustness  
 Extract and interpret key circuit parameters  
 Apply industry-standard design methodologies  

---

##  Repository Structure

```
.
├── README.md                           # This file
├── day1_spice_fundamentals.md          # SPICE & NMOS basics
├── day2_velocity_saturation.md         # Short-channel effects & VTC
├── day3_switching_threshold.md         # Vm analysis & dynamics
├── day4_noise_margin.md                # Noise margin robustness
├── day5_pvt_variations.md              # Supply & device variations
└── design/
    ├── day1_nfet_idvds_L2_W5.spice
    ├── day2_nfet_idvgs_L015_W039.spice
    ├── day3_inv_vtc_Wp084_Wn036.spice
    ├── day3_inv_tran_Wp084_Wn036.spice
    ├── day4_inv_noisemargin_wp1_wn036.spice
    ├── day5_inv_supplyvariation_Wp1_Wn036.spice
    ├── day5_inv_devicevariation_wp7_wn042.spice
    └── sky130_fd_pr/
        └── models/
            └── sky130.lib.spice
```

---

##  Credits

**Workshop Design:** VSD (VLSI System Design)  
**PDK:** SkyWater Technology and Google  
**Original Repository:** [kunalg123](https://github.com/kunalg123)

---
