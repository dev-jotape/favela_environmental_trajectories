# Environmental Trajectories of Brazilian Favelas

Capstone project for the Yale School of the Environment online certificate in Environmental Data Science (2026–2027).

## Table of Contents

1. [About the Project](#about-the-project)
2. [Research Question](#research-question)
3. [Data](#data)
4. [Built With](#built-with)
5. [Repository Structure](#repository-structure)
6. [How to Run](#how-to-run)
7. [License](#license)
8. [Contact](#contact)

## About the Project

In a previous study ([da Silva et al., 2026](https://doi.org/10.21203/rs.3.rs-9807561/v1)), we grouped the 12,344 favelas mapped in the 2022 Brazilian Census into three infrastructural regimes (Structured, Partially Structured and Unstructured), based on access to water, sanitation, living space and housing quality. That study only describes favelas in 2022.

This project looks at how these places changed over time. Using yearly land cover maps from 1985 to 2025, it estimates when each favela became urban and what was there before (forest, pasture, wetlands, water, etc.). The goal is to see whether this history is related to the infrastructure favelas have today.

The notebooks can run for a single city (pilot) or for all favelas in Brazil. The pilot uses São Paulo (1,357 favelas). See [Pilot or full run](#pilot-or-full-run).

## Research Question

**When was each favela urbanized, and what land cover did it replace? Is this history related to its current infrastructural regime?**

Hypothesis: favelas urbanized more recently, and on natural land cover such as forest or wetlands, are more often Partially Structured or Unstructured.

## Data

| Dataset | What it contains | Type | Source |
|---|---|---|---|
| The State of Brazilian Favelas | Favela boundaries from the 2022 Census, with infrastructure indices, terrain indicators (slope and height above the nearest drainage) and the regime of each favela | Spatial (polygons) and tabular | da Silva (2026), Mendeley Data, version 2: [doi:10.17632/x959s4g493](https://data.mendeley.com/datasets/x959s4g493/2) |
| MapBiomas Brazil, Collection 11 | Yearly land cover maps of Brazil, 30 m resolution, 1985–2025. Class codes follow the [Collection 11 legend](https://brasil.mapbiomas.org/codigos-de-legenda) | Spatial (raster), time series | [MapBiomas](https://brasil.mapbiomas.org), downloaded through Google Earth Engine on 2026-11-17 |

**The data files are not stored in this repository.** They are kept in the project folder on Google Drive. The favela dataset can be downloaded from Mendeley Data, and the MapBiomas data can be downloaded again with `01_download_mapbiomas.ipynb`.

Known issues:

- Pixels are 30 m wide, so very small favelas contain no pixel and have no land cover values.
- Pixels already urban in 1985 have an unknown year of urbanization (1985 or earlier).
- MapBiomas may take a few years to classify sparse or tree-covered occupation as urban.

## Built With

- R (readr, dplyr, tidyr, ggplot2, sf, rgee)
- Python (only to mount Google Drive and log in to Earth Engine)
- Google Colab, Google Drive and Google Earth Engine

## Repository Structure

```
favela_environmental_trajectories/
├── README.md
├── scripts/              notebooks, numbered in the order they run
│   ├── 01_download_mapbiomas.ipynb
│   ├── 02_data_clean.ipynb
│   └── 3_explore_data.ipynb
└── data/                 (Google Drive only)
    ├── raw/              original files, never edited
    └── processed/        files written by 02_data_clean.ipynb
```

### `01_download_mapbiomas.ipynb`

Downloads the MapBiomas land cover of every 30 m pixel inside the selected favelas, for every year from 1985 to 2025.

- **Reads:** `data/raw/state_of_brazilian_favelas.shp`
- **Writes:** `data/raw/mapbiomas_c11_favela_pixels_<run>.csv` and a metadata file

### `02_data_clean.ipynb`

Cleans the pixel data and summarizes it by favela:

1. Keeps the selected favelas and matches the favela code in both files.
2. Removes duplicated pixels and pixels with missing years.
3. Groups the MapBiomas classes into broad land cover types.
4. Finds the first year each pixel became urban and stayed urban for 5 years in a row, to ignore one-year mapping errors.
5. Records what each pixel was before it became urban.
6. Summarizes the pixels by favela and adds the regime of each favela.

- **Reads:** `data/raw/mapbiomas_c11_favela_pixels_<run>.csv`, `data/raw/state_of_brazilian_favelas.dbf`
- **Writes** (in `data/processed/`):
  - `2_favela_pixels_clean_<run>.csv`: one row per pixel
  - `2_favela_urbanization_<run>.csv`: one row per favela
  - `2_class_lookup.csv`: MapBiomas codes and their land cover groups
  - `2_data_dictionary.csv`: description of each column

### `3_explore_data.ipynb`

Plots and tables by regime: land cover over time, year of urbanization, land cover before urbanization and terrain.

- **Reads:** the files in `data/processed/`
- **Writes:** nothing

## How to Run

You need a Google account (Drive and Colab), a [Google Earth Engine](https://earthengine.google.com) account with a Cloud project (only for the first notebook), and the favela dataset saved in `data/raw/` on Drive.

Run the notebooks in order. Each one starts with a Python cell that mounts Google Drive. After running it, change the runtime to R (**Runtime → Change runtime type → R**) and run the rest.

1. `01_download_mapbiomas.ipynb`: the first cell also logs in to Earth Engine. Set your Earth Engine project in the settings cell.
2. `02_data_clean.ipynb`
3. `3_explore_data.ipynb`

### Pilot or full run

The `PILOT` setting in the first notebook chooses which favelas are used:

- `PILOT <- TRUE`: only the favelas of one municipality, set by `PILOT_MUN` (default: São Paulo, `3550308`). The favela polygons are sent directly to Earth Engine.
- `PILOT <- FALSE`: all favelas in Brazil. This run is too large to send directly, so the favela shapefile must first be uploaded to Earth Engine as an asset (set its path in `FAVELA_ASSET`).

The name of each data file ends with the run it comes from (`<run>` = `pilot` or `brazil`), so files from different runs do not overwrite each other. Use the same settings in all three notebooks.

## License

The code in this repository is released under the MIT License.

The data are not part of this repository and keep their own licenses: MapBiomas data and the favela dataset are licensed under CC-BY-4.0.

## Contact

João Pedro da Silva, jp.silva@usp.br
