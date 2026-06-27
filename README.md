# overture-tiles

Turn [Overture Maps](https://overturemaps.org/) places data into a self-hosted
PMTiles layer using DuckDB — no Microsoft Synapse or AWS Athena needed.

## Requirements

- [DuckDB](https://duckdb.org/)
- [felt/tippecanoe](https://github.com/felt/tippecanoe)

## Usage

1. Download the Overture Parquet dataset into an `overture/` directory
   ([how to access Overture data](https://github.com/OvertureMaps/data/blob/main/README.md#how-to-access-overture-maps-data)).

2. Extract places into `pois.csv` with DuckDB:

   ```sql
   COPY (
     SELECT json_extract(bbox, '$.minx')                   AS x,
            json_extract(bbox, '$.miny')                   AS y,
            json_extract_string(names, '$.common[0].value') AS name,
            json_extract_string(categories, '$.main')       AS category_main
     FROM read_parquet('overture/theme=places/type=place/*')
   ) TO 'pois.csv' (HEADER, DELIMITER ',');
   ```

3. Build vector tiles with tippecanoe:

   ```sh
   tippecanoe -o overture-pois.pmtiles pois.csv -M 80000 --drop-densest-as-needed
   ```

That's it — `overture-pois.pmtiles` is ready to serve.
