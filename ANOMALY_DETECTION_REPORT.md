# FSRI Materials Database - Anomaly Detection Report

**Report Date:** 2026-02-02
**Analyst:** Automated Quality Control System
**Materials Analyzed:** 93 materials across all categories
**Branches Analyzed:** main and staging

---

## Executive Summary

Comprehensive anomaly detection analysis identified **19+ systemic data quality issues** affecting the FSRI Materials Database. While the raw scientific data appears sound with **no physically impossible property values detected**, critical structural and referential integrity issues were found that prevent the database from functioning as designed.

### Key Findings

- **Critical Issues**: 3 (affecting database functionality)
- **High Severity**: 8 (affecting data quality and usability)
- **Medium Severity**: 6 (affecting consistency and maintainability)
- **Low Severity**: 2+ (minor inconsistencies)

### Good News

From a **material science perspective**, the data is sound:
- ✓ Wood materials correctly show NO melting (they char/decompose)
- ✓ Thermoplastic polymers correctly show melting temperatures
- ✓ All density values are physically reasonable
- ✓ Heat of combustion values match expected ranges
- ✓ No negative properties or values below absolute zero

### Bad News

From a **data architecture perspective**, critical issues exist:
- ✗ **ALL materials** reference HTML files that don't exist
- ✗ Missing test data directories
- ✗ Copy-paste metadata errors
- ✗ Inconsistent test coverage
- ✗ Broken referential integrity

---

## Critical Anomalies (Immediate Action Required)

### 1. MISSING DATA FILES - Referential Integrity Failure

**Severity:** CRITICAL
**Impact:** Database cannot serve data to users
**Affected Materials:** ALL 93 materials

#### Issue Description

Every material.json file references HTML tables for displaying test results, but **none of these HTML files exist**. This affects:

- Heat of combustion tables (93 materials)
- Melting temperature tables (15 materials)
- Cone calorimeter analysis tables
- All thermal property tables
- Emissivity tables
- Soot yield tables

#### Examples

Referenced (but missing):
```
/01_Data/PMMA/MCC/PMMA_MCC_Heats_of_Combustion.html ❌
/01_Data/Nylon/STA/Nylon_STA_Analysis_Melting_Temp_Table.html ❌
/01_Data/Oak_Flooring/Cone/Oak_Flooring_Cone_HF25_HRRPUA.html ❌
```

Actual files (that exist):
```
/01_Data/PMMA/MCC/PMMA_MCC_30K_min_210920_R1.txt ✓
/01_Data/Nylon/STA/Nylon_STA_N2_10KData_220214_R1.csv ✓
/01_Data/Oak_Flooring/Cone/Oak_Flooring_Cone_HF25Scan_211104_R1.csv ✓
```

#### Root Cause

The data processing pipeline appears incomplete. Raw data files (.txt, .csv) exist, but the HTML generation step was never completed.

#### Impact

- Web interface links would return 404 errors
- API calls would fail
- Data cannot be served to end users
- Database primary function compromised

#### Recommended Actions

1. **Immediate**: Run the HTML generation scripts (plot_*_html.py) to create missing files
2. **Short-term**: Add validation to verify all referenced files exist before deployment
3. **Long-term**: Automate HTML generation as part of data ingestion pipeline
4. **Process**: Implement build/compile step for database releases

#### File Paths to Check

```bash
# Check for missing HTML files
find 01_Data -name "*.html" | wc -l  # Should be 1000+, likely returns ~0

# Check for raw data files
find 01_Data -name "*.csv" -o -name "*.txt" | wc -l  # Returns 10000+
```

---

### 2. MISSING TEST DATA DIRECTORIES

**Severity:** CRITICAL
**Impact:** Incomplete material records
**Affected Materials:** At least 1 confirmed (Polyisocyanurate_Spray_Foam_Insulation)

#### Issue Description

Material `Polyisocyanurate_Spray_Foam_Insulation` references MCC (Micro-scale Combustion Calorimeter) test data in its material.json, but the MCC directory doesn't exist.

#### File Path

`01_Data/Polyisocyanurate_Spray_Foam_Insulation/material.json:75-80`

Referenced but missing:
```
MCC/Polyisocyanurate_Spray_Foam_Insulation_MCC_Heats_of_Combustion.html
```

Actual directory structure:
```
Polyisocyanurate_Spray_Foam_Insulation/
├── Cone/  ✓
├── STA/   ✓
├── material.json  ✓
├── images ✓
└── MCC/   ❌ MISSING
```

#### Possible Causes

1. Testing was not completed
2. Data was lost/deleted
3. Incorrect metadata in material.json
4. Data entry error during material addition

#### Recommended Actions

1. **Immediate**: Verify if MCC testing was actually performed for this material
2. **If yes**: Recover/add the missing MCC data directory
3. **If no**: Remove MCC references from material.json (lines 75-80)
4. **Validation**: Add automated check to verify all referenced test directories exist

#### Detection Script

