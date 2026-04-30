# DevOps Engineer Homework — Task D

**Author:** Phone Myint Mo  
**Date:** 2026-04-28  
**Repositories:**

| Repo | Purpose | URL |
|------|---------|-----|
| RepoA | Fork of grpc (C++ source) | https://github.com/EverestPhone/grpc |
| RepoC | Doxygen log parser (Python) | https://github.com/EverestPhone/doxygen-log-parser |
| This repo | Pipelines + README | https://github.com/EverestPhone/devops-homework |

---

## How did you test your pipelines?

### Strategy

Both pipelines were tested iteratively using a local Jenkins instance (Docker container), following this progression:

1. **Stage isolation** — each stage was developed and verified independently before being combined into the full pipeline.
2. **Dry-run validation** — `sh 'doxygen -g Doxyfile'` was run manually on the executor node first to confirm Doxygen was installed and functional before embedding the command in the pipeline.
3. **Config verification** — after the `sed` substitutions in the "Adjust Doxygen Config" stage, the pipeline prints the affected config lines to stdout. This made it easy to confirm the correct values were applied without inspecting the file manually.
4. **Artifact inspection** — after each successful run, `doc.tar.gz` was downloaded from the Jenkins artifacts page and extracted locally to verify the HTML documentation was correctly generated and complete.
5. **Warnings file verification** — in PipelineC, the warnings file path is printed to the console and a `cat` of the CSV output is shown before archiving. This confirms the Python parser ran and produced output.
6. **Failure path testing** — the `html/` directory check (`if [ -d html ]`) was tested by temporarily pointing `INPUT` at a non-existent folder to confirm the pipeline exits with a clear error rather than silently producing an empty archive.
7. **`cleanWs()` post-step** — the workspace is always cleaned after each run to ensure there is no state leak between builds.

### Jenkins setup used

Windows Installation

```powershell
# Download Jenkins MSI from https://www.jenkins.io/download/

# Or use Chocolatey (if installed)
choco install jenkins

# Start Jenkins service
Start-Service Jenkins

# Access Jenkins
Start-Process "http://localhost:8080"
```
## Required Plugins

### Installation Steps

1. Jenkins > Manage Jenkins > Manage Plugins
2. Available tab
3. Search and install:

#### Essential Plugins

```
- Pipeline (workflow-aggregator)
- Git plugin
- GitHub plugin
- GitHub Branch Source
- Docker plugin
- Docker Pipeline
- ws-cleanup
- Timestamper
```

## How did you test RepoC Python?

### Strategy

The parser (`doxygen_parser.py`) was tested using **pytest** with unit tests covering both the parsing logic and CSV output.

