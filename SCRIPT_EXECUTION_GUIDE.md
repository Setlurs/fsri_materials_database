# Script Execution Guide - FSRI Materials Database

**Date:** 2026-02-02
**Status:** Scripts fixed for compatibility, ready for execution

---

## Summary

The data processing and HTML generation scripts have been updated for compatibility with modern Python libraries (pandas 3.0, scipy 1.17). The scripts are now ready to run, but processing all 152+ materials will take significant time (estimated 2-6 hours depending on hardware).

---

## Compatibility Fixes Applied

### 1. scipy.integrate.trapz → scipy.integrate.trapezoid

**Issue:** `scipy.integrate.trapz` was deprecated in scipy 1.14 and removed in later versions.

**Fix Applied:** Replaced all instances in:
- `plot_MCC_data.py`
- `plot_MCC_data_html.py`
- `plot_STA_data.py`
- `plot_STA_data_html.py`
- `plot_IS_emissivity_data.py`
- `plot_IS_emissivity_data_html.py`

**Command used:**
```bash
sed -i '' 's/integrate\.trapz/integrate.trapezoid/g' plot_*.py
```

---

### 2. pandas 3.0 Type Strictness

**Issue:** Pandas 3.0 enforces strict type checking. Scripts were trying to set float values into string columns.

**Fix Applied in `plot_Cone_data.py`:**

**Before:**
```python
output_df.at['Peak HRRPUA (kW/m2)', label] = float("{:.2f}".format(max(data_temp_df['HRRPUA'])))
```

**After:**
```python
output_df.at['Peak HRRPUA (kW/m2)', label] = "{:.2f}".format(max(data_temp_df['HRRPUA']))
```

**Batch replacement command:**
```bash
sed -i '' 's/= float("{/= "{/g' plot_Cone_data.py plot_Cone_data_html.py
```

---

### 3. pandas 3.0 Chained Assignment

**Issue:** Pandas 3.0 throws errors for chained assignment which doesn't actually modify the original DataFrame.

**Fix Applied in `plot_Cone_data.py`:**

**Before:**
```python
data_temp_df['MLR'][data_temp_df['MLR'] > 5] = 0
data_temp_df['SPR'][data_temp_df['SPR'] < 0] = 0
```

**After:**
```python
data_temp_df.loc[data_temp_df['MLR'] > 5, 'MLR'] = 0
data_temp_df.loc[data_temp_df['SPR'] < 0, 'SPR'] = 0
```

---

### 4. NaN Value Handling in plot_HFM_data.py

**Issue:** Some materials have no HFM data, resulting in NaN values that can't be used in `math.floor()`.

**Fix Applied:**

**Before:**
```python
ylims[0] = 0.05 * (math.floor(y_min/0.05)-1)
ylims[1] = 0.05 * (math.ceil(y_max/0.05)+1)
```

**After:**
```python
if not math.isnan(y_min) and not math.isnan(y_max):
    ylims[0] = 0.05 * (math.floor(y_min/0.05)-1)
    ylims[1] = 0.05 * (math.ceil(y_max/0.05)+1)
```

---

## Virtual Environment Setup

A Python virtual environment has been created with all required packages:

**Location:** `/Users/anand/Projects/gh-stuff/fsri_materials_database/venv/`

**Packages Installed:**
```
pandas==3.0.0
numpy==2.4.2
scipy==1.17.0
matplotlib==3.10.8
plotly==6.5.2
GitPython==3.1.46
pybaselines==1.2.1
```

**To activate:**
```bash
source venv/bin/activate
```

---

## How to Run the Scripts

### Option 1: Run All Scripts (Recommended for Production)

This will process ALL materials and generate all PDF and HTML outputs:

```bash
cd 02_Scripts
source ../venv/bin/activate

# Generate PDF plots and analysis CSVs (Est. 1-2 hours)
bash run_all_data.sh

# Generate HTML interactive visualizations (Est. 1-2 hours)
bash run_all_data_html.sh

# Or run both sequentially
bash run_all_data.sh && bash run_all_data_html.sh
```

**Total estimated time:** 2-4 hours for all 152+ materials