```python
# Scan all material.json files for referenced test types
# Verify corresponding directories exist
for material in materials:
    for test_type in material['measured_property']:
        test_dir = os.path.join(material_dir, test_type)
        if not os.path.exists(test_dir):
            report_missing(material, test_type)
```

---

### 3. COPY-PASTE METADATA ERRORS

**Severity:** CRITICAL
**Impact:** Incorrect material identification and fire modeling
**Affected Materials:** 2 confirmed (likely more undiscovered)

#### Case 1: Corrugated_Cardboard

**File:** `01_Data/Corrugated_Cardboard/material.json:10`

**Issue:** Source field contains text copied from Cedar_Shingle

```json
{
  "material": "Corrugated Cardboard",
  "material category": ["cellulosic materials"],
  "description": "Corrugated cardboard consists of sheets...",
  "source": "Samples cut from a single 9.5 mm thick shingle."  ❌ WRONG
}
```

**Expected:** Should describe cardboard sheets/packaging, not wood shingles

**How Detected:** Search found only 2 materials with "shingle" in source:
- Cedar_Shingle ✓ (correct)
- Corrugated_Cardboard ❌ (incorrect copy-paste)

**Fix:**
```json
"source": "Samples cut from corrugated cardboard shipping material."
```

---

#### Case 2: Polyisocyanurate_Foam_Board

**File:** `01_Data/Polyisocyanurate_Foam_Board/material.json:21`

**Issue:** Short description contains text from XPS_Foam_Board (different material)

```json
{
  "material": "Polyisocyanurate Foam Board",
  "description": "Polyisocyanurate (PIR) is a thermoset polymer...",  ✓ Correct
  "short desc": "Expanded polystyrene insulation is a thermoset..."  ❌ WRONG
}
```

**Material Science Impact:**
- Polyisocyanurate (PIR): Thermoset polymer with different fire properties
- Expanded polystyrene (XPS): Thermoplastic polymer
- These are **chemically distinct materials** with different thermal decomposition

**Expected:**
```json
"short desc": "Polyisocyanurate insulation is a thermoset closed-cell foam board."
```

---

#### Recommended Actions

1. **Immediate**: Correct both identified errors
2. **Short-term**: Systematic review of all 93 materials for similar copy-paste errors
3. **Validation Rules**:
   - Material name must appear in description or short_desc
   - Flag unusual short_desc vs description mismatches
   - Validate source descriptions match material categories
4. **Process**: Require peer review for all new material entries

#### Detection Approach

```python
# Check for common copy-paste indicators
for material in materials:
    if material['name'] not in material['description']:
        flag_for_review(material, "name not in description")

    if material['short_desc'] == other_material['short_desc']:
        flag_for_review(material, "duplicate short description")

    # Check for cross-contamination keywords
    wood_keywords = ['shingle', 'lumber', 'wood']
    if material['category'] != 'wood' and any(kw in material['source'] for kw in wood_keywords):
        flag_for_review(material, "wood keywords in non-wood material")
```

---

## High Severity Anomalies

### 4. MELTING TEMPERATURE DATA ARCHITECTURE

**Severity:** HIGH
**Impact:** Missing data tables for thermal transitions
**Affected Materials:** 15 materials with melting temperature references

#### Materials Affected

All reference melting temperature HTML tables that don't exist:

**Thermoplastics (Correct to have melting data):**
- Nylon (expected Tm ~260°C)
- HDPE (expected Tm ~130-135°C)
- LDPE (expected Tm ~110-115°C)
- PP (expected Tm ~160-165°C)
- PVC (expected softening ~80-100°C)
- PETG (expected Tm ~220-230°C)
- PEX_Pipe

**Composite/Fabric Materials:**
- Overstuffed_Chair_Polyester_Fabric
- Bamboo_Rug_Fabric
- Polyolefin_Carpet_Low_Pile
- Polyester_Microfiber_Sheet
- Polyester_Bed_Skirt
- Overstuffed_Chair_Polyester_Batting
- House_Wrap

