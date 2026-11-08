# GoE Ministry of Finance — SSFF Power BI Star Schema, DAX & Dashboard Setup Guide

**Companion Workbook:** `GoE_SSFF_Budget_Mapping_&_Allocation_Model.xlsx`  
**Flat Export Tables:** `pbi_dim_ssff_categories.csv`, `pbi_fact_allocations.csv`, `pbi_fact_chapter6_screening.csv`  
**Analytical Cut-off:** `2026-11-03` (Africa/Cairo)

---

## 1. Data Ingestion & Star Schema Relationships

Import the three UTF-8 CSV files into **Power BI Desktop** (`Home -> Get Data -> Text/CSV`, UTF-8 encoding `65001`, promote first row as headers):

| Table Name | CSV File | Grain / Row Count | Primary / Foreign Keys |
|---|---|---|---|
| `Dim_SSFF_Categories` | `pbi_dim_ssff_categories.csv` | 14 verified categories (`G01`–`G07`, `S01`–`S07`) | PK: `Category_ID` |
| `Fact_Allocations` | `pbi_fact_allocations.csv` | 22 historical project allocations (7 Panda + 15 Green) | PK: `Allocation_UID` \| FK: `Category_ID` |
| `Fact_Chapter6_Screening` | `pbi_fact_chapter6_screening.csv` | 12 illustrative screening cases (`S01`–`S12`) | PK: `Case_ID` \| FK: `Category_ID` |

### Model View Relationships
1. `Dim_SSFF_Categories[Category_ID]` (`1`) $\rightarrow$ `Fact_Allocations[Category_ID]` (`*`), Single-direction filter.
2. `Dim_SSFF_Categories[Category_ID]` (`1`) $\rightarrow$ `Fact_Chapter6_Screening[Category_ID]` (`*`), Single-direction filter. *(Note: Case `S10` carries `EX02` to intentionally fall into an unmapped/excluded bucket during exclusion audits; add a dummy `EX02` row in Power Query if strict referential integrity is desired).*
3. Create two disconnected **What-If Parameter / Slicer Tables** (`Enter Data`):
   - `Param_ISS_Switch`: `{ (1, "MoF Reported Grouping (55.9% Transport)"), (2, "ISS-Corporate Strict Review (37.6% Transport)") }`
   - `Param_Scope_Switch`: `{ ("CH6", "Chapter 6 Only (Capital Investments)"), ("ALL", "All SSFF Permitted Chapters") }`

---

## 2. Core Institutional DAX Measures

Create an empty table named `_Measures` and add the following DAX measures:

```dax
-- =========================================================================
-- A. HISTORICAL ALLOCATION TELEMETRY (PANDA-2026 & GREEN BOND XS2241075014)
-- =========================================================================

Panda Allocated EGP Mn =
CALCULATE (
    SUM ( Fact_Allocations[Cumulative_Allocated_Native_Mn] ),
    Fact_Allocations[Instrument_ID] = "PANDA-2026"
)

Panda Eligible Headroom EGP Mn =
CALCULATE (
    SUM ( Fact_Allocations[Eligible_Budget_Native_Mn] )
        - SUM ( Fact_Allocations[Cumulative_Allocated_Native_Mn] ),
    Fact_Allocations[Instrument_ID] = "PANDA-2026"
)

Panda Analytical CNY Eq Mn =
DIVIDE ( [Panda Allocated EGP Mn], 4.2255, 0 )

Panda FX Residual EGP Mn =
[Panda Allocated EGP Mn] - ( 3500.0 * 4.2255 )
-- Evaluates to -7.25 EGP Mn (Forensic Check F10)

Panda Strict Clean Transport EGP Mn =
VAR SelectedMode = SELECTEDVALUE ( Param_ISS_Switch[Switch_ID], 1 )
RETURN
    IF (
        SelectedMode = 1,
        CALCULATE (
            SUM ( Fact_Allocations[Cumulative_Allocated_Native_Mn] ),
            Fact_Allocations[Instrument_ID] = "PANDA-2026",
            CONTAINSSTRING ( Fact_Allocations[MoF_Reported_Grouping], "Clean Transportation" )
        ),
        CALCULATE (
            SUM ( Fact_Allocations[Cumulative_Allocated_Native_Mn] ),
            Fact_Allocations[Instrument_ID] = "PANDA-2026",
            CONTAINSSTRING ( Fact_Allocations[ISS_Strict_Grouping], "Clean Transportation" )
        )
    )

Green Bond Allocated USD Mn =
CALCULATE (
    SUM ( Fact_Allocations[Cumulative_Allocated_Native_Mn] ),
    Fact_Allocations[Instrument_ID] = "XS2241075014"
)

Portfolio Normalized EGP Mn =
SUM ( Fact_Allocations[Normalized_Allocated_EGP_Mn] )

-- =========================================================================
-- B. ICMA HFIR (JUNE 2026) PRO-RATA ATTRIBUTION CONTROLS
-- =========================================================================

Headline Whole Project GHG tCO2e =
SUM ( Fact_Allocations[Whole_Project_GHG_tCO2e_yr] )

Attributed Bond GHG tCO2e =
SUM ( Fact_Allocations[Attributed_GHG_tCO2e_yr] )

Unattributed GHG Overstatement Gap tCO2e =
[Headline Whole Project GHG tCO2e] - [Attributed Bond GHG tCO2e]
-- Captures the +83,364.31 tCO2e/yr LRT sub-component overstatement (Controls C2 & F24)

Green Bond GHG per USD 100M =
DIVIDE (
    CALCULATE ( [Attributed Bond GHG tCO2e], Fact_Allocations[Instrument_ID] = "XS2241075014" ),
    DIVIDE ( [Green Bond Allocated USD Mn], 100, 0 ),
    0
)

-- =========================================================================
-- C. CHAPTER 6 PIPELINE SCREENING ENGINE (CASES S01–S12)
-- =========================================================================

Screening Gross Tested EGP Mn =
SUM ( Fact_Chapter6_Screening[Gross_Capex_EGP_Mn] )

Screening Dynamic Net Approved EGP Mn =
VAR ScopeMode = SELECTEDVALUE ( Param_Scope_Switch[Scope_Code], "CH6" )
RETURN
    IF (
        ScopeMode = "CH6",
        CALCULATE (
            SUM ( Fact_Chapter6_Screening[Net_Unencumbered_Capex_EGP_Mn] ),
            Fact_Chapter6_Screening[Default_Ch6_Pipeline_Verdict] = "ELIGIBLE"
        ),
        CALCULATE (
            SUM ( Fact_Chapter6_Screening[Net_Unencumbered_Capex_EGP_Mn] ),
            Fact_Chapter6_Screening[All_SSFF_Chapters_Verdict] = "ELIGIBLE"
        )
    )

Screening Rejected Gross EGP Mn =
VAR ScopeMode = SELECTEDVALUE ( Param_Scope_Switch[Scope_Code], "CH6" )
RETURN
    IF (
        ScopeMode = "CH6",
        CALCULATE (
            SUM ( Fact_Chapter6_Screening[Gross_Capex_EGP_Mn] ),
            Fact_Chapter6_Screening[Default_Ch6_Pipeline_Verdict] = "INELIGIBLE"
        ),
        CALCULATE (
            SUM ( Fact_Chapter6_Screening[Gross_Capex_EGP_Mn] ),
            Fact_Chapter6_Screening[All_SSFF_Chapters_Verdict] = "INELIGIBLE"
        )
    )

Screening Encumbrance Deductions EGP Mn =
SUM ( Fact_Chapter6_Screening[Overlapping_MDB_EGP_Mn] )
    + SUM ( Fact_Chapter6_Screening[Prior_Allocation_EGP_Mn] )
```

