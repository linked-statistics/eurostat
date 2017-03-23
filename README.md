# Eurostat dataset

About 
-----

This project is about publishing EuroStat as Linked Data on the Web. 


Design
------

1) Base URI would be http://eurostat.linked-statistics.org.

2) VoID discovery mechanism would be delivered through http://eurostat.linked-statistics.org/.well-known/void. Upon discovery, the system will serve the VoID description which would be like :

```turtle
@prefix meta: <http://eurostat.linked-statistics.org/meta#> . 
@prefix dss: <http://eurostat.linked-statistics.org/dss/> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix void: <http://rdfs.org/ns/void#> .

meta:Eurostat a void:Dataset;
	dcterms:title "EuroStat";
	void:subset
	dss:ds_1,
	dss:ds_2
	.
```

3) Upon de-referencing any dataset, we serve the DataSet Summary (DSS):

```turtle
@prefix data: <http://eurostat.linked-statistics.org/data/> .
@prefix dss: <http://eurostat.linked-statistics.org/dss/> .
@prefix dsd: <http://eurostat.linked-statistics.org/dsd#> .
@prefix qb: <http://purl.org/linked-data/cube#> .
@prefix void: <http://rdfs.org/ns/void#> .

dss:ds_1 a qb:DataSet, void:Dataset;
	qb:DataStructureDefinition dsd:dsd_1;
	void:dataDump data:ds_1.ttl
	.
```

Remarks
-------

1) Use N-Quads format to generate dataset triples.

