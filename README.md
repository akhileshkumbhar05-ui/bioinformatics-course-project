# Tumor–Normal Differential Expression, Pathways, and ML Classifier (TCGA-BRCA)

Team Members:
Akhilesh Arunkumar Kumbhar (1002248929)
Hritik Vivek Patil (1002244075)

Course project for a Bioinformatics class. This repository contains an end-to-end Python pipeline that goes from raw TCGA-BRCA RNA-seq STAR-Counts files to:

- Differential expression (DE) analysis with **PyDESeq2**
- Hallmark pathway enrichment with **GSEApy**
- Single-sample Hallmark pathway activity scores (ssGSEA)
- Logistic regression and random forest classifiers that separate tumor vs normal samples

The main goal is to show how a small, interpretable set of genes or pathways can accurately classify primary breast tumors vs solid-tissue normals.

---

## Repository structure

- `Bioinformatics_Course_Project.ipynb`  
  Main Colab notebook for the full pipeline (data download → DE → pathways → ML → plots).

- `Final_Report_Bioinformatics.docx`  
  Final written report with background, methods, results, discussion, and references.

- `README.md`  
  This file.

- `ai_usage.md`  
  Describes when and how AI tools were used and how outputs were checked.

- Generated outputs (not all are tracked in Git):  
  - `tcga_brca_star_metadata.csv` – sample-level metadata table  
  - `tcga_brca_star_counts_matrix.csv` – raw counts matrix (genes × samples)  
  - `tcga_brca_star_counts_filtered.csv` – filtered counts used for DE and ML  
  - `tcga_brca_deseq2_results.csv` – PyDESeq2 DE results  
  - `tcga_brca_gsea_hallmark_results.csv` – Hallmark GSEA results  
  - `metrics_genes_logreg.csv`, `metrics_genes_rf.csv`, `metrics_path_logreg.csv`, `metrics_path_rf.csv` – fold-wise CV metrics  
  - `metrics_summary_all_models.csv` – mean metrics across models  
  - `logreg_top_genes_positive.csv`, `logreg_top_genes_negative.csv` – top gene features  
  - `logreg_top_pathways_positive.csv`, `logreg_top_pathways_negative.csv` – top pathway features  
  - `gsea_hallmark/`, `ssgsea_hallmark/` – GSEApy result folders with extra tables and plots  

---

## Data

- **Project:** TCGA Breast Invasive Carcinoma (TCGA-BRCA)  
- **Source:** NCI Genomic Data Commons (GDC) Files API  
- **Data category:** Transcriptome Profiling  
- **Data type:** Gene Expression Quantification  
- **Workflow:** STAR – Counts  
- **Sample types used:**  
  - Primary Tumor  
  - Solid Tissue Normal  

For this project we work with a subset of 80 STAR-Counts files (set via `MAX_FILES` in the notebook/script). Only open-access RNA-seq counts are used.

Raw GDC data are **not** stored in the repository. They are downloaded on the fly by the code.

---

## Software and dependencies

Tested with Python 3.9 in Google Colab.

Key packages:

- `pandas`, `numpy`, `requests`, `tqdm`
- `matplotlib`, `seaborn`
- `pydeseq2` (DESeq2 in Python)
- `gseapy` (includes `Msigdb` and `ssgsea`)
- `mygene` (for Ensembl → gene symbol mapping)
- `scikit-learn`

Example install command (for local run):

```bash
pip install \
  pandas numpy requests tqdm \
  matplotlib seaborn \
  pydeseq2 gseapy mygene \
  scikit-learn
