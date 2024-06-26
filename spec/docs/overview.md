## Link to FnO: Overview

Instead of integrating a specific set of functions in [[RML]],
we combine [[RML]] with declarative function descriptions in [[FnO]].

Within [[FnO]], [=Function=]s and [=Execution=]s are described.
Within FNML, we refer to [=Execution=]s that link to specific [=Function=]s.

[=Triples Map=]s generate output triples from input data.
We use an intermediate [=Function-valued Expression Map=] to use a specific output (via a [=Return Map=]) of a [=Function Execution=].
That [=Function Execution=] specifies which [[FnO]] function to use (via a [=Function Map=])
and uses [=Input=]s to link input data (via regular [[RML]] [=Term Map=]s)
to [=Parameter=]s (via [=Parameter Map=]s).

<p class="note" data-format="markdown">
If an execution returns multiple returning outputs (eg, a result and a status code),
by referring to the same execution,
you can use both outputs in different locations of the same mapping.
If you leave out the intermediate [=Function-valued Expression Map=], you don't allow for reuse,
which means that you cannot specify the difference between
'using 2 outputs from one execution' vs 'use a different output from 2 different executions'.
</p>

### Function Example

We use [Example 1](#example-rml-mapping-without-data-transformations),
where we want to perform an uppercase operation to a set of fields.

The FnO description of the function [toUppercase](https://github.com/OpenRefine/OpenRefine/wiki/GREL-String-Functions#touppercasestring-s) is as follows:

```turtle "example": "toUppercase FnO description"
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix fno:     <https://w3id.org/function/ontology#> .
@prefix grel:    <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix rdfs:    <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .

grel:toUpperCase
    a                   fno:Function ;
    fno:name            "to Uppercase" ;
    rdfs:label          "to Uppercase" ;
    dcterms:description "Returns the input with all letters in upper case." ;
    fno:solves          grel:prob_ucase ;
    fno:expects         ( grel:valueParam ) ;
    fno:returns         ( grel:stringOut ) .

grel:valueParam
    a             fno:Parameter ;
    fno:name      "input value" ;
    rdfs:label    "input value" ;
    fno:predicate grel:valueParameter ;
    fno:type      xsd:string ;
    fno:required  "true"^^xsd:boolean .

grel:stringOut
    a             fno:Output ;
    fno:name      "output string" ;
    rdfs:label    "output string" ;
    fno:predicate grel:stringOutput ;
    fno:type      xsd:string .

```

The execution of such a function converts a string to its uppercase sibling,
so `test` becomes `TEST` and `This is an input STRING.` becomes `THIS IS AN INPUT STRING.`.
The latter would be described as follows using an FnO Execution description:

```turtle "example": "toUppercase FnO execution description"
@prefix fno:     <https://w3id.org/function/ontology#> .
@prefix grel:    <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix :        <http://example.com/> .

:exe a fno:Execution ;
    fno:executes grel:toUppercase ;
    grel:valueParameter "This is an input STRING." ;
    grel:stringOutput "THIS IS AN INPUT STRING." .
```

## FNML Example - shortcuts

To connect this function with the RML mapping document, we make use of FNML, see below for an example, which makes maximal use of shortcuts.

<figure>
<pre class="mermaid nohighlight override" style="width: 100%">
graph LR
    T3M([triples map])
    T3M-->|predicateObjectMap| POM([predicate-object map])
    T3M-->|subjectMap| FM
    POM -->|predicate| FM
    POM -->|objectMap| FM
    %% FM([Function term map]):::fnml
    FM([function-valued expression map]):::fnml
    %% FM -->|function value| Ex([triples map])
    FM -->|execution| Ex([Function Execution]):::fnml
    %% FM -->|execution| Ex([Function triples map]):::fnml
    %% FM -->|function value| Ex([Function triples map]):::fnml
    FM -->|return| J(grel:stringOut):::fno
    Ex -->|function| Fn(grel:toUpperCase):::fno
    %% Ex -->|predicateObjectMap| ExPOM([predicate-object map])
    %% ExPOM -->|predicate| ExP(fno:executes):::fno
    %% ExPOM -->|object map| ExO([Object map])
    %% ExO -->|constant| ExO1(grel:toUpperCase):::fno
    Ex -->|input| ParamPOM([Input]):::fnml
    %% Ex -->|predicateObjectMap| ParamPOM([predicate-object map])
    ParamPOM -->|parameter| P2(grel:valueParam):::fno
    ParamPOM -->|valueMap| O1(term map)
    %% ParamPOM -->|predicate| P2(grel:str_value):::fno
    %% ParamPOM -->|object map| O1([Object map])
    O1 --> |template| Ot1("{lastname}"):::kfno
    %% ParamPOM -->|objectMap| ROM([ReferencingObjectMap])
    %% ROM -->|parentTriplesMap| PT3M([TriplesMap]):::ls2
    %% ROM -->|joinCondition| JC([JoinCondition])
    %% ROM -->|joinResultTerm| JTM("{parentsource_value}"):::ls2
    %% JC -->|childTerm| ChTM([TermMap]):::ls2
    %% JC -->|parentTerm| PaTM([TermMap]):::ls2
    classDef fnml fill:#8F9
    classDef fno fill:#F89
    classDef rml fill:#89F
    classDef ls2 fill:#09F
</pre>
<figcaption>Visual overview of connections FNML</figcaption>
</figure>

```turtle "example": "using toUppercase in an RML mapping"
@prefix dbo: <http://dbpedia.org/ontology/> .
@prefix grel: <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix rr: <http://www.w3.org/ns/r2rml#> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;
    rr:subjectMap <#SubjectMap> ;
    rr:predicateObjectMap <#NameMapping> .

<#NameMapping>
    rr:predicate dbo:title ;
    rr:objectMap [                             # A function-valued expression map
        rml:functionExecution <#Execution> ;   # Link to a Function Execution
        rml:return grel:stringOut              # Specify which return of the referenced function to use, if omitted, the first specified return is used
    ] .

<#Execution> a rml:FunctionExecution ;         # A new class
    rml:function grel:toUppercase ;            # Specify which FnO function
    rml:input [                                # Specify the inputs
        a rml:Input ;                          # A new class
        rml:parameter grel:valueParam ;        # Specify this specific parameter
        rml:inputValueMap [                    # Link to the term map that creates the input value
            rml:reference "name"               # Specify the reference within the data source
        ]
    ] .
```

The `name`-value is not referenced directly,
instead, its value is used as `grel:valueParam`-parameter
for the `grel:toUppercase`-function.
After execution, the `grel:stringOut` result of that function is returned to generate the object
within the `<#NameMapping>`.
We make use of an intermediate [=Function-valued Expression Map=] so that we can reuse the returning output of an execution in multiple TermMaps.

## FNML Example - no shortcuts

The same example, but written without shortcuts, is as follows:

```turtle "example": "using toUppercase in an RML mapping without shortcuts"
@prefix dbo: <http://dbpedia.org/ontology/> .
@prefix grel: <http://users.ugent.be/~bjdmeest/function/grel.ttl#> .
@prefix rml: <http://w3id.org/rml/> .
@prefix rr: <http://www.w3.org/ns/r2rml#> .

<#Person_Mapping>
    rml:logicalSource <#LogicalSource> ;       # Specify the data source
    rr:subjectMap <#SubjectMap> ;              # Specify the subject
    rr:predicateObjectMap <#NameMapping> .     # Specify the predicate-object-map

<#NameMapping>
    rr:predicate dbo:title ;                   # Specify the predicate
    rr:objectMap [                             # Specify the object-map: a function-valued expression map
        rml:functionExecution <#Execution> ;   # Link to a Function Execution
        rml:returnMap [
            a rml:ReturnMap ;
            rr:constant grel:stringOut         # Specify which return of the referenced function to use
        ]
    ] .

<#Execution> a rml:FunctionExecution ;         # A new class
    rml:functionMap [
        a rml:FunctionMap ;
        rr:constant grel:toUppercase           # Specify which FnO function
    ] ;
    rml:input                                  # Specify the inputs
        [
            a rml:Input ;                      # A new class
            rml:parameterMap [
                a rml:ParameterMap ;
                rr:constant grel:valueParam ;  # Specify this specific parameter
            ] ;
            rml:inputValueMap [                    # Link to the term map that creates the input value
                a rr:TermMap ;
                rml:reference "name"           # Specify the reference within the data source
            ]
        ] .
```