2) Reuse code from [linked-eurostat](http://code.google.com/p/linked-eurostat/) for Dataset RDFication.

3) Use [our own code](https://github.com/linked-statistics/eurostat/tree/master/parser/src) for DSD RDFication.


Batch Scripts
-------------

Detailed description on each script can be found at this [page](https://github.com/linked-statistics/eurostat/wiki/Scripts)


Steps to RDFize EuroStat data
-----------------------------

The RDFication process can be found at this [page](https://github.com/linked-statistics/eurostat/wiki/Usage)


How to convert a single dataset to RDF
--------------------------------------

* The best way to test the RDFication process is to use `Main.sh` script. You are required to download the `*.zip` file(s) in a directory before running the script. It can be achieved by running `EurostatMirror.sh.`
* Change the directory path variables in the `Main.sh` to your desired directory paths. Make sure that the directories *exists* before running the script.
* How to run : `sh Main.sh -i ~/sdmx-code.ttl -l ~/logs/`


Example Query
-------------

Here is an examplary quert that joins two SDMX datasets:

```sparql
PREFIX rdfs: http://www.w3.org/2000/01/rdf-schema#
PREFIX qb: http://purl.org/linked-data/cube#
PREFIX e: http://ontologycentral.com/2009/01/eurostat/ns#
PREFIX sdmx-measure: http://purl.org/linked-data/sdmx/2009/measure#
PREFIX skos: http://www.w3.org/2004/02/skos/core#
PREFIX g: http://eurostat.linked-statistics.org/ontologies/geographic.rdf#
PREFIX dataset: http://eurostat.linked-statistics.org/data/

SELECT ?nuts2
SUM(xsd:decimal(?pop)) AS ?population
?wateruse
xsd:decimal(?wateruse)*1000000/SUM(xsd:decimal(?pop)) AS ?percapita 
WHERE { 
	?observation qb:dataset dataset:demo_r_pjanaggr3 ;
	e:time http://eurostat.linked-statistics.org/dic/time#2007;
	e:age http://eurostat.linked-statistics.org/dic/age#TOTAL;
	e:sex http://eurostat.linked-statistics.org/dic/sex#F;
	e:geo ?ugeo;
	sdmx-measure:obsValue ?pop.
	?ugeo g:hasParentRegion ?parent.
	?parent rdfs:label ?nuts2.
	?wuregion qb:dataset dataset:env_n2_wu ;
	e:geo ?parent;
	e:cons http://eurostat.linked-statistics.org/dic/cons#W18_2_7_2;
	e:time http://eurostat.linked-statistics.org/dic/time#2007;
	sdmx-measure:obsValue ?wateruse.
} 
GROUP BY ?nuts2 ?wateruse ORDER BY DESC(?percapita)
```

The query above uses dataset `demo_r_pjanaggr3`, which contains `Population by sex and age groups on 1 January - NUTS level 3 regions`. We need populations for NUTS level 2 and we therefore aggregate the dataset by using the NUTS vocabulary to find the parent regions.

We only want data for 2007, and both sexes. We then join the data with the `env_n2_wu` dataset, which contains `Water use (NUTS2) - mio m3`. We can then find the regions with the most domestic water (code W18_2_7_2) use per million inhabitants.

Another example query
---------------------

Below is a SPARQL query that combines 24 Eurostat datasets. It is a combined query on all national statistics for Albacore. The idea is to see if a species needs further protection in the form of fishing quotas etc. A similar query is used [here](http://eunis.eea.europa.eu/species/124054/linkeddata).

```sparql
PREFIX qb: <http://purl.org/linked-data/cube#>
PREFIX e: <http://ontologycentral.com/2009/01/eurostat/ns#>
PREFIX sdmx-measure: <http://purl.org/linked-data/sdmx/2009/measure#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX g: <http://eurostat.linked-statistics.org/ontologies/geographic.rdf#>
PREFIX dataset: <http://eurostat.linked-statistics.org/data/>
PREFIX eunis: <http://eunis.eea.europa.eu/rdf/species-schema.rdf#>

SELECT ?country ?year ?presentation ?landed ?unit
FROM <http://eurostat.linked-statistics.org/data/fish_ld_be.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_bg.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_cy.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_de.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_dk.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_ee.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_es.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_fi.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_fr.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_gr.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_ie.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_is.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_it.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_lt.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_lv.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_mt.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_nl.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_no.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_pl.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_pt.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_ro.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_se.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_si.rdf>
FROM <http://eurostat.linked-statistics.org/data/fish_ld_uk.rdf>
FROM <http://semantic.eea.europa.eu/home/roug/eurostatdictionaries.rdf>
WHERE {
  ?obsUri e:species <http://eurostat.linked-statistics.org/dic/species#ALB>;
	  e:pres <http://eurostat.linked-statistics.org/dic/pres#P00>, ?upresentation;
	  e:dest <http://eurostat.linked-statistics.org/dic/dest#D0>;
	  e:natvessr <http://eurostat.linked-statistics.org/dic/natvessr#TOTAL>;
	  e:unit <http://eurostat.linked-statistics.org/dic/unit#TPW>, ?uunit;
	  e:geo ?ucountry;
	  e:time ?uyear;
	  sdmx-measure:obsValue ?landed.
  ?ucountry skos:prefLabel ?country.
  ?uunit skos:prefLabel ?unit.
  ?uyear skos:prefLabel ?year.
  ?upresentation skos:prefLabel ?presentation.
} 
ORDER BY ?country ?year ?presentation
```


URIs for Eurostat identities
----------------------------

* The base URI for Eurostat is http://eurostat.linked-statistics.org.

* The Data Structure Definition (DSD) can be found under `http://eurostat.linked-statistics.org/dsd/`

	For example: http://eurostat.linked-statistics.org/dsd/bsbu_q.ttl

* The SDMX data sets can be found under `http://eurostat.linked-statistics.org/data/`

	For example: http://eurostat.linked-statistics.org/data/bsbu_q.rdf

* The dictionaries can be found under `http://eurostat.linked-statistics.org/dic/`

	For example: http://eurostat.linked-statistics.org/dic/geo.rdf


NameSpaces for Eurostat
-----------------------

* Namespace for SDMX datasets : `@prefix data:    <http://eurostat.linked-statistics.org/data/> .` 

* Namespace for Data Structure Definition (DSD) : `@prefix dsd:     <http://eurostat.linked-statistics.org/dsd/> .` 

* Namespace for dictionaries : `@prefix cl:      <http://eurostat.linked-statistics.org/dic/> .` 

* Namespace for dataset summaries is : `@prefix dss:     <http://eurostat.linked-statistics.org/dss#> .` 

* Namespace for the concepts defined in DSDs : `@prefix concept:  <http://eurostat.linked-statistics.org/concept#> .` 

* Namespace for the properties defined in DSDs : `@prefix property:  <http://eurostat.linked-statistics.org/property#> .`

* Namespace for titles of the datasets is : `@prefix title:   <http://eurostat.linked-statistics.org/title#> .`


License
-------

The software provided in this repository is Open Source.

To Do
-----

* Document the percentage of datasets that change per week, on average
* Explore how to generate a per-country subset of the data
* Load a .ie subset into data-gov.ie dataspace
* Interlinking
  * Regions: DBpedia, Geonames, LinkedGeoData
  * National regions: Data-gov.ie, GeoLinkedData.es, Ordnance Survey
  * Indicators: US Census
  * Topics/Subjects: DBpedia

Pitfalls
--------

There are few pitfalls in our current approach of converting eurostat datasets into RDF:

* Some datasets have time period associated to an observation which is not a date but instead represented as `LTAA (long term anual average)`, see [dataset] (http://epp.eurostat.ec.europa.eu/NavTree_prod/everybody/BulkDownloadListing?sort=1&file=data%2Fenv_watq1a.tsv.gz). Currently we dont know how to deal with `LTAA` in RDF because we dont have any extra information provided in the SDMX and DSD of that particular dataset. To deal with this case, our current solution provides a turtle file which contains the definition of `LTAA` and the time period for any particular observation value will be represented in RDF using that URI. For example, `<>  sdmx:timePeriod  <http://eurostat.linked-statistics.org/misc>`.
* Some datasets have time period associated to an obervation in the following way : `1989_1993`, see [dataset] (http://epp.eurostat.ec.europa.eu/NavTree_prod/everybody/BulkDownloadListing?sort=1&file=data%2Furb_ikey.tsv.gz). In the `.tsv` files, we dont have `TIME_FORMAT` value which actually tells if the observation value is for 1 year or X number of years. So for now, we are transforming values like `1989_1993` into `1989-01-01`.
* The `FREQ` value is missing in the `.tsv` files of all datasets, which is necessary to represent the observation values. In order to represent `FREQ` value for each observation value we extract the first `FREQ` value from the `sdmx` file of each dataset with the assumption that the `FREQ` value will remains same for all the observation values in that particular dataset.
* Certain observation values are represented as `-`, e.g. see [dataset] (http://epp.eurostat.ec.europa.eu/NavTree_prod/everybody/BulkDownloadListing?sort=1&file=data%2Fearn_mw_cur.tsv.gz). We assume these values to be `0` and replace values (i.e. `-`) with `0` in RDF. 


Further resources
-----------------

* Eurostat has [shapefiles for NUTS regions](http://epp.eurostat.ec.europa.eu/portal/page/portal/gisco_Geographical_information_maps/popups/references/administrative_units_statistical_units_1)
* [NUTS-RDF](http://nuts.geovocab.org/) is a browser for NUTS regions that has a nice map view attached. We have links to NUTS-RDF.
* [Linked NUTS](http://nuts.psi.enakting.org/) is an excellent conversion of the NUTS codes to RDF, including the various versions of NUTS that existed over time.
