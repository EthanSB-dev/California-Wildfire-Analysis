# California Wildfire Analysis

**Who starts California's wildfires, and which counties face rising risk?**

This project combines SQL, GIS, and machine learning in R to study how human development relates to where wildfires start in California. It compares human- and lightning-caused fires, measures housing growth in the wildland-urban interface (WUI), the zone where homes meet wildland vegetation, and predicts county ignition density with a random forest model.

Final project for GEOG 124 at UC Santa Barbara (Fall 2026).

## Research Question

How much do human factors (population, WUI housing growth, roads) explain where wildfires start in California, and can they predict which counties face rising ignition risk?

## Key Findings

- **Humans start most fires.** Among fires with a known cause, human-caused ignitions outnumber lightning-caused ignitions every year from 1992 to 2021. In 2020, 88.7% of known-cause fires were human-caused.
- **Human and lightning fires occur in different places.** Human-caused fires concentrate in the Bay Area, the Sierra Nevada foothills, and Southern California. Lightning fires cluster in remote northern and high-Sierra counties.
- **Fire activity and WUI growth go together.** Counties adding more homes at the wildland edge also tend to have more fires per area (Spearman's rho = 0.553, p < 0.001).
- **22 of 58 counties face rising risk.** They combine high ignition density with fast WUI housing growth. Contra Costa, Placer, and Alameda rank highest.
- **Fires start where people meet fuel.** A random forest predicted 2016–2020 ignition density better than linear regression (test R² = 0.430 vs. 0.255). Vegetation was the single most important feature, but human features together carried most of the predictive power.

## Methods

1. **SQL:** Queried a SQLite wildfire database to pull California ignitions and compare fires by cause over time.
2. **GIS:** Converted fire coordinates to spatial points and assigned each to a county with a spatial join. Calculated ignition density (fires per 100 km² per year) and WUI housing growth (new homes per 100 km², 2000–2020), and measured major road length per county.
3. **Statistics:** Tested the relationship between ignition density and WUI growth with a Spearman rank correlation, then classified counties into four risk groups.
4. **Machine learning:** Built a county-year dataset (57 counties × 21 years), trained linear regression and random forest models on 2000–2015, and tested them on 2016–2020.

## Tools

- **R / R Markdown:** analysis and presentation
- **SQL (SQLite):** `DBI`, `RSQLite`
- **GIS and mapping:** `sf`, `tmap`, `tigris`
- **Data wrangling:** `dplyr`
- **Machine learning:** `randomForest`
- **AI-assisted development:** Claude, used for help with code, debugging, and interpreting results

## Data

The raw data is too large for GitHub, so the `data/` folder is not included in this repository. To reproduce the analysis, download:

| Data | Source | File to download |
|---|---|---|
| Wildfire ignitions, 1992–2024 | Short, K.C. 2026. *Spatial wildfire occurrence data for the United States, 1992–2024* (FPA FOD, 7th Ed.). USDA Forest Service Research Data Archive. https://doi.org/10.2737/RDS-2013-0009.7 | SQLite format |
| WUI housing and vegetation, 1990–2020 | Radeloff, V.C. et al. 2023. *The 1990–2020 wildland-urban interface of the conterminous United States* (4th Ed.). USDA Forest Service Research Data Archive. https://doi.org/10.2737/RDS-2015-0012-4 | `RDS-2015-0012-4.zip` |
| County boundaries and major roads | U.S. Census TIGER/Line, loaded in R with `tigris` | No download needed |

Place the files like this:

```
California-Wildfire-Analysis/
├── California-Wildfire-Analysis.Rproj
├── Final-Presentation.Rmd
├── README.md
└── data/
    ├── OtherData/
    │   └── CONUS_WUI_block_1990_2020_change_v4.gdb
    └── RDS_SQL/
        └── FPA_FOD_20260615.sqlite
```

### One-time data prep

The national WUI file is about 4.65 GB. Run this once in the R Console to save a California-only copy before knitting:

```r
library(sf)

gdb <- "data/OtherData/CONUS_WUI_block_1990_2020_change_v4.gdb"
lyr <- st_layers(gdb)$name[1]

ca_wui <- st_read(gdb,
                  query = sprintf('SELECT * FROM "%s" WHERE BLK20 LIKE \'06%%\'', lyr))

st_write(ca_wui, "data/OtherData/ca_wui.gpkg")
```

Then open `Final-Presentation.Rmd` and knit.

## Limitations

- **Missing fire causes:** The share of fires with an unknown cause rose from about 21% in 2015 to 45% in 2020, and above 78% after 2021. Cause-based results are therefore limited to 2000–2020, and the model predicts total ignitions.
- **Interpolated housing:** WUI housing is measured every 10 years, so yearly values are estimated linearly between census years.
- **No weather data:** The model does not include temperature, drought, or wind, so it underpredicts extreme fire years.
- **County scale:** Counties are large, so risk within a county can vary a lot. San Francisco was excluded from the model because its small area inflates its densities.
- **Association, not causation:** The results show that human development and fire activity overlap, not that housing growth directly causes fires.

## Author

Ethan Anghelache, B.S. Statistics and Data Science, UC Santa Barbara
