# Egypt Sovereign Sustainable Financing Framework (SSFF) Tracker & Allocation Engine

An institutional financial allocation engine, capital expenditure screening model, and impact attribution tracker for the **Arab Republic of Egypt's Sovereign Sustainable Financing Framework (SSFF)**.

This repository provides an end-to-end framework to monitor the allocation of sustainable bond proceeds (2023 Sustainable Panda Bond and 2020 Sovereign Green Bond) against national budget expenditures, evaluate pipeline eligibility under Chapter 6 (Investments), and attribute environmental impacts without double-counting.

---

## Repository Structure

```text
MoF_SSFF_Tracker/
├── .gitignore                                          # Standard repository exclusions
├── README.md                                           # Project overview and technical guide
├── GoE_SSFF_Budget_Mapping_&_Allocation_Model.xlsx     # Master financial allocation & screening model
├── SSFF_Allocation_&_Eligibility_Report.md             # Research audit, source verification & legal rules
├── SSFF_PowerBI_DAX_&_Setup.md                         # Power BI data model blueprint & DAX measures
├── pbi_dim_ssff_categories.csv                         # Dimension: 14 eligible green & social categories
├── pbi_fact_allocations.csv                            # Fact: 22 bond project-level allocations
└── pbi_fact_chapter6_screening.csv                     # Fact: 12 capital project screening cases
```

---

## Financial Model Architecture (`.xlsx`)

The financial engine features **808 live formulas** and dynamic scenario toggles across six dedicated worksheets:

| Sheet | Worksheet Name | Scope & Functionality |
|:---:|---|---|
| **Tab 1** | `Readme_&_SSFF_Governance` | Governance perimeter, dynamic review switches (MoF Standard vs. ISS-Corporate Strict Review), lookback rules, and core parameters. |
| **Tab 2** | `Panda_&_Green_Bond_Ledger` | Project-level allocation ledgers for the 2023 Panda Bond (EGP 14,782M) and 2020 Green Bond (USD 750M) with currency conversions and category mappings. |
| **Tab 3** | `Chapter6_Eligibility_Engine` | Multi-step screening engine evaluating pipeline projects against Chapter 6 capital investment scope, lookback windows, and environmental safeguards. |
| **Tab 4** | `Impact_KPI_&_Attribution_Model` | Pro-rata impact attribution model (LRT, Monorail, wastewater plants) computing sovereign financing shares vs. total project outputs. |
| **Tab 5** | `Executive_Dashboard` | High-level KPI cards, doughnut charts (allocations by category), and capital absorption profiles. |
| **Tab 6** | `Forensic_Audit_&_QA_Log` | 10 live automated QA integrity gates (`OK - RECONCILED`) and 28 forensic audit discrepancy items (F01–F28) with reconciliation proofs. |

### Key Analytical Capabilities:
* **Dual Review Scenario Toggle:** Dynamic switch between MoF official reporting and ISS-Corporate strict environmental review (reclassifying Cairo–Assiut road proceeds from Clean Transportation to Affordable Basic Infrastructure).
* **Pro-Rata Impact Attribution:** Granular GHG reduction and clean transport passenger capacity allocations avoiding financing-gap distortions (e.g., LRT Stages 1-2 traction power allocation of EGP 238M representing 44.24% of the sub-project, rather than whole-project claim).
* **State Budget Lookback Windows:** Evaluation of capital expenditures based on national budget years rather than arbitrary rolling months.

---

## Power BI & Analytics Integration

The repository includes three export-ready fact and dimension tables and a complete DAX guide:
* **Star Schema Architecture:** 1-to-many relationships connecting `pbi_dim_ssff_categories` to allocation and screening facts.
* **Curated DAX Library:** Fully documented in [SSFF_PowerBI_DAX_&_Setup.md](SSFF_PowerBI_DAX_&_Setup.md), covering allocation rates, dynamic currency switches, and cumulative disbursements.

---

## Primary Regulatory Sources

The model and audit reconciliations are grounded in official disclosures:
* **Framework & Second Party Opinions:** Egypt Sovereign Sustainable Financing Framework (Nov 2022) and Moody's Investors Service Second Party Opinion (`SQS2 — Very Good`).
* **Allocation & Impact Reports:** MoF First Panda Allocation Report (Oct 2024), Compiled Allocation & Impact Reports (2025, 2026), and Green Bond Allocation Reports (2021, 2022).
* **External Assurance:** ISS-Corporate External Review Report (March 2025).
* **State Budget Legislation:** Official State Budget Laws for FY 2025/26 (Law 91/2025) and FY 2026/27 (Law 77/2026).

For complete security-by-security audit trails and legal reconciliation notes, see [SSFF_Allocation_&_Eligibility_Report.md](SSFF_Allocation_&_Eligibility_Report.md).

---

## Author

**Zyad Magdy**  
*Quantitative Research & Sovereign Debt Analysis*
