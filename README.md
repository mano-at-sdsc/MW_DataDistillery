# MW_DataDistillery
Information related the Metabolomics Workbench data (and other relevant data) shared with the CFDE Data Distillery Partnership

Data on three types of relationships has been shared with the CFDE Data Distillery Partnership. Purpose is to understand what metabolites may be regulated by various genes, which cell or tissue produce them and for what diseases they might be relevant. The procedure to generate the tables is briefly described below.

1. Gene-metabolite relationships: MW database tables based on KEGG and other resources

Of all the human genes, the one’s catalyzing various metabolic reactions were identified and then the related metabolites were identified by querying various MW database tables.  

2. Disease-metabolite relationships: A paper based on HMDB data; https://pubmed.ncbi.nlm.nih.gov/32426349/

Based on the paper https://pubmed.ncbi.nlm.nih.gov/32426349/ (which used HMDB data), metabolite names were harmonized using MW REFMET.The psql query used on MW DB was:

```postgres
\COPY (SELECT DISTINCT md.id as md_id, md.metabolite as md_metabolite, mb.name as mb_name, mb.sys_name as mb_sys_name, mb.refmet_name, mb.regno, md.disease, md.disclass, md.biospecimen,
        md.hmdb_id as md_hmdb_id, mb.hmdb_id as mb_hmdb_id, md.drugbank_id as md_drugbank_id, mb.drugbank_id as mb_drugbank_id, md.kegg_id as md_kegg_id, mb.kegg_id as mb_kegg_id, md.pubchem_cid as md_pubchem_cid, mb.pubchem_cid as mb_pubchem_cid from
(met_dis_PMID_32426349 md LEFT JOIN mb_names mb ON ((trim(lower(md.metabolite)) = trim(lower(mb.name))) OR ( (md.hmdb_id = mb.hmdb_id) AND ((md.pubchem_cid IS NULL) OR (mb.pubchem_cid IS NULL) OR (md.pubchem_cid = mb.pubchem_cid) OR (md.kegg_id='') OR (mb.kegg_id='') OR (md.kegg_id = mb.kegg_id) ) ) ) )
ORDER BY md.id, md.metabolite, mb.name, mb.sys_name, mb.refmet_name, mb.regno, md.disease, md.disclass, md.biospecimen,
        md.hmdb_id, mb.hmdb_id, mb.drugbank_id, mb.kegg_id, md.pubchem_cid, mb.pubchem_cid)
TO met_dis_PMID_32426349_nameHMDBkeggpc.tsv WITH DELIMITER E'\t' NULL '' CSV HEADER;
```
The resulting table was further cross-checked and updated as necessary in MS Excel. DOID and HPO IDs and UMLS IDs were obtained through cross-reference/search in respective resources using custom R scripts to generate the nodes and edges files. 

3. Cell-metabolite relationships: MW database tables for data submitted to NMDR

MW database tables were queried to identify which cell/tissue/anatomy (“sample source” in MW terminology) produced which metabolites across different studies: this was indirectly achieved by identifying the sample_source and the metabolites measured (listed) for all the public studies.

The resulting tables were further processed using custom R script (to be made available upon publication of a paper) to generate the list of nodes and edges in suitable format for ingestion into a neo4j database.

For any questions, please contact mano_at_sdsc_dot_edu.
