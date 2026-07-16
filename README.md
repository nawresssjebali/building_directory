# Doula UK — Directory Scraper & Dataset Cleaner
 
A Python/Colab pipeline that builds a dataset of Doula UK members (initially scoped to Greater London), scrapes their public profiles, and cleans the result into an analysis-ready CSV.
 
**Notebook:** [Open in Colab](https://colab.research.google.com/drive/1lW1qbnj-drMQvEk-NevYfuTCWDgl8B9J)
 
## What it does
 
The pipeline runs in three stages:
 
### 1. Build a London postcode list
Queries the [postcodes.io](https://postcodes.io) `/outcodes` reverse-geocode endpoint from 8 spread-out points across Greater London (25 km radius each, to work around the API's per-call cap), takes the union of results, and filters to only outcodes whose `admin_district` is an actual London borough — excluding postcodes that dip into neighbouring counties (e.g. Kent, Surrey, Essex, Herts).
 
### 2. Sweep the Doula UK directory
For each London postcode district, queries Doula UK's "Find a Doula" search (a Laravel Livewire endpoint) and keeps only doulas whose returned distance is **exactly 0 miles** for that district — meaning the district *is* their registered postcode, not just somewhere within their travel radius. This gives a confirmed list of members, their real registered district, and their profile slug for the next stage.
 
### 3. Scrape & clean profiles
Visits each doula's profile page to extract:
- Name, location, service type (birth / postnatal / both)
- Bio text
- Website, email, phone, social links
Then runs several cleaning/verification passes:
- Deduplication
- Website URL normalization and reachability checks (async, rate-limit aware, with manual re-verification for links that consistently 429 — e.g. Instagram)
- Categorization of contact links (`website`, `social`, `other`)
- Manual correction pass for a handful of hand-verified edge cases
## Output
 
A cleaned CSV (`doula_uk_cleaned_vX.csv`) with one row per doula, including profile details, cleaned/verified website info, and a `has_bio` flag.
 
## Requirements
 
```
requests
beautifulsoup4
pandas
httpx[http2]
```
 
> **Note:** This script was exported directly from Colab and still contains Colab-only lines (`!pip install`, `google.colab.files.upload()` / `.download()`). It's meant to be run as a Colab notebook, not as a standalone `.py` script — running it outside Colab will require adapting those lines to local file I/O and a plain `pip install`.
 
## Usage
 
Open the notebook in Colab (link above) and run cells top to bottom. Stage 2 is the slowest step (~300 district queries with a polite delay between calls) and can take a while.
 
## Notes
 
- Be respectful of Doula UK's servers: the script includes deliberate delays between requests. Please don't remove these.
- This dataset is scraped from publicly available directory listings for research/analysis purposes. Contact details are already public on doula.org.uk profile pages, but treat any personal data with care.
