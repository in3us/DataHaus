# DataHaus
### Massachusetts Real Estate Analytics

**Live tool:** [datahaus.claude.ai](#) <!-- replace with your published artifact URL -->

---

## What It Does

DataHaus is a browser-based real estate analytics tool that gives Massachusetts homebuyers a transparent, data-driven way to evaluate whether a listing is fairly priced relative to the current market. Most buyers rely on gut feel or ask their agent — DataHaus gives them an independent reference point built entirely from public market data.

The core premise is simple: a listing's fair value isn't a single number, it's a position within a distribution. Rather than guessing whether $850,000 is "too much" for a 3-bedroom condo in Somerville, DataHaus shows you exactly where that listing lands across seven market metrics — simultaneously, in context, against comparable active listings.

---

## The Data Visualization: Percentile Gradient Bars

The centerpiece of DataHaus is a set of horizontal gradient bars inspired by [Baseball Savant's](https://baseballsavant.mlb.com/) player percentile ranking design — one of the most effective sports analytics visualizations ever built. The adaptation to real estate works as follows:

Each bar spans a red-to-green gradient representing the full distribution of comparable listings in the market. A circular marker shows where the subject listing falls within that distribution, and a percentile label makes the position explicit.

**The insight this unlocks:** price alone is meaningless without context. A $700,000 condo might be a bargain in Cambridge and overpriced in Worcester. By showing price per square foot, total square footage, days on market, year built, and HOA fees as simultaneous percentile rankings — all filtered to comparable properties in the same geography — buyers can instantly see the full picture:

- Is this listing priced below or above comparable properties? *(List Price bar)*
- Is it getting good space for the money? *(Price per Sq Ft + Square Footage bars)*
- Has it been sitting because something is wrong, or is it freshly listed? *(Days on Market bar)*
- Is it a newer build commanding a premium, or an older home priced accordingly? *(Year Built bar)*
- Is the HOA eating into the apparent affordability? *(HOA/Month bar)*
- Is the per-bedroom value competitive? *(Price per Bedroom bar)*

Seven dimensions. One glance. No spreadsheet required.

The visualization is deliberately directional — higher on every bar is better for the buyer, regardless of which direction the underlying metric runs. Low price per square foot is good. High square footage is good. Short days on market means competition; long days on market means negotiating room. The inversion logic is handled automatically so buyers never have to think about which way is "better."

---

## Features

### Listing Analyzer
Enter a listing's details — price, square footage, bedrooms, bathrooms, and optionally year built, days on market, and HOA — and compare it against the live market. Filter by property type, bedroom count, and geography (All Massachusetts → By County → By Town) to narrow the comparison pool to the most relevant set of listings.

A pool size warning appears when fewer than 10 comparable listings are available, preventing misleading percentiles on thin data.

### Affordability Rankings
A ranked table of 154 Massachusetts towns by Price-to-Income Ratio, calculated as median home price (Zillow ZHVI, February 2026) divided by median household income (U.S. Census ACS 5-Year, 2024). Towns are color-coded from affordable (≤4x, green) to severely unaffordable (>8x, red). Click any row to pull up the full detail card for that town.

This tab answers the question buyers often ask before they even start searching: *which towns can I actually afford?*

---

## Tech Stack

### Data Collection — Python + Selenium
Active listing data was scraped from Redfin using a custom Python scraper built with **Selenium WebDriver** and **webdriver-manager**. The scraper navigates to each Massachusetts zip code's search results page, triggers the CSV download button, and moves the file to a local output directory. It handles login sessions, anti-bot detection via realistic browser behavior and randomized delays, progress checkpointing (resumable runs), and automatic CSV consolidation on completion.

**4,941 active listings** were collected across **216 zip codes**, **161 towns**, and **13 counties** in March 2026.

### Data Processing — Python + pandas
Raw Redfin CSVs were merged, deduplicated by address and zip code, cleaned of vacant land and non-residential property types, and enriched with town and county mappings via a hand-built Massachusetts zip code reference table. Numeric fields were standardized, outliers capped, and the final dataset serialized to a compressed indexed JSON format (221KB) suitable for browser embedding.

Affordability data was sourced separately:
- **Zillow Home Value Index (ZHVI)** — February 2026 median home values for 154 Massachusetts towns
- **U.S. Census ACS 5-Year Survey (2024)** — median household income by town

The two datasets were joined on town name, and Price-to-Income ratios were calculated for all 154 matched towns.

### Application — React (JSX)
The front end is a single-file React application with no external dependencies beyond React itself. The full listing dataset is embedded directly in the component as a compressed indexed payload, making the app fully self-contained with no API calls or backend required.

Key implementation details:
- **Percentile calculation** is computed client-side in real time using JavaScript array sorting — no pre-computed bins
- **Geography filtering** uses a cascading county → town drill-down, with town dropdowns that automatically filter to only towns within the selected county
- **Pool size validation** prevents analyses on fewer than 10 comparable listings
- **Color logic** is unified across all bars so higher percentile always means better buyer value, regardless of whether the underlying metric is inverted

### Development Environment
DataHaus was built with assistance from **Claude** (Anthropic), used as a development partner for data pipeline construction, React component architecture, data compression strategy, and iterative UI refinement. The scraper, data processing scripts, zip/county mapping, and front-end application were all developed collaboratively across an extended session.

---

## Data Sources & Freshness

| Source | Data | Date |
|--------|------|------|
| Redfin (scraped) | 4,941 active MA listings | March 2026 |
| Zillow ZHVI | Median home values, 154 MA towns | February 2026 |
| U.S. Census ACS | Median household income, 154 MA towns | 2024 (5-Year) |

The listing dataset is a snapshot and will drift from current market conditions over time. The affordability rankings update on a slower cycle and remain directionally accurate for 12–18 months.

---

## Limitations

- Listing data is a point-in-time snapshot from March 2026. Percentile rankings reflect market conditions at that time.
- Redfin data coverage is strong in Greater Boston and suburban Massachusetts but thinner in rural western MA.
- Affordability ratios use median values and do not account for property tax rates, insurance costs, or local amenities.
- Pool sizes in smaller towns may be limited, which can reduce the reliability of percentile rankings. The tool warns when fewer than 10 comparables are available.

---

## Author

**Steven Myette**
Data Professional — Boston, MA
[LinkedIn](#) · [GitHub](#) · [Tableau Public](https://public.tableau.com/app/profile/steven.myette)
