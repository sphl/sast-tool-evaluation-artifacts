# sast-tool-evaluation-artifacts

Artifacts for the paper: ***An Empirical Study on the Effectiveness of Static C Code Analyzers for Vulnerability Detection***

## Publication

<a href="https://raw.githubusercontent.com/sphl/sast-tool-evaluation-artifacts/main/paper_preprint.pdf"><img src="https://raw.githubusercontent.com/sphl/sast-tool-evaluation-artifacts/main/paper_thumbnail.png" align="right" width="280"></a>

- [Stephan Lipp](https://www.in.tum.de/i04/lipp/) (TU Munich)
- [Sebastian Banescu](https://github.com/banescusebi) (TU Munich)
- [Alexander Pretschner](https://www.in.tum.de/i04/pretschner/) (TU Munich)

```bibtex
@inproceedings{SASTEval2022,
    title     = {An Empirical Study on the Effectiveness of Static C Code Analyzers for Vulnerability Detection},
    author    = {Lipp, Stephan and Banescu, Sebastian and Pretschner, Alexander},
    year      = {2022},
    booktitle = {Proceedings of the ACM SIGSOFT International Symposium on Software Testing and Analysis},
    series    = {ISSTA'22},
    doi       = {10.1145/3533767.3534380},
    numpages  = {12}
}
```

### Overview

Static code analysis is often used to scan source code for security vulnerabilities. Given the wide range of existing solutions implementing different analysis techniques, it is very challenging to perform an objective comparison between static analysis tools to determine which ones are most effective at detecting vulnerabilities. Existing studies are thereby limited in that (1) they use synthetic datasets, whose vulnerabilities do not reflect the complexity of security bugs that can be found in practice and/or (2) they do not provide differentiated analyses w.r.t. the types of vulnerabilities output by the static analyzers. Hence, their conclusions about an analyzer's capability to detect vulnerabilities may not generalize to real-world programs.

We propose a methodology for automatically evaluating the effectiveness of static code analyzers based on CVE reports. We evaluate five free and open-source and one commercial static C code analyzer(s) against 27 software projects containing a total of 1.15 million lines of code and 192 vulnerabilities (ground truth). While static C analyzers have been shown to perform well in benchmarks with synthetic bugs, our results indicate that state-of-the-art tools miss in-between 47% and 80% of the vulnerabilities in a benchmark set of real-world programs. Moreover, our study finds that this false negative rate can be reduced to 30% to 69% when combining the results of static analyzers, at the cost of 15 percentage points more functions flagged. Many vulnerabilities hence remain undetected, especially those beyond the classical memory-related security bugs.

## Getting Started

### Required Software

The main evaluation of this work is made available in the form of an [R](https://www.r-project.org/about.html) Jupyter notebook called `analysis.ipynb`. To run this notebook, we recommend using [JupyterLab](https://jupyterlab.readthedocs.io/en/stable/getting_started/overview.html), a web-based interactive development environment for R (and Python) code. JupyterLab can be installed either via different package management tools (cf. the instructions [here](https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html)) or---as in this tutorial---via [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/), which provide ready-to-use Docker images containing Jupyter applications. For this, make sure you have Docker installed on your machine; if not, a detailed documentation explaining how to install Docker on various operating systems, including Windows, macOS, and GNU/Linux, can be found [here](https://runnable.com/docker/getting-started/).

### Installing and Running JupyterLab

To run the evaluation in `analysis.ipynb` using JupyterLab, follow the steps below.

#### Step 1: Creating a JupyterLab Docker Container

Run the command

```bash
docker run -p 8888:8888 -v <artefact-path>:/home/jovyan/work jupyter/r-notebook:latest
```

to download the latest `jupyter/r-notebook` image from [Docker Hub](https://hub.docker.com/r/jupyter/scipy-notebook).

Note that `<artefact-path>` must be replaced with the absolute path to the directory containing the artifacts. This directory will then be mounted to `/home/jovyan/work` within the Docker container.

Moreover, after downloading the image, a container with a Jupyter server is automatically started, with the container's internal port 8888 exposed to the same port on the host machine.

#### Step 2: Loading the `analysis.ipynb` Notebook

Launch JupyterLab by entering `http://<hostname>:8888/?token=<token>` (printed to `stdout`) in your web browser, where

- `<hostname>` is the name of the machine running Docker and

- `<token>` is the secret server token.

Next, navigate to the directory containing the artifacts in the file browser (left pane) and open the `analysis.ipynb` notebook.

#### Step 3: Installing Required R Packages

In the notebook, first run

```r
install.packages("pacman")
library(pacman)`
```

to install the R package manager [pacman](https://www.rdocumentation.org/packages/pacman/versions/0.5.1). Secondly, execute

```r
pacman::p_load(stringr, dplyr, tidyr, tidytext, ggplot2, ggtext, RColorBrewer)
```

to install and load all required R packages.

**Now, everything is set up to (re-)run the evaluation in the Jupyter notebook!**

## Detailed Description

The artifacts described in this section are intended to foster further comparable and evidence-based studies in the field of static code analysis.

### Static Code Analysis

#### Selected Analyzers

| **Analyzer** | **Version** | **CLI Options**                                                                                      |
|--------------|-------------|------------------------------------------------------------------------------------------------------|
| Flawfinder   | 2.0.11      | `--dataonly`, `--neverignore`, `--minlevel=2`, `--falsepositive`                                     |
| Cppcheck     | 2.3         | `--force`, `--enable=warning`                                                                        |
| Infer        | 0.14.0      | `--keep-going`, `--biabduction`, `--bufferoverrun`, `--liveness`, `--quandary`, `--siof`, `--uninit` |
| CodeChecker  | 6.12.0      |                                                                                                      |
| CodeQL       | 2.1.3       | `--language=cpp`                                                                                     |
| CommSCA      | -           | -                                                                                                    |

The queries that we used for CodeQL in this evaluation can be found in the query suite `codeql_query_pack/cpp/ql/src/codeql-suites/cpp-eval.qls`.

CommSCA is the only commercial tool used in this study. To protect the company behind this static analyzer, we anonymized its name and did not reveal the exact version and release date, nor the analysis technique implemented.

#### CWE Mapping and Grouping

`cwe_mapping` contains the following JSON files:

- `analyzers/{flawfinder,cppcheck,infer,codeql,codechecker}.json`: Different analysis tools use different names for the vuln. types they support. This makes it difficult to automatically assess if a tool refers to the correct vulnerability. Therefore, the respective files map each analyzer-specific vuln. identifier to the corresponding analyzer-agnostic [CWE](https://cwe.mitre.org/) ID.

- `buckets.json`: Many vuln. types are closely related. This means, comparing only low-level types would lead to a biased evaluation, as analyzers that do not output the exact, but related CWE IDs would be penalized. Therefore, we leverage the existing CWE hierarchy (stored in this file) to group related vuln. types into classes.

### Benchmark Dataset

`dataset` contains the raw benchmark data used in this study. For each benchmark program, i.e. *Binutils*, *FFmpeg*, *libpng*, *LibTIFF*, *Libxml2*, *OpenSSL*, *PHP*, *Poppler*, and *SQLite3*, the following artifacts are provided:

- `cve_data[.json]`: The directory contains detailed information about each vulnerability ([CVEs](https://cve.mitre.org/); queried with the [cve-search](https://github.com/cve-search/cve-search) tool) contained in the benchmark program (ground truth). To facilitate the evaluation, this data is summarized in the same-named JSON file.

- `functions.json`: Contains information, such as filename, line range, and lines of code (LoC), of every function in the target program. It is used to check if a line marked by a static analyzer corresponds to a vulnerable function (see `cve_data.json`).

- `sca_results.json`: Contains the outputs/flags of the employed static analyzer executed on the benchmark program. Each flag includes the line number of the marked instruction in the code, as well as the mapped vuln. type (CWE) output by the respective analyzer.

### Evaluation Script and Data

The CSV files listed below summarize the extensive benchmark data for convenient processing in the `analysis.ipynb` notebook.

- `data/cwe_distr.csv`: Contains for each benchmark program the number of included vulnerabilities (CVEs) per vuln. type/class (CWE). It is used to analyze the vulnerability distribution in our benchmark dataset.

- `data/fct_stats_1.csv`: Contains specific information about the functions in the benchmark programs, such as the source file containing the function, its lines of code (LoC), and if it contains a vulnerability, or not. It is employed to study certain characteristics of vulnerable functions.

- `data/fct_stats_2.csv`: Contains for each employed static analyzer (and combinations thereof) the ratio of marked functions per benchmark program. It is used to evaluate the trade-off between the ratio of detected vulnerabilities and flagged functions.

- `data/sca_data_1.csv`: Contains for each static analyzer (alone/combined) and evaluation scenario (cf. paper) the number of vulnerabilities detected per benchmark program. It is employed to analyze the effectiveness of the static analyzers/tool combinations.

- `data/sca_data_2.csv`: Contains for each static analyzer (and their combinations) the number of CWE-specific vulnerabilities detected across all target programs. It is used to analyze which vuln. types were detected best and worst by the employed analysis tools.