---

### Option 2: Run Individual Scripts (Recommended for Testing)

Test with individual apparatus types first:

```bash
cd 02_Scripts
source ../venv/bin/activate

# Test ATR data (fastest, ~1-2 minutes)
python plot_ATR_data.py

# Test MCC data (~5-10 minutes)
python plot_MCC_data.py

# Test Cone data (longest, ~30-60 minutes)
python plot_Cone_data.py

# Then HTML versions
python plot_ATR_data_html.py
python plot_MCC_data_html.py
python plot_Cone_data_html.py
```

---

### Option 3: Run in Background (Recommended for Long Jobs)

Run the scripts in background and monitor progress:

```bash
cd 02_Scripts
source ../venv/bin/activate

# Run in background
nohup bash run_all_data.sh > ../run_all_data.log 2>&1 &

# Monitor progress
tail -f ../run_all_data.log

# When complete, run HTML generation
nohup bash run_all_data_html.sh > ../run_all_data_html.log 2>&1 &
tail -f ../run_all_data_html.log
```

---

## What Each Script Does

### PDF Generation Scripts (plot_*_data.py)

| Script | Purpose | Output |
|--------|---------|--------|
| `plot_ATR_data.py` | FTIR ATR spectroscopy plots | PDF graphs, processed CSVs |
| `plot_Cone_data.py` | Cone calorimeter analysis | PDF graphs, analysis CSVs, notes |
| `plot_HFM_data.py` | Thermal properties (k, Cp) | PDF graphs, thermal data tables |
| `plot_MCC_data.py` | Micro-scale combustion | PDF graphs, heat of combustion tables |
| `plot_STA_data.py` | Thermal analysis (TGA/DSC) | PDF graphs, melting temp tables |
| `plot_IS_emissivity_data.py` | Integrating sphere emissivity | PDF graphs, emissivity tables |

### HTML Generation Scripts (plot_*_data_html.py)

| Script | Purpose | Output |
|--------|---------|--------|
| `plot_ATR_data_html.py` | Interactive ATR spectra | HTML Plotly visualizations |
| `plot_Cone_data_html.py` | Interactive cone cal. plots | HTML tables and graphs |
| `plot_HFM_data_html.py` | Interactive thermal plots | HTML tables and graphs |
| `plot_MCC_data_html.py` | Interactive MCC plots | HTML tables and graphs |
| `plot_STA_data_html.py` | Interactive TGA/DSC plots | HTML tables and graphs |
| `plot_IS_emissivity_data_html.py` | Interactive emissivity | HTML tables and graphs |
| `plot_Furniture_Cal_data_html.py` | Full-scale furniture tests | HTML visualizations |

---

## Expected Output Locations

### Generated in 01_Data/

For each material, generated files go into apparatus subdirectories:

```
01_Data/
└── [Material_Name]/
    ├── Cone/
    │   ├── *_Analysis_Data.csv    (NEW)
    │   ├── *_Notes.csv             (NEW)
    │   ├── *_CO_Yields.csv         (NEW)
    │   ├── *_Soot_Yields.csv       (NEW)
    │   └── *_*.html                (NEW - interactive plots)
    ├── HFM/
    │   ├── *_thermal_conductivity.html  (NEW)
    │   └── *_specific_heat.html         (NEW)
    ├── MCC/
    │   ├── *_Heats_of_Combustion.html   (NEW)
    │   └── *_Ignition_Temps.html        (NEW)
    └── STA/
        ├── *_Analysis_Melting_Temp_Table.html  (NEW)
        └── *_*.html                             (NEW)
```

### Generated in 03_Charts/

Mirror structure with PDF plots:

```
03_Charts/
└── [Material_Name]/
    ├── Cone/
    │   ├── *_HRRPUA.pdf
    │   ├── *_MLR.pdf
    │   ├── *_SPR.pdf
    │   └── *_Extinction_Coefficient.pdf
    ├── HFM/
    │   ├── *_Thermal_Conductivity.pdf
    │   └── *_Specific_Heat.pdf
    ├── MCC/
    │   └── *_HRR.pdf
    └── STA/
        ├── *_Mass_Loss.pdf
        ├── *_MLR.pdf
        ├── *_Heat_Capacity.pdf
        └── *_DSC_Derivative.pdf
```

