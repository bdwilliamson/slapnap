---
title: "`slapnap`: Super LeArner Prediction of NAb Panels"
author: "David Benkeser, Brian D. Williamson, Craig A. Magaret, Bhavesh R. Borate, Peter B. Gilbert"
date: "May 22, 2020"
bibliography: refs.bib
---



# Welcome {-}

The [`slapnap`](https://hub.docker.com/r/slapnap/slapnap) container is a tool for using the Compile, Analyze and Tally NAb Panels (CATNAP) database to develop predictive models of HIV-1 neutralization sensitivity to one or several broadly neutralizing antibodies (bNAbs). 

<img src="gp120.png" width="70%" style="display: block; margin: auto;" />
<div class="center">
<p>Crystal structure of HIV-1 gp120 glycoprotein. Highlighted residues indicating sites most-predictive of VRC01 neutralization resistance. <span class="citation">[@magaret2019prediction]</span></p>
</div>

In its simplest form, `slapnap` can be used simply to access and format data from CATNAP in a way that is usable for machine learning analysis. However, the tool also offers fully automated and customizable machine learning analyses based on up to five different neutralization endpoints, complete with automated report generation to summarize results and identify the most predictive features. 

This document serves as the user manual for the `slapnap` container. Here, we describe everything needed to utilize the `slapnap` container and understand its output. The documentation is organized into the following sections:
	
* Section \@ref(sec:docker) provides a brief overview of Docker, including information on installing Docker and downloading the `slapnap` container.
* Section \@ref(sec:catnap) provides a brief overview of the CATNAP database and the specifics of how the data were accessed to build the `slapnap` container. 
* Section \@ref(sec:runningcontainer) provides a detailed description of how to make calls to the slapnap repository, including descriptions of all options that are available.
* Section \@ref(sec:examples) includes several example calls to the `slapnap` container and descriptions of their output.
* Section \@ref(sec:data) provides a description of the data set created in the `slapnap` container. 
* Section \@ref(sec:methods) provides an overview of the methodology that is used in within the `slapnap` analysis. 

If you have any issues or questions about using `slapnap`, please [file an issue](https://github.com/benkeser/slapnap/issues) on GitHub. 

# Docker {#sec:docker}

Docker is a free platform for building containers. Containers are standard units of software that package code and all its dependencies, so that the code can be executed reliably irrespective of computing environment. The `slapnap` tool relies on machine learning implemented in the `R` language and relies on several packages. Achieving full reproducibility for such analyses is challenging in that it requires synchronization across the specific version of `R` and dependent packages. In other words, two users running two versions of `R` or two versions of the same `R` package may arrive at different output when running the same code. Containerization ensures that this does not happen. Any two runs of `slapnap` with the same input options will yield the same output every time. 

[Installing Docker](https://docs.docker.com/docker-for-windows/install/) is necessary for running the `slapnap` tool. While it is not necessary for execution of the `slapnap` container, readers interested in learning more about Docker should consult the [Docker documentation](https://docs.docker.com/get-started/) for information about getting started using Docker. 

Once Docker has been installed on your local computer, you can download `slapnap` using the following command. 


```bash
docker pull slapnap/slapnap
```

This command pulls the image from [DockerHub](https://hub.docker.com/). Once the image has been downloaded, we are ready to learn about how to execute `slapnap` jobs. The next section contains information on the source data used by `slapnap`. Users familiar with the CATNAP data may wish to skip directly to Section \@ref(sec:opts). 

# CATNAP Database {#sec:catnap}

The [CATNAP database](https://www.hiv.lanl.gov/components/sequence/HIV/neutralization/index.html) is a web server hosted by Los Alamos National Laboratory [@yoon2015catnap]. The database integrates antibody neutralization and HIV-1 sequence data from published studies. Neutralization is measured in terms of half maximal inhibitory concentration (IC$_50$) and 80\% inhibitory concentration (IC$_80$). These measures of neutralization against HIV envelope pseudoviruses are available for many broadly neutralizing antibodies (bNAbs) and for some combination bNAbs. Also available on each pseudovirus are amino acid (AA) sequence features for the gp160 protein. These are detailed in Section \@ref(sec:data). 

During each build of the `slapnap` container, all raw data are downloaded from CATNAP. At run time, the relevant data are selected and processed into a format that is amenable for predictive machine learning analyses. 

The date of last build of `slapnap` (and thus the last date data were downloaded from CATNAP) is May 22, 2020.  

# Running the `slapnap` container {#sec:runningcontainer}

To run the `slapnap` container, we make use of the [`docker run`](https://docs.docker.com/engine/reference/run/) command. Note that administrator (`sudo`) privileges are needed to execute this command. 

There are several options that are necessary to include in this command to control the behavior of `slapnap`. These are discussed in separate subsections below. 

## Mounting a local directory {#sec:mounting}

At the end of a `slapnap` run, user-specified output will be saved (see option `return` in Section \@ref(sec:opts)). To retrieve these files from inside the container, we [*mount*](https://docs.docker.com/storage/bind-mounts/) a local directory to an output directory (`/home/out/`) in the container using the `-v` option. That is, all files in the mounted local directory will be visible to programs running inside the container and any items saved to the output directory in the container (file path in the container `/home/out/`) will be available in the mounted directory. 

Suppose `/path/to/local/dir` is the file path on a local computer in which we wish to save the output files from a `slapnap` run. A `docker run` of `slapnap` would include the option `-v /path/to/local/dir:/home/out`. After a run completes, the requested output should be viewable in `/path/to/local/dir`. See Section \@ref(sec:examples) for full syntax. 

## `slapnap` options {#sec:opts}

The user has control over many aspects of `slapnap`'s behavior. These options are passed in using the `-e` option^[This sets an environment variable in the container environment. These variables are accessed by the various `R` and `bash` scripts in the container to dictate how the container executes code.]. Semi-colon separated strings are used to set options. For example, to provide input for the option `option_name`, we would used `-e option_name="a;semi-colon;separated;string"`. Note that there are no spaces between the option name and its value and no spaces after semi-colons in the separated list. See Section \@ref(sec:examples) for full syntax. 

Each description below lists the default value that is assumed if the option is not specified. Note that many of the default values are chosen simply so that na{\"i}ve calls to `slapnap` compile quickly. Proper values should be determined based on scientific context. 

__-e options for `slapnap`__ 

* __`nab`__: A semicolon-separated list of bNAbs (default = "`VRC01`"). A list of possible bNAbs can be found [here](https://www.hiv.lanl.gov/components/sequence/HIV/neutralization/main.comp). If multiple bNAbs are listed, it is assumed that the analysis should be of estimated `outcomes` for a combination of bNAbs (see Section \@ref(sec:outcomedefs) for details on how estimated outcomes for multiple bNAbs are computed). 
* __`outcomes`__: A semicolon-separated string of outcomes to include in the analysis. Possible values are `"ic50"` (default), `"ic80"`, `"iip"`, `"sens1"`, `"sens2"` (for definitions of outcomes see Section \@ref(sec:outcomedefs)). 
* __`learners`__: A semicolon-separated string of machine learning algorithms to include in the analysis. Possible values include `"rf"` (random forest, default), `"xgboost"` (eXtreme gradient boosting), and `"lasso"` (elastic net). If more than one algorithm is included, then it is assumed that a cross-validated-based ensemble (i.e., a super learner) is desired (see Section \@ref(sec:sldetails)). 
* __`cvtune`__: A boolean string (i.e., either `"TRUE"` or `"FALSE"` [default]) indicating whether the `learners` should be tuned using cross-validation and a small grid search. Defaults to `"FALSE"`. If multiple `learners` are specified, then the super learner ensemble includes three versions of each of the requested `learners` with different tuning parameters. 
* __`cvperf`__: A boolean string (i.e., either `"TRUE"` or `"FALSE"` [default]) indicating whether the `learners` performance should be evaluated using cross-validation. If `cvtune="TRUE"` or `learners` includes multiple algorithms, then nested cross-validation is used to evaluate the performance of the cross-validation-selected best value of tuning parameters for the specified algorithm or the super learner, respectively. 
* __`nfolds`__: A numeric string indicating the number of folds to use in cross-validation procedures (default = `"2"`). 
* __`importance_grp`__: A semicolon-separated string indicating which group-level variable importance measures should be computed. Possible values are none `""` (default), marginal `"marg"`, conditional `"cond"`. See Section \@ref(sec:biolimp) for details on these measures. 
* __`importance_ind`__: A semicolon-separated string indicating which individual-level variable importance measures should be computed. Possible values are none `""` (default), learner-level `"pred"`, marginal `"marg"` and conditional `"cond"`. The latter two take significant computation time to compute. 
* __`report_name`__: A string indicating the desired name of the output report (default = `report_[_-separated list of nabs]_[date].html`). 
* __`return`__: A semicolon-separated string of the desired output. Possible values are `"report"` (default), `"learner"` for the trained algorithm, `"data"` for the analysis dataset, `"figures"` for all figures from the report, and `"vimp"` for variable importance objects.

## Interactive sessions

To simply enter the container and poke around, use an interactive session by including `-it` and overriding the container's entry-point. 


```bash
docker run -it slapnap/slapnap /bin/bash
```

This will enter you into the container in a bash terminal. This may be useful for exploring the file structure, examining versions of `R` packages that are included in the container, etc.

# Examples {#sec:examples}

## A basic call to `slapnap`

A call to `slapnap` with all default options can be run using the following command.


```bash
docker run -v /path/to/local/dir:/home/out slapnap/slapnap
```

Note that this call mounts the local directory `path/to/local/dir` to receive output from the container (Section \@ref(sec:mounting)).

When this command is executed, the following operations 

## Super learning

## Train an algorithm

## Pull and clean data


# Report details {#sec:report}


# Data details {#sec:data}


# Method details {#sec:methods}

## Outcome definitions {#sec:outcomedefs}

## Super learner details {#sec:sldetails}

## Variable importance details 

### Biological importance {#sec:biolimp}

### Predictive importance

# References {#sec:refs}