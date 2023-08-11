# MW_DataDistillery
Information related the Metabolomics Workbench data (and other relevant data) shared with the CFDE Data Distillery Partnership

Data on three types of relationships has been shared with the CFDE Data Distillery Partnership. Purpose is to understand what metabolites may be regulated by various genes, which cell or tissue produce them and for what diseases they might be relevant. The procedure to generate the tables is briefly described below.

1. Gene-metabolite relationships: MW database tables based on KEGG and other resources

Of all the human genes, the ones catalyzing various metabolic reactions were identified and then the related metabolites were identified by querying various MW database tables through the postgres sql (psql) query:

```postgres
\COPY (SELECT DISTINCT mgpgk_kgrm.gene_id, mgp_gene.gene_symbol, mgpgk_kgrm.kegg_id, rf_mb.regno, rf_mb.pubchem_cid, rf_mb.drugbank_id, rf_mb.hmdb_id, rf_mb.refmet_name as refmet_name, mgpgk_kgrm.reactant_name as kegg_name from
( (SELECT DISTINCT rf.regno, rf.pubchem_cid, mb.kegg_id, mb.drugbank_id, mb.hmdb_id, mb.refmet_name FROM (refmet rf INNER JOIN mb_names mb ON (rf.regno = mb.regno AND rf.pubchem_cid = mb.pubchem_cid AND rf.name = mb.refmet_name AND (mb.kegg_id IS NOT NULL) ))) rf_mb INNER JOIN
(mgp_gene INNER JOIN
(SELECT distinct mgpgk.gene_id, mgpgk.kegg_id, kgrm.reactant_name from (kegg_reactants_metab kgrm INNER JOIN mgp_gene_to_keggid mgpgk ON mgpgk.kegg_id = kgrm.kegg_id) ORDER BY mgpgk.gene_id, mgpgk.kegg_id) mgpgk_kgrm
ON mgpgk_kgrm.gene_id = mgp_gene.gene_id)
ON mgpgk_kgrm.kegg_id = rf_mb.kegg_id)
ORDER BY mgpgk_kgrm.gene_id, mgpgk_kgrm.kegg_id, rf_mb.pubchem_cid, rf_mb.refmet_name, mgpgk_kgrm.reactant_name)
TO gene_metabolite_generic_KEGG_dbxrefs_refmet.tsv WITH DELIMITER E'\t' NULL '' CSV HEADER;
```

2. Disease-metabolite relationships: A paper based on HMDB data; https://pubmed.ncbi.nlm.nih.gov/32426349/

Based on the paper https://pubmed.ncbi.nlm.nih.gov/32426349/ (which used HMDB data), metabolite names were harmonized using MW REFMET. The postgres sql (psql) query used on MW DB was:

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

MW database tables were queried to identify which cell/tissue/anatomy (“sample source” in MW terminology) produced which metabolites across different studies: this was indirectly achieved by identifying the sample_source and the metabolites measured (listed) for all the public studies using the following psql query.

```postgres
\COPY (SELECT DISTINCT ss.study_id, ss.source as sample_source, sm.refmet_name, rf_mb.regno, rf_mb.pubchem_cid, rf_mb.name, rf_mb.kegg_id, rf_mb.drugbank_id, rf_mb.hmdb_id FROM
(study_status_prod sts INNER JOIN
(study_samplesource ss INNER JOIN
((SELECT DISTINCT study_id, refmet_name FROM study_metabolite where refmet_name IS NOT NULL) sm INNER JOIN
(SELECT DISTINCT rf.regno, rf.pubchem_cid, rf.name as name, rf.kegg_id, mb.drugbank_id, mb.hmdb_id, rf.name as refmet_name FROM
(refmet rf LEFT JOIN (select distinct refmet_name, pubchem_cid, drugbank_id, hmdb_id from mb_names) mb ON (rf.name = mb.refmet_name) ) WHERE (rf.pubchem_cid IS NOT NULL) ) rf_mb
ON (sm.refmet_name = rf_mb.refmet_name))
ON ss.study_id = sm.study_id)
ON ss.study_id = sts.study_id)
WHERE (sts.status=1)
ORDER BY ss.study_id, ss.source, rf_mb.regno, sm.refmet_name, rf_mb.pubchem_cid)
TO MW_samplesource_metabolite_expanded_sm.tsv WITH DELIMITER E'\t' NULL '' CSV HEADER;
```

The resulting tables were further processed using custom R script (to be made available upon publication of a paper) to generate the list of nodes and edges in suitable format for ingestion into a neo4j database.

For any questions, please contact mano_at_sdsc_dot_edu.