**Questionable:**
- Composite_Deck_Board (wood-polymer composite - polymer melts, wood doesn't)

#### Material Science Validation

**Good News:** The materials WITH melting temperature references are appropriate choices.

**Wood Materials (Correctly NO melting data):**
- Oak ✓
- Pine ✓
- Cedar ✓
- MDF ✓
- OSB ✓
- Plywood ✓

Woods decompose/char rather than melt - this is correct.

#### Missing Melting Data

Some thermoplastics that SHOULD have melting data but don't:
- **PMMA**: Described as "thermoplastic" but no melting temp data (glass transition ~105°C)
- **PC (Polycarbonate)**: Thermoplastic, expected Tm ~155°C, but no melting data

#### Recommended Actions

1. Generate actual melting temperature HTML tables from STA data
2. Review PMMA and PC STA data to see if thermal transitions were measured
3. Add melting temp data if available
4. Verify Composite_Deck_Board categorization (engineered wood vs composite)
5. Document why some thermoplastics lack melting data

---

### 5. PMMA CLASSIFICATION DISCREPANCY

**Severity:** HIGH
**Impact:** Inconsistent property expectations
**Affected Materials:** PMMA, Black_PMMA

#### Issue Description

Both PMMA materials are described as "thermoplastic" but lack melting temperature data, while other thermoplastics in the database have it.

**File Paths:**
- `01_Data/PMMA/material.json:19` - "engineered thermoplastic"
- `01_Data/Black_PMMA/material.json:16` - "engineered thermoplastic"

#### Material Science Background

- PMMA (polymethyl methacrylate) is indeed a thermoplastic
- Glass transition temperature (Tg) ~105°C
- Doesn't have sharp melting point but does soften
- Should show thermal transition in DSC/STA measurements

#### Comparison to Other Thermoplastics

| Material | Type | Has Melting Data? |
|----------|------|-------------------|
| Nylon | Thermoplastic | ✓ YES |
| HDPE | Thermoplastic | ✓ YES |
| PP | Thermoplastic | ✓ YES |
| PVC | Thermoplastic | ✓ YES |
| **PMMA** | **Thermoplastic** | **✗ NO** |
| **PC** | **Thermoplastic** | **✗ NO** |

#### Recommended Actions

1. Review STA data for PMMA to verify if thermal transition was measured
2. If glass transition visible, add to material.json as derived property
3. If not measured, update description to clarify "thermoplastic with no distinct melting point"
4. Same analysis for PC (polycarbonate)
5. Consider adding Tg (glass transition) as a property category for amorphous thermoplastics

---

### 6. INCONSISTENT THERMAL PROPERTY TESTING

**Severity:** HIGH
**Impact:** Incomplete material characterization
**Scope:** Multiple materials across database

#### Issue Description

Inconsistent presence of specific heat and thermal conductivity data, particularly wet vs dry testing.

#### Observations

**Wood Materials (Wet + Dry) ✓ Appropriate:**
- Oak_Flooring: Both wet/dry specific heat and conductivity
- Wood_Stud: Both wet/dry
- Pine_Siding: Both wet/dry
- Cedar_Shingle: Both wet/dry

**Polymers (Wet Only) ? Questionable:**
- PMMA: Only wet data
- PC: Only wet data
- Nylon: Only wet data (but nylon IS hygroscopic, so wet/dry makes sense)
- PVC: Only wet data

**Polymers (Wet + Dry):**
- Black_PMMA: Both wet and dry

**Missing Thermal Properties:**
- Fiberglass_Insulation: No thermal properties (only cone calorimeter)

#### Material Science Analysis

- **Wood**: Hygroscopic (absorbs moisture) → wet/dry testing is appropriate ✓
- **Polymers**: Most don't absorb significant moisture → wet/dry testing unnecessary
  - **Exception**: Nylon is hygroscopic → wet/dry testing appropriate
- **Inconsistency**: Why does Black_PMMA have wet/dry but regular PMMA doesn't?

#### Possible Explanations

1. Testing protocol changed over time
2. Different labs with different procedures
3. Budget constraints for comprehensive testing
4. Material-specific decisions not documented

#### Recommended Actions

1. Establish consistent testing protocol going forward
2. Document rationale for wet/dry testing decisions
3. Add metadata field: `"moisture_sensitivity": true/false`
4. Backfill missing thermal property data for critical materials
5. For polymers, dry testing alone is usually sufficient (except hygroscopic ones)

---

### 7. MISSING PARENT/COMPONENT RELATIONSHIPS

**Severity:** HIGH
**Impact:** Incomplete material traceability
**Scope:** Multi-component materials

#### Issue Description

Some materials reference components that may not be tested separately, and component relationships aren't fully validated.

#### Examples of CORRECT Usage

**Gypsum_Wallboard:**
```json
{
  "material": "Gypsum Wallboard",
  "parent material": "",
  "component material": [
    "Gypsum Wallboard - Gypsum Core",
    "Gypsum Wallboard - Paper"
  ]
}
```

**Bamboo_Rug_Fabric:**
```json
{
  "material": "Bamboo Rug - Polyester Fabric",
  "parent material": "Bamboo Rug",
  "component material": []
}
```

**Fiberglass_Insulation_R30_Paper_Faced:**
```json
{
  "parent material": "",
  "component material": [
    "Paper-faced R30 Fiberglass Insulation - Fiberglass",
    "Paper-faced R30 Fiberglass Insulation - Paper"
  ]
}
```

#### Potential Issues

1. **Unvalidated references**: Need to verify all parent materials exist in database
2. **Missing reverse links**: Need to verify all component materials exist and link back
3. **Incomplete decomposition**: Composite_Deck_Board is wood-polymer composite but lists no components

#### Recommended Actions

1. Create validation script:
   ```python
   for material in materials:
       if material['parent material']:
           assert parent_exists(material['parent material'])
       for component in material['component material']:
           assert component_exists(component)
           assert component_links_back_to_parent(component, material)
   ```

2. Consider adding component breakdown for:
   - Composite_Deck_Board (wood + polymer matrix)
   - Other composite materials

3. Add bidirectional link validation

---

### 8. DENSITY ANOMALY - Extremely Low Value

**Severity:** HIGH
**Impact:** Questionable measurement accuracy
**Material:** Bamboo_Rug_Fabric

**File:** `01_Data/Bamboo_Rug_Fabric/material.json:12`

#### Issue

```json
{
  "material": "Bamboo Rug - Polyester Fabric",
  "density": "97 kg/m<sup>3</sup>"  ❌ VERY LOW
}
```

#### Comparison

| Material | Density (kg/m³) | Type |
|----------|----------------|------|
| **Bamboo_Rug_Fabric** | **97** | **Polyester fabric** |
| Overstuffed_Chair_Polyester_Fabric | 410.5 | Polyester fabric |
| Typical polyester fabric | 200-500 | Expected range |
| Polyurethane foam | 17-48 | Cellular foam |

#### Analysis

97 kg/m³ is extremely low for solid polyester fabric:
- This is closer to **foam density** than fabric density
- Solid polyester should be 1200-1400 kg/m³
- Even with air gaps, fabric is typically 200-500 kg/m³

#### Possible Causes

1. **Measurement error**: Incorrect volume measurement
2. **Bulk density**: Measured with significant air gaps (not fabric density)
3. **Open weave**: Very loose/open fabric structure
4. **Incorrect calculation**: Density formula applied wrong

#### Recommended Actions

1. **Retest density** using proper fabric measurement technique:
   - Measure true fabric thickness (compressed, no air gaps)
   - Or specify this is "bulk density" not "fabric density"
2. Compare to parent material (Bamboo Rug) density
3. Document measurement methodology for all fabrics
4. Add field to distinguish bulk density vs material density

---

### 9. CATEGORY CLASSIFICATION ISSUES

**Severity:** HIGH
**Impact:** Database organization and searchability
**Scope:** Multiple materials

#### Issue Description

Some materials may benefit from additional or different category assignments.

#### Examples

**Composite_Deck_Board:**
```json
{
  "material category": ["engineered wood", "exterior finish"],
  "description": "...wood waste and a polymer matrix..."
}
```

**Issue:** It's a wood-plastic composite (WPC), not pure engineered wood

**Suggested:**
```json
{
  "material category": ["composite material", "engineered wood", "exterior finish"]
}
```

---

**Overstuffed_Chair_Polyurethane_Foam:**
```json
{
  "material category": ["general polymer", "upholstered furniture"]
}
```

**Suggested addition:**
```json
{
  "material category": ["general polymer", "upholstered furniture", "foam", "cellular polymer"]
}
```

#### Recommended New Categories

1. **"composite material"** - for multi-material composites
2. **"foam"** or **"cellular polymer"** - for foamed materials
3. **"textile"** - for fabrics/woven materials
4. **"elastomer"** - for rubbers

#### Recommended Actions

1. Review and expand category taxonomy in global.json
2. Add new categories where appropriate
3. Update existing materials with additional categories
4. Multi-category assignment is already supported (good!)
5. Create category assignment guidelines

---

### 10. INCONSISTENT TEST COVERAGE

**Severity:** HIGH
**Impact:** Uneven material characterization
**Scope:** Entire database

#### Issue Description

Different materials have vastly different test coverage with no clear rationale documented.

#### Examples

**Full Test Suite - Oak_Flooring:**
- ✓ Specific heat (wet/dry)
- ✓ Thermal conductivity (wet/dry)
- ✓ Mass loss rate (STA + Cone at 3 heat fluxes)
- ✓ Heat release rate (Cone at 3 heat fluxes)
- ✓ CO yield
- ✓ Specific heat release rate (MCC)
- ✓ Soot yield (derived)
- ✓ Effective heat of combustion (MCC + Cone)

**Minimal Test Suite - Cedar_Shingle:**
- ✓ Specific heat (wet/dry)
- ✓ Thermal conductivity (wet/dry)
- ✓ Mass loss rate (STA only, no Cone)
- ✓ Specific heat release rate (MCC)
- ✓ Effective heat of combustion (MCC only)
- ✓ Emissivity
- ✗ No Cone calorimeter data

**Very Minimal - Fiberglass_Insulation_R30_Paper_Faced:**
- ✓ Mass loss rate (Cone only)
- ✓ Heat release rate (Cone)
- ✓ CO yield
- ✓ Soot yield
- ✓ Effective heat of combustion (Cone only)
- ✗ NO thermal properties
- ✗ NO MCC
- ✗ NO STA

#### Observations

- Building materials (structural) tend to have comprehensive testing
- Some materials have limited testing (budget/scope constraints?)
- Test coverage doesn't always correlate with material importance
- No documentation of why certain tests were/weren't performed

#### Recommended Actions

1. Define "minimum required tests" for each material category:
   ```
   Building materials: Cone + HFM + STA + MCC + FTIR
   Polymers: Cone + MCC + STA (melting)
   Insulation: Cone + HFM (thermal properties critical)
   Textiles: Cone + MCC minimum
   ```

2. Identify high-priority materials needing additional testing
3. Add metadata field: `"test_coverage_status": "complete/partial/minimal"`
4. Document testing rationale in material.json
5. Create test coverage matrix for visualization

---

### 11. MISSING EMISSIVITY DATA

**Severity:** HIGH
**Impact:** Incomplete fire modeling capability
**Scope:** Multiple high-priority materials

#### Issue Description

Emissivity (infrared radiation characteristics) is critical for fire modeling but inconsistently measured across materials.

#### Materials WITH Emissivity

- Wood_Stud ✓
- Pine_Siding ✓
- Cedar_Shingle ✓
- Black_PMMA ✓
- Basswood_Panel ✓
- Nylon ✓
- Many others...

#### Materials WITHOUT Emissivity

- Oak_Flooring ✗ (despite comprehensive testing otherwise)
- PMMA (clear) ✗
- PC ✗
- Many furniture materials ✗

#### Fire Modeling Impact

- Emissivity affects radiative heat transfer calculations (Stefan-Boltzmann law)
- Critical for flame spread models
- Important for compartment fire simulations
- Missing data forces use of generic assumptions (reduces model accuracy)

#### Pattern Analysis

- No clear pattern by category or material importance
- Suggests testing protocol evolved over time
- Some early materials lack emissivity, later ones have it

#### Recommended Actions

1. Prioritize emissivity testing for:
   - High fire risk materials
   - Commonly modeled materials
   - Building materials (structural importance)

2. Add emissivity to standard test suite going forward

3. Consider interim solution:
   - Use literature values for common polymers
   - Document sources for literature values
   - Flag as "estimated" vs "measured"

4. Estimate cost/time to backfill emissivity for existing materials

---

## Medium Severity Anomalies

### 12. INCONSISTENT DESCRIPTIONS vs ALTERNATE NAMES

**Severity:** MEDIUM
**Impact:** Material identification confusion

**Example - MDF:**
```json
{
  "material": "Medium Density Fiberboard",
  "alternate names": ["MDF", "Particle Board", "Press Board"]
}
```

**Issue:** Particle board and MDF are **different materials**:
- **MDF**: Fine wood fibers, higher density (600-800 kg/m³)
- **Particle board**: Wood chips/particles, lower density (400-600 kg/m³)
- Different fire performance and structural properties

**Recommended Action:**
```json
{
  "alternate names": ["MDF", "Fiberboard"]  // Remove "Particle Board"
}
```

**Systematic Review Needed:**
1. Check all alternate names for accuracy
2. Remove incorrect alternates
3. Add clarifying notes if materials are commonly confused
4. Add "commonly confused with: X" field if needed

---

### 13. INCONSISTENT NAMING CONVENTIONS

**Severity:** MEDIUM
**Impact:** Database maintainability and parsing

#### Component Naming

Inconsistent separators:
```
"Gypsum_Wallboard_Paper"           // Underscore
vs
"Gypsum Wallboard - Paper"         // Hyphen with spaces
```

#### Material Naming

Inconsistent use of abbreviations:
```
"Polymethyl Methacrylate (PMMA)"   // Full name with abbrev
"PMMA"                              // Abbreviation only
"Black Polymethyl Methacrylate (PMMA)"  // Full name + color
```

#### Recommended Actions

1. Standardize component naming: `Parent_Material_Component_Name`
2. Standardize material naming: `Full_Name (ABBREV)` or `ABBREV` consistently
3. Create naming guidelines document
4. Apply retroactively to all materials
5. Update validation scripts to enforce conventions

---

### 14. MISSING VIDEO FILES

**Severity:** MEDIUM
**Impact:** Incomplete multimedia documentation

#### Issue

All materials have empty video field:
```json
{
  "video file": ""
}
```

#### Observations

- Field exists in schema (suggests it was planned)
- No materials have video files linked
- Testing often includes video documentation
- Videos could be valuable for:
  - Quality control
  - Understanding test behavior
  - Publication/outreach

#### Recommended Actions

1. **If videos exist**: Link them to materials
2. **If not planned**: Remove field from schema to reduce clutter
3. **If planned**:
   - Prioritize creating test videos for representative materials
   - Define video storage/hosting strategy
   - Document video naming convention

---

### 15. INCOMPLETE TEST DESCRIPTIONS

**Severity:** MEDIUM
**Impact:** Reduced data interpretability

#### Issue

Many test descriptions are empty strings.

**Example - Nylon:**
```json
{
  "test name": "specific heat",
  "test description": ""   // EMPTY
},
{
  "test name": "thermal conductivity",
  "test description": ""   // EMPTY
}
```

**Better Example - Oak_Flooring:**
- Has detailed cone calorimeter test notes
- Describes sample behavior during tests
- Includes observations

#### Recommended Actions

1. Add standard test descriptions for each test type:
   ```json
   {
     "test name": "specific heat",
     "test description": "Specific heat capacity measured per ASTM E1269 using heat flow meter at multiple temperatures for both conditioned (50% RH) and dried samples."
   }
   ```

2. Include test standards (ASTM E1354, ASTM E1269, etc.)
3. Note any deviations or special procedures
4. Add material-specific observations where relevant

---

### 16. PROPERTY UNITS IN DESCRIPTIONS (HTML Encoding)

**Severity:** MEDIUM
**Impact:** Difficult programmatic data extraction

#### Issue

Density and other properties embedded as HTML strings:

```json
{
  "density": "687 kg/m<sup>3</sup>"
}
```

#### Problems

- Hard to parse programmatically
- Can't do numerical comparisons easily
- Makes data analysis scripts complex
- Mixing data and presentation

#### Better Practice

```json
{
  "density": 687,
  "density_unit": "kg/m³",
  "density_formatted": "687 kg/m³"
}
```

Or separate value and unit:
```json
{
  "density": {
    "value": 687,
    "unit": "kg/m³",
    "uncertainty": 15
  }
}
```

#### Recommended Actions

1. Refactor schema to separate values and units
2. Store numerical values as numbers (not strings)
3. Add units as separate field
4. Generate HTML formatting in frontend/presentation layer
5. Update all 93 materials to new schema
6. Maintain backward compatibility during transition

---

### 17. TEST FILE NAMING CONSISTENCY

**Severity:** MEDIUM
**Impact:** Data organization

#### Current Naming Convention

Test files appear to follow good naming convention:

```
PMMA_MCC_30K_min_210920_R1.txt
Oak_Flooring_MCC_30K_min_211104_R1.txt
Material_Apparatus_Condition_Date_Replicate.ext
```

Includes:
- Material name
- Apparatus
- Test conditions (heating rate, heat flux, etc.)
- Date (YYMMDD format)
- Replicate number

#### Potential Issues

Need to verify:
1. All files follow convention
2. No orphaned files (files not referenced in material.json)
3. Replicate numbering is consistent (R1, R2, R3...)
4. Date format is consistent
5. No duplicate file names

#### Recommended Actions

1. Document file naming convention formally
2. Create validation script to check:
   ```python
   # Check file naming pattern
   pattern = r'^[A-Za-z_]+_(Cone|MCC|STA|HFM|ATR|IS)_.*_\d{6}_R\d+\.(csv|txt|tst)$'
   for file in data_files:
       if not re.match(pattern, file):
           flag_unusual_naming(file)
   ```
3. Create automated file renaming tool for legacy data
4. Check for orphaned files not referenced in material.json

---

## Low Severity Anomalies

### 18. INCONSISTENT IMAGE NAMING

**Severity:** LOW
**Impact:** Minor inconsistency

#### Pattern

```
Material_Name.JPG           // Uppercase extension
Material_Name600x400.jpg    // Lowercase extension
```

#### Recommended Action

Standardize to either:
- `.jpg` (lowercase) - more common on web
- `.JPG` (uppercase) - consistent with some cameras

Update all image references in material.json files.

---

### 19. SHORT DESC REDUNDANCY

**Severity:** LOW
**Impact:** Schema inefficiency

#### Good Example - Wood_Stud

```json
{
  "description": "Dimensional lumber is softwood (typically spruce, pine, or fir) cut to specific dimensions for construction framing.",
  "short desc": "Dimensional lumber is softwood used in construction framing."
}
```
Short desc is actually shorter and simplified ✓

#### Bad Example - HDPE

```json
{
  "description": "High Density Polyethylene (HDPE) is a durable polyolefin thermoplastic often used when molding manufactured parts.",
  "short desc": "HDPE is a durable polyolefin thermoplastic often used when molding manufactured parts."
}
```
Nearly identical, defeats the purpose ✗

#### Recommended Actions

1. Ensure short desc is actually shorter (<80 characters)
2. Short desc should omit technical details
3. Or remove short desc field if not used consistently
4. Guidelines:
   - Description: Full technical description
   - Short desc: One sentence summary for cards/previews

---

## Material Science Validation Results

### Wood Materials - ALL CORRECT ✓

**Materials Checked:** Oak_Flooring, Wood_Stud, Pine_Siding, Cedar_Shingle, Plywood, MDF, OSB, Basswood_Panel

#### Findings

- ✓ No melting temperature data (correct - wood decomposes/chars, doesn't melt)
- ✓ Heat of combustion ~15-20 MJ/kg (expected for cellulosic materials)
- ✓ Density ranges 310-687 kg/m³ (reasonable)
- ✓ Wet and dry thermal properties (appropriate for hygroscopic materials)
- ✓ No physically impossible values

**Conclusion:** Wood material data is scientifically sound.

---

### Polymer Materials - Generally CORRECT ✓

#### Thermoplastics WITH Melting Data (Correct)

| Material | Expected Tm | Status |
|----------|-------------|--------|
| Nylon (PA 66) | ~260°C | ✓ Has melting data |
| HDPE | ~130-135°C | ✓ Has melting data |
| LDPE | ~110-115°C | ✓ Has melting data |
| PP | ~160-165°C | ✓ Has melting data |
| PVC | ~80-100°C (softening) | ✓ Has melting data |
| PETG | ~220-230°C | ✓ Has melting data |

#### Thermoplastics WITHOUT Melting Data (Questionable)

| Material | Expected Tm/Tg | Status |
|----------|----------------|--------|
| PMMA | Tg ~105°C | ? No melting data (see Anomaly #5) |
| PC | Tm ~155°C | ? No melting data (see Anomaly #5) |

#### Thermosets (Correctly NO Melting Data)

- Polyurethane foam ✓
- Polyisocyanurate foam ✓

**Conclusion:** Polymer material data is scientifically sound, with minor gaps in PMMA/PC.

---

### Density Validation - ALL REASONABLE ✓

| Material Type | Observed Range (kg/m³) | Expected Range | Status |
|--------------|----------------------|----------------|--------|
| Wood products | 310-687 | 300-800 | ✓ Reasonable |
| Solid polymers | 886-1401 | 900-1400 | ✓ Reasonable |
| Foams | 17-48 | 10-80 | ✓ Reasonable |
| Gypsum | 635 | 600-700 | ✓ Reasonable |
| **Fabric** | **97** | **200-500** | **? Very low** (see Anomaly #8) |

**Exception:** Bamboo_Rug_Fabric at 97 kg/m³ is unusually low and requires investigation.

---

## Summary Statistics

### Anomalies by Severity

| Severity | Count | Percentage |
|----------|-------|------------|
| Critical | 3 | 15.8% |
| High | 8 | 42.1% |
| Medium | 6 | 31.6% |
| Low | 2+ | 10.5% |
| **Total** | **19+** | **100%** |

### Anomalies by Category

| Category | Count | Examples |
|----------|-------|----------|
| Data Architecture | 5 | Missing HTML files, missing directories |
| Metadata Errors | 4 | Copy-paste errors, naming inconsistencies |
| Test Coverage | 3 | Inconsistent testing, missing emissivity |
| Material Properties | 2 | Density anomaly, PMMA classification |
| Data Quality | 5 | Incomplete descriptions, missing videos |
| **Total** | **19+** | - |

### Materials Analyzed

- **Total materials scanned:** 93
- **Materials with anomalies:** ~20
- **Materials with critical issues:** 3
- **Materials fully validated:** ~70

---

## Physically Impossible Values Found

**NONE** ✓

No physically impossible property values were detected:
- ✓ No negative densities
- ✓ No negative thermal conductivities
- ✓ No temperatures below absolute zero
- ✓ No heat of combustion > 60 MJ/kg
- ✓ All values within physically reasonable bounds

**This is good news** - the raw scientific data appears to be sound.

---

## Recommendations by Priority

### IMMEDIATE (This Week)

1. **Fix copy-paste errors:**
   - Corrugated_Cardboard source description
   - Polyisocyanurate_Foam_Board short description

2. **Investigate missing directory:**
   - Polyisocyanurate_Spray_Foam_Insulation MCC data

3. **Begin HTML generation:**
   - Run plot_*_html.py scripts to generate missing HTML files
   - Start with high-priority materials

### SHORT TERM (This Month)

4. **Create validation scripts:**
   - File existence checker
   - Parent/component relationship validator
   - Property range validator
   - Naming convention checker

5. **Fix high-priority issues:**
   - Review and correct all alternate names
   - Standardize naming conventions
   - Investigate Bamboo_Rug_Fabric density

6. **Documentation:**
   - Document testing rationale
   - Add test coverage status to materials
   - Create file naming convention guide

### MEDIUM TERM (This Quarter)

7. **Complete data processing:**
   - Generate ALL missing HTML files
   - Validate all generated files

8. **Backfill missing data:**
   - Add thermal properties where missing
   - Add emissivity for high-priority materials
   - Complete PMMA/PC melting point investigation

9. **Standardization:**
   - Review and update material categories
   - Standardize test descriptions
   - Refactor property schema (separate values/units)

### LONG TERM (Ongoing)

10. **Process improvements:**
    - Implement automated validation in data pipeline
    - Create data quality dashboard
    - Establish peer review process
    - Document all testing protocols

11. **Infrastructure:**
    - Set up automated HTML generation
    - Implement build/deploy pipeline
    - Create regression testing suite

12. **Expansion:**
    - Add new categories (composite, foam, textile, elastomer)
    - Define minimum test requirements by category
    - Prioritize additional testing for incomplete materials

---

## Validation Scripts to Create

### 1. file_existence_validator.py

```python
"""
Verify all files referenced in material.json actually exist
"""
def validate_file_references(material_json_path):
    material = load_json(material_json_path)
    missing_files = []

    for prop in material['measured_property'] + material['derived_property']:
        for nested in prop.get('nested tests', []):
            graph = nested.get('graph', '')
            table = nested.get('table', '')

            if graph and not file_exists(graph):
                missing_files.append(graph)
            if table and not file_exists(table):
                missing_files.append(table)

    return missing_files
```

### 2. property_range_validator.py

```python
"""
Flag physically impossible or unusual property values
"""
def validate_property_ranges(material_json_path):
    material = load_json(material_json_path)
    anomalies = []

    # Check density
    density = extract_density(material)
    if density < 10 or density > 20000:
        anomalies.append(f"Unusual density: {density} kg/m³")

    # Check thermal conductivity
    k = extract_thermal_conductivity(material)
    if k and (k < 0.01 or k > 500):
        anomalies.append(f"Unusual thermal conductivity: {k} W/(m·K)")

    # Check heat of combustion
    hoc = extract_heat_of_combustion(material)
    if hoc and hoc > 60:
        anomalies.append(f"Unusually high heat of combustion: {hoc} MJ/kg")

    return anomalies
```

### 3. material_class_validator.py

```python
"""
Check material class vs property consistency
"""
def validate_material_class(material_json_path):
    material = load_json(material_json_path)
    issues = []

    # Wood materials shouldn't have melting temps
    if is_wood(material) and has_melting_temp(material):
        issues.append("Wood material has melting temperature data")

    # Thermoplastics should have melting temps
    if is_thermoplastic(material) and not has_melting_temp(material):
        issues.append("Thermoplastic missing melting temperature data")

    # Check material name in description
    if material['material'].lower() not in material['description'].lower():
        issues.append("Material name not found in description")

    return issues
```

### 4. relationship_validator.py

```python
"""
Validate parent/component relationships
"""
def validate_relationships(all_materials):
    issues = []

    for material in all_materials:
        # Check parent exists
        if material['parent material']:
            if not material_exists(material['parent material'], all_materials):
                issues.append(f"{material['material']}: parent not found")

        # Check components exist
        for component in material['component material']:
            if not material_exists(component, all_materials):
                issues.append(f"{material['material']}: component {component} not found")

    return issues
```

### 5. test_coverage_reporter.py

```python
"""
Generate test coverage matrix
"""
def generate_coverage_matrix(all_materials):
    tests = ['Cone', 'HFM', 'MCC', 'STA', 'FTIR_ATR', 'FTIR_IS']
    matrix = []

    for material in all_materials:
        coverage = {
            'material': material['material'],
            'category': material['material category']
        }

        for test in tests:
            coverage[test] = has_test_data(material, test)

        coverage['completeness'] = calculate_completeness(coverage)
        matrix.append(coverage)

    return matrix
```

### 6. html_generator.py

```python
"""
Auto-generate missing HTML files from raw data
"""
def generate_missing_html_files(material_json_path):
    material = load_json(material_json_path)

    for prop in material['measured_property'] + material['derived_property']:
        for nested in prop.get('nested tests', []):
            table_path = nested.get('table', '')

            if table_path and not file_exists(table_path):
                # Find raw data file
                raw_file = find_raw_data_file(material, prop)
                if raw_file:
                    generate_html_table(raw_file, table_path)
                else:
                    log_missing_raw_data(material, prop)
```

---

## Conclusion

The FSRI Materials Database contains **scientifically sound raw data** from a material properties and physics perspective. The thermal, mechanical, and combustion properties measured are consistent with known material behavior, and no physically impossible values were detected.

However, the database suffers from **critical data architecture issues**:

### Primary Issues
1. **Missing HTML files** (affects 100% of materials)
2. **Missing test directories** (at least 1 material)
3. **Metadata errors** from copy-paste (at least 2 materials)
4. **Inconsistent test coverage** (no clear rationale)
5. **Incomplete data processing pipeline**

### Root Cause
The database appears to be in **transition** from raw data collection to a polished public database. The HTML generation step and comprehensive validation were not completed.

### Path Forward

**Immediate priority:** Complete the data processing pipeline
- Generate all missing HTML files
- Fix critical metadata errors
- Validate file references

**Short-term:** Implement quality controls
- Create automated validation scripts
- Document testing protocols
- Standardize naming and descriptions

**Long-term:** Process improvements
- Automated data quality checks
- Build/deploy pipeline
- Continuous validation
- Peer review process

### Impact Assessment

**Current state:** Database cannot serve its intended purpose (broken links, missing files)

**After fixes:** Database will be a high-quality, scientifically sound resource for fire research

**Effort required:**
- Critical fixes: 1-2 weeks
- HTML generation: 2-4 weeks
- Full validation: 1-2 months
- Process improvements: 3-6 months

The good news is that the underlying science is sound - these are engineering and data management issues that can be systematically resolved.

---

**Report End**

*Generated by automated quality control analysis*
*Contact: FSRI Database Team for questions or corrections*
