# Data cube

## Rationale

This repository contains the functionality to process occurrence data and create aggregated **occurrence data cubes** at species level for European countries. An occurrence data cube is a multi-dimensional array of values. In our context we have three dimensions, ``N = 3``:

1. taxonomic (taxon)
2. temporal (year)
3. spatial (cell code)

For each triplet the stored value represent the number of occurrences found in [GBIF](www.gbif.org/occurrence).

As the data cubes are used as input for modelling and risk assessment, we store the smallest geographic coordinate uncertainty of the occurrences assigned to a certain cell code as value as well. The occurrences are first reassigned randomly within their uncertainty circle before assigning them to a cell. If uncertainty is not available, a default 1000m radius is assigned. Due, to the random assignment, the same occurrence data could result in slightly different data cubes if iterated.

Using a tabular structure (typical of R data.frames), a cube would look like this:

taxon | year | cell code | number of occurrences | minimal coordinate uncertainty
--- | --- | --- | --- | ---
2366634 | 2002 | 1kmE3872N3101 | 8 | 250
2382155 | 2002 | 1kmE3872N3101 | 3 | 250
2498252 | 2002 | 1kmE3872N3149 | 2 | 1000
5232437 | 2002 | 1kmE3872N3149 | 4 | 1000

where number of columns is equal to number of dimensions, ``N``, plus number of values. In our case we have three dimensions and two values.

The work in this repository can be seen as a natural extension of the data cube of [**alien species** in Belgium](https://github.com/trias-project/occ-cube-alien). 
The main difference is that the data cubes created in this repository include native species as well and are totally based on an aggregation at species level (`speciesKey`). If a taxon has taxonomic status `ACCEPTED`  or  `DOUBTFUL`, i.e. it's not a synonym, then GBIF returns not only the occurrences linked directly to it, but also the occurrences linked to its synonyms and its infraspecific taxa.

As example, consider the species [_Reynoutria japonica Houtt._`](https://www.gbif.org/species/2889173). If you search for its occurrrences wordwide you will get all the occurrences from the synonyms and infraspecies too.

taxonKey | scientificName | numberOfOccurrences | taxonRank | taxonomicStatus
--- | --- | --- | --- | ---
5652243 | Fallopia japonica f. colorans (Makino) Yonek. 41 | FORM | SYNONYM
5652241 | Fallopia japonica var. compacta (Hook.fil.) J.P.Bailey | 52 | VARIETY | SYNONYM
2889173 | Reynoutria japonica Houtt. | 39576 | SPECIES | ACCEPTED
4038356 | Reynoutria japonica var. compacta (Hook.fil.) Buchheim | 19 | VARIETY | SYNONYM
4033014 | Tiniaria japonica (Houtt.) Hedberg | 28 | SPECIES | SYNONYM
5652236 | Fallopia japonica var. uzenensis (Honda) K.Yonekura & Hiroyoshi Ohashi | 212 | VARIETY | SYNONYM
5334352 | Polygonum cuspidatum Sieb. & Zucc. | 1570 | SPECIES | SYNONYM
7291566 | Polygonum japonicum (Houttuyn) S.L.Welsh | 2 | SPECIES | SYNONYM
5334357 | Fallopia japonica (Houtt.) Ronse Decraene | 110742 | SPECIES | SYNONYM
7291912 | Reynoutria japonica var. japonica | 2199 | VARIETY | ACCEPTED
6709291 | Reynoutria compacta (Hook.fil.) Nakai | 1 | SPECIES | SYNONYM
7413860 | Reynoutria japonica var. terminalis (Honda) Kitag. | 13 | VARIETY | SYNONYM
8170870 | Reynoutria japonica var. uzenensis Honda | 32 | VARIETY | SYNONYM
7128523 | Fallopia japonica var. japonica | 1560 | VARIETY | DOUBTFUL
5651605 | Polygonum compactum Hook.fil. | 28 | SPECIES | SYNONYM
5334355 | Pleuropterus zuccarinii Small | 1 | SPECIES | SYNONYM
4038371 | Reynoutria henryi Nakai | 14 | SPECIES | SYNONYM
8361333 | Fallopia compacta (Hook.fil.) G.H.Loos & P.Keil | 24 | SPECIES | SYNONYM
7291673 | Polygonum reynoutria (Houtt.) Makino | 3 | SPECIES | SYNONYM

Table based on this [GBIF download](https://doi.org/10.15468/dl.rej1cz).

By aggregating we would loose this information, so we provide aside the cubes, e.g. `be_species_cube.csv` for Belgium, a kind of taxonomic compendiums, e.g. `be_species_info.csv`. They include for each taxa in the cube all the synonyms or infraspecies whose occurrences contribute to the total count. Differently from data cube of alien species, these data cubes are completely built upon the taxonomic relationships of [GBIF Backbone Taxonomy](https://www.gbif.org/dataset/d7dddbf4-2cf0-4f39-9b2a-bb099caae36c). Both data cubes and taxonomic compendiums are saved in `data/processed`.

For example, _Aedes japonicus (Theobald, 1901)_ is an accepted species present in the Belgian cube: based on the information stored in `occ_belgium_taxa.tsv`, its occurrences include occurrences linked to the following taxa:
1. [Aedes japonicus (Theobald, 1901)](https://www.gbif.org/species/1652212)
2. [Ochlerotatus japonicus (Theobald, 1901)](https://www.gbif.org/species/4519733)
3. [Aedes japonicus subsp. japonicus](https://www.gbif.org/species/7346173)

We provide the occurrence cube and correspondent taxonomic compendium of the following European countries:

country | countryCode
--- | ---
Belgium | BE
Italy | IT
Slovenia | SI
Lithuania | LT

## Repo structure

The repository structure is based on [Cookiecutter Data Science](http://drivendata.github.io/cookiecutter-data-science/). Files and directories indicated with `GENERATED` should not be edited manually.

```
├── README.md            : Description of this repository
├── LICENSE              : Repository license
├── occ-cube.Rproj : RStudio project file
├── .gitignore           : Files and directories to be ignored by git
│
├── data
│   ├── raw              : Occurrence data as downloaded from GBIF GENERATED
│   ├── interim          : big sqlite and text files, stored locally  GENERATED
│   └── processed        : occurrence data cubes and related taxa informations GENERATED
│
└── src
    ├── 1_download.Rmd    : Script to trigger a download of occurrences in a country
    ├── 2_create_db.Rmd   : Script to genereate a sqlite file and perform basic filtering
    ├── 3_assign_grid.Rmd : Script to assign cell code to occurrences
    ├── 4_aggregate.Rmd   : Script to aggregate data and make the Belgian data cube
```

## Installation

Clone this repository to your computer and open the RStudio project file,  `occ-processing.Rproj`.

### Generate occurrence data cube

You can generate a national occurrence data cube by running the [R Markdown files](https://rmarkdown.rstudio.com/) in `src` following the order shown here below:

1. `1_download.Rmd`: trigger a GBIF download for a specific country and add it to the list of triggered downloads
2. `2_create_db.Rmd`: create a sqlite database and perform basic data cleaning
3. `3_assign_grid.Rmd`: assign geographic cell code to occurrence data
4. `4_aggregate.Rmd`: aggregate occurrences per taxon, year and cell code, the _national occurrence data cube_

The data cubes are authomatically generated in  folder `/data/processed/`.

Install any required package, first.

## Contributors

[List of contributors](https://github.com/trias-project/occ-cube/contributors)

## License

[MIT License](https://github.com/trias-project/unified-checklist/blob/master/LICENSE) for the code and documentation in this repository.