---

## Progress Monitoring

### Monitor Script Output

```bash
# If running in foreground, you'll see:
Running Data Processing Scripts
***** plot_ATR_data.py
MDF ATR
OSB ATR
...

***** plot_Cone_data.py
ABS Cone
Black_PMMA Cone
...
```

### Check Generated Files

```bash
# Count generated HTML files
find 01_Data -name "*.html" -newer 01_Data/README.md | wc -l

# Count generated PDF files
find 03_Charts -name "*.pdf" -newer 03_Charts/README.md | wc -l

# List recently modified materials
ls -lt 01_Data/*/Cone/*Analysis_Data.csv | head -10
```

---

## Known Issues and Warnings

### 1. Long Execution Time

Processing all materials takes **2-6 hours** depending on hardware:
- Cone calorimeter: Slowest (multiple heat fluxes, many calculations)
- STA: Moderate (melting point detection with baseline correction)
- HFM: Moderate (UTF-16 file parsing)
- MCC: Fast (simple integration)
- ATR: Fastest (simple averaging)

### 2. Memory Usage

Some materials have large datasets:
- Peak memory usage: ~2-4 GB
- Mostly matplotlib figure generation
- Should be fine on modern systems

### 3. Disk Space

Generated outputs:
- HTML files: ~500 KB each × ~1500 files = ~750 MB
- PDF files: ~100 KB each × ~2000 files = ~200 MB
- Total: ~1 GB of new files

### 4. Materials Without Data

Some materials may not have all apparatus data:
- Scripts will skip materials without required files
- Check logs for "No files found" messages
- This is normal and expected

### 5. Warnings You Can Ignore

These warnings are cosmetic and don't affect output:
- `FutureWarning: The behavior of DataFrame concatenation...` (pandas)
- `UserWarning: Matplotlib is currently using agg...` (matplotlib backend)
- Git warnings about detached HEAD (if repo is in detached state)

---

## Verification Steps

After running all scripts, verify outputs:

### 1. Check for Generated HTML Files

```bash
# Should find 1000+ HTML files
find 01_Data -name "*.html" | wc -l

# Check specific material
ls -lh 01_Data/PMMA/MCC/*.html
ls -lh 01_Data/Oak_Flooring/Cone/*.html
```

**Expected:**
- Each material should have HTML files in apparatus directories
- Tables for heat of combustion, melting temps, etc.
- Interactive plots for all test types

### 2. Check for Generated PDF Files

```bash
# Should find 2000+ PDF files
find 03_Charts -name "*.pdf" | wc -l

# Check specific material
ls -lh 03_Charts/PMMA/
ls -lh 03_Charts/Oak_Flooring/Cone/
```

**Expected:**
- Each material should have PDF plots in 03_Charts
- Multiple plots per apparatus type

### 3. Check for Analysis CSV Files

```bash
# Should find 100+ analysis CSV files
find 01_Data -name "*_Analysis_Data.csv" | wc -l
find 01_Data -name "*_Notes.csv" | wc -l
```

**Expected:**
- Cone materials should have Analysis_Data.csv and Notes.csv
- Files should contain tabular results

### 4. Check File Sizes

```bash
# HTML files should be reasonable size (1-500 KB)
find 01_Data -name "*.html" -exec du -h {} \; | head -20

# PDF files should be reasonable size (10-500 KB)
find 03_Charts -name "*.pdf" -exec du -h {} \; | head -20
```

**Red flags:**
- HTML files < 1 KB (might be empty/error)
- PDF files > 5 MB (might indicate issue)

---

## Troubleshooting

### Issue: Script Fails with "No module named 'X'"

**Solution:**
```bash
source venv/bin/activate
pip install X
```

### Issue: Script Fails with "KeyError: 'Column Name'"

**Cause:** Data file format doesn't match expected structure

**Solution:**
- Check if raw data file exists
- Verify column names in raw data file
- Material may have different format (older/newer version)
- Skip that material or update script for new format

