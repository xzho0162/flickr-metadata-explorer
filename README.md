<h1 align="center">📸 Flickr Metadata Explorer</h1>

<p align="center">
  <strong>Two messy files. Two formats. Thirty-two thousand photos. One clean, analysis-ready story.</strong>
</p>

<p align="center">
  <em>An end-to-end data-wrangling and exploratory-data-analysis journey that takes raw, multilingual,<br>
  tag-soup Flickr metadata — scattered across an XML dump and a JSON dump — and turns it into<br>
  a single, trustworthy dataset ready for machine learning.</em>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white">
  <img alt="pandas" src="https://img.shields.io/badge/pandas-data%20wrangling-150458?logo=pandas&logoColor=white">
  <img alt="Jupyter" src="https://img.shields.io/badge/Jupyter-notebook-F37626?logo=jupyter&logoColor=white">
  <img alt="Regex" src="https://img.shields.io/badge/Regex-text%20cleansing-FF6F00">
  <img alt="License" src="https://img.shields.io/badge/License-MIT-green">
</p>

---

## 🎬 The Story

Real-world data almost never arrives clean, and it almost never arrives in one place. This project
begins exactly there: **two raw exports of the same Flickr photo collection** — one in **XML**, one
in **JSON** — that disagree on field names, mix data types, hide emojis and HTML inside text fields,
and carry titles and tags written in a dozen languages and three different alphabets.

The mission is the bread and butter of a data engineer: **parse it, reconcile it, scrub it, and
understand it.** By the end of the notebook, those two chaotic files have become a single tidy table
of **32,382 photos described by 18 attributes** — and, more importantly, a set of *insights* and
*machine-learning research questions* grounded in what the data actually says.

---

## ✨ What Makes This Project Worth a Look

🧬 **Two formats, one schema.** XML calls it `PostID`, JSON calls it `Post date` — and neither agrees
with the other on casing or spacing. The pipeline parses both sources natively
(`xml.etree.ElementTree` + `json`), maps every field onto a single canonical schema, stacks them, and
deduplicates on the photo id so no picture is counted twice.

🌍 **Regex surgery on multilingual text.** The five free-text columns (`Title`, `City`, `Country`,
`Tags`, `Description`) are a minefield: HTML tags, emojis, Japanese, Chinese, Cyrillic, and accented
European characters all tangled together. Using **regular expressions and Unicode classification**,
the pipeline strips markup and emojis, *removes non-Latin scripts while carefully preserving accented
Latin text* (so `españa` survives but `日本語` is cleanly dropped), lowercases everything, and replaces
every flavour of "empty" with a single honest `NaN` sentinel.

🔬 **EDA that asks "is this real, or is this a data artifact?"** Every analysis follows a disciplined
**Purpose → Method → Finding → Risk** structure — because a good analyst doesn't just report a spike,
they ask whether the spike is a genuine pattern or a collection bias.

🧠 **From observations to ML questions.** The notebook closes by translating concrete EDA findings into
well-posed supervised, unsupervised, and time-series machine-learning questions — each with a named
target, candidate predictors, and a justification rooted in the data.

---

## 🔭 A Few Things the Data Revealed

> These are real findings from the notebook — and just as importantly, the *caveats* that come with them.

- 🏷️ **Tagging and titling are independent habits.** More tags does **not** mean a longer title — the
  cloud of points is wide and a huge population of zero-tag posts still sports titles of every length.
  Two separate user behaviours, not one.
- 📅 **France "peaks" every December — but it's a mirage.** Posting activity for France looks wildly
  seasonal until you notice the December spike is really a *data-collection bias*, not a cultural one.
  A textbook reminder to distrust a tidy peak.
- 🌐 **The leading country shifts over time.** France dominates the collection in 2019–2020, then the
  United States takes the crown in 2021 — a genuine temporal signal worth modelling.
- 🕳️ **Geography is mostly missing.** `City` and `Country` are sparsely populated, while precise
  `Latitude`/`Longitude` are far more complete — a critical fact for any location-based model.

---

## 🗂️ Project Structure

