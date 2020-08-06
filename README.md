# ![nf-core/modules](docs/images/nfcore-modules_logo.png)

![GitHub Actions Coda Linting](https://github.com/nf-core/modules/workflows/Code%20Linting/badge.svg)
[![Get help on Slack](http://img.shields.io/badge/slack-nf--core%20%23modules-4A154B?logo=slack)](https://nfcore.slack.com/channels/modules)

> THIS REPOSITORY IS UNDER ACTIVE DEVELOPMENT. SYNTAX, ORGANISATION AND LAYOUT MAY CHANGE WITHOUT NOTICE!

A repository for hosting [Nextflow DSL2](https://www.nextflow.io/docs/latest/dsl2.html) module files containing tool-specific process definitions and their associated documentation.

## Table of contents

- [Using existing modules](#using-existing-modules)
- [Adding a new module file](#adding-a-new-module-file)
    - [Testing](#testing)
    - [Documentation](#documentation)
    - [Uploading to `nf-core/modules`](#uploading-to-nf-coremodules)
- [Terminology](#terminology)
- [Help](#help)
- [Citation](#citation)

## Using existing modules

The module files hosted in this repository define a set of processes for software tools such as `fastqc`, `bwa`, `samtools` etc. This allows you to share and add common functionality across multiple pipelines in a modular fashion.

We have written a helper command in the `nf-core/tools` package that uses the GitHub API to obtain the relevant information for the module files present in the `software/` directory of this repository. This includes using `git` commit hashes to track changes for reproducibility purposes, and to download and install all of the relevant module files.

1. [Install](https://github.com/nf-core/tools#installation) the latest version of `nf-core/tools` (`>=1.10.2`)
2. List the available modules:

    ```console
    $ nf-core modules list

                                              ,--./,-.
              ___     __   __   __   ___     /,-._.--~\
        |\ | |__  __ /  ` /  \ |__) |__         }  {
        | \| |       \__, \__/ |  \ |___     \`-._,-`-,
                                              `._,._,'

        nf-core/tools version 1.10.2



    INFO      Modules available from nf-core/modules (master):                                                                                                                  modules.py:51

    bwa/index
    bwa/mem
    deeptools/computematrix
    deeptools/plotfingerprint
    deeptools/plotheatmap
    deeptools/plotprofile
    fastqc
    ..truncated..
    ```

3. Install the module in your pipeline directory:

    ```console
    $ nf-core modules install . fastqc

                                              ,--./,-.
              ___     __   __   __   ___     /,-._.--~\
        |\ | |__  __ /  ` /  \ |__) |__         }  {
        | \| |       \__, \__/ |  \ |___     \`-._,-`-,
                                              `._,._,'

        nf-core/tools version 1.10.2



    INFO      Installing fastqc                                                                                                                                                 modules.py:62
    INFO      Downloaded 3 files to ./modules/nf-core/software/fastqc                                                                                                           modules.py:97
    ```

4. Import the module in your Nextflow script:

    ```nextflow
    #!/usr/bin/env nextflow

    nextflow.enable.dsl = 2

    include { FASTQC } from './modules/nf-core/software/fastqc/main'
    ```

5. We have plans to add other utility commands to help developers install and maintain modules downloaded from this repository so watch this space!

    ```console
    $ nf-core modules --help

    ...truncated...

    Commands:
      list     List available software modules.
      install  Add a DSL2 software wrapper module to a pipeline.
      update   Update one or all software wrapper modules.             (NOT YET IMPLEMENTED)
      remove   Remove a software wrapper from a pipeline.              (NOT YET IMPLEMENTED)
      check    Check that imported module code has not been modified.  (NOT YET IMPLEMENTED)
    ```

## Adding a new module file

> **NB:** The definition and standards for module files are still under discussion
but we are now gladly accepting submissions :)

If you decide to upload a module to `nf-core/modules` then this will
ensure that it will become available to all nf-core pipelines,
and to everyone within the Nextflow community! See
[`nf-core/modules/software`](https://github.com/nf-core/modules/tree/master/software)
for examples.

### Current guidelines

The key words "MUST", "MUST NOT", "SHOULD", etc. are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

#### General

* Software that can be piped together SHOULD be added to separate module files
unless there is a run-time, storage advantage in implementing in this way. For example,
using a combination of `bwa` and `samtools` to output a BAM file instead of a SAM file:

    ```bash
    bwa mem | samtools view -B -T ref.fasta
    ```

* Where applicable, the usage/generation of compressed files SHOULD be enforced as input/output e.g. `*.fastq.gz` and NOT `*.fastq`, `*.bam` and NOT `*.sam` etc.
* A module MUST NOT contain a `when` statement.
* Where applicable, each module command MUST emit a file `<SOFTWARE>.version.txt` containing a single line with the software's version in the format `<VERSION_NUMBER>` or `0.7.17`:

    ```bash
    echo \$(bwa 2>&1) | sed 's/^.*Version: //; s/Contact:.*\$//' > ${software}.version.txt
    ```

If the software is unable to output a version number on the command-line then a variable called `VERSION` can be manually specified to create this file e.g. [homer/annotatepeaks module](https://github.com/nf-core/modules/blob/master/software/homer/annotatepeaks/main.nf).

#### Naming conventions

* The directory structure for the module name must be all lowercase e.g. `software/bwa/mem/`. The name of the software (i.e. `bwa`) and tool (i.e. `mem`) MUST be all one word.
* The process name in the module file MUST be all uppercase e.g. `process BWA_MEM {`. The name of the software (i.e. `BWA`) and tool (i.e. `MEM`) MUST be all one word separated by an underscore.
* All parameter names MUST follow the `snake_case` convention.
* All function names MUST follow the `camelCase` convention.

#### Module parameters

* A module file SHOULD only define input and output files as command-line parameters to be executed within the process.
* All other parameters MUST be provided as a string i.e. `options.args` where `options` is a Groovy Map that MUST be provided in the `input` section of the process.
* If the tool supports multi-threading then you MUST provide the appropriate parameter using the Nextflow `task` variable e.g. `--threads $task.cpus`.
* Any parameters that need to be evaluated in the context of a particular sample e.g. single-end/paired-end data MUST also be defined within the process.

#### Input/output options

* Named file extensions MUST be emitted for ALL output channels e.g. `path "*.txt", emit: txt`.
* Optional inputs are not currently supported by Nextflow. However, "fake files" MAY be used to work around this issue.

#### Resource requirements

* An appropriate resource `label` MUST be provided for the module as listed in the [nf-core pipeline template](https://github.com/nf-core/tools/blob/master/nf_core/pipeline-template/%7B%7Bcookiecutter.name_noslash%7D%7D/conf/base.config#L29) e.g. `process_low`, `process_medium` or `process_high`.
* If the tool supports multi-threading then you MUST provide the appropriate parameter using the Nextflow `task` variable e.g. `--threads $task.cpus`.

#### Software requirements

[BioContainers](https://biocontainers.pro/#/) is a registry of Docker and Singularity containers automatically created from all of the software packages on [Bioconda](https://bioconda.github.io/). Where possible we will use BioContainers to fetch pre-built software containers and Bioconda to install software using Conda.

* Software requirements SHOULD be declared within the module file using the Nextflow `container` directive e.g. go to the [BWA BioContainers webpage](https://biocontainers.pro/#/tools/bwa), click on the `Pacakages and Containers` tab, sort by `Version` and get the portion of the link after the `docker pull` command where `Type` is Docker. You may need to double-check that you are using the latest version of the software because you may find that containers for older versions have been rebuilt more recently.

* If the software is available on Conda the software should also be defined using the Nextflow `conda` directive. Software MUST be pinned to the channel (i.e. `bioconda`) and version (i.e. `0.7.17`) e.g. `bioconda::bwa=0.7.17`. Pinning the build too e.g. "bioconda::bwa=0.7.17=h9402c20_2" is not currently a requirement.

* If required, multi-tool containers may also be available on BioContainers e.g. [`bwa` and `samtools`](https://biocontainers.pro/#/tools/mulled-v2-fe8faa35dbf6dc65a0f7f5d4ea12e31a79f73e40). It is also possible for a multi-tool container to be built and added to BioContainers by submitting a pull request on their [`multi-package-containers`](https://github.com/BioContainers/multi-package-containers) repository.

* If the software is not available on Bioconda a `Dockerfile` MUST be provided within the module directory. We will use GitHub Actions to auto-build the containers on the [GitHub Packages registry](https://github.com/features/packages).

### Uploading to `nf-core/modules`

[Fork](https://help.github.com/articles/fork-a-repo/) the `nf-core/modules` repository to your own GitHub account. Within the local clone of your fork add the module file to the [`nf-core/modules/software`](https://github.com/nf-core/modules/tree/master/software) directory.

Commit and push these changes to your local clone on GitHub, and then [create a pull request](https://help.github.com/articles/creating-a-pull-request-from-a-fork/) on the `nf-core/modules` GitHub repo with the appropriate information.

We will be notified automatically when you have created your pull request, and providing that everything adheres to nf-core guidelines we will endeavour to approve your pull request as soon as possible.

## Terminology

The features offered by Nextflow DSL2 can be used in various ways depending on the granularity with which you would like to write pipelines. Please see the listing below for the hierarchy and associated terminology we have decided to use when referring to DSL2 components:

- *Module*: A `process` that can be used within different pipelines and is as atomic as possible i.e. cannot be split into another module. An example of this would be a module file containing the process definition for a single tool such as `FastQC`. At present, this repository has been created to only host atomic module files that should be added to the `software/` directory along with the required documentation and tests.
- *Sub-workflow*: A chain of multiple modules that offer a higher-level of functionality within the context of a pipeline. For example, a sub-workflow to run multiple QC tools with FastQ files as input. Sub-workflows should be shipped with the pipeline implementation and if required they should be shared amongst different pipelines directly from there. As it stands, this repository will not host sub-workflows although this may change in the future since well-written sub-workflows will be the most powerful aspect of DSL2.
- *Workflow*: What DSL1 users would consider an end-to-end pipeline. For example, from one or more inputs to a series of outputs. This can either be implemented using a large monolithic script as with DSL1, or by using a combination of DSL2 individual modules and sub-workflows.

## Help

For further information or help, don't hesitate to get in touch on [Slack `#modules` channel](https://nfcore.slack.com/channels/modules) (you can join with [this invite](https://nf-co.re/join/slack)).

## Citation

If you use the module files in this repository for your analysis please you can cite the `nf-core` publication as follows:

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
> ReadCube: [Full Access Link](https://rdcu.be/b1GjZ)


<!---

#### Publishing results

- The module MUST accept the parameters `params.out_dir` and `params.publish_dir` and MUST publish results into `${params.out_dir}/${params.publish_dir}`.
- The `publishDirMode` MUST be configurable via `params.publish_dir_mode`
- The module MUST accept a parameter `params.publish_results` accepting at least
    - `"none"`, to publish no files at all,
    - a glob pattern which is initalized to a sensible default value.

    It MAY accept `"logs"` to publish relevant log files, or other flags, if applicable.

- To ensure consistent naming, files SHOULD be renamed according to the `$name` variable before returning them.

#### Testing

- Every module MUST be tested by adding a test workflow with a toy dataset.
- Test data MUST be stored within this repo. It is RECOMMENDED to re-use generic files from `tests/data` by symlinking them into the test directory of the module. Specific files MUST be added to the test-directory directly. Test files MUST be kept as tiny as possible.

#### Documentation

- A module MUST be documented in the `meta.yml` file. It MUST document `params`, `input` and `output`. `input` and `output` MUST be a nested list. [Exact detail need to be elaborated. ]


### Configuration and parameters

The module files hosted in this repository define a set of processes for software tools such as `fastqc`, `trimgalore`, `bwa` etc. This allows you to share and add common functionality across multiple pipelines in a modular fashion.

> The definition and standards for module files are still under discussion amongst the community but hopefully, a description should be added here soon!

### Offline usage

If you want to use an existing module file available in `nf-core/modules`, and you're running on a system that has no internet connection, you'll need to download the repository (e.g. `git clone https://github.com/nf-core/modules.git`) and place it in a location that is visible to the file system on which you are running the pipeline. Then run the pipeline by creating a custom config file called e.g. `custom_module.conf` containing the following information:

```bash
include /path/to/downloaded/modules/directory/
```

Then you can run the pipeline by directly passing the additional config file with the `-c` parameter:

```bash
nextflow run /path/to/pipeline/ -c /path/to/custom_module.conf
```

> Note that the nf-core/tools helper package has a `download` command to download all required pipeline
> files + singularity containers + institutional configs + modules in one go for you, to make this process easier.

-->
