# RefSeq Pipeline – Full Technical Documentation
This document explains how to run, understand, and extend the RefSeq Pipeline.
It covers all modules inside refseq_pipeline/core/ and refseq_pipeline/cli/, execution order, internal logic, and code snippets.

# Overview 
The RefSeq Pipeline is a Spark with Delta Lake workflow that: <br>
	1.	Downloads and parses the latest RefSeq Assembly Index <br>
	2.	Fetches annotation hashes or MD5 checksums <br>
	3.	Writes Delta snapshot tables <br>
	4.	Compares snapshots and detects changed assemblies <br>
	5.	Parses genome assembly reports into a CDM-compliant schema <br>

# Files that do not need to be run in Core:
## config.py (Configuration)
The config.py file centralizes all constants, schema definitions, and shared parameters required by other components in the pipeline, ensuring consistency and maintainability across modules.<br> 

## datasets_api.py (Fetching genome reports from NCBI datasets API)
The RefSeq pipeline retrieves genome assembly metadata directly from the NCBI Datasets API, which serves as the authoritative and up-to-date source for RefSeq assembly reports. <br> 
This module implements a robust, retry-enabled API client that streams assembly reports for a given taxonomic ID. <br> 
### API endpoint 
All requests are made against the NCBI Datasets V2 API: <br>
http://api.ncbi.nlm.nih.gov/datasets/v2 <br> 

Genome reports are fetched from: <br> 
/genome/taxon/{taxon}/dataset_report 

## 



# Operating Sequence 



