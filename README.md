# AmbitionBox Analysis

An end-to-end data science project on Indian company data. Company profiles were
scraped from [AmbitionBox](https://www.ambitionbox.com/), cleaned and structured
into a single dataset, analysed, and finally served through an interactive Flask
web app where the whole thing can be filtered and visualised.

The project follows the full data science lifecycle: **data collection ->
data preprocessing -> data analysis -> data visualization.**

- **Companies:** 64,210 (after cleaning)
- **Cities scraped:** 10 major Indian hiring hubs
- **Industries:** 84  ·  **Locations:** 371
- **Interface:** a 4-page Flask app (Home, Explore, Dashboard, About)


## Table of contents

- [What it does](#what-it-does)
- [The data science pipeline](#the-data-science-pipeline)
- [The dataset](#the-dataset)
- [How the raw data was cleaned](#how-the-raw-data-was-cleaned)
- [The web app](#the-web-app)
- [Key findings](#key-findings)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [API reference](#api-reference)
- [Filters](#filters)
- [Notes and caveats](#notes-and-caveats)
- [Author](#author)
- [Disclaimer](#disclaimer)


## What it does

The app turns a raw pile of scraped listings into something you can actually
explore:

- **Filter** 64,210 companies by name, rating, industry, size, type, age and
  location, with searchable multi-selects and dual range sliders.
- **Browse** matching companies in a sortable, paginated table, and export any
  filtered slice to CSV.
- **Visualise** the same filtered slice through twelve charts that redraw the
  instant you change a filter, including a section that breaks down what
  actually correlates with higher company ratings.


## The data science pipeline

### 1. Data collection
Company profiles were web-scraped from AmbitionBox across ten major Indian hiring
hubs: Ahmedabad, Bangalore, Chennai, Gurugram, Hyderabad, Indore, Jaipur, Mumbai,
Noida and Pune. Each city produced its own CSV with the company name, its overall
rating, and a single free-text field holding the rest of the details. That came to
**94,580 rows** in total.

### 2. Data preprocessing
The ten city files were combined, and **30,370 exact duplicate rows** (companies
that appear in more than one city) were removed. The messy free-text details field
was then parsed into five clean columns, ages were converted to numbers, and
missing values were left as blanks rather than guessed. The result is a tidy
dataset of **64,210 unique companies**. See
[How the raw data was cleaned](#how-the-raw-data-was-cleaned) for the details.

### 3. Data analysis
The cleaned data was profiled across 84 industries and 371 locations to look at
rating distributions, company age and size mixes, ownership types, and, most
usefully, which of those factors line up with higher ratings.

### 4. Data visualization
Everything is wrapped in a Flask app with a live dashboard, so the analysis is
interactive rather than a static notebook. Change a filter and every chart and
number updates.


## The dataset

The cleaned dataset lives at `combined_companies_cleaned.csv` (and a copy is
bundled with the app at `ambitionbox_app/data/companies.csv`).

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `company_name` | text | Company name | Zydus Lifesciences |
| `company_rating` | float | Overall AmbitionBox rating, 1.0 - 5.0 | 4.2 |
| `industry` | text | Primary industry | Pharma |
| `size` | text | Employee band | 10k-50k Employees |
| `type` | text | Ownership / classification | Public |
| `years_old` | integer | Company age in years | 72 |
| `location` | text | Head-office location | Ahmedabad |

Coverage and completeness:

- `location` and `company_rating` are present for almost every row (only 82
  companies have no rating).
- `size` is blank for roughly 34% of rows, `industry` for roughly 43%,
  `years_old` for roughly 49%, and `type` for roughly 85%. These gaps come
  straight from the source listings and were deliberately left empty.
- `type` only takes one of nine values: Public, Forbes Global 2000, Startup,
  Fortune India 500, Conglomerate, Indian Unicorn, Central, State and MNC.


## How the raw data was cleaned

The hardest part of preprocessing was the free-text details field. A typical value
looked like this:

```
Pharma , 10k-50k Employees , Public , 72 years old , Ahmedabad +152 more
```

It had to become five separate columns:

```
industry = Pharma
size     = 10k-50k Employees
type     = Public
years_old = 72
location = Ahmedabad
```

The tricky part is that the field is not fixed. Anywhere from one to five of those
pieces can be present, so you cannot rely on position alone. For example, some rows
have no `type` and no `industry`:

```
BPO , 5k-10k Employees , 16 years old , Indore +24 more
```

So each piece is identified by its content rather than its position:

- **size** is the piece that contains the word "Employees".
- **years_old** is the piece that contains "years old" (the number is then
  extracted as an integer).
- **type** is matched against the fixed set of nine known ownership values.
- **location** is always the last piece, with the trailing "+N more" count
  stripped off.
- **industry** is whatever remains at the front.

This approach parses every row with no leftover or misread pieces, and a
row-by-row reconciliation against the raw source confirmed zero mismatches.


## The web app

Four pages, all sharing one filter engine so a selection stays consistent as you
move between them.

| Page | Route | What it does |
|------|-------|--------------|
| Home | `/` | Landing page with animated counters, a feature overview and a short how-it-works. |
| Explore | `/explore` | The filter panel plus a sortable, paginated results table. Export to CSV. |
| Dashboard | `/dashboard` | KPI cards and twelve live charts, filtered through a slide-out drawer. |
| About | `/about` | The project write-up (the data science pipeline) and contact details. |

### Dashboard charts

Five KPI cards (companies, average rating, average age, industry count, top type)
sit above twelve charts:

**Distributions and mixes**
1. Top industries by company count
2. Company type breakdown
3. Rating distribution
4. Top locations
5. Company size distribution
6. Company age distribution
7. Average rating by industry
8. Rating vs age (scatter)

**What drives the ratings** (average rating broken down by dimension, on a zoomed
3-5 axis so the differences are visible)
9. Rating by company size
10. Rating by type
11. Rating by company age
12. Rating by city

A short insight line above this section summarises the current selection in plain
English and updates with the filters.


## Key findings

Across the full dataset:

- **Ownership tracks rating the most.** Fortune India 500 (4.09), central-government
  bodies (4.08) and Forbes Global 2000 firms (4.04) rate highest. Established,
  large organisations score better than the average.
- **Cities differ.** Jaipur (3.95) edges out Hyderabad and Indore (3.92) as the
  best-rated hub among the scraped cities.
- **Age is mostly flat**, but the youngest firms (0-5 years, 4.02) rate slightly
  higher than older ones - a mild honeymoon effect.
- **Size barely matters.** Average ratings sit in a narrow 3.82-3.89 band across
  every employee size.


## Tech stack

- **Python 3** with **Flask** for the backend and JSON API
- **pandas** and **NumPy** for cleaning and aggregation
- **Chart.js** for the charts
- **Tom Select** for searchable multi-selects and **noUiSlider** for range sliders
- Vanilla HTML, CSS and JavaScript on the front end (no build step)

The full dataset is held in memory when the app starts, so filtering and chart
updates return in milliseconds without a database.


## Project structure

```
Ambition_box/
|-- RawData/                       # the 10 scraped per-city CSVs
|   |-- ahmedabad.csv
|   |-- bangalore.csv
|   |-- ... (8 more)
|   +-- pune.csv
|-- combined_companies_cleaned.csv # cleaned, merged dataset (64,210 rows)
|-- README.md                      # this file
+-- ambitionbox_app/               # the Flask web app
    |-- app.py                     # backend + JSON API
    |-- requirements.txt
    |-- run.bat                    # one-click launcher (Windows)
    |-- README.md                  # app-specific quickstart
    |-- data/
    |   +-- companies.csv          # dataset the app reads
    |-- templates/
    |   |-- base.html              # shared layout (nav, footer, script tags)
    |   |-- index.html             # Home
    |   |-- explore.html           # Explore
    |   |-- dashboard.html         # Dashboard
    |   +-- about.html             # About
    +-- static/
        |-- css/
        |   +-- style.css          # the "Midnight Analytics" dark theme
        +-- js/
            |-- filters.js         # shared filter panel
            |-- landing.js         # home-page animations
            |-- explore.js         # table, sorting, pagination
            |-- dashboard.js       # the twelve charts
            +-- creator.js         # click-to-copy contact helper
```


## Getting started

**Prerequisites:** Python 3.9 or newer. The front end loads Chart.js, Tom Select,
noUiSlider and Google Fonts from public CDNs, so the first page load needs an
internet connection. The data and everything else run locally.

**Windows (easiest):** double-click `ambitionbox_app/run.bat`. It installs the
dependencies, opens the browser and starts the server.

**Any operating system (manual):**

```bash
cd ambitionbox_app
pip install -r requirements.txt
python app.py
```

Then open http://127.0.0.1:5000.

**Using a different dataset:** replace `ambitionbox_app/data/companies.csv` with a
CSV that has the same seven columns and restart the app.


## API reference

The front end talks to four JSON endpoints. They all accept the same filter query
parameters (see [Filters](#filters)).

| Endpoint | Returns |
|----------|---------|
| `GET /api/meta` | Filter options (industries, sizes, types, locations, ranges) and headline totals. |
| `GET /api/companies` | Filtered, sorted, paginated rows. Extra params: `page`, `page_size`, `sort`, `order`. |
| `GET /api/analytics` | Aggregations for every dashboard chart and KPI. |
| `GET /api/export` | The current filtered selection as a downloadable CSV. |


## Filters

All filters combine with AND logic and are shared by Explore and the Dashboard.

| Filter | Query param(s) | Notes |
|--------|----------------|-------|
| Company name | `company_name` | Case-insensitive substring match |
| Industry | `industry` | Repeatable; pick any number |
| Location | `location` | Repeatable |
| Size | `size` | Repeatable |
| Type | `type` | Repeatable |
| Rating | `rating_min`, `rating_max` | 1.0 - 5.0 |
| Age | `years_min`, `years_max`, `include_unknown_age` | Slider capped at 150+; keeps unknown-age companies unless turned off |

The age slider is capped at 150 because a single very old outlier (Oxford
University Press, founded in 1478) would otherwise stretch the whole control. At
the top of the slider there is no upper limit, so those old companies still show.


## Notes and caveats

- The star ratings and the numbers come directly from AmbitionBox listings at the
  time of scraping; they are a snapshot, not live data.
- Missing `type` and `years_old` values are common. This is a property of the
  source, not a bug, and range filters on age include an "unknown age" toggle so
  those rows are not silently dropped.
- Average-rating charts use a zoomed 3-5 axis on purpose. Almost every average
  falls between 3.5 and 4.3, so a full 0-5 axis would make every bar look
  identical.


## Disclaimer

This project is for analytical and educational purposes. The underlying company
names, ratings and details belong to AmbitionBox and their respective companies.
It is not affiliated with or endorsed by AmbitionBox.
