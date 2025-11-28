# AI Usage Documentation

This file documents how AI tools were used in this project, how their outputs were verified, and any relevant ethical or data-privacy considerations.

---

## 1. AI tools used

- **ChatGPT (OpenAI)** – used as a conversational assistant for:
  - Designing the analysis pipeline.
  - Writing and refactoring Python code for the notebook and script.
  - Debugging errors and suggesting fixes.
  - Polishing written text (report sections, README, and this file).
  - Generating short figure-caption style summaries of results.

No other code-completion or AI writing tools were used on this repository.

---

## 2. Where AI contributed

### 2.1 Project design and planning

ChatGPT was used to help sketch an analysis plan consistent with the course template:

- Use TCGA-BRCA STAR-Counts data from the GDC.
- Build a gene × sample count matrix and metadata table.
- Perform differential expression with **PyDESeq2**.
- Perform pathway analysis with **GSEApy** and Hallmark gene sets.
- Compute single-sample pathway scores (ssGSEA).
- Train logistic regression and random forest classifiers with stratified 5-fold CV.
- Produce the specific visualizations requested in the assignment (QC plots, MA/volcano, pathway tables, ROC/confusion matrices, heatmaps).

These plans were then implemented and adjusted based on what actually worked in the notebook.

### 2.2 Code scaffolding and refactoring

ChatGPT assisted in writing and/or refactoring code for the following parts of `Bioinformatics_Course_Project.ipynb` / `bioinformatics_course_project.py`:

- **GDC data access**
  - Building the GDC Files API query for TCGA-BRCA STAR-Counts.
  - Chunked download and extraction of TSV gene-level count files.
  - Logging and sanity checks (e.g., number of files, sample types).

- **Counts matrix and metadata**
  - Functions to:
    - Parse STAR count TSVs.
    - Strip Ensembl version suffixes.
    - Drop non-gene rows.
    - Merge counts into a unified gene × sample matrix.
  - Construction of the sample metadata table from the GDC manifest / API response.

- **QC and filtering**
  - Computation of library sizes and plotting:
    - Histogram of total counts per sample.
    - Boxplot of log10 library size by condition.
  - Simple low-count filtering rule (`MIN_COUNT`, `MIN_SAMPLES`) and saving the filtered matrix.

- **Differential expression (PyDESeq2)**
  - Setting up the PyDESeq2 dataset with design `~ condition`.
  - Running DESeq2, collecting results, and saving them to CSV.
  - Generating MA and volcano plots from the DE table.

- **Gene-symbol mapping and Hallmark analysis**
  - Using **MyGene.info** to map Ensembl IDs to gene symbols.
  - Handling duplicates and building a symbol-indexed expression matrix.
  - Using **GSEApy** to:
    - Load MSigDB Hallmark gene sets.
    - Run pre-ranked GSEA with a DE-based statistic.
    - Run ssGSEA to get per-sample Hallmark scores.
  - Extracting and sorting the top Hallmark pathways by FDR q-value.

- **Machine-learning features and models**
  - Building feature matrices:
    - `X_genes` = top 100 DE genes.
    - `X_pathways` = Hallmark ssGSEA scores.
  - Implementing the cross-validated evaluation helper:
    - StratifiedKFold.
    - StandardScaler inside each fold.
    - Logistic regression and random forest models.
    - Computation of ROC-AUC, PR-AUC, accuracy, and F1.
  - Saving per-fold metrics to CSV and summarizing mean metrics.
  - Extracting top logistic-regression coefficients for genes and pathways.

- **Final visualization summary cell**
  - Single cell that re-generates all required plots at the end of the notebook:
    - Library size histogram and boxplot.
    - MA and volcano plots.
    - Top Hallmark pathways table.
    - ROC curves and confusion matrices.
    - Heatmaps for top 25 genes and top 25 pathways.

### 2.3 Writing and editing

ChatGPT helped edit:

- Parts of the **final report** (method descriptions, results summaries, figure-caption style sentences).
- The **README.md** and **ai_usage.md** files.
- Short interpretations of plots (one-sentence insights).

All scientific claims were checked by reading the underlying results in the notebook.

---

## 3. What was *not* delegated to AI

- AI was **not** used to fabricate data, p-values, or metrics.  
  All numerical outputs (DE results, GSEA results, ML metrics) come from running the Python code on real TCGA-BRCA data.
- AI was **not** used to upload or process any non-public data. Only open-access TCGA RNA-seq data obtained directly from the GDC were used.
- Interpretation and final wording of conclusions were reviewed and adjusted by the student to ensure they matched the actual results.

---

## 4. How AI-generated outputs were verified

The student verified AI-assisted code and text in several ways:

1. **Execution and error checking**
   - Every non-trivial block of AI-suggested code was pasted into the notebook and executed.
   - Runtime errors and shape mismatches were fixed iteratively.
   - If code could not be made to run correctly or produced nonsensical outputs, it was discarded.

2. **Sanity checks on data shapes and counts**
   - Confirmed that the number of samples and sample IDs matched between:
     - Counts matrix, metadata, and label vectors.
   - Checked that filtered gene counts decreased as expected after QC.
   - Verified that ssGSEA feature matrices had reasonable dimensions (samples × pathways).

3. **Consistency of statistical results**
   - Inspected the DE result table:
     - Reasonable number of significantly changed genes.
     - Log2 fold changes with both up- and down-regulated genes.
   - Examined GSEA results:
     - Biologically plausible Hallmark pathways (e.g., E2F targets, estrogen response, MYC targets) at low FDR.
   - Confirmed that ML metrics matched the printed confusion matrices and ROC curves.

4. **Cross-validation behavior**
   - Checked that each model was evaluated with stratified 5-fold CV.
   - Confirmed that ROC-AUC, PR-AUC, accuracy, and F1 were in a consistent range across folds (not random or contradictory).

5. **Text verification**
   - Compared AI-drafted sentences with the actual figures and tables.
   - Edited text where necessary so that it describes what the plots and tables *actually* show.

Only code and explanations that passed these checks remain in the repository.

---

## 5. Reproducibility notes

- All analysis steps are captured in the notebook and the Python script in this repository.
- Random seeds (`random_state`) are set in key places (subsampling, cross-validation, random forest) to improve reproducibility.
- The environment used for development was Google Colab with Python 3.9. Library versions may differ slightly in other environments, which can cause minor numeric differences but should not change the overall conclusions.

---

## 6. Ethical and data-privacy considerations

- The project uses **public, de-identified** TCGA-BRCA RNA-seq STAR-Counts data from the NCI GDC.  
  No protected health information (PHI) or controlled-access data were used.
- No private datasets, patient records, or confidential information were uploaded to ChatGPT.
- When citing literature, only short paraphrased text was generated; full copyrighted articles were not copied into the prompt or repository.
- All usage of AI followed the course guidelines and is transparently documented in this file.

---
