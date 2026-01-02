# Wikipedia API Extraction — Cross-Lingual Semantic Difference (EN ↔ ZH)

This notebook (`Wikipedia_API_Extraction.ipynb`) supports a research project that investigates **how English and Chinese Wikipedia pages differ semantically** in **sensitive categories** (e.g., **race, gender/sex, nationality, age, religion**) and explores which language edition may exhibit **more potential bias signals**.

It is designed as a **data extraction + candidate generation pipeline**:
- **Extract** paired EN/ZH Wikipedia pages for a curated keyword taxonomy
- **Compute semantic similarity** (document- and sentence-level) using multilingual embeddings
- **Surface low-similarity “mismatch” sentence pairs** as candidates for downstream bias review and analysis
- **Export structured CSV outputs** for annotation, qualitative review, and quantitative modeling

---

## Research Goal

### Core questions
1. For sensitive categories (race, gender, nationality, age, religion, etc.), **how different are EN vs ZH Wikipedia pages** about the same concept?
2. Are certain concepts/categories **systematically more divergent** across languages?
3. Do those divergences correlate with **potential bias indicators** (framing, omissions, emphasis, or lexical cues) in one language more than the other?

### What this notebook does
- Pull EN pages from Wikipedia for your keyword list
- Map each EN page to a ZH counterpart (when available)
- Extract plain text summaries/content via the MediaWiki API
- Compute multilingual embedding similarity
- Output **ranked mismatch candidates** and per-page similarity summaries

---

## Pipeline Overview (What the notebook does)

1. **Keyword taxonomy**
   - Input is `KEYWORDS_BY_CATEGORY`: a dictionary mapping sensitive categories → keyword lists.

2. **Wikipedia page discovery (EN)**
   - Uses MediaWiki API search (`list=search`) to find candidate EN pages per term.

3. **Cross-language linking (EN → ZH)**
   - For each EN page, queries language links (`langlinks`) to retrieve the corresponding **ZH title** if it exists.

4. **Text extraction (EN & ZH)**
   - Uses `extracts` to pull plain-text content (typically the intro / extract; exact scope depends on API parameters).

5. **Cleaning + sentence splitting**
   - Cleans raw extracts and splits into sentences for comparison.

6. **Multilingual embedding**
   - Encodes EN and ZH sentences with a multilingual sentence transformer (default in the notebook is typically:
     `paraphrase-multilingual-MiniLM-L12-v2`).

7. **Similarity + mismatch candidate generation**
   - For each EN sentence, finds the **most similar ZH sentence** and records cosine similarity.
   - Flags **low-similarity pairs** (below a threshold, e.g., ≤ 0.5) as **semantic divergence candidates**.

8. **Aggregation**
   - Produces page-level summary statistics (mean similarity, quantiles, counts below threshold, etc.).

9. **Export**
   - Writes CSVs for downstream analysis and optionally zips outputs for download (useful in Colab).

---

## Outputs

All outputs are written to an output directory (commonly something like `wiki_sem_diff_outputs/`).

Typical files include:

- `summary_all.csv`
  - **One row per EN↔ZH page pair**
  - Page metadata + aggregated similarity stats (e.g., mean/median/min similarity, counts of low-similarity sentences)

- `diffs_all.csv`
  - **Sentence-level candidate mismatches**
  - Contains EN sentence, best-match ZH sentence, cosine similarity, and category/term/page identifiers

- Per-category exports (optional)
  - e.g., `diffs_race.csv`, `diffs_religion.csv`, etc.

- Zipped bundle (optional)
  - A `.zip` of the entire output folder for easy download from Colab

---

## Requirements

### Recommended environment
- Python 3.9+ (3.10+ recommended)
- Jupyter / Colab

### Key libraries
- `requests`, `pandas`, `numpy`, `tqdm`
- `sentence-transformers`
- `scikit-learn` (for cosine similarity)
- (Optional) `nltk` / `spacy` if you extend sentence segmentation

### Configuration / Key Parameters
- KEYWORDS_BY_CATEGORY
  - Your category → keyword lists
- Search controls
  - max search results per term (to avoid pulling too many pages)
  - whether to restrict to exact title matches vs search results
- Similarity model
  - multilingual SBERT model name (trade-off: speed vs quality)
- Divergence selection
  - similarity threshold (e.g., 0.5)
  - top-K lowest similarity sentence pairs per page/category
  - maximum sentences per page (for runtime control)
- Output path
  - output directory name
  - whether to zip outputs
