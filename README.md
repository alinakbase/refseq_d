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

# Core 
## Refseq Pipeline - Configuration (config.py) 
The config.py file centralizes all constants, schema definitions, and shared parameters required by other components in the pipeline, ensuring consistency and maintainability across modules.<br> 

## The configuration module defines:
	•	Global constants used across the RefSeq ingestion and parsing pipeline <br>
	•	Default URLs for accessing NCBI RefSeq metadata <br>
	•	Expected CDM (Common Data Model) fields <br>
	•	The CDM schema used when creating Spark DataFrames <br>
	•	The UUID namespace for deterministic CDM ID generation <br>

This ensures all modules reference a single source of truth instead of duplcating configuration. 

# Operating Sequence 



