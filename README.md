# RefSeq Pipeline – Full Technical Documentation
This document explains how to run, understand, and extend the RefSeq Pipeline.
It covers all modules inside refseq_pipeline/core/ and refseq_pipeline/cli/, execution order, internal logic, and code snippets.

# Overview 
The RefSeq Pipeline is a Spark + Delta Lake workflow that:
	1.	Downloads and parses the latest RefSeq Assembly Index
	2.	Fetches annotation hashes or MD5 checksums
	3.	Writes Delta snapshot tables
	4.	Compares snapshots and detects changed assemblies
	5.	Parses genome assembly reports into a CDM-compliant schema


