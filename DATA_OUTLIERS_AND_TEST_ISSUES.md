# Data Outliers and Potential Test Equipment Failures

**Analysis Date:** 2026-02-03  
**Analyst:** Material Science Knowledge Base Analysis  
**Purpose:** Identify gross outliers requiring test reruns

---

## Executive Summary

Analysis of test data across all apparatus types reveals **several critical data quality issues** that strongly suggest test equipment failures, data acquisition errors, or incorrect test procedures. These outliers fall well outside physically reasonable ranges and should be investigated immediately.

### Critical Findings

**Must Rerun (Equipment Failure Suspected):**
- 7 materials with **negative heat of combustion** (MCC equipment failure)
- 3 materials with **zero ignition time** when ignition occurred (Cone data logging error)
- 2 materials with **extremely high peak HRRPUA** (>3000 kW/m²) suggesting instrument saturation

**Should Investigate:**
- House_Wrap extremely low peak HRRPUA (3-4 kW/m²) despite being PE (should be 600+)
- PMMA zero ignition time at 15 kW/m² tests (likely did not ignite)
- Multiple materials with very slow ignition (>10 minutes) at low heat flux

---

## Critical Issue #1: NEGATIVE HEAT OF COMBUSTION (MCC)

### Severity: CRITICAL - Equipment Malfunction

**Physical Impossibility:** Heat of combustion CANNOT be negative. This represents energy *released* during combustion, which is always positive for organic materials.

### Affected Materials (MUST RERUN):

| Material | HOC (MJ/kg) | Category | Expected Range |
|----------|-------------|----------|----------------|
| **Mineral_Wool_Insulation** | -47.50 | Insulation | Should be ~1-5 (if binder present) |
| **Fiberglass_Insulation_R13_Paper_Faced_Fiberglass** | -17.33 | Insulation | Should be ~1-5 (if binder present) |
| **Cinder_Block** | -14.11 | Building Materials | Should be ~0-2 (essentially inert) |
| **Concrete_Block** | -4.84 | Building Materials | Should be ~0-2 (essentially inert) |
| **Gypsum_Wallboard_Gypsum_Core** | -3.19 | Building Materials | Should be ~0-1 (essentially inert) |
| **Fiber_Cement_Board** | -0.58 | Building Materials | Should be ~0-2 (essentially inert) |

### Root Cause Analysis

**Most Likely Cause:** MCC instrument calibration error or baseline drift

1. **Baseline Error**: Heat flow baseline not properly zeroed before test
2. **Calibration Issue**: Temperature or heat flow sensor out of calibration
3. **Sample Preparation**: Contamination or incorrect sample mass
4. **Software Bug**: Data processing error in heat integration

### Why This is Critical

- Negative values are physically impossible
- Indicates systematic error in MCC apparatus
- All MCC tests during this period are suspect
- Other materials tested around same time may also be affected

### Recommended Actions

1. **Immediate**: Check MCC calibration records and maintenance logs
2. **Immediate**: Re-baseline MCC instrument
3. **Immediate**: Run PMMA or other standard reference material to verify instrument
4. **High Priority**: Rerun ALL six materials listed above
5. **Medium Priority**: Review ALL other MCC tests from same time period

---

## Critical Issue #2: EXTREMELY HIGH PEAK HRRPUA (Cone Calorimeter)

### Severity: HIGH - Likely Instrument Saturation

**Physical Concern:** Peak HRRPUA values above 3000 kW/m² are extremely rare and suggest instrument saturation or flaming/dripping artifacts.

### Affected Materials (INVESTIGATE/RERUN):

**Polyolefins at 75 kW/m² heat flux:**

| Material | Peak HRRPUA | Expected | Deviation |
|----------|-------------|----------|-----------|
| **PP** | 3452.8 kW/m² | 600-1200 | **3x too high** |
| **LDPE** | 3397.1 kW/m² | 600-1200 | **3x too high** |
| **HDPE** | 3321.4 kW/m² | 600-1200 | **3x too high** |

### Analysis

**Literature Values for Polyolefins:**
- PE/PP at 50 kW/m²: 600-1000 kW/m²
- PE/PP at 75 kW/m²: 800-1400 kW/m²
- **Maximum well-documented**: ~1500 kW/m² for thin PP films

