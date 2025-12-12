# RefSeq Pipeline – Architecture & Developer Guide

# Overview 
## The RefSeq Pipeline is a Spark + Delta Lake–based data ingestion and update system design
1. Fetch genome assembly metadata from NCBI Datasets API <br>
2. Track content-level changes using hash snapshots <br>
3. Support incremental updates instead of full reprocessing <br>
4. Normalize heterogeneous NCBI responses into a stable CDM (Common Data Model) <br>
This document focuses on architecture, module responsibilities, and execution flow, rather than end-user CLI usage. <br>


# Design Principle
1. Deterministic IDs: CDM IDs are UUIDv5-based and stable across runs <br>
2. Incremental by default: Hash snapshots determine what actually changed <br>
3. Pure Spark execution: No Pandas dependency in the core pipeline <br>
4. Schema-first: All outputs conform to CDM_SCHEMA <br>
5. Separation of concerns: API access, parsing, hashing, and storage are isolated <br>

# Core Modules (Execution Order) 
## config.py (Global Configuration & Schema)
### Responsibility 
Central configuration for the entire pipeline <br>
### Defines 
	•	CDM_NAMESPACE (UUID namespace for stable IDs)
	•	NCBI API base URL
	•	EXPECTED_COLS
	•	CDM_SCHEMA (Spark StructType)
	
from refseq_pipeline.core.config import CDM_SCHEMA, EXPECTED_COLS <br>
This file is imported by almost every other module.
It defines the data contract of the pipeline.

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



