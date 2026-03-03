# APSIM Barley Dataset

This dataset is derived from the **APSIM Next Generation Barley** validation suite (covering **Australia** and **New Zealand**). It harmonizes APSIM simulation outputs (`.db`, `.apsimx`, `.met`) into a standardized **FieldBigData (Level 1)** structure compliant with ICASA standards. The dataset covers experiments focusing on nitrogen response, irrigation strategies, sowing windows, and cultivar performance.

#### Raw Dataset Link  
https://github.com/APSIMInitiative/ApsimX/tree/master/Tests/Validation/Barley

#### Harmonized Dataset Link  
*(Insert Link Here)*

## Authorship

| Role | Details |
| :--- | :--- |
| **Publishing Organization** | **APSIM Initiative** (apsim@csiro.au) |
| **Dataset Owners** | **CSIRO** (https://www.csiro.au/) <br> **Plant and Food Research NZ** (https://www.plantandfood.com/) |
| **Data Card Author** | **Nikos Fragkos** |
---
## Dataset Overview

### Snapshot

| Dimension | Details |
| :--- | :--- |
| **Crop** | Barley (*Hordeum vulgare*) |
| **Volume** | **91** Plotcycles (Simulations) <br> **4,526** Crop Observations |
| **Geographic Scope** | Australia, New Zealand |
| **Data Structure** | Relational (joined by `UID`) |

---

### Harmonized Variable Availability

The following tables detail the variables present in the extracted CSVs. 


  
**Status Legend:**
* <img src="https://img.shields.io/badge/Available-brightgreen" align="absmiddle" /> : The variable was successfully extracted, mapped to the ICASA standard, and is included in the harmonized CSV files.
* <img src="https://img.shields.io/badge/Unavailable-red" align="absmiddle" /> : The variable is part of the standard ICASA schema but was not recorded or provided in the source experiments.
* <img src="https://img.shields.io/badge/Available_only_in_raw_data-yellow" align="absmiddle" /> : The variable exists in the original source files but was omitted from the harmonized ICASA CSVs (e.g., due to lack of a direct ICASA mapping or processing scope).

#### 1. Metadata (`meta.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `UID`, `EID`, `PLTCID`, `FL_LAT`, `FL_LONG`, `CRP_NAME`, `CUL_NAME`, `CRID`, `CUL_ID`, `PDATE`, `HADAT`, `IR_TOT`, `FEN_TOT`, `FEP_TOT`, `FEK_TOT`, `ICRDAT`, `ICDAT`, `ICPCR`, `ICRAG`, `ICSW`, `ICIN`, `SC_IC`, `PLRS`, `PLPA`, `DSOURCE` |
| **Unavailable** | `TRDATE`, `OM_TOT_N`, `OM_TOT`, `CYC_NUM`, `PLSP` |

#### 2. Weather (`weather.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `UID`, `W_DATE`, `SRAD`, `TMAX`, `TMIN`, `RAIN`, `WIND`, `VPRSD` |
| **Unavailable** | `ACO2`, `RHAVD`, `DEW` |

#### 3. Plant Observations (`obs_plant.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `GSTZD`, `LWAD`, `WLVG`, `LDAD`, `SWAD`, `GWAD`, `CHWAD`, `CWAD`, `LAID`, `CNAD`, `LGNAD`, `SNAD`, `GNAD`, `LGN%D`, `SN%D`, `GN%D`, `NE%D`, `CN%D`, `SLAD`, `GWGD`, `G#AD`, `COVER` |
| **Unavailable** | `GSTBD`, `GSTHD`, `CHTD`, `LWADM`, `DWAD`, `CRAD`, `PNAD`, `LIWAD`, `RWAD`, `UWAD`, `TDWA`, `TWAD`, `SAID`, `GAID`, `EAID`, `TAID`, `PAID`, `T#AD`, `LNAD`, `LDNAD`, `UNAD`, `CHNAD`, `PNNAD`, `VNAD`, `TUNA`, `RNAD`, `LN%D`, `LDN%D`, `VN%D`, `CHN%`, `UN%D`, `RN%D`, `CL%D`, `CLG%D`, `CLD%D`, `CS%D`, `GC%D`, `CV%D`, `CE%D`, `CTU%D`, `CR%D`, `CC%D`, `PARI`, `SNLD`, `SWLD`, `G#ED`, `PLPAD`, `SPAD`, `Ear_PD#AD`, `BRANCHD`, `CN_CD`, `ICRZN` |
| **Available in raw data only** | `------------` |

#### 4. Static Soil (`static_soil.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `SLDUB`, `SLDLB`, `SLDUL`, `SLSAT`, `SLCLY`, `SLSIL`, `SLSND`, `SLCF`, `SLBDM` |
| **Unavailable** | *(None)* |
| **Available in raw data only** | `------------` |

#### 5. Soil Water Observations (`obs_soil_water.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `UID`, `OBSDATE`, `SLDUB`, `SLDLB`, `SWLD` |
| **Unavailable** | *(None)* |

#### 6. Irrigation (`irrigation.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `UID`, `IDATE`, `IRVAL` |
| **Unavailable** | *(None)* |

#### 7. Fertilisation (`fertilisation.csv`)
| Status | Variables |
| :--- | :--- |
| **Available** | `UID`, `FEDATE`, `FEAMN` |
| **Unavailable** | `FEAMP`, `FEAMK`, `OMDAT` |

---

## ETL Pipeline & Transformation Logic

The harmonization process is managed by a Python ETL (Extract, Transform, Load) script (`APSIM_Barley_new.py`). The workflow acts on the `Barley.apsimx` file and the associated SQLite database `Barley.db`.

### 1. Pre-processing & Configuration
* **Data Ingestion:** The script includes an optional downloader to pull the raw APSIM Barley validation dataset directly from Zenodo.
* **Path Resolution:** The script parses the JSON structure of the `.apsimx` file and dynamically replaces `%root%` macros with absolute local system paths. This ensures `ExcelInput` and `.met` (weather) files referenced in factorial nodes can be successfully located and read.
* **Source Selection:** Filters simulations to only include specific validation folders: `NewZealand`, `CPT`, `MCVP`, `Australia`, `Gatton1984`, `Hermitage1990`, `HermitageRS`, `Roma1988`, `Wellcamp`, `NPIField2019`, and `NPIField2020`.

### 2. Data Extraction
* **Database Ingestion:** Connects to the SQLite `Barley.db` and pulls raw tables: `_Factors`, `_Messages`, `_InitialConditions`, `_Simulations`, `Observed`, `HarvestReport`, and `DailyReport`.
* **Text Mining (Messages):** Because management events (like fertilization and irrigation) are not stored in dedicated tables, the script uses Regex to mine the `_Messages` table to extract dates, amounts, and types of applications.
* **Weather Ingestion:** Identifies `.met` files referenced in the simulation metadata and reads them directly. 

### 3. Transformation Rules (Harmonization)
* **Metadata (`meta.csv`):** * Generates a unique identifier `UID` (`APSIM_Barley_<SimulationName>`).
    * Parses Latitude (`FL_LAT`) and Longitude (`FL_LONG`) from the text headers of the `.met` files.
    * Aggregates total irrigation (`IR_TOT`) and fertilizer separated by N, P, and K (`FEN_TOT`, `FEP_TOT`, `FEK_TOT`).
    * Converts initial soil nutrients (`NO3`, `NH4`, `Urea`) into a single `ICIN` metric using stoichiometric conversion factors.
* **Soil (`static_soil.csv`):**
    * Extracts physical soil parameters (`DUL`, `SAT`, `BD`, etc.).
    * Explodes multi-layer array strings (e.g., `"0-10, 10-30"`) into individual depth rows (`SLDUB`, `SLDLB`).
    * Mathematically adjusts soil textures (`ParticleSizeClay`, `Sand`, `Silt`) based on the rock fraction (`SLCF`).
    * **Format:** Reshaped to a "Long" format (`UID`, `SLDUB`, `SLDLB`, `FEATURE`, `VALUE`).
* **Plant Observations (`obs_plant.csv`):**
    * Maps APSIM variables to ICASA standards (e.g., `Barley.Leaf.Live.Wt` → `WLVG`, `Barley.Grain.Wt` → `GWAD`).
    * Derives complex traits (like Nitrogen Concentrations) mathematically from total mass and dry weight.
    * **Imputation:** If `OBSDATE` is missing but the crop stage is exactly `HarvestRipe`, the script aligns the observation date with `HADAT` (Harvest Date).
* **QC Filtering:**
    * **RPWO (Remove Plotcycles Without Observations):** Actively drops any `UID` from all tables if it does not have corresponding plant observation data.

### 4. Export
* **Standardization:** All datasets are standardized to include `UID` as the primary foreign key.
* **Output:** Data is exported as cleaned CSVs to `format_dataset_level1/APSIM/Barley/`.

### 5. Specific Operations & Complex Logic
*(Details of non-standard workarounds, hardcoded logic, or specific error handling)*

* **Regex Mining for Management Data:** Fertilizer amounts, nitrogen types (NO3, Urea, etc.), irrigation volumes, plant population (`PLPA`), and row spacing (`PLRS`) are not stored as columns in APSIM. The script uses complex regular expressions to read natural language logs in the `_Messages` table (e.g., parsing `"sown... population of 150 plants/m2"`) to populate the ICASA metadata.
* **Controlled Environment Fallback:** For indoor/controlled simulations that do not read from a `.met` weather file, the script implements a fallback mechanism to extract daily weather conditions directly from the internal `DailyReport` SQLite table.
* **Weather Unit Standardization:** Raw APSIM weather data is mathematically converted to match ICASA requirements (e.g., Wind is converted from $m/s$ to $km/day$; Vapor Pressure is converted from $hPa$ to $kPa$).

## Additional Comments
*None*


## Data Quality & validation
*(Section to be filled upon quality assessment)*

## License
*(Insert License Info)*

## How to cite
*(Insert Citation Info)*