**Measured values 2-3x literature maximums**

### Possible Causes

1. **Instrument Saturation**: Cone oxygen analyzer or load cell saturated
   - Check if instrument has upper limit ~1000-1500 kW/m²
   - Values may be extrapolated beyond valid range

2. **Sample Dripping/Flaming**: Molten polymer dripped causing secondary ignition
   - Very common with polyolefins at high heat flux
   - Burning drips create artificially high localized HRR

3. **Sample Holder Issue**: Wire mesh failure or sample overflow
   - Polymer melted and flowed outside sample holder
   - Increased burning area → artificially high HRRPUA

4. **Data Processing Error**: Peak detection algorithm error
   - Spike in data mistaken for actual peak
   - Outlier not filtered from raw data

### Recommended Actions

1. **Review**: Examine raw data files for spikes or anomalies
2. **Review**: Check test videos if available - look for dripping/flowing
3. **Compare**: Check 50 kW/m² results for same materials (should be lower)
4. **Decision**: 
   - If 50 kW/m² values are reasonable (800-1200), accept those
   - Rerun 75 kW/m² tests with edge frame to prevent dripping
   - Or document that 75 kW/m² data is unreliable for polyolefins

---

## Critical Issue #3: ZERO IGNITION TIME WITH COMBUSTION

### Severity: HIGH - Data Logging Error

**Physical Impossibility:** If material actually ignited and released heat (HRRPUA > 0), ignition time cannot be zero.

### Affected Materials:

| Material | Ignition Time | Peak HRRPUA | Issue |
|----------|---------------|-------------|-------|
| **PC** | 0.0 s at 25 kW/m² | 190-230 kW/m² | Ignited but time not recorded |
| **Gypsum_Wallboard** | 0.0 s at 25 kW/m² | 20-30 kW/m² | Paper facing ignited |
| **PMMA** | 0.0 s at 15 kW/m² | 0.0 kW/m² | Actually did NOT ignite (correct) |

### Analysis

**PC (Polycarbonate):**
- Peak HRRPUA 190-230 kW/m² indicates sustained flaming combustion
- Zero ignition time is impossible - likely data logging error
- Expected ignition time at 25 kW/m²: 60-120 seconds
- **Action**: Examine raw data files, manually determine ignition time

**Gypsum Wallboard:**
- Peak HRRPUA 20-30 kW/m² indicates paper facing burned
- Zero ignition time is impossible
- Expected ignition time for paper: 20-40 seconds
- **Action**: Examine raw data files, manually determine ignition time

**PMMA at 15 kW/m² (NO ISSUE):**
- Peak HRRPUA is 0.0 → material did NOT ignite
- Zero ignition time is correct (no ignition occurred)
- PMMA critical heat flux is ~20 kW/m², so no ignition at 15 expected
- **No action needed** - this is correct

### Root Cause

**Most Likely**: Cone data acquisition software did not detect ignition
- Manual ignition used but not logged
- Automatic detection threshold set incorrectly
- Operator forgot to mark ignition time

### Recommended Actions

1. **Immediate**: Review raw Cone data files for these materials
2. **Manual Analysis**: Determine ignition time from HRRPUA curve (when HRR first rises)
3. **Update**: Correct ignition times in analysis spreadsheets
4. **Future**: Verify ignition detection settings on Cone apparatus

---

## Critical Issue #4: EXTREMELY LOW PEAK HRRPUA FOR POLYMERS

### Severity: HIGH - Test Procedure Error or Non-Ignition

**Physical Concern:** House_Wrap (polyethylene) showing 3-4 kW/m² peak HRRPUA when literature values are 600-1000 kW/m².

### Affected Material:

**House_Wrap (Polyethylene)**
- Measured: 3.4 - 4.0 kW/m² at 25 and 50 kW/m²
- Expected: 600-800 kW/m² at 50 kW/m²
- **150x too low**

### Analysis

**This is a ~99.5% discrepancy - indicates fundamental test problem**

**Possible Causes:**

1. **Material Did Not Actually Ignite**
   - Spark igniter failed
   - Material melted and flowed away from heat source
   - Test terminated before ignition

