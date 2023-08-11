# MW_DataDistillery
Information related the Metabolomics Workbench data (and other relevant data) shared with the CFDE Data Distillery Partnership

Data on three types of relationships has been shared with the CFDE Data Distillery Partnership. Purpose is to understand what metabolites may be regulated by various genes, which cell or tissue produce them and for what diseases they might be relevant. The procedure to generate the tables is briefly described below.

1. Gene-metabolite relationships: MW database tables based on KEGG and other resources

Of all the human genes, the one’s catalyzing various metabolic reactions were identified and then the related metabolites were identified by querying various MW database tables. The resulting table was further processed using the R script .

2. Disease-metabolite relationships: A paper based on HMDB data; https://pubmed.ncbi.nlm.nih.gov/32426349/

Based on the paper https://pubmed.ncbi.nlm.nih.gov/32426349/ (which used HMDB data), metabolite names were harmonized using MW REFMET. DOID and HPO IDs and UMLS IDs were obtained through cross-reference/search in respective resources using custom R scripts. 

3. Cell-metabolite relationships: MW database tables for data submitted to NMDR

MW database tables were queried to identify which cell/tissue/anatomy (“sample source” in MW terminology) produced which metabolites across different studies: this was indirectly achieved by identifying the sample_source and the metabolites measured (listed) for all the public studies.


Briefly, the following approach was followed to generate the tables.

