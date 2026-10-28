# Blog Post 2: Web Scraping for Actionable Insights

## Research question

**Which day or days of the week offer the most opportunities among events currently listed on the Drexel University Events Calendar for 2026?**

This project uses R to collect event listings, clean the records, and compare the number of **listed event opportunities** across weekdays. An opportunity is an event occurrence on a listed date, not a unique event or a confirmed attendance.

- **Source:** [Drexel University Events Calendar](https://calendar.drexel.edu/EventList.aspx)
- **Study period:** January 1–December 31, 2026
- **Observation date:** Stored in the `scraped_at` column of `data/raw/drexel_events_raw.csv`. The calendar can change, so a new scrape may produce different results.

## Repository structure

```text
RodgersYe_6400_Blog02/
├── RodgersYe_6400_Blog02.Rproj
├── README.md
├── .gitignore
├── code/
│   ├── 01_scrape_events.R
│   └── 02_clean_analyze.R
├── data/
│   ├── raw/
│   │   └── drexel_events_raw.csv
│   └── processed/
│       ├── drexel_events_clean.csv
│       ├── weekday_summary.csv
│       ├── weekday_robustness_comparison.csv
│       └── monthly_counts_clean.csv
└── figures/
    ├── events_by_weekday.png
    └── weekday_robustness_check.png
```

The scripts use `here()` to locate files relative to the project root. The folders and file names above match the paths used in the code.

## Requirements

- R 4.1 or later (the scripts use the native `|>` pipe).
- RStudio is recommended: `01_scrape_events.R` includes an interactive `View()` call.
- An internet connection is needed **only if you choose to scrape the calendar again**.

Install the required R packages once:

```r
install.packages(c(
  "here", "rvest", "robotstxt", "tidyverse", "lubridate", "xml2"
))
```

## How to reproduce the analysis

1. Clone or download this repository.
2. Open **`RodgersYe_6400_Blog02.Rproj`** in RStudio so the project root is recognized.
3. Choose one of the workflows below and run the commands in the RStudio Console.

### Option A: Reproduce the analysis from the included raw data

Use this option to regenerate the cleaned datasets, summary tables, and figures **without contacting the calendar website**. Keep `data/raw/drexel_events_raw.csv` unchanged.

```r
source(here::here("code", "02_clean_analyze.R"))
```

### Option B: Collect a new snapshot and rerun the full workflow

Use this option if you want the results to reflect the calendar listings available **when you run the scraper**. The first script overwrites `data/raw/drexel_events_raw.csv`.

```r
source(here::here("code", "01_scrape_events.R"))
source(here::here("code", "02_clean_analyze.R"))
```

Run the scripts **in that order**. The scraping script checks the site's robots rules, retrieves the calendar month by month, and pauses two seconds between monthly requests. Availability and content of the source website are outside this repository's control.

## Data collection and analysis

**Collection (`01_scrape_events.R`)**

The scraper requests the 2026 monthly calendar views and extracts the event ID, information ID, title, date, URL, source month, and scrape date, along with text from the event listing row. It saves the combined records to `data/raw/drexel_events_raw.csv`.

**Cleaning (`02_clean_analyze.R`)**

The analysis script:

1. Removes records whose titles explicitly indicate cancellation (`cancelled` or `canceled`).
2. Removes repeated records with the same `event_id`, `information_id`, and `event_date`.
3. Identifies multi-day events while retaining separate listed dates as potential attendance opportunities in the main analysis.
4. Constructs weekday, month, and whether-the-event-was-in-the-future-at-scrape variables.

**Weekday comparison**

For each weekday, the main measure is:

```text
Average listed events per weekday =
  Number of cleaned event occurrences on that weekday in 2026
  / Number of times that weekday occurs in the 2026 calendar
```

The script also computes weekday rankings and the share of all listed event occurrences.

**Robustness checks**

- **Observed by scrape date:** Restricts events to dates on or before `scraped_at` and divides by the number of corresponding weekdays from January 1 through the scrape date.
- **Excluding labeled multi-day events:** Repeats the weekday comparison after excluding titles marked `(Multi-Day Event)`.

## Generated outputs

| File | Description |
| --- | --- |
| `data/raw/drexel_events_raw.csv` | Raw event records produced by the scraper; input to the analysis. |
| `data/processed/drexel_events_clean.csv` | Cleaned event occurrences and analysis variables. |
| `data/processed/weekday_summary.csv` | Weekday event counts, calendar-day denominators, averages, shares, and ranks. |
| `data/processed/weekday_robustness_comparison.csv` | Weekday averages under the main and two robustness specifications. |
| `data/processed/monthly_counts_clean.csv` | Cleaned event occurrence counts by month. |
| `figures/events_by_weekday.png` | Main chart of average listed events per weekday. |
| `figures/weekday_robustness_check.png` | Comparison chart across analysis specifications. |

Both figures and all four processed CSV files are regenerated by `02_clean_analyze.R`.

## Interpretation and limitations

- These results describe **events listed on the calendar at the time of collection**, not all events that actually occurred or how many people attended.
- Future events may be missing or added later. This particularly affects months after the scrape date; the observed-by-scrape-date robustness check helps assess this issue.
- An event listed on several dates can contribute several attendance opportunities. The separate multi-day robustness check assesses the effect of explicitly labeled multi-day events.
- **Snapshot reproducibility:** Option A uses the included raw CSV for a stable input; Option B creates a new snapshot and may produce different counts, rankings, or figures.
- **Current script-label caveat:** In `02_clean_analyze.R`, the robustness figure's legend text `Events through September 21` is hard-coded from an older run. The underlying observed-event filter uses the actual `scraped_at` date, but this label does **not** update automatically after a new scrape. Figure titles and subtitles also describe a particular weekday pattern; verify that they still match newly generated results.
- **Current script-comment caveat:** The opening comment in `01_scrape_events.R` mentions an older October 2026–September 2027 date range. The executed `start_date` and `end_date` values, used by both scripts, are January 1–December 31, 2026.
