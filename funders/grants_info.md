# Structured data on grants

## European projects

All the grants and their associated metadata are available in the dataset https://data.enseignementsup-recherche.gouv.fr/explore/dataset/fr-esr-all-projects-signed-informations/table/?disjunctive.free_keywords

In particular, the different frameworks (from FP6 to Horizon Europe) and programs (ERC, Marie-Curie etc.) are described. The grant ids are also in the file but they are difficult to catch (eg `001516`)

## ANR

ANR provides open data to describe their grants:

 - https://www.data.gouv.fr/datasets/anr-01-projets-anr-dos-et-dgds-detail-des-projets-et-des-partenaires (cf https://www.data.gouv.fr/datasets/anr-01-projets-anr-dos-et-dgds-detail-des-projets-et-des-partenaires?resource_id=87d29a24-392e-4a29-a009-83eddcff3e66)

 - https://www.data.gouv.fr/datasets/anr-02-projets-anr-dgpie-detail-des-projets-et-des-partenaires ( cf https://www.data.gouv.fr/datasets/anr-02-projets-anr-dgpie-detail-des-projets-et-des-partenaires?resource_id=aca6972b-577c-496a-aa26-009f81256dcb)

The grant id (called Code Decision) are easier to spot et suive une regex du type `\bANR-\d{2}-[A-Z0-9]{3,5}-\d{4}\b` 