```
flickr-metadata-explorer/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── 01_data_processing_and_eda.ipynb   ← the full journey, with rendered outputs
├── src/
│   └── flickr_pipeline.py                  ← the pipeline as a runnable script
└── data/
    └── samples/
        ├── flickr_metadata_sample.json     ← 20 raw JSON records
        ├── flickr_metadata_sample.xml      ← 20 raw XML records
        └── flickr_dataset_sample.csv       ← 50 rows of the cleaned output
```

---

## 🧱 The Dataset

Two raw inputs describe the same population of public Flickr photos:

| File | Records | Format | Shape |
| ---- | ------: | ------ | ----- |
| `flickr_metadata.json` | ~29,440 | JSON array | one object per photo |
| `flickr_metadata.xml`  | ~14,720 | XML       | one `<record>` per photo |

After reconciliation and deduplication, they become **one dataset of 32,382 records × 18 attributes**:

| Column | Type | What it tells us |
| ------ | ---- | ---------------- |
| `Post_ID` | int | unique Flickr photo id (the dedup key) |
| `User_ID` | str | photo owner (`NSID`) |
| `Secret` | str | photo secret token |
| `Server` / `Farm` | int | Flickr serving-infrastructure ids |
| `Title` | str · lowercased | user-supplied photo title |
| `Is_Public` / `Is_Friend` / `Is_Family` | bool | visibility flags |
| `City` / `Country` | str · lowercased | optional geocoded location (often missing) |
| `Post_Date` / `Taken_Date` | datetime | upload time vs. capture time |
| `Tags` | str | comma-separated tag list |
| `Latitude` / `Longitude` | float | GPS coordinates |
| `Description` | str · lowercased | optional user description |
| `Min_Taken_Date` | datetime | earliest valid "taken" date |

> 🪶 The full ~32k-record corpus isn't tracked in git (it's large). Instead, **runnable samples of all
> three files** live in [`data/samples/`](data/samples) so you can execute the notebook end-to-end in
> seconds. Drop the full `flickr_metadata.json` / `flickr_metadata.xml` beside the notebook to scale up.

---

## 🚀 Quick Start

```bash
# 1 — isolate
python3 -m venv .venv && source .venv/bin/activate

# 2 — install
pip install -r requirements.txt

# 3 — explore
jupyter lab notebooks/01_data_processing_and_eda.ipynb
```

Copy the three files from `data/samples/` next to the notebook and run every cell — that's the whole
demo.

---

## 🧭 The Journey, Step by Step

### Step 1 · Load, parse & merge — *taming two formats*
Read JSON with `json.load` and XML with `ElementTree`; rename every field onto one canonical schema
(`PostID → Post_ID`, `Post date → Post_Date`, …); concatenate; deduplicate on `Post_ID`; then perform
**regex-driven text cleansing** — strip HTML/emojis, remove non-Latin scripts while keeping accented
Latin, lowercase the five text fields, and standardise nulls to `'NaN'`. The result is validated
against a reference schema.

### Step 2 · Exploratory data analysis — *interrogating the data*
- **Univariate** — distribution shapes, skewness & kurtosis, IQR outlier flags, and a per-column
  missing-value audit.
- **Bivariate** — tag usage vs. title and description length, France's monthly posting rhythm,
  the top-country race across years, and the latitude/longitude spatial relationship.
- **Multivariate** — a correlation view plus a derived *tourism signal* that fuses country, time,
  and tag content.

### Step 3 · Research questions — *turning insight into a plan*
Each finding is forged into a concrete ML question with a stated target, predictors, and the best-suited
technique (classification, clustering, anomaly detection, time-series forecasting…).

---

## 🛠️ Techniques Showcased

`Multi-format ingestion (XML + JSON)` · `Schema reconciliation` · `Deduplication` ·
`Regex & Unicode text cleansing` · `Multilingual / non-Latin filtering` · `Missing-data profiling` ·
`Univariate / bivariate / multivariate EDA` · `Skewness · kurtosis · IQR` · `Correlation analysis` ·
`Feature engineering (upload lag, tourism signal)` · `Insight-to-ML-question translation`

**Stack:** Python · pandas · NumPy · Matplotlib · seaborn · SciPy · scikit-learn

---

## 📄 License

Released under the [MIT License](LICENSE) — free to learn from, build on, and share.
