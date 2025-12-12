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

## Step 1. config.py (Global Configuration & Schema)
### Responsibility 
Central configuration for the entire pipeline <br>
### Defines 
1. CDM_NAMESPACE (UUID namespace for stable IDs)
2. NCBI API base URL
3. EXPECTED_COLS
4. CDM_SCHEMA (Spark StructType) <br>

This file is imported by almost every other module. It defines the data contract of the pipeline.


## Step 2. datasets_api.py (NCBI Datasets API Client)
### Responsibility
1. Fetch genome dataset reports from NCBI
2. Handle pagination, retries, rate limits
   
### Key API 
fetch_reports_by_taxon(taxon_id: str) -> Iterable[dict]

### Notes
The RefSeq pipeline retrieves genome assembly metadata directly from the NCBI Datasets API, which serves as the authoritative and up-to-date source for RefSeq assembly reports. <br> 
This module implements a robust, retry-enabled API client that streams assembly reports for a given taxonomic ID. <br> 

### API endpoint 
All requests are made against the NCBI Datasets V2 API: <br>
http://api.ncbi.nlm.nih.gov/datasets/v2 <br> 

Genome reports are fetched from: <br> 
/genome/taxon/{taxon}/dataset_report 



## Step 3. refseq_io.py (RefSeq FTP & Index Utilities) 
### Responsibility
1. Load RefSeq assembly index
2. Resolve: accession → ftp_path; accession → taxid
3. Fetch remote files: annotation hash files; MD5 checksums <br> 
This module bridges NCBI metadata and FTP content.

	
## Step 4. cdm_parse.py (Normalize Reports into CDM) 
### Responsibility
1. Normalize raw NCBI reports into CDM-aligned records
2. Generate stable CDM IDs
3. Perform safe type conversions


## Step 5. spark_delta.py (Spark & Delta Lake I/O layer)


## hashes_diff.py 
Compares two snapshots and finds which taxa changed. 
Core logic
	•	full outer join on (accession, kind)
	•	detect new / missing / modified hashes
	•	map changed accessions → taxon IDs
This enables incremental taxon-level reprocessing.

## hashes_snapshot.py (build hash snapshots) 





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



