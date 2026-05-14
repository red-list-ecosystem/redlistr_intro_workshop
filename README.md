# redlistr Introductory Workshop

This repository contains materials for a basic introductory workshop to the `redlistr` package version `2.0`

A beginner workshop for using the `{redlistr}` package in R to calculate the key spatial metrics used in the **IUCN Red List of Ecosystems** and **IUCN Red List of Threatened Species** workflows — from ecosystem distribution rasters and other spatial data formats.

> **Workshop dataset:**
>
> Mangrove distributions from Western Port Bay (French Island), Victoria, Australia — two time points: 2000 and 2017.
>
> The occurrence records for alpine plant species (primarily *Richea continentis*) from south-eastern Australia (NSW/VIC ranges). Columns: `species`, `lon`, `lat`, `uncertainty` (metres), `date`, `year`, `month`. Records span 1910–2006 from herbarium and survey sources.

------------------------------------------------------------------------

## Prerequisites

Install the following software **before the workshop**. All are free.

| Software | Purpose | Download |
|-------------------------|-----------------------|-------------------------|
| **R** (≥ 4.2) | The analysis language | <https://cran.r-project.org> |
| **RStudio** | IDE (code editor) | <https://posit.co/download/rstudio-desktop> |
| **Quarto** | Workshop document format (optional) | <https://quarto.org/docs/get-started> |

