# FSRI Materials Database - Repository Analysis

**Generated:** 2026-02-02
**Repository:** fsri_materials_database
**Branch:** main
**Last Commit:** d492829 (Merge pull request #264 from ulfsri/staging)

---

## NEW: Staging Branch Update

**MAJOR UPDATE IN PROGRESS**: The `staging` branch contains **29 new Personal Protective Equipment (PPE) materials** ready to be merged to main. This represents the largest single expansion of the database focused on structural firefighter equipment.

**Key Highlights:**
- **29 new PPE materials** covering helmet components, SCBA parts, and turnout gear
- **524 files changed** with 1,169,455 lines added
- All new materials tested with STA (Simultaneous Thermal Analyzer)
- Comprehensive coverage of firefighter equipment from major manufacturers (Cairns, MSA, Drager, Honeywell, etc.)
- Repository will grow from 152 to **181+ materials** upon merge

See Section 10.2 for complete details on staging branch activity.

---

## Table of Contents

1. [Overall Purpose](#1-overall-purpose)
2. [Project Structure](#2-project-structure)
3. [Data Structure](#3-data-structure)
4. [Code Organization](#4-code-organization)
5. [Key Technologies](#5-key-technologies)
6. [Data Processing Pipeline](#6-data-processing-pipeline)
7. [Configuration](#7-configuration)
8. [Documentation](#8-documentation)
9. [Entry Points & Usage](#9-entry-points--usage)
10. [Recent Activity & Git History](#10-recent-activity--git-history)
11. [Material Categories & Database Coverage](#11-material-categories--database-coverage)
12. [Measured vs. Derived Quantities](#12-measured-vs-derived-quantities)
13. [Key Data Characteristics](#13-key-data-characteristics)
14. [File Paths Reference](#14-file-paths-reference)

---

## 1. Overall Purpose

The **FSRI Materials and Products Database** is a comprehensive, open-source repository of materials and products properties and fire test data developed by the Fire Safety Research Institute (part of UL Research Institutes).

### Primary Objectives

- **Improve fire modeling**: Provide empirical data from standardized fire tests to enhance fire behavior prediction models
- **Support fire investigation**: Enable investigators and researchers to compare materials' fire performance characteristics
- **Democratize fire research data**: Make experimental results freely available to researchers, engineers, and practitioners worldwide
- **Funding**: Supported by Award No. 2019-DU-BX-0018 from the National Institute of Justice

### Database Contents

The database contains experimental results from multiple apparatus types testing various material categories including:
- Polymers
- Composites
- Wood products
- Insulations
- Fabrics
- Complete assemblies (furniture)

---

## 2. Project Structure

The repository is organized in a hierarchical, highly structured manner:

```
fsri_materials_database/
├── 01_Data/                    # Raw and processed experimental data
├── 02_Scripts/                 # Python data processing and visualization scripts
├── 03_Charts/                  # Generated graphs and visualizations (output)
├── 04_Documentation/           # Technical guides and references
├── README.md                   # Main project documentation
├── CITATION.cff                # Citation metadata (CFF format)
├── global.json                 # Global configuration for the entire database
└── .gitignore                  # Git ignore patterns
```

### 01_Data/ Directory

- **Contains**: Raw experimental data organized by material
- **Size**: 152 material directories
- **Structure**:
  - Each material gets its own subdirectory (e.g., `MDF/`, `Oak_Flooring/`, `Polyester_Fabric/`)
  - Within each material directory: subdirectories for each test apparatus (Cone, HFM, MCC, STA, FTIR)
  - Each apparatus directory contains: raw data files (.csv, .txt, .tst) and generated summary data
  - Each material includes: `material.json` (metadata), photos (JPG files), and optional component materials

### 02_Scripts/ Directory

- **Purpose**: Python scripts for data processing and visualization
- **Main components**:
  - **Plot scripts** (12 total): `plot_[APPARATUS]_data.py` and `plot_[APPARATUS]_data_html.py`
  - **Utilities/**: Helper scripts for metadata management and validation
  - **Deprecated/**: Older scripts no longer in active use
  - **Batch runners**: Shell and batch scripts to execute all processing scripts

- **Key subdirectories**:
  - `Utilities/`: Contains JSON writers, validators, image resizers, property collectors
  - `Utilities/material_headers/`: Pre-generated JSON header templates for 87 materials
  - `Utilities/schematics/`: Equipment setup diagrams (JPG files)

### 03_Charts/ Directory

- **Purpose**: Output directory for generated graphs and visualizations
- **Contents** (auto-generated):
  - Material subdirectories created when scripts execute
  - Each contains PDF plots and interactive HTML visualizations
  - Organized by apparatus type within each material
- **Not tracked in git** (in .gitignore) to avoid bloat

### 04_Documentation/ Directory

- **User_Guide/**: `FSRI_MaP_User_Guide.tex` - LaTeX source for user documentation with figures
- **Technical_Reference_Guide/**: `FSRI_MaP_Tech_Guide.tex` - Detailed technical documentation with experimental methodology
- **Bibliography/**: `MaP_Database_Bibliography.bib` - LaTeX bibliography file with research references, logos, common commands

---

## 3. Data Structure

### 3.1 Data Organization Hierarchy

```
01_Data/
└── [MATERIAL_NAME]/
    ├── material.json              # Metadata and content links
    ├── [Material_Name].JPG        # Full-size material photo
    ├── [Material_Name]600x400.jpg # Thumbnail photo
    ├── Cone/                      # Cone Calorimeter test data
    │   ├── [Material]_Cone_*.csv  # Raw scan data
    │   ├── *_Notes.csv            # Test notes (generated)
    │   ├── *_Analysis_Data.csv    # Processed data (generated)
    │   └── *.html                 # Interactive visualizations (generated)
    ├── HFM/                       # Heat Flow Meter data
    ├── MCC/                       # Micro-scale Combustion Calorimeter
    ├── STA/                       # Simultaneous Thermal Analyzer
    ├── FTIR/                      # Fourier Transform Infrared
    │   ├── ATR/                   # Attenuated Total Reflectance method
    │   └── IS/                    # Integrating Sphere method
    └── [Optional component directories]
```

### 3.2 File Naming Conventions

The project uses highly descriptive, systematic file naming for traceability:

#### Cone Calorimeter Example
`MDF_Cone_HF25Scalar_210323_R1.csv`
- `MDF` = Material name
- `Cone` = Apparatus
- `HF25` = Heat flux 25 kW/m²
- `Scalar` = Scalar data (parameters) vs `Scan` (raw output)
- `210323` = Test date (March 23, 2021)
- `R1` = Replicate number 1

#### HFM Example
`OSB_HFM_Dry_Conductivity_210315_R3.tst`
- Includes condition (Dry/Wet) and property (Conductivity/Specific_heat)

#### MCC Example
`Polyester_Fabric_MCC_30K_min_210308_R1.txt`
- Includes heating rate (30 K/min)

#### STA Example
`Polyester_Batting_STA_N2_3KData_210215_R1.csv`
- Includes atmosphere (N2), heating rate, and Data/Meta designation

#### FTIR/ATR Example
`MDF_ATR_210323_R5.csv`

#### FTIR/IS Example
`OSB_IS_REFLECT_MEAS_210623_R2.csv`
- Includes measurand type (REFLECT, TRANSMIT, etc.)

### 3.3 File Formats

- **CSV files** (.csv): Comma-separated values with headers, typically containing time-series data
- **TST files** (.tst): UTF-16 encoded text files from HFM apparatus
- **TXT files** (.txt): MCC data with headers and metadata
- **HTML files** (.html): Interactive Plotly visualizations (auto-generated)
- **PDF files** (.pdf): Static matplotlib plots (auto-generated)
- **JPG files** (.jpg): Material photographs

### 3.4 Data Schemas

#### material.json Structure

```json
{
  "material": "Full Material Name",
  "alternate names": ["Name1", "Name2"],
  "material_dir": "Directory_Name",
  "parent material": "For component materials",
  "component material": [],
  "material category": ["engineered wood", "structural", ...],
  "image file": "Material.JPG",
  "image 600x400 file": "Material600x400.jpg",
  "description": "Full description text",
  "short desc": "Short description",
  "source": "Material source information",
  "density": "Value with unit markup",
  "property scales": ["milligram", "bench scale"],
  "measured property": [
    {
      "test name": "specific heat",
      "nested tests": [
        {
          "display name": "Unconditioned",
          "graph": "HFM/Material_HFM_Wet_specific_heat.html",
          "table": "HFM/Material_HFM_Wet_specific_heat.html"
        }
      ]
    }
  ],
  "derived property": [
    {
      "test name": "effective heat of combustion",
      "nested tests": [...]
    }
  ]
}
```

#### global.json Structure

Centralized configuration file containing:
- Material category order (12 categories)
- Repository metadata (GitHub URL, branch)
- Measured properties (13 types) with descriptions
- Derived properties (8 types) with descriptions
- UI display configuration and flags

---

## 4. Code Organization

### 4.1 Main Python Scripts (02_Scripts/)

Total of 12 apparatus-specific script pairs (3,902 lines of code total):

| Script Name | Lines | Purpose |
|-------------|-------|---------|
| `plot_Cone_data.py` | 312 | Process cone calorimeter data, generate PDFs |
| `plot_Cone_data_html.py` | 449 | Create interactive cone calorimeter visualizations |
| `plot_HFM_data.py` | 346 | Process heat flow meter data (thermal properties) |
| `plot_HFM_data_html.py` | 331 | Interactive HFM visualizations |
| `plot_MCC_data.py` | 256 | Process micro-scale combustion calorimeter data |
| `plot_MCC_data_html.py` | 231 | Interactive MCC visualizations |
| `plot_STA_data.py` | 344 | Process simultaneous thermal analyzer data |
| `plot_STA_data_html.py` | 327 | Interactive STA visualizations |
| `plot_ATR_data.py` | 150 | Process FTIR ATR spectroscopy data |
| `plot_ATR_data_html.py` | 91 | Interactive ATR visualizations |
| `plot_IS_emissivity_data.py` | 375 | Process integrating sphere emissivity data |
| `plot_IS_emissivity_data_html.py` | 344 | Interactive integrating sphere visualizations |
| `plot_Furniture_Cal_data_html.py` | 346 | Full-scale furniture calorimetry visualization |

### 4.2 Utility Scripts (02_Scripts/Utilities/)

- **`json_writer.py`** (39.5 KB, ~2000+ lines):
  - Primary utility for maintaining material.json files
  - Scans processed HTML outputs to auto-populate JSON metadata
  - Generates standardized material.json structure
  - Reads from: `test_description.csv`, `test_description.json`, `global.json`
  - Creates material header JSON files

- **`json_validation.py`** (1.9 KB):
  - Validates all material.json files for correctness
  - Reports valid, invalid, and missing JSON files
  - Ensures data integrity across repository

- **`collect_thermophysical_properties.py`** (9.9 KB):
  - Extracts thermal conductivity and heat capacity data from HFM files
  - Parses UTF-16 encoded .tst files from instrument
  - Aggregates results across replicates
  - Generates summary tables

- **`reduce_image_sizes.py`** (1.5 KB):
  - Batch resize material photographs
  - Creates 600x400 thumbnail versions
  - Maintains aspect ratios

- **`check_versions.sh`** (386 B):
  - Shell script for version checking

### 4.3 Batch Execution Scripts

- **`run_all_data.sh`** (347 B): Unix shell script to execute all plot_*_data.py scripts sequentially
- **`run_all_data.bat`** (335 B): Windows batch script equivalent
- **`run_all_data_html.sh`** (598 B): Unix shell for all HTML generation scripts
- **`run_all_data_html.bat`** (578 B): Windows batch equivalent

### 4.4 Deprecated Scripts (02_Scripts/Deprecated/)

- `heat_capacity.py` - Replaced by HFM processing
- `ignition_temp.py` - Replaced by MCC processing
- `plot_IS_data.py` - Legacy integrating sphere script
- `plot_IS_data_html.py` - Legacy integrating sphere HTML script

---

## 5. Key Technologies

### 5.1 Languages

- **Python 3.x**: Core processing and visualization
- **Shell/Batch**: Automation and orchestration
- **LaTeX**: Documentation preparation
- **JSON**: Data configuration and metadata

### 5.2 Key Libraries

| Library | Purpose |
|---------|---------|
| `pandas` | Data wrangling and CSV processing |
| `numpy` | Numerical and mathematical analysis |
| `scipy` | Statistical analysis and signal processing (Savitzky-Golay filters, integration) |
| `matplotlib` | Static PDF plot generation and styling |
| `plotly` | Interactive HTML visualizations with hover/zoom/pan |
| `GitPython` | Repository introspection (appends commit hash to plots) |
| `pybaselines` | Baseline correction for STA melting analysis |

### 5.3 External Tools

- **Git/GitHub**: Version control and collaborative development
- **Matplotlib rendering**: PDF generation for publication-quality graphics
- **Plotly rendering**: Interactive web visualizations in HTML5

---

## 6. Data Processing Pipeline

### 6.1 Overall Workflow

```
Raw Instrument Data
        ↓
    [Stored in 01_Data/]
        ↓
    Execution: run_all_data.sh / .bat
        ├─→ plot_Cone_data.py
        ├─→ plot_HFM_data.py
        ├─→ plot_MCC_data.py
        ├─→ plot_STA_data.py
        ├─→ plot_ATR_data.py
        └─→ plot_IS_emissivity_data.py
        ↓
    Processed CSV Data & PDF Graphs → 01_Data/ & 03_Charts/
        ↓
    Execution: run_all_data_html.sh / .bat
        ├─→ plot_Cone_data_html.py
        ├─→ plot_HFM_data_html.py
        ├─→ plot_MCC_data_html.py
        ├─→ plot_STA_data_html.py
        ├─→ plot_ATR_data_html.py
        ├─→ plot_IS_emissivity_data_html.py
        └─→ plot_Furniture_Cal_data_html.py
        ↓
    Interactive HTML Graphs & Tables → 01_Data/ & 03_Charts/
        ↓
    json_writer.py
        ↓
    Updated material.json Files
        ↓
    Uploaded to GitHub + Website Frontend
```

### 6.2 Apparatus-Specific Processing Details

#### Cone Calorimeter (plot_Cone_data.py)

**Input:** CSV files with columns for time, temperatures, mass, extinction coefficient, gas concentrations

**Processing:**
- Applies Savitzky-Golay filtering (window=31, polynomial order=3)
- Calculates derived quantities: HRRPUA, MLR, SPR, Extinction Coefficient
- Computes smoke extinction from optical density
- Calculates effective heat of combustion

**Output:**
- PDF graphs: HRRPUA, MLR, SPR, Extinction Coefficient (for each heat flux: 25, 50, 75 kW/m²)
- CSV tables: Analysis data, notes, CO yield, soot yield
- HTML interactive plots with mean and standard deviation bands

#### HFM - Heat Flow Meter (plot_HFM_data.py)

**Input:** UTF-16 encoded .tst files from instrument

**Processing:**
- Reads binary files with specific encoding handling
- Extracts "Results Table -- SI Units" section
- Parses thermal conductivity and specific heat capacity
- Separates wet vs. dry conditions
- Aggregates across temperature points and replicates

**Output:**
- PDF graphs: Thermal conductivity vs temperature (wet/dry), Specific heat vs temperature
- CSV tables: Mean ± std dev for each condition
- HTML interactive plots by temperature

#### MCC - Micro-scale Combustion Calorimeter (plot_MCC_data.py)

**Input:** TXT files with metadata headers and time-series data

**Processing:**
- Extracts calibration coefficients
- Applies Savitzky-Golay filtering to heat release rate
- Reads final mass from accompanying FINAL_MASS.txt files
- Calculates effective heat of combustion from integration

**Output:**
- PDF: Specific HRR vs temperature
- CSV: Heats of combustion, ignition temperature
- HTML: Interactive HRR plots

#### STA - Simultaneous Thermal Analyzer (plot_STA_data.py)

**Input:** CSV files with temperature, mass, DSC signal data

**Processing:**
- Applies dynamic window Savitzky-Golay filtering
- Uses `pybaselines` library for baseline correction on DSC derivative
- Calculates apparent heat capacity, normalized mass loss rate
- Integrates to find heat of reaction and heat of gasification
- Detects melting temperature and enthalpy

**Output:**
- PDF graphs: Mass loss, MLR, heat capacity, DSC derivative (by heating rate: 3, 10, 30 K/min)
- CSV: Melting temperature and enthalpy tables
- HTML: Interactive visualizations

#### FTIR/ATR - Attenuated Total Reflectance (plot_ATR_data.py)

**Input:** CSV files with wavelength and ATR signal

**Processing:**
- Calculates mean and standard deviation across replicates
- Applies Savitzky-Golay smoothing

**Output:**
- PDF: ATR signal vs wavenumber
- HTML: Interactive spectra plots

#### FTIR/IS - Integrating Sphere Emissivity (plot_IS_emissivity_data.py)

**Input:** CSV files with wavelength and reflectance/transmittance

**Processing:**
- Calculates band-averaged emissivity from reflectance
- Aggregates across replicates

**Output:**
- PDF/HTML: Emissivity vs temperature/wavelength
- CSV: Emissivity tables

### 6.3 Post-Processing: JSON Generation

#### json_writer.py workflow

1. Reads `material_status.csv` (generated by HTML scripts)
2. Scans existing HTML output files for each material
3. Constructs JSON entries for each measured/derived property
4. Links to appropriate graph and table HTML files
5. Maintains hierarchical "nested tests" structure for multi-condition properties
6. Appends repository metadata (commit hash, branch)

---

## 7. Configuration

### 7.1 global.json (Main Configuration File)

Located at: `./global.json`

#### Key sections:

- **material category order**: Defines 12 material classifications:
  - cord and cable, engineered wood, exterior finish, general polymer, insulation, interior finish, plumbing, roofing, sleeping product, structural, upholstered furniture, cellulosic materials

- **repository**: GitHub URL and active branch (main)

- **measured property** (13 types with configuration):
  - Each includes: test name, display name, test description, table/graph flags, nesting structure
  - Examples: specific heat, thermal conductivity, mass loss rate, HRRPUA, CO yield, heat release rate, plume temperature, plume velocity

- **derived property** (8 types):
  - Examples: soot yield, effective heat of combustion, heat of reaction, heat of gasification, ignition temperature, melting temperature/enthalpy, band averaged emissivity

### 7.2 test_description.json & test_description.csv

Located in: `./02_Scripts/Utilities/`

- Maps each property to descriptive text for display
- Provides test methodology descriptions
- Used by json_writer.py to populate material.json descriptions

### 7.3 material_headers/ Directory

Pre-generated JSON header templates for 87 materials containing:
- Material name and aliases
- Category classification
- Image references
- Source information
- Initial property list structure

### 7.4 .gitignore Configuration

Excludes from version control:
- LaTeX compilation artifacts (*.aux, *.log, *.pdf)
- Generated output files:
  - All files in `03_Charts/` (dynamically generated)
  - Generated CSV summary files (*_Notes.csv, *_Analysis_Data.csv, etc.)
  - Generated HTML files (*_*.html)
- Material_Status.csv from Utilities
- Compiled documentation PDFs

---

## 8. Documentation

### 8.1 README Files

Located at:
- `./README.md` (Root) - Main project overview, database structure, usage instructions, package requirements
- `./01_Data/README.md` - Data directory structure and file naming conventions
- `./03_Charts/README.md` - Output directory structure

### 8.2 Technical Documentation

Located in: `./04_Documentation/`

#### User Guide

- File: `./04_Documentation/User_Guide/FSRI_MaP_User_Guide.tex`
- LaTeX source document for end-user documentation
- Includes figures demonstrating how to use the database on the website
- Covers material browsing, data interpretation, citation guidelines

#### Technical Reference Guide

- File: `./04_Documentation/Technical_Reference_Guide/FSRI_MaP_Tech_Guide.tex`
- Comprehensive technical documentation covering:
  - Experimental methodologies for each apparatus
  - Data processing procedures
  - Derived quantity calculations
  - Example figures showing processed data
  - Calibration and uncertainty information
- Includes figures directory with sample plots and apparatus images

#### Bibliography

- File: `./04_Documentation/Bibliography/MaP_Database_Bibliography.bib`
- BibTeX format bibliography for all referenced research
- Includes logos (FSRI_Logo_white.png, UL_RI.png)
- Common LaTeX commands file (commoncommands.tex)

### 8.3 In-Code Documentation

Each Python script includes:
- Header comments explaining purpose
- Issue tracker link (GitHub issues)
- Usage notes section detailing inputs/outputs/directory structure
- Function docstrings describing data transformations

### 8.4 Citation Information

Located in: `./CITATION.cff` (Citation File Format)

Provides structured citation metadata:

```
Authors: McKinnon (M.), Weinschenk (C.), McCoy (C.), Dow (N.), DiDomizio (M.), Madrzykowski (D.)
Version: 1.0.0
Date Released: 2023-01-01
Organization: Fire Safety Research Institute, UL Research Institutes
```

#### Recommended citations:

- **Database**: "McKinnon, M., Weinschenk, C., Dow, N., DiDomizio, M., and Madrzykowski, D. Materials and Products Database (Version 1.0.0), Fire Safety Research Institute, UL Research Institutes. Columbia, MD 21045, 2023."
- **Technical Guide**: "McKinnon, M., Weinschenk, C., and Madrzykowski, D. Materials and Products Database - Technical Reference Guide..."
- **User Guide**: "McKinnon, M., and Weinschenk, C. Materials and Products Database - User Guide..."

---

## 9. Entry Points & Usage

### 9.1 Primary Entry Points

#### For Processing Raw Data

```bash
# Execute all data processing scripts (PDF generation)
cd /path/to/fsri_materials_database/02_Scripts
./run_all_data.sh              # Unix/Mac
# OR
run_all_data.bat               # Windows

# Execute all HTML visualization scripts
./run_all_data_html.sh         # Unix/Mac
# OR
run_all_data_html.bat          # Windows
```

#### For Individual Apparatus Processing

```bash
# Process only cone calorimeter data
python plot_Cone_data.py
python plot_Cone_data_html.py

# Process only HFM data
python plot_HFM_data.py
python plot_HFM_data_html.py

# Similar for MCC, STA, ATR, IS emissivity
```

#### For Utility Operations

```bash
# Validate all material.json files
cd 02_Scripts/Utilities
python json_validation.py

# Generate/update material.json files from processed outputs
python json_writer.py

# Extract thermophysical properties from HFM data
python collect_thermophysical_properties.py

# Resize material images to thumbnails
python reduce_image_sizes.py
```

### 9.2 Permission Requirements

For Unix/Mac execution:

```bash
chmod +x run_all_data.sh
chmod +x run_all_data_html.sh
chmod +x Utilities/check_versions.sh
```

### 9.3 Environment Setup

#### Required Python packages (install via pip)

```bash
pip install pandas              # Data wrangling
pip install numpy               # Numerical computing
pip install scipy               # Scientific computing (signal processing)
pip install matplotlib          # PDF plot generation
pip install plotly              # Interactive HTML plots
pip install GitPython           # Git integration (repo hash in plots)
pip install pybaselines         # Baseline correction (STA melting analysis)
```

#### External requirements

- Python 3.x interpreter
- Git (for GitPython functionality)
- Sufficient disk space for output files (03_Charts can grow large)

---

## 10. Recent Activity & Git History

### 10.1 Recent Commits on Main Branch (Last 10)

| Commit | Author | Message | Changes |
|--------|--------|---------|---------|
| d492829 | - | Merge pull request #264 from ulfsri/staging | Merged staging branch |
| 1fa356d | - | Merge pull request #265 from mckinnonm/staging | Merged staging branch |
| ba047dd | mckinnonm | Added data for Corrugated_Cardboard and Polyisocyanurate_Spray_Foam_Insulation | 70 files, 103,828 insertions: 2 new materials with Cone, STA, MCC data and images |
| 6389370 | - | Merge pull request #262 from mckinnonm/staging | Merged staging branch |
| 6d1183f | mckinnonm | Modified plotting scripts to accommodate changes to file format | Updated plot_Cone_data_html.py and plot_STA_data.py |
| ff14025 | mckinnonm | Added STA data for XPS foam | Thermal analysis data |
| 2c947e8 | mckinnonm | Added STA data for Vinyl Tile | Thermal analysis data |
| f109af9 | mckinnonm | Added STA data for PIR foam | Thermal analysis data |
| 65a1d6d | mckinnonm | Added STA data for Plastic Laminate Countertop components | Thermal analysis data |
| fb32feb | mckinnonm | Add STA data for PU Aerogel | Thermal analysis data |

### 10.2 Staging Branch Activity - MAJOR PPE EXPANSION

**Status**: Staging branch is ahead of main by **40+ commits** with significant new content

**Overall Changes**: 524 files changed, 1,169,455 insertions(+), 749 deletions(-)

#### Key Developments in Staging:

1. **29 New Personal Protective Equipment (PPE) Materials Added**
   - Comprehensive firefighter equipment testing dataset
   - All materials have STA (Simultaneous Thermal Analyzer) data
   - 11 materials include melting temperature/enthalpy analysis
   - Density range: 400-1840 kg/m³

2. **PPE Material Categories**:

   **Helmet Components (13 materials)**:
   - Fiber Reinforced Helmet Shell (1840 kg/m³)
   - Leather Helmet Shell (1340 kg/m³)
   - Polycarbonate Helmet Shell (1215 kg/m³)
   - Thermoplastic Helmet Shell (1290 kg/m³)
   - Helmet Brim Edging
   - Helmet Shield (leather front patch)
   - Helmet Shield Bracket
   - Helmet Strap
   - Helmet Buckle
   - Helmet Tetrahedron (reflective adhesive)
   - Helmet Earflap

   **Helmet Face Shields/Bourkes (4 materials)**:
   - High Temperature Bourke (polycarbonate visor)
   - Polycarbonate Bourke
   - Nylon Bourke
   - Face Shield

   **SCBA Components (7 materials)**:
   - EPDM SCBA Face Seal (950 kg/m³)
   - Neoprene SCBA Face Seal (1500 kg/m³)
   - High Temperature Nylon SCBA Nosecup (1140 kg/m³)
   - Nylon SCBA Nosecup (1060 kg/m³)
   - NFPA 1981-2007 SCBA Lens
   - NFPA 1981-2019 SCBA Lens
   - SCBA Bracket

   **Turnout Gear Components (5 materials)**:
   - Turnout Gear Shell (PBI fibers, 640 kg/m³)
   - Turnout Gear Reflector (retroreflective polymer)
   - Two Layer Turnout Gear Moisture Barrier (400 kg/m³ - lightest material)
   - Three Layer Turnout Gear Moisture Barrier (420 kg/m³)
   - Nomex Turnout Gear Thermal Liner (675 kg/m³)
   - Kevlar/Nomex Turnout Gear Thermal Liner (700 kg/m³)

   **Miscellaneous PPE (2 materials)**:
   - Protective Hood (Nomex/PBI blend)
   - UEBSS (Universal Emergency Breathing Supply System hose)

3. **Equipment Manufacturers Represented**:
   - Helmet brands: Cairns, Ben, Franklin, Bullard, Phenix, First Due, New Yorker, Honeywell
   - SCBA brands: Drager, Scott, MSA
   - Turnout gear brands: MSA, Globe, Lion, Viking, Honeywell, FDNY, LAFD, Baltimore FD, Stedair

4. **Material Types Tested**:
   - Polymers: Polycarbonate, Nylon, various thermoplastics
   - Elastomers: EPDM rubber, Neoprene rubber
   - High-performance fibers: Kevlar, Nomex, Nomex/Kevlar blends, PBI (polybenzimiazole)
   - Leather
   - Fiber-reinforced composites (FRP)

5. **Recent Script Improvements** (also in staging):
   - Updated MCC plotting script (commit 426bd64)
   - Modified utility scripts (commit 6deab8d, commit cc54957)
   - Fixed material.json header files and validation issues
   - JSON linter checks added by nicholas-dow
   - Cone notes and STA notes generation improvements
   - Added FTIR data for Polycarbonate (PC)

6. **Contributors on Staging Branch**:
   - **mckinnonm (Mark McKinnon)**: Primary contributor for PPE materials and data additions
   - **nicholas-dow**: Bug fixes, JSON validation improvements, cone/STA notes fixes

#### Staging Branch Commit Sequence (PPE Addition):

The 29 PPE materials were systematically added in alphabetical batches:

| Commit | Message | Materials Added |
|--------|---------|-----------------|
| 8fa016f | added 5 PPE materials A-H | First batch (A-H range) |
| 81b6ace | added 6 PPE materials H | Helmet materials (H range) |
| f2e6517 | added 5 PPE materials K-N | Mid-alphabet materials |
| 45c145f | added 6 PPE materials N-P | Continuation (N-P range) |
| bee0e3b | added 7 PPE materials S-U | Final batch (S-U range) |

### 10.3 Development Patterns

- **Primary contributor**: mckinnonm (Mark McKinnon) adding material test data
- **Workflow**: Features developed on `staging` branch, merged to `main` via pull requests
- **Activity focus**: Recent commits emphasize adding new materials and STA (thermal analysis) data
- **Current emphasis**: Major expansion into structural firefighter PPE testing
- **Frequency**: Regular additions of material data (multiple commits per month visible)
- **Quality control**: Pull request-based workflow suggests code review process

### 10.4 Data Growth

Recent commit ba047dd (on main) shows typical data expansion:
- Added 2 new materials (Corrugated_Cardboard, Polyisocyanurate_Spray_Foam_Insulation)
- 70 new data files (raw test scans, scalars, metadata)
- Images for both materials
- Cone calorimeter data (25, 50, 75 kW/m² heat fluxes)
- STA data (3K, 10K, 30K heating rates with data and metadata files)
- MCC data (micro-scale combustion tests)
- material.json metadata files

**Upcoming data growth (from staging merge)**:
- **29 new PPE materials** will be added
- Estimated **200+ new STA data files** (multiple heating rates per material)
- **29 new material.json files** with complete metadata
- **58+ new image files** (full-size and thumbnail for each material)
- Total repository will grow to **181+ materials** (from current 152)

### 10.5 Repository Metrics

**Current (Main Branch)**:
- **Total materials**: 152
- **Total material.json files**: 93
- **Test apparatus types**: 5 primary (Cone, HFM, MCC, STA, FTIR with ATR and IS variants)
- **Python code**: 3,902 lines across 12 main scripts
- **Utilities code**: 41+ KB in Utilities directory
- **Documentation**: LaTeX source files for user and technical guides

**After Staging Merge (Projected)**:
- **Total materials**: 181+ (29 new PPE materials)
- **Material categories**: Original 12 categories plus emphasis on structural firefighter PPE
- **Test data files**: 1.1+ million lines added to repository
- **New research focus**: Comprehensive structural firefighter equipment thermal characterization

---

## 11. Material Categories & Database Coverage

### 11.1 Defined Material Categories (from global.json)

1. **Cord and cable** - Electrical wiring, cables
2. **Engineered wood** - MDF, OSB, plywood, particleboard
3. **Exterior finish** - Siding, shingles, exterior coatings
4. **General polymer** - Pure polymer materials (ABS, PVC, PE, PP, etc.)
5. **Insulation** - Fiberglass, mineral wool, polyurethane, spray foam
6. **Interior finish** - Wallboard, paint, interior panels
7. **Plumbing** - PVC pipes, foam insulation, fittings
8. **Roofing** - Roof felt, membranes, shingles
9. **Sleeping product** - Bedding, mattresses, pillows
10. **Structural** - Load-bearing materials, lumber, concrete products
11. **Upholstered furniture** - Chairs, sofas, cushions
12. **Cellulosic materials** - Paper, cardboard, straw

### 11.2 Sample Materials in Database

- **Polymers**: ABS, HDPE, LDPE, PMMA, PP, PVC, PET, PETG, Nylon, Polycarbonate
- **Wood**: Oak, Pine, MDF, OSB, Plywood, Engineered flooring
- **Composites**: FRP panels, Corian countertops
- **Insulation**: Fiberglass (R30), Mineral wool, Cellulose, Polyisocyanurate, Polyurethane
- **Textiles**: Cotton, Polyester, Nylon carpets, Fabrics
- **Assemblies**: Furniture (Chairs, Sofas), Gypsum wallboard with components
- **Recently added**: Corrugated cardboard, Spray foam insulation

---

## 12. Measured vs. Derived Quantities

### 12.1 Measured Properties (13 types)

Properties directly from instrument output:

1. **Specific heat** [J/(kg·K)]
   - Apparatus: HFM
   - Conditions: 10°C, 20°C, 30°C, 40°C (both conditioned and dried)

2. **Thermal conductivity** [W/(m·K)]
   - Apparatus: HFM
   - Conditions: 15°C with 45°C or 65°C, both wet and dry

3. **Mass loss rate** [1/s] and [g/s]
   - Apparatus: STA (1/s) and Cone (g/s)
   - Heating rates: 3, 10, 30 K/min (STA); Heat fluxes: 25, 50, 75 kW/m² (Cone)

4. **Heat release rate per unit area (HRRPUA)** [kW/m²]
   - Apparatus: Cone calorimeter
   - Conditions: 20°C, 50% RH; Three heat fluxes

5. **Test videos**
   - Full-scale open calorimeter experiments (HD and infrared)
   - Links to external video hosting

6. **Heat release rate (full-scale)** [kW]
   - Full-scale open calorimeter tests on upholstered furniture

7. **Heat flux to target** [kW/m²]
   - Schmidt-Boelter gauge measurements in full-scale tests

8. **Plume centerline temperature** [°C]
   - Measured at 3 elevations above test article

9. **Plume centerline velocity** [m/s]
   - Vertical velocity at multiple heights

10. **Carbon monoxide (CO) yield** [kg/kg]
    - Cone calorimeter at 25, 50, 75 kW/m²

11. **Specific heat release rate** [W/g]
    - Micro-scale combustion calorimeter at 30 K/min

### 12.2 Derived Properties (8 types)

Quantities calculated from measured data:

1. **Soot yield** [g/g]
   - Calculated from smoke obscuration data
   - Cone calorimeter at three heat fluxes

2. **Effective heat of combustion (ΔHc)** [MJ/kg]
   - From MCC and Cone data integration
   - Nested by apparatus type

3. **Heat of reaction** [kJ/kg]
   - Calculated from STA data integration

4. **Heat of gasification** [kJ/kg]
   - Calculated from STA thermograms

5. **Ignition temperature (Tign)** [°C]
   - Derived from MCC experiments
   - Identified from HRR inflection

6. **Melting temperature (Tmelt) and enthalpy** [°C, kJ/kg]
   - From STA with baseline correction
   - Detected from DSC derivative peaks

7. **Band averaged emissivity**
   - Calculated from FTIR integrating sphere reflectance
   - Multiple wavelength bands

---

## 13. Key Data Characteristics

### 13.1 Replicates

- Most materials tested in **R1-R5** (5 replicates) or **R1-R3** (3 replicates)
- Tests with variations have multiple replicates per condition
- Mean and standard deviation computed across replicates

### 13.2 Conditions Tested

- **Thermal properties**: Wet vs. Dry, multiple temperatures
- **Combustion**: Multiple heat fluxes/heating rates for comparison
- **Spectroscopy**: Multiple measurement methods (ATR vs. IS)

### 13.3 Data Quality Control

- `material.json` validation via `json_validation.py`
- Notes files (*_Notes.csv) document any experimental anomalies
- Physical photos (full and thumbnail) for visual verification
- Systematic naming prevents data confusion

---

## 14. File Paths Reference

### Key Configuration Files

- `./global.json` - Main configuration
- `./CITATION.cff` - Citation metadata

### Main Scripts

- `./02_Scripts/plot_*_data.py` - Processing scripts
- `./02_Scripts/plot_*_data_html.py` - HTML visualization scripts
- `./02_Scripts/run_all_data.sh` - Batch processor

### Utilities

- `./02_Scripts/Utilities/json_writer.py` - JSON metadata generator
- `./02_Scripts/Utilities/json_validation.py` - JSON validator
- `./02_Scripts/Utilities/collect_thermophysical_properties.py` - Property extractor

### Documentation

- `./README.md` - Main documentation
- `./04_Documentation/User_Guide/FSRI_MaP_User_Guide.tex`
- `./04_Documentation/Technical_Reference_Guide/FSRI_MaP_Tech_Guide.tex`

### Data

- `./01_Data/` - 152 material directories
- `./03_Charts/` - Generated visualizations (auto-created)

---

## Summary Table

| Category | Details |
|----------|---------|
| **Purpose** | Fire research database combining experimental data and visualization tools |
| **Total Materials (main)** | 152 materials |
| **Total Materials (staging)** | 181+ materials (29 new PPE materials pending merge) |
| **Test Apparatus** | 5 types (Cone Calorimeter, HFM, MCC, STA, FTIR with ATR/IS variants) |
| **Data Format** | CSV, TST, TXT (raw); CSV (processed); HTML (interactive); PDF (static graphics) |
| **Python Scripts** | 12 apparatus processing pairs (3,902 lines) + 4 utility scripts |
| **Key Libraries** | pandas, numpy, scipy, matplotlib, plotly, GitPython, pybaselines |
| **Documentation** | README files, LaTeX guides, in-code comments, BibTeX bibliography |
| **Version Control** | Git/GitHub with pull request workflow, main + staging branches |
| **Data Categories** | 12 material types across polymers, wood, composites, textiles, assemblies, PPE |
| **Measured Properties** | 13 types including thermal, fire, optical properties |
| **Derived Properties** | 8 types calculated from measured data |
| **Recent Activity** | Major PPE expansion in staging (29 materials); regular STA data additions |
| **Staging Branch Status** | 40+ commits ahead, 524 files changed, 1.17M lines added |

---

**End of Repository Analysis**

*This comprehensive analysis documents the complete structure, functionality, and organization of the FSRI Materials Database repository as of February 2, 2026. The database represents a significant open-source contribution to fire research, providing standardized experimental protocols and processed data for 152+ materials (181+ including staging) across multiple fire test apparatus types.*

*The staging branch contains a major expansion focusing on structural firefighter Personal Protective Equipment (PPE), representing the most comprehensive thermal characterization dataset of firefighter gear components to date.*
