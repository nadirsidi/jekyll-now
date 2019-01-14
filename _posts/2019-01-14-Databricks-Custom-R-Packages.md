---
layout: post
title: Custom R Packages on Databricks
tags: databricks R
---
## Problem

Need to use custom packages or packages on GitHub on a Databricks cluster, but Databricks doesn't support adding an R library from a local file.

## Why Not Just Install via Notebook

Want to make sure packages are installed on ALL nodes. Init scripts will run for each machine in cluster.

## Set-Up

* Databricks CLI to access dbfs
* Desired package as local repo

## Steps Using Cluster-Specific Init Script

1. In R, use devtools::build() to build local package repo into tar.gz file
2. Copy tar.gz file to the dbfs with `dbfs cp ./tar.gz dbfs:/<path>`
3. Compare package dependencies (see package metadata) with R packages listed in Databricks runtime environment.
4. Write bash script to invoke R and install necessary dependencies from CRAN, and install local package from dbfs.

```
sample bash script here 
```

## Monitoring Init Scripts

### Turn on Logging

### Copying Log File
