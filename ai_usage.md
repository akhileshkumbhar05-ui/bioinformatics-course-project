# AI Usage Documentation

This file describes **how AI (ChatGPT)** was used in this repository, with a focus on the
**data collection / matrix construction code** for the TCGA-BRCA RNA-seq project, and how we
verified any AI-generated outputs.

---

## 1. AI tools used

- **ChatGPT (OpenAI, GPT-5.1-class model)**  
  Used via the ChatGPT web interface while developing the project.

No other code-generation tools (e.g., GitHub Copilot, CodeWhisperer) were used for this
repository.

---

## 2. Where AI was used in the data collection pipeline

AI was used as a *coding assistant* and *editor*, not as an autonomous agent. The main
places it contributed are:

### 2.1. GDC API query and metadata construction

- Helped draft the initial Python code to:
  - Call the GDC **`/files`** endpoint with a JSON filter for:
    - `project_id = "TCGA-BRCA"`
    - data category = `Transcriptome Profiling`
    - data type = `Gene Expression Quantification`
    - workflow type = `STAR - Counts`
    - access = `open`
  - Request expansion of `cases.samples` and `cases.demographic`.
  - Convert the JSON response into a `pandas.DataFrame`.

- We **manually edited** this code to:
  - Clean up column names and select only the fields we actually need.
  - Restrict to `Primary Tumor` and `Solid Tissue Normal` sample types.
  - Add `MAX_FILES` logic to down-sample for Colab memory limits.

### 2.2. Chunked download of STAR counts

- ChatGPT suggested the pattern of:
  - Grouping file IDs into batches (size 20).
  - Sending POST requests to the GDC **`/data`** endpoint.
  - Streaming responses to `.tar.gz` archives and extracting them.

- We modified the generated code to:
  - Add progress printing and basic error handling.
  - Clean up temporary archives after extraction.
  - Choose directory names and constants that match our repo.

### 2.3. Robust STAR counts reader

- ChatGPT helped design the `read_star_counts_file()` helper function that:
  - Tries to read both GDC `gene_counts.tsv` and STAR `ReadsPerGene.out.tab` formats.
  - Removes non-gene summary rows.
  - Strips Ensembl version suffixes (e.g., `ENSG000001234.1` → `ENSG000001234`).
  - Returns a `pandas.Series` of counts indexed by `gene_id`.

- We reviewed and edited this function to:
  - Make the format checks more explicit.
  - Ensure that only **unstranded** counts are used.
  - Remove any leftover debug code and tighten up `try/except` blocks.

### 2.4. Memory-safe count-matrix construction

- ChatGPT proposed using:
  - A loop over TSV files where each file is loaded into memory one at a time.
  - A list of per-sample `Series` objects.
  - A single `pd.concat(series_list, axis=1)` at the end.

- We:
  - Integrated this into `build_counts_matrix()`.
  - Added explicit mapping from `file_name` → `sample_submitter_id`.
  - Chose how to handle missing values (fill with zero, cast to `int`).
  - Decided which genes to optionally drop (e.g., pseudo-genes with `"PAR_Y"`).

### 2.5. Documentation and README text

- ChatGPT helped draft:
  - The initial version of `README.md`.
  - This `ai_usage.md` file.
  - Status-report text describing the data-collection pipeline.

- We edited all text for accuracy, course requirements, and clarity.

---

## 3. What was done **without** AI

The following choices and implementations were made by us, not suggested by AI:

- Deciding to focus on **TCGA-BRCA** and on **STAR – Counts** rather than raw FASTQ data.
- Choosing which sample types to include (`Primary Tumor`, `Solid Tissue Normal`) and how
  to down-sample to 80 samples for prototyping.
- All decisions about **project goals**, especially:
  - Tumor vs normal DE analysis.
  - Pathway analysis with Hallmark gene sets.
  - Building an interpretable logistic-regression classifier.
- The specific directory structure and naming conventions for files in the repo.
- The overall organization of the notebook and high-level analysis plan.

---

## 4. Verification of AI-generated code

All AI-assisted code was **manually verified** before being kept in the repository.

Checks we performed:

1. **Small-scale tests of the GDC query**
   - Ran the API call on a small number of files.
   - Confirmed that the response contained the expected fields (`project_id`,
     `sample_type`, `case_id`, etc.).
   - Printed `value_counts()` for `sample_type` to ensure tumor/normal labels looked
     reasonable.

2. **File download sanity checks**
   - Verified that the number of downloaded TSV files matches the number of selected
     metadata rows (or the intended `MAX_FILES` subset).
   - Manually inspected a few TSVs to confirm they are valid STAR count files.

3. **STAR counts parsing**
   - For each branch in `read_star_counts_file()`:
     - Tested it on example files of each format.
     - Confirmed that:
       - The `index` is a clean set of gene IDs without version suffixes.
       - Summary rows (`N_unmapped`, etc.) are gone.
       - Counts are integers and non-negative.
   - Compared the number of genes reported by different files to ensure consistency.

4. **Matrix alignment and shape checks**
   - After running `build_counts_matrix()`:
     - Checked that `tcga_brca_star_counts_matrix.csv` has one column per expected
       sample and > 50k genes.
     - Confirmed that the matrix columns exactly match the
       `sample_submitter_id` values in `tcga_brca_star_metadata.csv`.
     - Spot-checked a few genes across multiple samples for obviously wrong patterns
       (e.g., all zeros).

5. **Re-running in a fresh runtime**
   - Cloned the repository into a new Colab environment.
   - Ran the notebook from top to bottom.
   - Confirmed that the same metadata and matrix files are produced without errors.

If any AI-generated snippet behaved unexpectedly, we either **deleted it** or rewrote it
manually.

---

## 5. Ethical and privacy considerations

- Only **open-access** TCGA-BRCA data from the GDC are used; no controlled-access or
  dbGaP-protected files are downloaded.
- The data contain no direct personal identifiers; analyses are carried out at the level
  of cohort-level gene expression.
- ChatGPT never received raw data files as uploads; only **code snippets, error
  messages, and textual descriptions** of the data were shared.
- Final responsibility for correctness, interpretation, and ethical use of all analyses
  remains with the project team.

---

## 6. Planned AI use for later stages

For future stages (DESeq2, GSEA, and ML), we may use ChatGPT to:

- Suggest boilerplate code for standard analyses (e.g., DESeq2 setup, scikit-learn
  cross-validation loops).
- Help draft figure captions and report text.

For each such use, we will:

- Run and debug all code ourselves.
- Sanity-check results (e.g., number of significant genes, behavior of well known
  pathways).
- Update this `ai_usage.md` file to reflect any additional AI involvement.

