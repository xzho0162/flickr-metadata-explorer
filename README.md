# Flickr Photo Metadata — Exploratory Data Analysis

End-to-end exploratory data analysis (EDA) on a corpus of ~32,000 Flickr photo
metadata records collected from two heterogeneous sources (XML and JSON). The
project covers raw data parsing, schema reconciliation, regex-based text
cleansing, univariate / bivariate / multivariate analysis, and the formulation
of machine-learning research questions grounded in the observed data.

## Highlights

- **Heterogeneous ingestion** — parses XML and JSON dumps that describe the
  same entity with slightly different field names, and reconciles them into a
  single canonical schema.
- **Regex-driven text wrangling** — strips XML/HTML tags, emojis, non-Latin
  characters, and normalises casing across `Title`, `City`, `Country`,
  `Tags`, and `Description`.
- **Comprehensive EDA** — univariate distributions (skewness, kurtosis, IQR),
  bivariate relationships (tag vs. title length, spatial patterns, country
  trends), and multivariate exploration with correlation and dimensionality
  views.
- **Research questions** — the notebook derives concrete supervised,
  unsupervised, and time-series ML questions that the cleaned dataset can
  realistically support.

## Project Structure

```
flickr-photo-metadata-eda/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── 01_data_processing_and_eda.ipynb     # the main analysis notebook
├── src/
│   └── flickr_pipeline.py                    # script export of the pipeline
└── data/
    ├── samples/
    │   ├── flickr_metadata_sample.json       # first 20 raw JSON records
    │   ├── flickr_metadata_sample.xml        # first 20 raw XML records
    │   └── flickr_dataset_sample.csv         # 50-row cleaned output sample
    └── processed/
        └── (full processed CSV not tracked — see "Data" below)
```

## Dataset

The raw inputs are two files describing public Flickr photos:

| File | Records | Format | Notes |
| ---- | ------- | ------ | ----- |
| `flickr_metadata.json` | ~29,440 | JSON array | one object per photo |
| `flickr_metadata.xml`  | ~14,720 | XML       | one `<record>` per photo |

After deduplication and cleansing, the unified dataset contains **32,382
records × 18 attributes**:

| Column | Type | Description |
| ------ | ---- | ----------- |
| `Post_ID` | int | unique Flickr photo id |
| `User_ID` | str | owner id (`NSID`) |
| `Secret` | str | photo secret token |
| `Server` / `Farm` | int | Flickr serving infrastructure id |
| `Title` | str (lowercased) | user-supplied photo title |
| `Is_Public` / `Is_Friend` / `Is_Family` | bool | visibility flags |
| `City` / `Country` | str (lowercased) | optional geocoded location |
| `Post_Date` / `Taken_Date` | datetime | upload time and capture time |
| `Tags` | str | comma-separated tag list |
| `Latitude` / `Longitude` | float | photo GPS coordinates |
| `Description` | str (lowercased) | optional user description |
| `Min_Taken_Date` | datetime | earliest valid "taken" date |

Small samples of all three files (raw JSON, raw XML, cleaned CSV) live in
[`data/samples/`](data/samples) so the notebook can be run end-to-end without
the full corpus. To work with the full dataset, place `flickr_metadata.json`
and `flickr_metadata.xml` under `data/raw/` and re-run the first stage of the
notebook.

## Quick Start

```bash
# Create an isolated environment
python3 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Launch the notebook
jupyter lab notebooks/01_data_processing_and_eda.ipynb
```

The notebook will read its inputs from the current working directory by
default. The simplest way to get going is to copy the files in
`data/samples/` next to the notebook and execute every cell.

## Analysis Walkthrough

The notebook is organised in three steps that mirror the broader EDA lifecycle:

### Step 1 — Load, parse and merge

- Reads JSON with `json.load` and XML with `xml.etree.ElementTree`.
- Standardises column names across the two sources (e.g. `PostID` →
  `Post_ID`, `Post date` → `Post_Date`).
- Concatenates the two frames, then deduplicates on `Post_ID`.
- Cleans text fields with regular expressions: removes XML/HTML tags,
  emojis and non-Latin code points, lower-cases the five text columns,
  represents nulls as the sentinel `'NaN'`, and validates output against a
  reference schema.

### Step 2 — Exploratory data analysis

- **Univariate**: distribution shapes, skewness / kurtosis, IQR-based
  outlier flags, and missing-value profiles per attribute.
- **Bivariate**: tag usage vs. title and description length, temporal
  distribution within key countries, top-country trends across the
  most active years, and the latitude/longitude spatial relationship.
- **Multivariate**: correlation heatmap and a tourism-signal derivation
  that joins country, time, and tag content.

### Step 3 — Research questions

The notebook closes with concrete ML questions formulated from observed
patterns. Each question lists the target variable, the candidate
predictors, and the ML technique most suited to it (supervised
classification, clustering, anomaly detection, etc.).

## Tech Stack

- Python 3.10+
- `pandas`, `numpy` — data manipulation
- `matplotlib`, `seaborn` — visualisation
- `scipy` — statistical tests and transformations
- `scikit-learn` — preprocessing utilities

## License

Released under the [MIT License](LICENSE).