---

## 3. Recommended 3-Page Institutional Power BI Layout

1. **Page 1 — Sovereign Sustainable Portfolio Overview:**
   - Top Slicers: `Param_ISS_Switch` (MoF vs ISS-Corporate Strict Review) & `Instrument_ID`.
   - 4 KPI Cards: `Panda Allocated EGP Mn` (`EGP 14,782M`), `Panda Strict Clean Transport EGP Mn` (`8,264M` $\leftrightarrow$ `5,564M`), `Green Bond Allocated USD Mn` (`USD 750.00M`), `Attributed Bond GHG tCO2e` (`131,635 tCO2e/yr` combined attributed vs `214,999 tCO2e/yr` headline).
   - Stacked Column Chart: `Project_ID` on X-axis; `Allocated_Year1_Native_Mn`, `Allocated_Year2_Native_Mn`, and `Panda Eligible Headroom EGP Mn` as stacked series.
   - Donut Visual: `Portfolio Normalized EGP Mn` by `Dim_SSFF_Categories[SSFF_Eligible_Category]`.

2. **Page 2 — Chapter 6 Pipeline Screening & Anti-Double-Allocation Gate:**
   - Mandatory Banner Badge: `"ILLUSTRATIVE SCREENING CASES (S01–S12) — NOT ACTUAL DISBURSEMENTS"`.
   - Top Slicers: `Param_Scope_Switch` (`Chapter 6 Only` vs `All SSFF Chapters`) & `Candidate_Instrument` (`U34`, `J31C`, `J36D`).
   - Waterfall / Clustered Bar Visual: `Gross_Capex_EGP_Mn` (`850M`) $\rightarrow$ `Rejected Gross` (`-400M`) $\rightarrow$ `Eligible Case MDB/Prior Deductions` (`-70M`) $\rightarrow$ `Net Approved` (`380M` under Ch.6; `430M` under All SSFF Chapters).
   - Matrix Table with conditional formatting icons on `Primary_Audit_Reason`.

3. **Page 3 — ICMA HFIR Attribution & Forensic Audit Exceptions:**
   - Side-by-side Clustered Bar comparing `Headline Whole Project GHG tCO2e` vs `Attributed Bond GHG tCO2e` on `CT1` (LRT electrical traction component: `149,500` vs `66,136 tCO2e/yr`).
   - Forensic Exception Callout Cards for `F04/F05` (`+1 EGP Mn` MCIT capital rounding), `F08` (`EGP 2,700M` Cairo–Assiut Road ISS dispute), `F10` (`-7.25 EGP Mn` FX residual), and `F19` (`250k vs 500k` C9 TP beneficiary conflict).