No prior R experience is assumed. If you are new to R, work through [R for Beginners](https://cran.r-project.org/doc/contrib/Paradis-rdebuts_en.pdf) chapters 1–3 before the workshop.

------------------------------------------------------------------------

## Installation {#installation}

### Step 1 — Install R packages

Run this **once** on your computer before the workshop. You do not need to repeat this every session.

``` r
# Spatial and visualisation packages (from CRAN)
install.packages(c(
  "sf",       # vector spatial data (points, lines, polygons)
  "terra",    # raster spatial data
  "leaflet"   # interactive web maps
))
 
# redlistr — install the development version from GitHub
# (CRAN release v1.0.4 has older function signatures — use GitHub for this workshop)
install.packages("remotes")
remotes::install_github("red-list-ecosystem/redlistr")
```

> **If prompted to update packages**, enter `1` to update ALL and click **Yes** in the popup window.

### Step 2 — Download the workshop repository

1.  Go to the [workshop GitHub page](https://github.com/red-list-ecosystem/redlistr_intro_workshop)
2.  Click the green **Code** button → **Download ZIP**
3.  Unzip and save the folder somewhere convenient (e.g., `Desktop/redlistr_workshop/`)
4.  Open RStudio → File → Open Project → select `redlistr_intro_workshop.Rproj` Opening the `.Rproj` file automatically sets your working directory to the workshop folder. All file paths in this workshop assume that.

------------------------------------------------------------------------

## Repository Structure {#repository-structure}

```         
redlistr_intro_workshop/
├── Data/
│   ├── WhiteAshForest/         ← shapefile dataset for exercises
│   │   └── (spatial files)
│   └── Plant_species_points.csv  ← species point occurrences
├── Exercises/
│   ├── Exercises_part1.qmd     ← Lessons 1–2 exercises
│   ├── Exercises_part2.qmd     ← Lessons 3–4 exercises
│   ├── Exercises_part1_ANSWERS.qmd
│   └── Exercises_part2_ANSWERS.qmd
├── redlistr_intro_workshop.Rproj  ← open this in RStudio
└── README.md
```

# Theory

**What is Criterion A?**\
Criterion A assesses **decline in ecosystem extent** over a 50-year window (past, present, or future). Two decline rate methods are used:

-   **ARD** (Absolute Rate of Decline): constant km²/yr loss
-   **PRD** (Proportional Rate of Decline): constant % per year loss, like compound interest in reverse
-   **ARC** (Annual Rate of Change): log-linear rate (Puyravaud 2003) **IUCN thresholds for Criterion A:**

| Category | Decline over 50 yrs |
|----------|---------------------|
| CR       | ≥ 80%               |
| EN       | ≥ 50%               |
| VU       | ≥ 30%               |

**What is EOO?**\
EOO is the area of the **minimum convex polygon** (smallest polygon with no inward angles) drawn around all known occurrences of an ecosystem or species. It is not the same as the actual occupied area — it captures the spatial breadth of the distribution and vulnerability to broad-scale threats (e.g., climate shifts, regional land clearing).

**IUCN thresholds for Criterion B1:**

| Category                   | EOO           |
|----------------------------|---------------|
| Critically Endangered (CR) | \< 2,000 km²  |
| Endangered (EN)            | \< 20,000 km² |
| Vulnerable (VU)            | \< 50,000 km² |

**What is AOO?**\
AOO is the number of **2 km × 2 km grid cells** occupied by the ecosystem or species. The IUCN mandates the 2 km grid for standardisation. Because AOO depends on grid placement, `{redlistr}` uses grid-shifting to find the **minimum AOO** (most conservative estimate).

**IUCN thresholds for Criterion B2:**

| Category | AOO (km²) |
|----------|-----------|
| CR       | \< 2 km²  |
| EN       | \< 20 km² |
| VU       | \< 50 km² |

## Data Formats Supported

`{redlistr}` accepts three spatial formats. Use whichever matches your data source.

| Format | Class | Typical Source | Read with |
|-----------------|-----------------|---------------------|-----------------|
| **Raster** (GeoTIFF, etc.) | `SpatRaster` (terra) | Remote sensing, ALA gridded layers | `terra::rast()` |
| **Polygon** (Shapefile, GeoPackage) | `sf` | TERN, state agency shapefiles | `sf::st_read()` |
| **Points** (CSV with lat/lon) | `sf` (after conversion) | ALA species records, GBIF | `read.csv()` + `st_as_sf()` |

## Tips

-   All formats must be in a **projected CRS (metres)** before being passed to any `{redlistr}` function. **Rule of thumb:** For small study areas (\< 500 km across), almost any projected CRS will give consistent results. For national-scale analyses, test whether different CRS choices affect your outputs.

-   `{redlistr}` requires coordinates in **metres** (projected CRS), not degrees (geographic CRS). The package will throw an error if you pass in longitude/latitude data.

-   **What is `|>`?** The pipe operator passes the result of one step directly into the next function, without saving intermediate objects. It makes code easier to read top-to-bottom.

-   **Note on function naming:** The dev version uses `getEOO()` (not `makeEOO()`). The returned object has slots accessible with `@` (e.g., `EOO@EOO` for the area, `EOO@spatial` for the polygon geometry). Toggle layers on/off in the map to compare the two time points visually.

-   For small study areas (\< 500 km across), almost any projected CRS will give consistent results. For national scale analyses, test whether different CRS choices affect your outputs.

## Common Errors & Fixes

| Error message | Cause | Fix |
|-------------------------------------|------------------|-----------------|
| `Input raster has a longitude/latitude CRS` | Data is in degrees (EPSG:4326) | `project(r, "EPSG:32755")` for rasters; `st_transform(v, 32755)` for vectors |
|  |  |  |
|  |  |  |
|  |  |  |
| `could not find function "makeEOO"` | Using old function name | Use `getEOO()` in the dev version |
| `crs(r1) == crs(r2)` returns `FALSE` | Two rasters in different CRS | Reproject one to match: `project(r2, crs(r1))` |

------------------------------------------------------------------------

## References

IUCN (2024). *Guidelines for the application of IUCN Red List of Ecosystems Categories and Criteria, Version 2.0*. Keith, D.A., Ferrer-Paris, J.R., Ghoraba, S.M.M., Henriksen, S., Monyeki, M., Murray, N.J., Nicholson, E., Rowland, J., Skowno, A., Slingsby, J.A., Storeng, A.B., Valderrábano, M. & Zager, I. (Eds.). IUCN, Gland, Switzerland. <https://doi.org/10.2305/CJDF9122>

Keith, D.A., Rodríguez, J.P., Rodríguez-Clark, K.M., et al. (2013). Scientific foundations for an IUCN Red List of Ecosystems. PLOS ONE, 8(5), e62111. <https://doi.org/10.1371/journal.pone.0062111>

Puyravaud, J.-P. (2003). Standardizing the calculation of the annual rate of deforestation. *Forest Ecology and Management*, 177(1–3), 593–596. <https://www.sciencedirect.com/science/article/pii/S0378112702003353>
