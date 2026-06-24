# QLStat

Analyze batches of real-world projects with declarative static analysis provided by CodeQL for empirical study and statistical analysis, gaining insights into patterns in real-world projects.

## Features Overview

QLStat provides a comprehensive framework for large-scale empirical analysis of software projects using CodeQL. Key features include:

- **Batch Processing**: Clone, build, and analyze multiple repositories in parallel
- **Flexible Configuration**: YAML-based configuration for defining analysis targets and parameters
- **Extensible Analysis**: Support for custom external predicates (e.g., escape analysis data)
- **Scalable Query Execution**: Parallel execution of CodeQL queries across repositories
- **Comprehensive Logging**: Detailed logging at each stage of the analysis pipeline
- **Data Collection**: Aggregation of results from multiple repositories into unified datasets
- **Language Support**: Currently focused on Go, with extensibility for other languages supported by CodeQL

## Setup

- Install [CodeQL CLI](https://docs.github.com/en/code-security/how-tos/scan-code-for-vulnerabilities/scan-from-the-command-line/setting-up-the-codeql-cli#1-download-the-codeql-cli-tar-archive) and add codeql to your PATH.
- Install [Go](https://go.dev/dl/).

## Demo

- [`demo.sh`](./demo.sh): A demo script to run QLStat on a sample configuration file.
  - Results are in `./codeqlResult/escape_ext/heapvar_should_move`. You will find optimization variables allocated in heap.
- [`demo.yaml`](./demo.yaml): The sample configuration file demonstrating the usage of QLStat.

## Usage

### 1. Configuration

Create your `stat.yaml` config file according to [`example.yaml`](./example.yaml), [`demo.yaml`](./demo.yaml) or YAML files in [`yaml-examples/`](./yaml-examples/). The configuration supports several key sections:

- `repositories`: Define repositories with URL prefixes and optional directory layout
- `language`: Specify the programming language for analysis (e.g., go)
- `buildGrps`: Configure build groups with timeout, build commands, and optional extgen scripts
- `queryconfig`: Set up query execution with parallelization options
- `queryGrps`: Define query groups with specific queries and target repositories

### 2. Database Creation

Run `go run ./cmd/batch_clone_build stat.yaml` to clone repositories and create CodeQL databases:

```bash
go run ./cmd/batch_clone_build stat.yaml
```

Key options:
- `-noclone`: Skip cloning if repositories already exist
- `-nobuild`: Skip database creation if databases already exist
- `-noextgen`: Skip generation of external predicates

The tool supports three main phases:
1. **Cloning**: Download repositories from specified sources
2. **Building**: Create CodeQL databases using appropriate build commands
3. **External Predicate Generation**: Generate additional data sources like escape analysis results

### 3. Query Development

Create your queries in the [`qlsrc`](./qlsrc/) directory. Queries should follow CodeQL conventions and can leverage external predicates when needed.

### 4. Query Execution

Run `go run ./cmd/codeql_qdriver -collect stat.yaml` to execute queries on the created databases:

```bash
go run ./cmd/codeql_qdriver -collect stat.yaml
```

Available options:
- `-format`: Specify output format (text, csv, json, bqrs) - default: csv
- `-decode-only`: Only decode existing bqrs files without running queries
- `-collect`: Collect all CSV results into a single file with repository names

Results are processed in three stages:
1. **Query Execution**: Run CodeQL queries on each database
2. **Decoding**: Convert bqrs results to specified format (CSV, JSON, etc.)
3. **Collection**: Aggregate results from all repositories into a single dataset

## Extensions

### Go Escape Analysis Extension

QLStat supports extending CodeQL with escape analysis data through the escape adapter:

1. Configure `extgenScript: goescape` in a build group in your YAML (where `goescape` runs `go build -a -gcflags=all=-m=2 .`).
   - You can also specify your own script, as long as it generates `m2.log` in `$logRoot/extgen/path/to/repo/m2.log`.
2. This generates escape analysis data during the build phase.
3. Reference the external predicate in your query group with `externals: [movedToHeap, newEscapesToHeap]`.
4. Use the external predicate in your CodeQL queries.

For more details about how the escape analysis extension works, see [Escape Analysis Documentation](doc/adapters/escape_analysis.md).

## Architecture

For detailed information about the storage structure and architecture, please refer to the [Architecture Documentation](doc/arch.md).

# Contribution

Contributions are welcome! If you encounter any issues while using QLStat, or have ideas for improvements, feel free to submit an [Issue](https://github.com/Lslightly/QLStat/issues) or open a [Pull Request](https://github.com/Lslightly/QLStat/pulls).

# Citation

```bibtex
@software{Li_QLStat,
    author = {Li, Qingwei and Ding, Boyao and Zhang, Yu and Chen, Jinbao},
    license = {Apache-2.0},
    title = {{QLStat}},
    url = {https://github.com/s4plus/QLStat}
}

@article{li2026empiricalMemPerfSafetyGo,
    title = {Go语言程序的内存性能与安全问题实证研究},
    author = {李清伟 and 丁伯尧 and 张昱 and 陈金宝},
    journal = {软件学报},
    volume = {37},
    number = {3},
    pages = {1197},
    numpages = {28},
    year = {2026},
    doi = {10.13328/j.cnki.jos.007464},
    publisher = {科学出版社}
}
```
