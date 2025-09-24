# Junior Data Scientist Assessment

## Overview
This repository contains solutions to the Junior Data Scientist Assessment at **Vanilla Steel**.  
The project consists of two main parts:  

1. **Supplier Data Cleaning** (Excel files → unified dataset).  
2. **RFQ Similarity** (enriching RFQs with reference properties and computing similarity).  

## Repository Structure
```
├── notebooks/                   # Jupyter notebooks for analysis
│   ├── task1_pipeline.ipynb  # Scenario A: Supplier data cleaning
│   ├── task2_pipeline.ipynb     # Scenario B: RFQ and reference data engineering
│
├── data/                        # Input data
│   ├── task_1
│   │   ├──supplier_data1.xlsx
│   │   ├── supplier_data2.xlsx
│   ├── task 2
│   │   ├── rfq.csv
│   │   ├── reference_properties.tsv
│
├── outputs/                     # Generated outputs
│   ├── inventory_dataset.csv    # Deliverable A
│   ├── (top3.csv)                 # Deliverable B missing. 
├── requirements.txt             # Python dependencies
├── README.md                    # Project documentation
```
## Installation
```bash
# Clone repository with SSH. 
git clone <git@github.com:myosang/steel-data-engineering.git>
cd <steel-data-engineering>

# Create virtual environment
python -m venv .venv
source .venv/bin/activate   # Linux/Mac
.venv\Scripts\activate      # Windows

# Install dependencies
pip install -r requirements.txt
```
## Usage

### Run via Notebook
1. Launch Jupyter Lab or Jupyter Notebook.  
2. Navigate to the `notebooks/` folder.  
3. Execute (Run all):
   - `task1_pipeline.ipynb` → outputs `inventory_dataset.csv`  
   - `task2_pipeline.ipynb` → outputs in the code and `top3.csv`  

## Tasks Completed

- **Task A.1** → Cleaned and joined supplier datasets into `inventory_dataset.csv`.  
- **Task B.1** → Normalized grade keys, parsed ranges, and joined RFQs with reference properties.  
- **Task B.2** → Engineered:
  - Interval features for dimensions (with overlap metric).  
  - Define the funciton for exact-match categorical features.  
  - Midpoints for numeric grade properties.  
- **Task B.3** → **Not completed** due to the time limits. Aggregated similarity metric and generated `top3.csv` should have been delivered.


## Report (Assumptions and Notes)
### Task A
1. **Text Standardization**  
   - Translated categorical values (e.g., German → English) using a manual mapping.  
   - Normalized strings to lowercase and stripped whitespace.  
   - Missing values labeled as `"NULL VALUE FOUND"`, unexpected values flagged as `"EXCEPTIONS"`.  

2. **Numerical Cleaning**  
   - Converted `Thickness (mm)` and `Width (mm)` to `float`, and `Quantity` to `int` (floored).  
   - Missing numerical values replaced with `"NULL VALUE FOUND"`.  
   - No numerical fixes required for `supplier_data2`.  

3. **Column Alignment & Integration**  
   - Unified `Grade` (from `supplier_data1`) and `Material` (from `supplier_data2`) into a single column: `Grade/Material`.  
   - Renamed `Gross weight (kg)` → `Weight (kg)` for consistency across datasets.  
   - Added missing columns to ensure both datasets share the same schema before merging.  

4. **Final Dataset Construction**  
   - Checked for duplicates in both datasets for awareness.  
   - Concatenated into a unified schema (`inventory`).  
   - Exported final dataset as **`inventory_dataset.csv`** 

### Task B
1. **Grade Normalization & Reference Join**  
   - Grades were normalized to uppercase and split into base + suffix.  
   - RFQs without a matching grade in the reference were flagged (`missing_in_ref = True`).  
   - Range values (e.g., `150–440 MPa`, `≥280 MPa`, `≤20 mm`) were parsed into `(min, max, unit)`.  
   - Reference properties were cleaned: columns lowercased, spaces replaced with underscores, and joined with RFQs on `grade_joined`.  

2. **Missing Values Handling**  
   - Rows missing identifiers or grades were dropped/flagged (`missing_grade = True`).  
   - Kept-null for categorical attributes (e.g., coating, finish).  
   - For numerical ranges, if only `min` or `max` existed, the missing side was imputed with the available value.  

3. **Feature Engineering**  
   - **Dimensions:** Represented as intervals `(min, max)` with fallback to singletons if one bound was missing. Overlap measured using *Intersection over Union (IoU)*.  
   - **Categorical Features:** Defined similarity as exact match (1/0) across attributes like coating, finish, surface_type, and form.  
   - **Grade Properties:** Converted ranges into midpoints for numerical features (e.g., chemical compositions). Sparse features (<10% coverage) were dropped.  

4. **Similarity Framework**  
   - Dimensions contribute overlap scores (IoU).  
   - Categoricals contribute averaged exact-match scores.  
   - Grade properties contribute midpoint-based numeric similarity.  
   - This framework forms the basis for the aggregate similarity score used in Task B.3.  