Test file: [`test_doxygen_parser.py`](https://github.com/EverestPhone/doxygen-log-parser/blob/main/test_doxygen_parser.py)

Run tests with:

```bash
python -m pytest test_doxygen_parser.py -v

python -m pytest test_doxygen_parser.py --cov=. --cov-report=html
```

testing 
<img width="975" height="421" alt="image" src="https://github.com/user-attachments/assets/077c943c-0ebc-464f-abbf-44e3f6ff9a99" />

python -m pytest test_doxygen_parser.py --cov=. --cov-report=html
<img width="975" height="562" alt="image" src="https://github.com/user-attachments/assets/5e9f520d-6c86-480c-a721-8415c94bfcfb" />

### Test cases covered

| Test | What it verifies |
|------|-----------------|
| `test_standard_warning_with_line_number` | Correctly parses file, line, and message from a full warning line |
| `test_standard_warning_without_line_number` | Handles warnings with no line number (empty `Line` field) |
| `test_non_standard_lines_are_ignored` | Doxygen banners, info lines, and blank lines are silently skipped |
| `test_empty_log_returns_empty_list` | Empty input produces empty output without errors |
| `test_multiple_warnings_parsed` | Multiple warnings in one file are all captured |
| `test_missing_file_raises_exit` | Non-existent file causes a clean `SystemExit` with error message |
| `test_csv_has_correct_headers` | Output CSV always has the correct column headers |
| `test_csv_rows_match_input` | CSV rows match the parsed input exactly |
| `test_empty_records_writes_header_only` | Zero warnings produces a valid CSV with headers only |

### Manual testing

The parser was also tested end-to-end by running PipelineC on the actual grpc fork, collecting the real `doxygen_warnings.log`, and confirming the CSV output was correctly structured and all warnings were accounted for.

---

## RepoA-doc contains binaries — what is the advantage of using Git LFS?

### The problem

Doxygen generates binary files inside the `html/` output — primarily images (`.png`, `.svg`) and compiled search data (`.js` binary blobs). When these are committed directly to a standard Git repository:

- Every version of every binary is stored in the `.git/objects` pack — the repository grows without bound over time.
- `git clone` and `git fetch` always download the full binary history, even if you only need the latest version.
- Diffs and log output become noisy and meaningless for binary content.

### How Git LFS helps

[Git LFS (Large File Storage)](https://git-lfs.github.com/) replaces large binary files in the repository with small **pointer files** (plain text, ~130 bytes). The actual binary content is stored on a separate LFS server and downloaded on demand.

**Key advantages:**

| Concern | Without LFS | With LFS |
|---------|------------|----------|
| Repo clone size | Downloads all binary history | Downloads only pointers; binaries fetched on demand |
| CI/CD speed | Slow — full binary blobs on every checkout | Fast — only current binaries pulled |
| Repository growth | Unbounded — every binary commit adds permanent size | Controlled — old binary versions stay on LFS server |
| Collaboration | Large pushes/pulls slow for all contributors | Efficient for all team members |
| Diffs | Meaningless binary diffs | Clean pointer diffs; content versioned separately |

This is especially important for documentation repositories that are regenerated on every CI run — without LFS, the repo would grow by several MB per pipeline execution.

---

## How to adjust RepoA to support Git LFS?

### Method 1 — Git command line (recommended)

```bash
# 1. Install Git LFS on your machine (once)
git lfs install

# 2. Clone (or navigate into) RepoA
git clone https://github.com/YOUR_USERNAME/grpc.git
cd grpc

# 3. Track the binary file types generated by Doxygen
git lfs track "*.png"
git lfs track "*.svg"
git lfs track "*.js"
git lfs track "doc.tar.gz"

# 4. This creates/updates .gitattributes — commit it
git add .gitattributes
git commit -m "chore: enable Git LFS tracking for Doxygen binary outputs"

# 5. From now on, any push of tracked files goes to LFS automatically
git add html/
git commit -m "docs: add generated Doxygen HTML documentation"
git push origin main
```

Reference: [GitHub Docs — Configuring Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/configuring-git-large-file-storage)

### Method 2 — GitHub web interface (easier alternative)

GitHub allows you to enable LFS per repository without any local CLI setup:

1. Go to your repository on GitHub → **Settings** → **General**
2. Scroll to **Features** and enable **Git LFS**
3. GitHub will prompt you to purchase LFS storage if your files exceed the free tier (1 GB)

This enables the server-side LFS endpoint but you still need to add `.gitattributes` tracking rules locally (step 3–4 above) to tell Git which files to redirect to LFS.

Reference: [GitHub Docs — About Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage)

### Method 3 — BFG Repo Cleaner (retroactive, for existing repos)

If binaries were already committed to a repo without LFS and you want to migrate them retroactively:

```bash
# Migrate existing committed binaries to LFS
git lfs migrate import --include="*.png,*.svg,doc.tar.gz" --everything
git push --force
```

Reference: [Git LFS migrate docs](https://github.com/git-lfs/git-lfs/blob/main/docs/man/git-lfs-migrate.adoc)

### Other (easier) alternatives to LFS

| Alternative | Description | Trade-off |
|------------|-------------|-----------|
| **Jenkins `archiveArtifacts`** | Store `doc.tar.gz` as a Jenkins build artifact instead of committing to Git | Not version-controlled in Git; lives only in Jenkins |
| **GitHub Releases** | Attach `doc.tar.gz` as a release asset via GitHub API or `gh` CLI | Good for tagged releases; not suitable for per-commit docs |
| **GitHub Pages** | Push the `html/` folder to a `gh-pages` branch — GitHub hosts it as a website | Best for live browsable docs; branch grows over time without LFS |
| **S3 / Blob Storage** | Upload artifacts to AWS S3 or Azure Blob from the pipeline | No Git involvement; requires cloud credentials in Jenkins |
| **Nexus / Artifactory** | Enterprise artifact repository — store `doc.tar.gz` with version metadata | Overkill for small projects; ideal for large organisations |


