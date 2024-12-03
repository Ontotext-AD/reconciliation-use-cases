# Candidate KG RDF datasets

This folder contains the source data from the banks use case 
in RDF, following the [model](../kb/model) of the target graph
and without entity matching. 

* [candiate-kb-all-random-ids.ttl](candiate-kb-all-random-ids.ttl) is a basic transformation 
where each entity from each column is assigned a random ID.
* [candiate-kb-city-state-dedup.ttl](candiate-kb-city-state-dedup.ttl) is a slightly more sophisticated transformation, 
where entities form teh `CITY` and `STATE` columns are assigned unique ids for the state and city/state pairs. 

```ttl
<urn:uuid:719ef457-cb3b-4da9-a233-0171b47f70cc> a s:Place;
  s:name "1st National Bank";
  gn:featureCode gn:P.PPL;
  ex:charter "8709";
  s:additionalType wd:Q18761864;
  s:containedInPlace <http://example.com/base/32a0bdf614243a5c01da52a736ab596fc0a1731f580317f64d3793b3d26aa376> .

<http://example.com/base/32a0bdf614243a5c01da52a736ab596fc0a1731f580317f64d3793b3d26aa376>
  a gn:Feature;
  gn:name "Lebanon";
  skos:altLabel "Lebanon";
  gn:parentADM1 <http://example.com/base/bba36f226cdd988cfb4953ec680b214e315b120a0de4ba0fb12363487f7e1ad6> .

<http://example.com/base/bba36f226cdd988cfb4953ec680b214e315b120a0de4ba0fb12363487f7e1ad6>
  a gn:Feature;
  gn:featureCode gn:A.ADM1;
  gn:countryCode "US";
  skos:altLabel "OH" .
```


