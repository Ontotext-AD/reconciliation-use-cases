# Knowledge Base 

Here is the data and queries used to build the knowledge base.

* The full dataset is on Google Drive in [geonames-recon-kb.zip](https://drive.google.com/drive/folders/1UIEBh34MudmsObYSSR2qWPFUWgpHRe3d?usp=sharing) 
* Load data from the zip file in GraphDB
* To work on the KB, download and unzip in this folder but *do not commit in the repository!*
* If rebuilding the KB upload only the zip to the Google Drive without the individual `.ttl` files
* The files used for the model are in the [model](model) folder.

## Building procedure

This is the building procedure for the Knowledge Base
It depends on
* having the full geonames RDF dump loaded (available here [TODO])
* having access to Wikidata Truthy RDF endpoint (currently at <https://reconcile.ontotext.com/graphdb/>)

From full geonames RDF dump, select only relevant feature codes

### Geonames Administrative entities

* constructs the entire administrative hierarchy for the world.
* run on geonames repo
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
* run on geonames repo
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

### Altrenative Lables 

* query constructs alternative labels from wikidata for all objets with a Geonames id
* resulting RDF is with geonemaes URI and ready to integrate
* run on WDtruthy endpoint.
* slow and very large output - available on gdrive [here](https://drive.google.com/file/d/1kMYWoc5ZpXFx-uVc3j-__iGRXCZ054Nl/view?usp=sharing)
* to be loaded in separate repository and integrated through inter repository federation (see below)

This is an example of the result

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