### Issue: "Permission denied" on Script Execution

**Solution:**
```bash
chmod +x run_all_data.sh
chmod +x run_all_data_html.sh
```

### Issue: Scripts Running Very Slowly

**Cause:** Normal for large datasets

**Solution:**
- Run in background: `nohup bash run_all_data.sh &`
- Or run overnight
- Or process subset of materials first

### Issue: "Disk full" Error

**Cause:** Generating ~1 GB of output files

**Solution:**
- Free up disk space
- Or process materials in batches
- Or generate to external drive

---

## After Running Scripts

### 1. Re-run Anomaly Detection

After HTML generation completes, the anomaly report needs updating:

**Critical Anomaly #1 should be RESOLVED:**
- ✓ HTML files now exist
- ✓ Referential integrity restored
- ✓ Database can serve data

**Remaining anomalies to check:**
- Missing MCC directory (Polyisocyanurate_Spray_Foam_Insulation)
- Copy-paste metadata errors (2 materials)
- Density anomaly (Bamboo_Rug_Fabric)
- Other high/medium severity issues

### 2. Validate Generated Files

Run the proposed validation scripts:

```bash
cd 02_Scripts/Utilities
source ../../venv/bin/activate

# Validate all material.json references now work
python json_validation.py

# Check for remaining missing files
python file_existence_validator.py  # (to be created)
```

### 3. Update Material Status

Check which materials now have complete data:

```bash
# Find materials with complete test coverage
for mat in 01_Data/*/; do
    matname=$(basename "$mat")
    echo -n "$matname: "
    ls "$mat"/Cone/*.html 2>/dev/null | wc -l | xargs echo -n "Cone:"
    ls "$mat"/HFM/*.html 2>/dev/null | wc -l | xargs echo -n " HFM:"
    ls "$mat"/MCC/*.html 2>/dev/null | wc -l | xargs echo -n " MCC:"
    ls "$mat"/STA/*.html 2>/dev/null | wc -l | xargs echo -n " STA:"
    echo
done
```

### 4. Commit Generated Files (Optional)

**Note:** The `.gitignore` currently excludes generated HTML files. You may want to:

**Option A:** Keep generated files out of git (recommended for development)
```bash
# Files stay local, regenerate as needed
```

**Option B:** Commit generated files (recommended for deployment)
```bash
# Edit .gitignore to allow HTML files
git add 01_Data/*/Cone/*.html
git add 01_Data/*/HFM/*.html
git add 01_Data/*/MCC/*.html
git add 01_Data/*/STA/*.html
git commit -m "Add generated HTML data tables and interactive visualizations"
```

---

## Summary of What Was Fixed

✓ **scipy compatibility:** `trapz` → `trapezoid` in 6 scripts
✓ **pandas 3.0 compatibility:** Fixed type errors in Cone data scripts
✓ **pandas 3.0 compatibility:** Fixed chained assignment warnings
✓ **NaN handling:** Fixed HFM data script for materials without thermal data
✓ **Virtual environment:** Created with all required packages
✓ **Scripts ready:** All scripts updated and ready to execute

---

## Next Steps

1. **Run the scripts** (choose option based on time available):
   - Option 1: Full run (2-6 hours) - `bash run_all_data.sh && bash run_all_data_html.sh`
   - Option 2: Test run (10-30 min) - Run individual scripts
   - Option 3: Background run - `nohup bash run_all_data.sh &`

2. **Verify outputs:**
   - Check HTML files exist in 01_Data
   - Check PDF files exist in 03_Charts
   - Verify file sizes are reasonable

3. **Re-run anomaly detection:**
   - Critical Anomaly #1 should be resolved
   - Generate updated ANOMALY_DETECTION_REPORT.md
   - Document remaining issues

4. **Fix remaining anomalies:**
   - Copy-paste errors (2 materials) - quick fix
   - Missing MCC directory (1 material) - needs investigation
   - Density anomaly (1 material) - needs retesting

---

**End of Guide**

*For questions or issues, refer to REPOSITORY_ANALYSIS.md and ANOMALY_DETECTION_REPORT.md*
