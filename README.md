# RASO: RASter Ontology

RASO is a lightweight ontology for describing raster datasets and their spatial, thematic, and temporal structure.

It represents the grid shared by one or more rasters, the cells forming that grid, the pixels corresponding to those cells, and the measurements recorded for the raster bands. RASO was originally developed to represent the climate and weather rasters distributed by [WorldClim](https://www.worldclim.org/), but its core model can be reused with other raster datasets.

- **Ontology IRI:** <http://raster.gsic.uva.es/ontology/>
- **Preferred prefix:** `raso`
- **Current version:** 0.4
- **Ontology file:** [`raster_ontology.ttl`](raster_ontology.ttl)
- **Author:** [Guillermo Vega Gorgojo](https://www.gsic.uva.es/members/guiveg)

## Purpose and scope

A geospatial raster is composed of pixels arranged in a rectangular grid. Each pixel stores data for the spatial cell at the corresponding grid position.

RASO separates:

1. the **spatial structure**, represented by grids and cells; and
2. the **raster data**, represented by raster datasets, bands, pixels, variables, and measurements.

This separation allows multiple raster datasets, representing different variables or time periods, to share the same spatial grid.

RASO supports:

- regular grids with different spatial resolutions and geographic extents;
- cells identified by row and column;
- cell geometries and centroids;
- raster datasets associated with grids;
- climate and weather raster specializations;
- single-band and multi-band rasters;
- variables, units, and temporal metadata;
- measurements associated with pixels and bands; and
- compact representations that attach values directly to pixels.

## Conceptual model

The main relationships defined by RASO are:

```text
Grid ── containsCell ─────> Cell

RasterDataset ── hasGrid ───────> Grid
RasterDataset ── containsPixel ──> Pixel
RasterDataset ── hasBand ───────> Band
RasterDataset ── hasVariable ───> Variable

Pixel ── hasCell ───────────> Cell
Pixel ── hasMeasurement ────> Measurement

Measurement ── belongsToBand ──> Band
Measurement ── hasValue ───────> literal

Band ── hasVariable ────────> Variable
Variable ── hasUnit ────────> literal
```

A `raso:Grid` defines a regular spatial subdivision and contains `raso:Cell` entities. Each cell is identified by its row and column and may have a polygon geometry and a centroid.

A `raso:RasterDataset` is associated with a grid and contains pixels and bands. Each `raso:Pixel` corresponds to a grid cell and may have one or more `raso:Measurement` entities. Each measurement belongs to a band and records the value observed for that pixel and band.

RASO also defines shortcut properties such as `raso:hasValueYear` and `raso:hasValueMonth01` to `raso:hasValueMonth12`. These properties support a more compact representation by attaching values directly to pixels.

## Main classes

| Class | Description |
|---|---|
| `raso:Grid` | A regular coordinate mesh covering a geographic area. |
| `raso:Cell` | A rectangular spatial unit within a grid. |
| `raso:RasterDataset` | A gridded dataset composed of pixels. It specializes `dcat:Dataset`. |
| `raso:ClimateRaster` | A raster dataset containing climate data. |
| `raso:WeatherRaster` | A raster dataset containing weather data. |
| `raso:Pixel` | The raster data corresponding to a cell of the associated grid. |
| `raso:Band` | A sample dimension or layer represented in the raster pixels. |
| `raso:Measurement` | A value observed for a particular pixel and band. |
| `raso:Variable` | The phenomenon represented by a raster or band, such as precipitation or temperature. |

## Main properties

| Property | Description |
|---|---|
| `raso:containsCell` | Associates a grid with one of its cells. |
| `raso:hasGrid` | Identifies the grid used by a raster dataset. |
| `raso:containsPixel` | Associates a raster dataset with one of its pixels. |
| `raso:hasCell` | Associates a pixel with its corresponding spatial cell. |
| `raso:hasBand` | Associates a raster dataset with one of its bands. |
| `raso:hasMeasurement` | Associates a pixel with one of its measurements. |
| `raso:belongsToBand` | Identifies the band associated with a measurement. |
| `raso:hasVariable` | Identifies the phenomenon represented by a raster or band. |
| `raso:hasValue` | Records the value of a measurement. |
| `raso:hasUnit` | Specifies the measurement unit of a variable. |

Grid dimensions are described through properties such as `raso:rows`, `raso:columns`, `raso:cellWidth`, and `raso:cellHeight`. Cell locations are identified through `raso:row` and `raso:column`.

Cell geometries are represented using GeoSPARQL terms such as `geo:hasGeometry`, `geo:hasCentroid`, and `geo:asWKT`.

## Reused vocabularies

RASO reuses established vocabularies whenever possible:

- [DCAT](https://www.w3.org/TR/vocab-dcat-3/) for dataset descriptions and temporal resolution;
- [GeoSPARQL](https://www.ogc.org/standard/geosparql/) and Simple Features for spatial features and dublincore.org/specifications/dublin-core/dcmi-terms/) for titles, creators, and spatial and temporal coverage;
- [FOAF](http://xmlns.com/foaf/0.1/) for organizations and agents; and
- [SKOS](https://www.w3.org/TR/skos-reference/) for documentation annotations.

## Example

The following example describes a grid, one of its cells, a raster dataset, and a pixel corresponding to that cell.

```turtle
@prefix ex:   <http://example.org/data/> .
@prefix raso: <http://raster.gsic.uva.es/ontology/> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .

ex:Grid_10m a raso:Grid ;
    raso:columns 2160 ;
    raso:rows 1080 ;
    raso:cellWidth "0.16666667"^^xsd:decimal ;
    raso:cellHeight "0.16666667"^^xsd:decimal ;
    raso:containsCell ex:Cell_10m_r290c1051 .

ex:Cell_10m_r290c1051 a raso:Cell ;
    raso:row 290 ;
    raso:column 1051 .

ex:Raster_10m_prec a raso:ClimateRaster ;
    raso:hasGrid ex:Grid_10m ;
    raso:hasVariable ex:Variable_prec ;
    raso:containsPixel ex:Pixel_10m_r290c1051_prec .

ex:Variable_prec a raso:Variable ;
    raso:hasUnit "mm" .

ex:Pixel_10m_r290c1051_prec a raso:Pixel ;
    raso:hasCell ex:Cell_10m_r290c1051 ;
    raso:hasMeasurement ex:Measurement_10m_r290c1051_prec .

ex:Measurement_10m_r290c1051_prec a raso:Measurement ;
    raso:hasValue "44"^^xsd:decimal .
```

## Use with WorldClim

RASO was developed as part of a workflow for converting WorldClim climate and weather rasters into RDF. Two supporting tools are available:

- **Grid Generator**, which generates RDF descriptions of regular grids and their cells; and
- **Raster RDFizer**, which generates raster metadata and, for selected rasters, RDF descriptions of pixels and measurements.

The tools and conversion workflow are available in the [worldclim2rdf repository](https://github.com/guiveg/worldclim2rdf).

[SCRAPI](https://github.com/guiveg/scrapi) provides a semantic API over the same model. It can generate representations of raster entities and retrieve raster values without materializing the complete RDF dataset.

## Repository contents

- raster_ontology.ttl: RASO ontology in Turtle.
- README.md: ontology overview and usage documentation.