2. **Wrong Material Tested**
   - Sample labeled "House_Wrap" but actually something else
   - Tested backing material instead of polymer

3. **Instrument Malfunction**
   - Oxygen analyzer not working
   - Load cell failure
   - Gas flow rate incorrect

4. **Sample Preparation Error**
   - Sample area much larger than standard 100 cm²
   - Sample not properly exposed to heat flux

### Evidence Supporting Non-Ignition

- Value (3-4 kW/m²) similar to non-combustible insulation materials
- Feather_Pillow_Feathers shows similar low values (1.5-3.9 kW/m²)
- Both materials may have melted/shrunk away without sustaining flame

### Recommended Actions

1. **Immediate**: Review test videos and operator notes
2. **Verify**: Confirm material identity (is it really polyethylene House_Wrap?)
3. **Retest**: Run new Cone tests with edge frame to prevent shrinkage
4. **Compare**: Test virgin LDPE/HDPE film under same conditions as control

---

## Issue #5: EXTREMELY SLOW IGNITION TIMES

### Severity: MEDIUM - May Be Correct But Should Verify

**Materials with ignition > 10 minutes at 25 kW/m²:**

| Material | Ignition Time | Category | Concern |
|----------|---------------|----------|---------|
| PET | 1102.5 s (18.4 min) | Plastic | Very slow for thin polymer |
| Nylon_Carpet_High_Pile | 1003.8 s (16.7 min) | Carpet | Slow but possible if thick |
| Overstuffed_Chair_Polyester_Batting | 994.0 s (16.6 min) | Furniture | Slow but possible if thick |

### Analysis

**These MAY be correct if:**
- Materials are very thick (>10 mm)
- Materials have high density/thermal inertia
- Test properly conducted but material is inherently resistant

**However, should verify:**
- PET (polyester) typically ignites in 60-180 seconds at 25 kW/m²
- 18 minutes is unusually slow - suggests:
  - Very thick sample
  - Operator intervention/test disruption
  - Pilot flame extinguishment

### Recommended Actions

1. **Review**: Check sample thickness and preparation notes
2. **Review**: Examine HRRPUA curves - is ignition clearly marked?
3. **Compare**: Look at same materials at 50 kW/m² (should be much faster)
4. **Decision**: If thickness explains slow ignition, document. Otherwise retest.

---

## Issue #6: ANOMALOUS HEAT OF COMBUSTION - VERY LOW VALUES

### Severity: MEDIUM - May Indicate Inert Contamination

**Materials with HOC < 5 MJ/kg:**

| Material | HOC (MJ/kg) | Expected | Issue |
|----------|-------------|----------|-------|
| Fiberglass_Insulation_R30_Paper_Faced_Fiberglass | 0.83 | 1-3 | Very low, check if pure fiberglass |
| Lightweight_Gypsum_Wallboard_Gypsum_Core | 2.03 | 0-2 | Reasonable (mostly inert) |
| Vinyl_Tile | 3.04 | 15-20 | **Much too low for PVC** |
| Pine_Lap_Siding_Paint | 4.17 | 5-10 | Low but possible if thin paint |

### Analysis

**Vinyl_Tile Most Concerning:**
- PVC should have HOC of 15-20 MJ/kg
- Measured 3.04 MJ/kg is 80% too low
- Suggests:
  - Mostly inert filler (calcium carbonate, talc)
  - Very low actual PVC content
  - OR sample contamination
  - OR MCC sample too small/incomplete combustion

### Recommended Actions

1. **Vinyl_Tile**: Rerun MCC test with proper sample size
2. **Check**: Verify Vinyl_Tile material composition
3. **Compare**: Test pure PVC as control

---

## Issue #7: ANOMALOUS HEAT OF COMBUSTION - TOO HIGH

### Severity: LOW-MEDIUM - Possibly Correct

**Materials with HOC > 42 MJ/kg:**

| Material | HOC (MJ/kg) | Expected | Assessment |
|----------|-------------|----------|------------|
| Coffee_Machine | 44.08 | 30-40 | Contains mostly PE/PP components |
| Air_Fryer | 42.69 | 30-40 | Contains mostly PE/PP components |

### Analysis

