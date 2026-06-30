# Sample Data

These small files let the notebook run end-to-end without the full corpus.

| File | Rows | Source |
| ---- | ---- | ------ |
| `flickr_metadata_sample.json` | 20 | first 20 records of the raw JSON dump |
| `flickr_metadata_sample.xml` | 20 | first 20 records of the raw XML dump |
| `flickr_dataset_sample.csv` | 50 | first 50 rows of the cleaned dataset |

To rebuild the cleaned CSV against the samples, copy `flickr_metadata_sample.json`
and `flickr_metadata_sample.xml` into the notebook's working directory and run
Step 1. To work with the full ~32k-record corpus, place the complete
`flickr_metadata.json` and `flickr_metadata.xml` files in the same location.
