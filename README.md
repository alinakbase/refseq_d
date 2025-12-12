# RefSeq Pipeline – Full Technical Documentation
This document explains how to run, understand, and extend the RefSeq Pipeline.
It covers all modules inside refseq_pipeline/core/ and refseq_pipeline/cli/, execution order, internal logic, and code snippets.

# Overview 
Detect genome changes by hashing authoritative RefSeq files (annotation/md5), then perform snapshot diffs to identify updated or newly added assemblies. <br>
The RefSeq Pipeline is a Spark with Delta Lake workflow that: <br>
	1.	Downloads and parses the latest RefSeq Assembly Index <br>
	2.	Fetches annotation hashes or MD5 checksums <br>
	3.	Writes Delta snapshot tables <br>
	4.	Compares snapshots and detects changed assemblies <br>
	5.	Parses genome assembly reports into a CDM-compliant schema <br>

# High-level architecture 
NCBI Datasets API --> datasets_api.py (fetch assembly reports by taxon) --> cdm_parse.py (from parse reports to CDM schema) --> Delta lake <br> 
From delta lake, it divided by two scripts: <br> 
1. hashes_snapshot.py (snapshot content hashes)
2. snapshot_utils.py (compare snapshots) <br>
--> hashes_diff.py (map changed accession to taxon ID) 

# Core Concepts 
## Hash-based snapshot 
RefSeq assemblies change over time due to:
	•	re-annotation
	•	contamination fixes
	•	metadata corrections

Instead of comparing full files, this pipeline:
	•	fetches annotation_hashes.txt or md5checksums.txt
	•	computes a SHA256 fingerprint
	•	stores the fingerprint in Delta Lake

If the fingerprint changes, the assembly has changed.

# Files in Core:
## config.py (Configuration)
Defines constants and schemas shared across the pipeline. 
### Responsibilities 
	•	Define CDM UUID namespace
	•	Define RefSeq URLs
	•	Define expected output columns
	•	Define Spark schema for assembly stats
This file is not meant to be executed, it is imported by other modules. 

## datasets_api.py (Fetching genome reports from NCBI datasets API)
The RefSeq pipeline retrieves genome assembly metadata directly from the NCBI Datasets API, which serves as the authoritative and up-to-date source for RefSeq assembly reports. <br> 
This module implements a robust, retry-enabled API client that streams assembly reports for a given taxonomic ID. <br> 
### API endpoint 
All requests are made against the NCBI Datasets V2 API: <br>
http://api.ncbi.nlm.nih.gov/datasets/v2 <br> 

Genome reports are fetched from: <br> 
/genome/taxon/{taxon}/dataset_report 

What it does
	•	Handles pagination
	•	Retries on transient failures
	•	Optionally filters:
	•	RefSeq-only
	•	current assemblies only
	
## cdm_parse.py (Normalize Reports into CDM) 
This is the semantic core of the pipeline. <br> 
Responsibilities
	•	Normalize inconsistent JSON fields (snake_case / camelCase)
	•	Safely convert numeric fields
	•	Generate deterministic CDM IDs
	•	Produce Spark DataFrames compatible with Delta Lake <br>
Output schema: <br>
Matches CDM_SCHEMA exactly.


## spark_delta.py 


## hashes_diff.py 
Compares two snapshots and finds which taxa changed. 
Core logic
	•	full outer join on (accession, kind)
	•	detect new / missing / modified hashes
	•	map changed accessions → taxon IDs
This enables incremental taxon-level reprocessing.

## hashes_snapshot.py (build hash snapshots) 


## refseq_io.py (Refseq index and FTP access) 
Handles RefSeq assembly metadata and FTP access. <br> 
Responsibilities
	•	Download & parse assembly_summary_refseq.txt
	•	Map accession → ftp_path / taxid
	•	Fetch:
	•	annotation_hashes.txt
	•	md5checksums.txt

This module is used by hash snapshot generation.


## snapshot_utils.py 



## debug_snapshot.py 
A minimal runnable script that verifies:
	1.	Spark + Delta setup
	2.	RefSeq index download
	3.	FTP hash fetching
	4.	Snapshot creation
	5.	Delta write & SQL query
Run it: 
python -m refseq_pipeline.core.debug_snapshot


# Operating Sequence 