**These values may be CORRECT:**
- Both appliances contain predominantly polyolefin (PE/PP) components
- PE/PP have HOC of 43-46 MJ/kg
- If tested components were mostly PE/PP, high values are expected

**However, should verify:**
- What specific component was tested?
- Were metal/ceramic parts removed?
- Did sample include electrical components?

### Recommended Actions

1. **Document**: Clarify exactly what part of appliance was tested
2. **Low Priority**: Retest if specific component identification is unclear

---

## Summary of Required Actions

### MUST RERUN (Equipment Failure):

1. **Mineral_Wool_Insulation** - MCC (negative HOC)
2. **Fiberglass_Insulation_R13_Paper_Faced_Fiberglass** - MCC (negative HOC)
3. **Cinder_Block** - MCC (negative HOC)
4. **Concrete_Block** - MCC (negative HOC)
5. **Gypsum_Wallboard_Gypsum_Core** - MCC (negative HOC)
6. **Fiber_Cement_Board** - MCC (negative HOC)
7. **House_Wrap** - Cone (peak HRRPUA 150x too low)

### SHOULD INVESTIGATE/CORRECT:

8. **PP, HDPE, LDPE** - Cone at 75 kW/m² (peak HRRPUA 3x too high - likely saturation)
9. **PC** - Cone ignition time (zero but ignited - data logging error)
10. **Gypsum_Wallboard** - Cone ignition time (zero but ignited - data logging error)
11. **Vinyl_Tile** - MCC (HOC 80% too low)

### VERIFY BUT MAY BE CORRECT:

12. **PET, Nylon_Carpet, Overstuffed_Chair_Batting** - Very slow ignition (check thickness)
13. **Coffee_Machine, Air_Fryer** - HOC slightly high (document component tested)

---

## Equipment Recommendations

### MCC Apparatus:

1. **Immediate**: Full calibration check
2. **Immediate**: Baseline verification with PMMA standard
3. **Check**: Maintenance records around dates of negative HOC tests
4. **Implement**: More frequent calibration verification (weekly with PMMA)

### Cone Calorimeter:

1. **Check**: Oxygen analyzer calibration and range limits
2. **Implement**: Edge frames for all thermoplastic tests at high heat flux
3. **Verify**: Data acquisition software ignition detection settings
4. **Procedure**: Require operator to manually log ignition time as backup

---

## Validation Strategy

### For MCC Retests:

1. Run PMMA standard first (should get 25-26 MJ/kg)
2. If PMMA correct, proceed with problem materials
3. If PMMA incorrect, recalibrate and repeat

### For Cone Retests:

1. Use edge frame for all thermoplastics
2. Record video of all tests
3. Manually log ignition time in addition to automatic detection
4. Check for instrument saturation by examining raw data

---

## Data Quality Impact

**Percentage of Materials Affected:**
- MCC: 6 materials with negative values out of 126 tested = **4.8% critical failures**
- Cone: 7-10 materials with questionable results out of 79 tested = **9-13% issues**

**Overall Assessment:**
- **~93% of data appears reliable**
- **~7% requires investigation or retest**
- Issues concentrated in specific material types (inert, polyolefins) and test conditions

**Root Causes:**
1. MCC instrument calibration issue (specific time period)
2. Cone data acquisition settings (ignition detection)
3. Polyolefin behavior at high heat flux (known challenge)

---

## Conclusion

While the majority of test data is of high quality and within expected ranges, **7 materials require mandatory retesting** due to physically impossible results (negative heat of combustion) or gross outliers (House_Wrap). An additional 5-6 materials should be investigated for data logging errors or unexpected behavior.

The pattern of errors suggests:
- **Systematic MCC calibration issue** during specific testing period
- **Cone data acquisition settings** need adjustment
- **Test procedures for thermoplastics** at high heat flux need improvement

**Priority Actions:**
1. Verify MCC calibration immediately
2. Rerun 6 materials with negative HOC
3. Investigate House_Wrap Cone results
4. Correct ignition time data logging errors

---

**Report Date:** 2026-02-03  
**Materials Requiring Retest:** 7 (5.5% of tested materials)  
**Materials Requiring Investigation:** 5-6 (4-5% of tested materials)  
**Data Quality:** 90% reliable, 10% requires attention

---

**End of Data Outliers Report**
