## Exporting odML to RDF


## Mapping odML to RDF terms

odML ... namespace should be a URL pointing to where the RDF definition of odML can be found.

looked up all the different resources for odML that are still out there and came up with the following existing
classes and properties - I am sure many of them can be removed, but then they should also be removed from the
resources to reduce confusion.

NO idea how to deal with different terminologies, if RDF graphs of different odML documents would be merged.

ALL of the properties can be fine grained into subproperties to give more semantic meaning to the connection
between nodes.

e.g.
    section hasSection section
 could become
    section hasExperimenter section
    
 here hasExperimenter is a subclass of hasSection ... a search for hasExperimenter would return only sections connected
 via this specific property, a search for hasSection would return all sections connected by `hasSection` and `hasExperimenter`.


### Hub
I think this is something we would need, to merge different odML documents into an existing graph.
Because we start from a hierarchical document it is ok to connect them via a central hub node.

    odml                                    RDF                             RDF alternative
    ---------------------------------------------------------------------------------------
    -                                       odml:Hub                        RDF type odml:Hub
    -                                       odml:hasDocument

### Root section

    odml                                    RDF                             RDF alternative
    ---------------------------------------------------------------------------------------
    odml                                    odml:Document                   RDF type odml:Document
    
    id                                      use uuid as instance name
    author                                  odml:author
    date                                    odml:date
    version (odml)                          odml:version
    version (document)                      odml:doc-version
    repository (deprecated?)                odml:hasExternalSection
    Sections                                odml:hasSection



### Section

    odml                                    RDF                             RDF alternative
    ---------------------------------------------------------------------------------------
    Section                                 odml:Section                    RDF type odml:Section
    
    id (to be implemented)                  RDF Section instance name
    name                                    odml:name                       RDFS:label
    type                                    odml:type
    definition                              odml:description                RDFS:comment
    repository (deprecated?)                odml:hasExternalSection         ...
    mapping (deprecated)                    odml:mapsTo                     RDF subProperty odml:hasSection
    include (deprecated?)                   odml:hasExternalSection
    link (deprecated?)                      odml:hasLink                    RDF subProperty odml:hasSection
            ... link to an external section! requires the IRI of this section.
    reference (deprecated?)                 odml:hasReference               RDF subProperty odml:hasSection
    Sections                                odml:hasSection
            ... should this be specified to hasSubSection and already be a subclass of hasSection?
                     there are already different connectors between sections/property 
                     and section which could not be distinguished otherwise e.g. mapsToSection
    Properties                              odml:hasProperty


### Property

    odml                                    RDF                             RDF alternative
    ---------------------------------------------------------------------------------------
    Property                                odml:Property                   RDF type odml:Property
    
    id (to be implemented)                  RDF Property instance name
    name                                    odml:name                       RDFS:label
    definition                              odml:definition                 RDFS:comment
    mapping (deprecated)                    odml:mapsTo                     RDF subProperty odml:hasSection
    dependency                              odml:dependency                 RDF subProperty odml:hasProperty
    dependencyValue                         -
            ... is not required, since access to value is there via property link!
    Values                                  odml:hasValue
    Synonym (deprecated?)



### Value

    odml                                    RDF                             RDF alternative
    ---------------------------------------------------------------------------------------
    Value                                   odml:Value                      RDF type odml:Value
    
    id (?)                                  generate uuid as instance name or use blank node
    unit                                    odml:unit                       si:unit (needs to be another node)
    dtype                                   odml:dtype
            ... problem to link, if its not a basic type, but one defined in a terminology; 
                    has to be different from Section/type
    definition (not in NIX)                 odml:definition                 RDFS:comment 
    uncertainty                             odml:uncertainty
    filename (deprecated)
    encoder (deprecated)
    checksum (deprecated)
    reference (deprecated)
    data                                    odml:data
     ... the actual value
    defaultFilename (deprecated?)


## RDF libraries

## RDF libraries
- [RDFlib](https://github.com/RDFLib/rdflib) 
    - Actively developed, many RDF project seem to converge on RDFlib
    - Comes with decent documentation
    - py2 and py3.
- [librdf](http://librdf.org/): C library with bindings to various language bindings including Python.


## odML Resources that are out there:
A lot of the resources that can still be found have very conflicting content. This makes it quite difficult to
properly discuss even within teams, if everyone has different "base of knowledge" about odML.

### Documents, Documentation, Tutorials
- [odML Roadmap (google doc)](https://docs.google.com/document/d/16usqrMgk6muqgkSmQ0qonXsGrC0S9nhyHdzgWAJyoFg/edit#heading=h.ycmlvplf2l2s)
- odML metadata future document
- [python-odml documentation + tutorial](https://g-node.github.io/python-odml/)
- [odML terminologies](http://portal.g-node.org/odml/terminologies/v1.0/terminologies.xml)
- [odML Schema](http://www.g-node.org/projects/odml/odMLSchema.png/view)

### Papers
- [A bottom-up approach to data annotation in neurophysiology](http://journal.frontiersin.org/article/10.3389/fninf.2011.00016/full)
- [Handling Metadata in a Neurophysiology Laboratory](http://journal.frontiersin.org/article/10.3389/fninf.2016.00026/full)

### Projects using odML
- [python-odml](https://github.com/G-Node/python-odml)
- [odml-ui](https://github.com/G-Node/odml-ui)
- [odml-terminologies](https://github.com/G-Node/odml-terminologies)
- [odML Tables](https://github.com/INM-6/python-odmltables)


## Unsorted notes:

Difference between python-odml and NIX odML? which fields are actually different?
- File: Version and Author and Date for odML root is missing in Nix implementation
- NIX does not support terminologies
- in NIX: section - reference & include is missing-
... figure out the current state of fields and functions of core odML and odML tables.

Section:
- what is mapping, how does it work?
- check out all odML Terms and how they could be mapped to RDF terms.

- NIX values and odML Values are differently structured!
- NIX values don't have a `dtype` explicitly, but implicitly via the actual data ... does not support odML.Modules
custom data types ... problem for import.
- NIX does not seem to support the way odML handles values.

Check out which RDFs:xxx would fir for "subset" and check, how this would be stored in "normal" odML ... would this break existing files? 

odML namespace

Maybe a supported table for different odML versions + NIX + odML tables would be useful!

RDF ... how to do the reverse  o p s thing?

Conversion odML -> RDF seems to be easy.
Conversion RDF -> odML ... how to root the graph? Here a single RootSection node would help!

How to structure convergence of projects?  see what needs to be done on which side (NIX/odML)

NIX lookout:
- do we actually want to import odML into NIX files or just export?
    -> then only support one format and do conversions in odML core.


Regarding the conversion and how the process should be started:

We have problems on multiple levels:
- we are in the process of refactoring odML.
- we have no proper overview of which odML versions are out there and no good understanding of how they are currently used.
    - don't have even proper example odML files from any of the users!
- we have no one who actually is proficient in RDF, knows how the class system works and how to best design it
    - to specify: to which degree should we use RDF terms that are already there, or should we use just our own defined RDF terms.
- we have no good idea, how the people from the CRCNS actually work with RDF and which format would be good for them
    - how would they like to search the data.
    - which RDF terminologies are they using, etc.
    - if they use validators for RDF graphs

