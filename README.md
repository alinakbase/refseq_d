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


