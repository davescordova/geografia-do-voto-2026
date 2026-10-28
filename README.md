# Geografia do Voto 2026 - Presidential Runoff by Voting Venue, Neighborhood and Municipality

Interactive map of the 2026 Brazilian presidential runoff (2nd round, 25 Oct 2026) built from official TSE ballot box reports, georeferenced at the level of each voting venue and aggregated into IBGE 2022 neighborhoods, municipalities and overseas cities.

**Live map:** https://davescordova.github.io/geografia-do-voto-2026/

## Contents

| File | Description |
|---|---|
| `index.html` | Self-contained interactive map (Leaflet / Folium) |
| `Geografia_do_voto.ipynb` | Full reproducible pipeline (Python, Jupyter) |
| `LICENSE` | MIT license (code) and third-party data attribution |

## Map layers

- **Bairros - vencedor:** IBGE 2022 Census neighborhoods colored by the winner among the voting venues located inside each neighborhood (venues with exact or street-level coordinates only).
- **Municípios - vencedor:** municipalities colored by the winner among all Brazilian venues inside each municipality.
- **Limites estaduais:** state boundaries.
- **Cidades no exterior - vencedor:** overseas cities (TSE `ZZ`) painted on their OpenStreetMap municipal boundary.
- **Locais de votação:** clustered venue markers with section-level results in the popup; hollow dashed markers flag approximate coordinates.

Colors: Lula in red, the other contender in blue, ties in gray. Fill intensity grows with the winner's margin over the two-candidate vote (full intensity at 40 p.p. or more).

## Data sources

| Source | Dataset | Use |
|---|---|---|
| TSE | Eleitorado por local de votação 2026 (`eleitorado_local_votacao_2026`) | Voting venues, sections, coordinates, registered voters |
| TSE | Resultados 2026 - Boletim de Urna (`bweb_2t_<UF>`) | Presidential votes per section (2nd round) |
| IBGE | Malha de bairros - Censo 2022 (`BR_bairros_CD2022`) | Neighborhood polygons |
| IBGE (via `geobr`) | Municipalities 2022, states 2020, municipal seats | Municipality and state polygons, validation |
| IBGE (via `geocodebr`) | CNEFE | Offline geocoding by address and CEP |
| OpenStreetMap | Nominatim | Venue-name geocoding, overseas city boundaries |

## Pipeline

1. **Voting venues:** consolidation of all state files plus overseas voting (`ZZ`), keeping the 2nd-round venue when a section moved between rounds.
2. **Registered voters per section:** out-of-core aggregation with DuckDB.
3. **First merge and audit:** classification of coordinate problems (sentinel `-1`, missing values, out-of-Brazil coordinates, aggregated sections, overseas voting).
4. **Geocoding of missing coordinates:** cascade by venue name, address, CEP and municipality centroid; each coordinate keeps its origin in `GEO_METODO`.
5. **Re-audit, manual corrections and validation:** point-in-polygon checks against IBGE municipalities, distance checks for overseas venues, and refinement of CEP/municipality centroids with `geocodebr`.
6. **Votes per section:** TSE ballot box reports (`bweb`) filtered to `DS_CARGO_PERGUNTA = 'Presidente'`, stored as Parquet and aggregated with DuckDB.
7. **Map:** spatial joins (venue → neighborhood / municipality), winner and margin per area, simplified geometries, HTML export.

## Methodological notes

- Aggregated sections have no ballot box of their own; their votes are reported in the principal section.
- Sections whose venue had no TSE coordinate were geocoded; approximate coordinates (CEP, locality or municipality centroids) are excluded from the neighborhood layer to avoid assigning votes to the wrong neighborhood.
- IBGE neighborhoods exist only where municipalities have an official intra-urban division; elsewhere the municipality layer provides coverage.
- Overseas boundaries come from OpenStreetMap and may correspond to a district or county rather than the city proper; the tooltip shows the OSM unit used.
- Vote shares in polygons are computed over the two-candidate vote (blank and null votes excluded).

## Reproducing

Requirements: Python 3.12, `pandas`, `numpy`, `duckdb`, `requests`, `geopandas`, `geobr`, `folium`, and R with `geocodebr` (called through `rpy2`).

The notebook downloads all TSE and IBGE inputs automatically. Paths are set in the first cell (`base_dir`); large intermediate files (ZIP, CSV, Parquet) are not versioned in this repository.

## Author

**Daves de Azevedo Cordova** - Economist, doctoral researcher at POSGEO/UFMT (spatial inequality and territorial analysis, Mato Grosso, Brazil).

## License

Code and map rendering: MIT (see `LICENSE`). Data: TSE (CC BY), IBGE, OpenStreetMap contributors (ODbL), CARTO and Esri basemaps.
