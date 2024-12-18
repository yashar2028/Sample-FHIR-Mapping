# Sample-FHIR-Mapping
This is a very simple example of mapping from an invalid Care-Unit resource to FHIR Location resource using the FHIR Mapping Language (FML). The exact rules are also implemented in python ([notebook](https://github.com/yashar2028/Sample-FHIR-Mapping/conversion.ipynb)). 

[See the map file](https://github.com/yashar2028/Sample-FHIR-Mapping/tree/main/input/maps)

## Methods for conversion of mapping files:
1- Using the validator tool: After downloading the validator tool (validator_cli.jar), invoke it as follows
```
java -jar <path to your validator tool> \  
  -ig <path to your mapping file> \
  -compile <url for defined in your mapping file as StructureMap> \
  -version 4.0.1 \
  -output <path to see the output .json>
```
Check the [confluence page](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Mapping+Language) for more information.

2- Invoking the `$convert` operation on StructureMap with body being your mapfile content:
```http
POST /StructureMap/$convert
Content-Type: application/plaintext 

Body: mapping file rules
```
## Performing the mapping
There are different ways to get the mapping done. All is needed is the mapping file (or StructureMap), source to be mapped, and source/target StructureDefinition:

1- Invoking the `$transform` operation on your StructureMap (its id must be specified) with body being the source content that is going to be mapped:
```http
POST /StructureMap/{id}/$transform
Content-Type: application/json

Body: source json
```
Along with API clients such as Insomnia or Postman, python fhir client and server libraries can perform the same thing. Make sure that your FHIR server support `$convert` and `$transform` operation on StructureMap

2- Using the validator tool and invoking the `$transform` operation. Confluence page provides more information on this (see link above).

3- Using python and [fhir.resources](https://pypi.org/project/fhir.resources) module to directly perform the mapping. This is of course the fastest and most straightforward approach which is also used here.

## Explanation
You can find the generated StructureMap resource [here](https://github.com/yashar2028/Sample-FHIR-Mapping/blob/main/output/map.json). Note that the validator tool generated this StructureMap in FHIR R4 definition (so validate against R4 validator). However, the Location resource used inside this StructureMap is FHIR R5 definition. The validator may raise issue regarding this incompatibility (e.g. Location.contact in R5 is location.telecome in R4 (with different type)), so the best practise was to use the R4 definition of Location resource as well. The StructureMap in R5 was valid and compatible with map file which also passed the [compiler](https://www.antvaset.com/fhir-mapping-language-compiler). [Generated output](https://github.com/yashar2028/Sample-FHIR-Mapping/blob/main/output/map.json) from the python script was validated using https://validator.fhir.org.

### Considarations when using validator tool:
Mapping engine was not able to read the status metadata for StructureMap so it was manually added. I changed '///' to '//' since validator tool was not able to parse the metadata information.

When invoking the `$convert` with validator tool there are possibilities that it cannot connect to FHIR terminology server at http://tx.fhir.org. In this case try again after some seconds.


## Notes
- This project uses the [HAPI FHIR Validator Tool](https://github.com/hapifhir/org.hl7.fhir.core) (version 6.5.1) only to convert the mapping file to corresponding StructureMap resource.
- [FHIRPath](https://build.fhir.org/ig/HL7/FHIRPath/) expressions in the mapping file can be potentially used to improve the type safety and control flow.
- The main purpose of this mapping is to provide a possible an example on possible usage of FML and serve as a reference.
- This mapping is not under any IG, package or organization.

