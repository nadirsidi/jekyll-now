---
layout: post
title: Custom R Packages on Databricks
tags: databricks R
---
## Problem Overview

The [Databricks](https://databricks.com/) platform provides a great solution for data wonks to write polyglot notebooks that leverage tools like Python, R, and most-importantly Spark. It is easy to experiment in a notebook and then scale it up to a solution that is more production-ready, leveraging features like scheduled, AWS clusters.

In my case, I need to use an ecosystem of custom, in-house R packages, hosted on our internal GitHub Enterprise server, to interact with various internal services. Databricks allows users to manage packages using [Libraries](https://docs.databricks.com/user-guide/libraries.html), but currently only R packages that are hosted on a CRAN server can be installed.

In this post I will go through my process for POSTing a custom R package to the [Databricks File System (dbfs)](https://docs.databricks.com/user-guide/dbfs-databricks-file-system.html) and installing it on each node of a cluster using a [Cluster Node Initialization Script (init script)](https://docs.databricks.com/user-guide/clusters/init-scripts.html).

## Why Not Just Install Using `R install.packages()`?

At first, a Databricks user might be tempted to install a package directly in a notebook using `install.packages()` in an R code block. While you can do this, the drawback is that it will only install the package on the node that runs that R command. In a distributed setting, such as using a R-Spark binding, you could end up sending work to a node that doesn't have the package installed.

While there are other ways around this constraint, by setting up an init script to run when our cluster is initialized, this ensures our R package is installed on every node in the cluster. This is especially useful if we are going to take advantage of auto-scaling and spot instances, which Databricks easily supports.

## Set-Up

In order to reproduce my approach, you'll need to ensure you have set-up the [Databricks command-line interface (CLI)](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html). The CLI is just a python package, so I found it was cleanest to put it in a new virtualenv.

I am assuming you have access to the custom R package repository. By this I mean the directory with the `DESCRIPTION` file.

This process will also work if you already have the tar.gz file for the package. You can use a package binary too, but you'll need to ensure you use the correct `install.packages()` arguments.

## Install a Custom R Package Using an Init Script

1. If you do not already have a [bundled package](http://r-pkgs.had.co.nz/package.html), use the R function `devtools::build()` to build a local bundled package tar.gz file.
2. Use the dbfs CLI to cp the tar.gz file to your desired path in the dbfs. For example, I copied my tar.gz using the command:
```
dbfs cp ./my_package.tar.gz dbfs:/home/nadir/custom_R_packages/
```
3. We'll need to ensure we first install all the package dependencies from a CRAN mirror. You can compare your custom package dependencies listed in the [DESCRIPTION](http://r-pkgs.had.co.nz/description.html) file to the already-installed R packages in each Databricks runtime environment. Here is the list of [R packages in the Databricks 4.3 runtime enviroment](https://docs.databricks.com/release-notes/runtime/4.3.html#installed-r-libraries).
4. Write a bash script to invoke R, install all necessary dependencies from CRAN, and install your local package from the dbfs. Here is my `install_my_package.sh` init script. Note, you'll need to specify a CRAN repo as there doesn't seem to be a default. Also, note the different syntax for referring to the dbfs with the CLI versus in the script:

  ```
  #!/bin/bash

  # Install necessary dependencies from CRAN for your custom package
  # Dependencies I need that are already in Databricks runtime env 4.3:
  # data.table
  # timeDate
  # xml2

  R -e 'install.packages(c("rvest", "evaluate"), repos = "https://cloud.r-project.org")'

  # Install the local package code for your package
  R -e 'install.packages("/dbfs/home/nadir/custom_R_packages/my_package.tar.gz", repos = NULL)'
  ```

## Monitoring Init Scripts

If you need to debug your init script, Databricks will create [init script logs](https://docs.databricks.com/user-guide/clusters/init-scripts.html#init-script-logs) of the console output and save them to a location of your choice. You will first need to ensure you turn on cluster logging in the cluster configuration as shown in the screenshot below:

![screenshot]({{ site.baseurl }}/images/databricks-cluster-logging.png "Screenshot of clustering logging configuration")

You can use the dbfs CLI to copy the logs to your local machine to read them. The automated logging naming convention is a bit cumbersome, and there is no auto-complete using the CLI, but I found this to be an easy way to copy down the logs.

Alternatively, you could change your logging destination to AWS S3 and access them in S3.

## Using Your Custom R Environment

If everything worked correctly, you should now be able to use your custom package each time the cluster starts-up. This should work for both interactive clusters and clusters you launch for batch jobs. If you can successfully load your custom package, `library(my_package)` you should be good to go forth and code.

Note, you may need to attach the package into the namespace on each worker node-- I haven't experimented with this. If that is the case, just make sure to run `library(my_package)` before you run distributed jobs that need to use your package functions. You could also explicitly load your package each time you call a function, such as `my_package::my_function()`.
