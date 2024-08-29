# Knowledge Base 

Here is the data and queries used to build the knowledge base in the form of `.ttl` files 
The files used for the model diagrams are in the [model](model) folder.

## Building procedure

This is the building procedure for the Knowledge Base
It depends on
* having the full geonames RDF dump loaded in a repository named `geonames`. The full data is available on Google Drive [here](https://drive.google.com/file/d/1EGteG-kQYzcKVIBd-82BrTANQDzMtZxm/view?usp=drive_link)
* having read access to Wikidata Truthy RDF endpoint (currently at <https://reconcile.ontotext.com/graphdb/>)


### Geonames Administrative entities
From full geonames RDF dump  select only relevant feature classes `A` and `P` for a selection of countries.

* constructs the entire administrative hierarchy for the world.
* run on `geonames` repository
* file: [geonames-a.ttl](geonames-a.ttl)

```sparql
PREFIX gn: <http://www.geonames.org/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
construct {
    ?s ?p ?o .
}
where {
    ?s gn:featureClass <https://www.geonames.org/ontology#A> ;
    ?p ?o .
    filter(?p not in(	
            gn:alternateName,
            rdfs:seeAlso,
            gn:locationMap,
            rdfs:isDefinedBy,
            gn:childrenFeatures
    )
    )
}
```

### Geonames Populated places

* query extracts all the populated places in europe and North America
* run on `geonames` repository
* file: [geonames-p.ttl](geonames-p.ttl)

```sparql
PREFIX gn: <http://www.geonames.org/ontology#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
construct {
?s ?p ?o .
}
where {
    ?s gn:featureClass <https://www.geonames.org/ontology#P> ;
    gn:countryCode ?country ;
    ?p ?o .
    filter(?p not in(
        gn:alternateName,
        rdfs:seeAlso,
        gn:locationMap,
        rdfs:isDefinedBy,
        gn:childrenFeatures
)
    )
    filter(?country in("US","CA","AL", "AD", "AT", "BY", "BE", "BA", "BG", "HR", "CZ", "DK", "EE", "FI", "FR", "DE", "GR", "HU", "IS", "IE", "IT", "LV", "LI", "LT", "LU", "MT", "MC", "ME", "NL", "MK", "NO", "PL", "PT", "RO", "SM", "RS", "SK", "SI", "ES", "SE", "CH", "UA", "GB", "VA"))
}
```
###  Buildings From wikidata 

* constructs data about buildings from Wikidata and attaches the to their locations expressed as geonames IDs
* run on WDtruthy
* file [buildings.ttl](buildings.ttl)

```sparql
PREFIX s: <http://schema.org/>
PREFIX onto: <http://www.ontotext.com/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
construct {
?S a s:Place ;
s:additionalType ?type ;
s:name ?label ;
s:description ?desc ;
skos:altLabel ?altLabel ;
s:containedInPlace ?gn_uri ;
.
?type a skos:Concept ;
s:name ?type_label ;
s:description ?type_desc ;
.
}
from onto:explicit
where {
    ?s  wdt:P131|wdt:P276 ?loc ;
    wdt:P31/wdt:P279 wd:Q41176 ;
    wdt:P31 ?type ;
    rdfs:label ?label ;
    s:description ?desc ;
    .
    filter(lang(?label)="en")
    filter(lang(?desc)="en")
    ?type rdfs:label ?type_label ;
    s:description ?type_desc .
    filter(lang(?type_label)="en")
    filter(lang(?type_desc)="en")
    ?loc wdt:P1566 ?gn_id
    optional {?s skos:altLabel ?altLabel.  filter(lang(?altLabel)="en")
    }
    BIND(uri(concat("https://example.org/",sha1(str(?s)))) AS ?S )
    bind(uri(concat("https://sws.geonames.org/",str(?gn_id),"/")) as ?gn_uri)
} 
```

### Alternative Labels 

* query constructs alternative labels from wikidata for all objets with a Geonames id with geonames URI and ready to integrate
* run on `WDtruthy` endpoint.
* slow and very large output - available on gdrive [here](https://drive.google.com/file/d/1CfFa11Na5fo0rVZKB2hWh7JuSKVELj5q/view?usp=sharing)
* to be loaded in separate repository (called here `wdlabels`) and integrated through inter repository federation (see below)

This is a `.ttl` example of the result

```turtle
<https://sws.geonames.org/6252001/> skos:altLabel "United States of America"@en, "United States"@en, "America"@en, "the United States of America"@en,
        "USA"@en, "US"@en, "Murica"@en, "the States"@en, "US of America"@en, "U.S.A."@en,
        "the US"@en, "the U.S."@en, "the US of A"@en, "U.S. of America"@en, "the US of America"@en,
        "the USA"@en, "U.S."@en, "the U.S.A."@en, "the U.S. of A"@en, "US of A"@en, "the U.S. of America"@en,
        "U. S."@en, "U. S. A."@en, "the United States"@en, "Merica"@en, "America (United States)"@en .
```

```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX s: <http://schema.org/>

construct {
   ?gn_uri 
        s:description ?feat_desc ;
    	s:additionalType ?type ;
        skos:altLabel ?feat_label , ?alt_label , ?native_label , ?nickname , ?official_name , ?shortname .
   
   ?type a skos:Concept ;
      s:name ?type_label ;
      s:description ?type_desc ;
   .     
}
where {
    ?feat wdt:P1566 ?gn_id ;
        wdt:P31 ?type ;
        rdfs:label ?feat_label ;
        s:description ?feat_desc ;
    .
    filter(lang(?feat_label)="en")
    filter(lang(?feat_desc)="en")
    
    optional {?feat skos:altLabel ?alt_label ; filter(lang(?alt_label)="en")}
    
    ?type rdfs:label ?type_label ;
    	  s:description ?type_desc .
    filter(lang(?type_label)="en")
    filter(lang(?type_desc)="en")
    
    optional{?feat wdt:P1705 ?native_label. filter(lang(?native_label)="en")}
    optional{?feat wdt:P1449 ?nickname. filter(lang(?nickname)="en")}
    optional{?feat wdt:P1448 ?official_name. filter(lang(?official_name)="en")}
    optional{?feat wdt:P1813 ?shortname. filter(lang(?shortname)="en")}
    
    bind(uri(concat("https://sws.geonames.org/",str(?gn_id),"/")) as ?gn_uri)
} 
```

This query constructs  [geonames-alt-labels.ttl](geonames-alt-labels.ttl)
using inter repository federation, provided the result from the previous query ([wd-more-labels.ttl](https://drive.google.com/file/d/1CfFa11Na5fo0rVZKB2hWh7JuSKVELj5q/view?usp=drive_link)
is loaded in a repo called `wdlabels`
```sparql
PREFIX s: <http://schema.org/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX gn: <http://www.geonames.org/ontology#>
construct {
    ?s skos:altLabel ?altlabel ;
} where {
    ?s a gn:Feature .
    service <repository:wdlabels> {
        ?s skos:altLabel ?altlabel 
    }
}
```

### Banks 

In order to support the reconciliation use case, we have converted part of the source data to RDF and added it to the Knowledge Graph.

This is th OR mapping query used for the conversion


```sparql
BASE <http://example.com/base/>
PREFIX s: <http://schema.org/>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX ex: <https://example.org/>
CONSTRUCT {
	?ID a s:Place ;
    	s:name ?NAME ;
    	s:containedInPlace ?gn_uri ;
    	s:additionalType wd:Q18761864 ;
    	ex:charter ?c_charter_in_data ;
    .
} 
WHERE {
  BIND( ?c_charter_in_data AS ?charter_in_data ) .
  BIND( uri(concat("https://example.org/",?c_ID)) AS ?ID ) .
  BIND( ?c_NAME AS ?NAME ) 
  bind(uri(concat("https://sws.geonames.org/",str(?c_gn_id),"/")) as ?gn_uri)
 }
```
