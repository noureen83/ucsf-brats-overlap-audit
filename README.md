# UCSF-PDGM / BraTS 2021 patient-overlap audit

Code and derived metadata for the paper **"Hidden Patient Overlap Between Public Glioma MRI Cohorts: An Identifier-Level Audit of UCSF-PDGM and BraTS and Its Consequences for External Validation"** (N. Hameed, T. F. Khan, M. Hussain, S. Saeed, A. Rauf; IEEE conference submission, 2026).

The audit links the 501 scans (495 patients) of UCSF-PDGM v5 to BraTS 2021 using the identifiers published in the UCSF-PDGM metadata. It reports overlap by WHO grade, the follow-up scans that hide overlap from a scan-level check, and how removing overlap changes cohort composition. It also includes an exploratory, stratified zero-shot segmentation analysis.

## Contents

| Path | What it is |
|---|---|
| `audit/audit_overlap.py` | Stand-alone audit (pandas + scipy, CPU, seconds). Reproduces Tables I–III of the paper and, with a split file, the strata S1–S3. |
| `metadata/` | Audit outputs derived from UCSF-PDGM v5 metadata: patient-level BraTS 2021 linkage, overlap by grade, follow-up scans, composition with standardized differences and tests. |
| `metadata/split_manifest.csv` | The fixed BraTS 2021 split (875 / 188 / 188) used by the models (`patient_level_split_1251.csv`). |
| `notebooks/ucsf_grade4_overlap_strata.ipynb` | Colab notebook: strata counts, export of UCSF-PDGM grade-4 cases, image-fingerprint check, zero-shot inference with 3D U-Net and Swin UNETR, stratum table and tests (GPU). |
| `notebooks/review_checks_bootstrap_fingerprint.ipynb` | Colab notebook: patient-bootstrap CIs, fingerprint reconciliation, within-BraTS duplicate search (CPU). |
| `figures/make_fig_overlap.py` | Regenerates Fig. 2 from the metadata CSV. |
| `results/` | Per-case and summary result files of the paper's run (see `results/README.md`). |

## Reproduce the audit

```bash
pip install -r requirements.txt
# download UCSF-PDGM-metadata_v5.csv from TCIA (UCSF-PDGM collection)
python audit/audit_overlap.py --meta UCSF-PDGM-metadata_v5.csv --split metadata/split_manifest.csv --out audit_out
python figures/make_fig_overlap.py UCSF-PDGM-metadata_v5.csv
```

Expected output: 501 scans of 495 patients. 296 patients are linked to BraTS 2021 (260 in the training cohort, 36 in the validation cohort), all of them WHO grade 4. With our split, the grade-4 strata are S1 = 185, S2 = 36, S3 = 100, and 75 patients are excluded.

### Audit rules
- **Matching.** Metadata IDs (`UCSF-PDGM-004`) and image folders (`UCSF-PDGM-0004_nifti`) are matched on the integer case number. All scans with the same number belong to one patient, and patient attributes come from the baseline row.
- **Linkage.** A patient is linked if *any* scan has a `BraTS21 ID`. Patients with two IDs keep both.
- **Strata.**
  - Seen in training → S1.
  - Otherwise, any validation or BraTS validation-cohort link → excluded.
  - Only in test → S2.
  - Grade 4 with no link → S3.
- **Missing values.** "unknown" and "indeterminate" are treated as missing and excluded from denominators.

## Data and licences
- **UCSF-PDGM** (Calabrese et al., 2022) is hosted by The Cancer Imaging Archive. Image data and the clinical table must be downloaded from TCIA under its licence; check the collection page for the current terms. This repository redistributes only identifier-level derived tables and aggregate statistics.
- **BraTS 2021** data come from the RSNA-ASNR-MICCAI BraTS 2021 challenge, under the challenge's terms.
- **Code**: MIT licence (see `LICENSE`).

## Models
The baselines (3D U-Net, 16.6 M parameters; MONAI Swin UNETR, feature size 48, 62.2 M parameters) were trained with the settings listed in the notebook's `CFG` and in Sec. III-C of the paper. MONAI 1.6.1 was installed at evaluation time; the training-time version was not recorded. Checkpoints are not included.

## Citation
See `CITATION.cff`. After the Zenodo release, cite the DOI it assigns.
