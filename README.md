# RefSeq Pipeline – Architecture & Developer Guide

The RefSeq Pipeline is a Spark- and Delta Lake–based data ingestion and update system designed to
efficiently track and process genome assembly data from NCBI RefSeq.

Key capabilities include:

- Fetching genome assembly metadata from the NCBI Datasets API
- Tracking content-level changes using hash-based snapshots
- Supporting incremental updates instead of full reprocessing
- Normalizing heterogeneous NCBI responses into a stable Common Data Model (CDM)

This document focuses on the internal architecture, module responsibilities, and execution flow,
rather than end-user CLI usage.


## Design Principles

The RefSeq Pipeline follows a set of explicit design principles to ensure
scalability, reproducibility, and maintainability:

- **Deterministic IDs**  
  CDM identifiers are UUIDv5-based and remain stable across runs given the same input.

- **Incremental by default**  
  Hash-based snapshots are used to detect content-level changes, avoiding unnecessary reprocessing.

- **Pure Spark execution**  
  The core pipeline avoids Pandas and relies exclusively on Spark for scalability.

- **Schema-first design**  
  All outputs strictly conform to the predefined `CDM_SCHEMA`.

- **Separation of concerns**  
  API access, parsing, hashing, and storage are implemented as independent modules.

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
### Responsibility
1. Build SparkSession with Delta support
2. Write DataFrames to:
   managed Delta tables <br> 
   external Delta paths <br> 
4. Handle:
   schema evolution <br> 
   overwrite / append <br> 
   deduplication <br> 
   cleanup <br> 
   table registration <br>
This module is pure infrastructure. 


## Step 6. hashes_snapshot.py (build hash snapshots) 
### Responsibility
1. Generate content fingerprints for assemblies
2. Fetch annotation hashes and MD5 checksums (fallback)
3. Compute SHA256
4. Output Spark DataFrame

### Purpose
Detect real biological content changes, not metadata noise.



## Step 7. hashes_diff.py (Incremental Change Detection) 
### Responsibility
1. Compare two hash snapshots
2. Detect new assemblies, updated assemblies and removed assemblies
3. Map changed accessions → taxonomy IDs
   
This module determines what needs to be reprocessed.


## Step 8. snapshot_utils.py (Snapshot Comparison Helpers)
### Responsibility
1. Lightweight helpers for changed accessions, new accessions and removed accessions
2. Operates on Delta paths instead of metastore tables
Often used by CLI or orchestration scripts.


## Step 9. debug_snapshot.py (Debug and validation script) 
### Responsibility
A minimal runnable script that verifies:
	1.	Spark + Delta setup<br> 
	2.	RefSeq index download<br> 
	3.	FTP hash fetching<br> 
	4.	Snapshot creation<br> 
	5.	Delta write & SQL query<br> 
### Run it: 
python -m refseq_pipeline.core.debug_snapshot <br> 
Not part of production flow. Recommended to run once during setup. 

## Step 10. driver.py (Pipeline Orchestration) 
### Responsibility
1. High-level pipeline execution
2. Coordinates metadata fetch, snapshot creation, diff detection, CDM parsing and Delta writes. 

This is typically the main entry point.

## What should run directly: 
1. driver.py<br> 
2. debug_snapshot.py<br>

## Incremental Update Workflow 
1. Create new hash snapshot 
2. Compare with previous snapshot
3. Identify changed taxids
4. Fetch metadata only for affected taxa
5. Re-parse and overwrite CDM tables (deduplicated)

## Result
1. Orders-of-magnitude faster than full re-ingest
2. Deterministic, reproducible updates

# Command-Line Interface (CLI) Modules 
The RefSeq Pipeline exposes a small set of CLI-oriented entry points designed for:<br> 
	•	Incremental updates<br> 
	•	Snapshot comparison<br> 
	•	Operational debugging<br> 
	•	Automation (cron / Airflow / CI jobs)<br> 

CLI modules do not implement business logic themselves.<br> 
They orchestrate functionality from the core/ modules.<br> 













