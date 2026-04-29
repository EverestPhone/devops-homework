# Setup & Testing Guide

**Project:** DevOps Engineer Homework — Doxygen Log Parser Pipeline
**Author:** Phone Myint Mo
**Date:** 2026-04-26

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start](#quick-start)
4. [Phase 1: Local Development](#phase-1-local-development)
5. [Phase 2: GitHub Setup](#phase-2-github-setup)
6. [Phase 3: Jenkins Setup](#phase-3-jenkins-setup)
7. [Unit Testing](#unit-testing)
8. [Integration Testing](#integration-testing)
9. [Pipeline Testing](#pipeline-testing)
10. [Manual Testing Procedures](#manual-testing-procedures)
11. [Performance Testing](#performance-testing)
12. [CI/CD Integration (GitHub Actions)](#cicd-integration-github-actions)
13. [Troubleshooting](#troubleshooting)
14. [Final Validation Checklist](#final-validation-checklist)

---

## Project Overview

This project implements two Jenkins pipelines and a Python-based Doxygen warning log parser:

- **PipelineB** — Clones RepoA (grpc fork), generates Doxygen HTML docs, archives `doc.tar.gz`
- **PipelineC** — Extends PipelineB; captures Doxygen warnings, parses them with the Python parser, and archives a CSV report
- **RepoC** — Python parser (`doxygen_parser.py`) that converts Doxygen warning logs to CSV

### Expected Results

| Component | Result |
|-----------|--------|
| RepoC unit tests | 9/9 passing, coverage > 95% |
| PipelineB | `doc.tar.gz` archived in Jenkins |
| PipelineC | `doc.tar.gz` + `warnings.csv` archived in Jenkins |
| Execution time (PipelineB) | 2–5 minutes |
| Execution time (PipelineC) | 5–10 minutes |

---

## Prerequisites

### Tools Required

| Tool | Version | Install |
|------|---------|---------|
| Python | 3.8+ | https://www.python.org/downloads/ |
| Git | Any recent | https://git-scm.com/ |
| Docker | Any recent | https://www.docker.com/products/docker-desktop |
| Doxygen | Any recent | https://www.doxygen.nl/download.html |
| Jenkins | LTS | https://www.jenkins.io/download/ |

---

## Quick Start

### Linux / Mac

```bash
# Clone the pipeline repo
git clone https://github.com/EverestPhone/devops-homework.git
cd devops-homework

# Set up Python environment for RepoC
cd repoC
pip install -r requirements.txt

# Run all tests
python -m pytest test_doxygen_parser.py -v

# Test with sample data
python doxygen_parser.py sample_warnings.log --output output.csv
```

### Windows

```cmd
git clone https://github.com/EverestPhone/devops-homework.git
cd devops-homework\repoC
pip install -r requirements.txt
python -m pytest test_doxygen_parser.py -v
```

---

## Phase 1: Local Development

### Python Setup

```bash
cd repoC
pip install -r requirements.txt
```

### Verify Parser Works

```bash
# Test with a sample warnings log
python doxygen_parser.py sample_warnings.log --output output.csv
cat output.csv
```

### Code Quality Checklist

- [ ] Python 3.8+ installed
- [ ] Virtual environment created and activated
- [ ] All dependencies installed via `pip install -r requirements.txt`
- [ ] All 9 unit tests passing
- [ ] Coverage > 95%
- [ ] Sample data tested
- [ ] CSV output validated

---

## Phase 2: GitHub Setup

### Repository Structure

You need **3 repositories**:

| Repo | Purpose | Branch |
|------|---------|--------|
| `grpc` (fork) | RepoA — C++ source for Doxygen | `main` |
| `doxygen-log-parser` | RepoC — Python parser | `main` |
| `devops-homework` | Pipelines + README | `main`, `TaskB`, `TaskC`, `TaskD` |

### Branch Structure for Pipeline Repo

```
devops-homework/
├── main          ← README.md
├── TaskB         ← taskB/Jenkinsfile
├── TaskC         ← taskC/Jenkinsfile
└── TaskD         ← taskD/README.md
```


### GitHub Setup Checklist

- [ ] RepoA: grpc repository forked
- [ ] RepoC: `doxygen-log-parser` repository created and code pushed
- [ ] Pipeline repo created with correct branch structure
- [ ] `.gitignore` configured for each repo
- [ ] README.md present in each repo
- [ ] Branch protection rules configured (optional but recommended)

---

## Phase 3: Jenkins Setup

### Running Jenkins in Docker

```bash
# Start Jenkins
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Installing Tools on the Jenkins Agent

```bash
docker exec -u root jenkins bash -c "
  apt-get update &&
  apt-get install -y doxygen python3 python3-pip git
"
```

### Required Jenkins Plugins

| Plugin | Purpose |
|--------|---------|
| `git` | Cloning repositories |
| `workflow-aggregator` (Pipeline) | Declarative pipeline support |
| `archiveArtifacts` | Built-in step, no extra plugin needed |
| `ws-cleanup` | `cleanWs()` post step |
| `GitHub Branch Source` | GitHub webhook integration |

### Jenkins Job Setup

For each pipeline job (PipelineB, PipelineC):

1. **New Item** → **Pipeline**
2. Under **Pipeline** → **Definition**: select `Pipeline script from SCM`
3. SCM: `Git`
4. Repository URL: your `devops-homework` GitHub URL
5. Branch: main
6. Script Path: `taskB/Jenkinsfile` or `taskC/Jenkinsfile`

### Jenkins Checklist

- [ ] Jenkins running on port 8080
- [ ] Doxygen installed on agent
- [ ] Python 3 installed on agent
- [ ] All required plugins installed
- [ ] GitHub credentials configured
- [ ] PipelineB job created and tested
- [ ] PipelineC job created and tested
- [ ] Artifacts visible after each build

---

## Unit Testing

### Running All Tests

```bash
cd repoC

# Basic run
python -m pytest test_doxygen_parser.py -v

# With detailed output
python -m pytest test_doxygen_parser.py -vv

# With coverage report
python -m pytest test_doxygen_parser.py --cov=. --cov-report=term-missing --cov-report=html

# Run specific test class
python -m pytest test_doxygen_parser.py::TestParseWarnings -v
python -m pytest test_doxygen_parser.py::TestWriteCsv -v

# Run a single test
python -m pytest test_doxygen_parser.py::TestParseWarnings::test_standard_warning_with_line_number -v
```

### Expected Output

```
test_doxygen_parser.py::TestParseWarnings::test_standard_warning_with_line_number PASSED
test_doxygen_parser.py::TestParseWarnings::test_standard_warning_without_line_number PASSED
test_doxygen_parser.py::TestParseWarnings::test_non_standard_lines_are_ignored PASSED
test_doxygen_parser.py::TestParseWarnings::test_empty_log_returns_empty_list PASSED
test_doxygen_parser.py::TestParseWarnings::test_multiple_warnings_parsed PASSED
test_doxygen_parser.py::TestParseWarnings::test_missing_file_raises_exit PASSED
test_doxygen_parser.py::TestWriteCsv::test_csv_has_correct_headers PASSED
test_doxygen_parser.py::TestWriteCsv::test_csv_rows_match_input PASSED
test_doxygen_parser.py::TestWriteCsv::test_empty_records_writes_header_only PASSED

============================== 9 passed in 0.06s ==============================
```

### Test Coverage

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

---

## Integration Testing

### Test RepoC with Real Doxygen Output

#### Step 1: Generate Sample Doxygen Warnings

```bash
# Clone RepoA
git clone https://github.com/EverestPhone/grpc.git repoA
cd repoA

# Generate a Doxyfile and set input to src
doxygen -g Doxyfile
sed -i 's/^INPUT.*/INPUT = src/' Doxyfile
sed -i 's/^WARN_FILE.*/WARN_FILE = warnings.log/' Doxyfile

# Run Doxygen and capture warnings
doxygen Doxyfile 2>&1 | tee warnings.log
```

#### Step 2: Run Parser on Real Warnings

```bash
cd ../repoC
python doxygen_parser.py ../repoA/warnings.log --output real_warnings.csv

# Verify
head -20 real_warnings.csv
wc -l real_warnings.csv
```

#### Step 3: Validate CSV Format

```bash
python << 'EOF'
import csv
with open('real_warnings.csv') as f:
    reader = csv.DictReader(f)
    print(f"Headers: {reader.fieldnames}")
    rows = list(reader)
    print(f"Total warnings: {len(rows)}")
    if rows:
        print(f"First record: {rows[0]}")
        print(f"Last record:  {rows[-1]}")
EOF
```

---

## Pipeline Testing

### Simulate PipelineB Locally (No Jenkins Required)

```bash
#!/bin/bash
set -e

WORKSPACE=$(mktemp -d)
cd $WORKSPACE

# Stage 1: Clone RepoA
git clone https://github.com/EverestPhone/grpc.git repoA
cd repoA

# Stage 2: Generate Doxygen config
doxygen -g Doxyfile

# Stage 3: Adjust config
sed -i 's/^INPUT.*/INPUT = src/' Doxyfile
sed -i 's/^GENERATE_HTML.*/GENERATE_HTML = YES/' Doxyfile
sed -i 's/^GENERATE_LATEX.*/GENERATE_LATEX = NO/' Doxyfile
sed -i 's/^GENERATE_MAN.*/GENERATE_MAN = NO/' Doxyfile

# Stage 4: Run Doxygen
doxygen Doxyfile

# Stage 5: Pack output
if [ -d html ]; then
  tar -czf doc.tar.gz html/
  echo "✓ doc.tar.gz created"
else
  echo "✗ html/ directory not found" && exit 1
fi

ls -lh doc.tar.gz
```

### Simulate PipelineC Locally (No Jenkins Required)

```bash
#!/bin/bash
set -e

WORKSPACE=$(mktemp -d)
cd $WORKSPACE

# Run PipelineB stages (as above)
git clone https://github.com/EverestPhone/grpc.git repoA
cd repoA
doxygen -g Doxyfile
sed -i 's/^INPUT.*/INPUT = src/' Doxyfile
sed -i 's/^GENERATE_HTML.*/GENERATE_HTML = YES/' Doxyfile
sed -i 's/^WARN_FILE.*/WARN_FILE = doxygen_warnings.log/' Doxyfile
doxygen Doxyfile

# Clone RepoC and install dependencies
cd ..
git clone https://github.com/EverestPhone/doxygen-log-parser.git repoC
cd repoC
pip install -r requirements.txt

# Run the parser
python doxygen_parser.py ../repoA/doxygen_warnings.log --output warnings.csv

echo "✓ Warnings parsed"
wc -l warnings.csv
head -5 warnings.csv
```

---

## Manual Testing Procedures

### Test 1: Basic Parser Functionality

```bash
cd repoC

cat > test.log << 'EOF'
Doxygen version 1.9.8
Searching for include files...

/src/file.cpp:42: warning: Member not documented.
/src/header.h: warning: Function missing docs.
EOF

python doxygen_parser.py test.log --output test_output.csv
cat test_output.csv
# Expected: header row + 2 warning rows
```

### Test 2: Edge Cases

```bash
# Empty log
echo "" > empty.log
python doxygen_parser.py empty.log --output empty_out.csv
# Expected: header row only, 0 warnings

# Non-standard lines only
cat > nostandard.log << 'EOF'
Doxygen version 1.9.8
Searching...
Processing...
Done
EOF
python doxygen_parser.py nostandard.log --output nostandard_out.csv
# Expected: header row only, 0 warnings

# Mixed content
cat > mixed.log << 'EOF'
Doxygen version 1.9.8
/src/a.cpp:10: warning: Missing docs
Searching for files...
/src/b.h:20: warning: Undocumented member
Processing complete
EOF
python doxygen_parser.py mixed.log --output mixed_out.csv
# Expected: header row + 2 warning rows
```

### Test 3: Error Handling

```bash
# Non-existent file
python doxygen_parser.py /nonexistent/file.log 2>&1
# Expected: exit code 1, clear error message printed

# UTF-8 content
echo "Line with UTF-8: ñ, é, ü" > utf8.log
echo "/src/file.cpp:1: warning: Test" >> utf8.log
python doxygen_parser.py utf8.log --output utf8_out.csv
# Expected: parses successfully without encoding errors
```

---

## Performance Testing

```bash
cd repoC

python << 'EOF'
import time
from doxygen_parser import parse_warnings

# Create a large test file
with open('large_test.log', 'w') as f:
    f.write("Doxygen version 1.9.8\n")
    for i in range(10000):
        f.write(f"/src/file{i}.cpp:{i}: warning: Test warning {i}\n")

start = time.time()
records = parse_warnings('large_test.log')
elapsed = time.time() - start

print(f"Parsed {len(records)} warnings in {elapsed:.3f}s")
print(f"Average: {1000*elapsed/len(records):.3f}ms per warning")
EOF
```

**Expected performance benchmarks:**

| Warnings | Expected Time |
|----------|--------------|
| 1,000 | ~10ms |
| 10,000 | ~100ms |
| 100,000 | ~1,000ms |

---

## CI/CD Integration (GitHub Actions)

Create `.github/workflows/test.yml` in RepoC:

```yaml
name: Test RepoC

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.8', '3.9', '3.10', '3.11', '3.12']

    steps:
    - uses: actions/checkout@v3

    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run tests with coverage
      run: python -m pytest test_doxygen_parser.py -v --cov=. --cov-report=xml

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage.xml
```

---

## Troubleshooting

### `ModuleNotFoundError: No module named 'pytest'`

```bash
pip install pytest
# or
pip install -r requirements.txt
```

### `FileNotFoundError: Log file not found`

```bash
# Verify the file exists
ls -la warnings.log

# Use an absolute path
python doxygen_parser.py /absolute/path/to/warnings.log --output output.csv
```

### `UnicodeDecodeError`

```bash
# Convert file to UTF-8 first
iconv -f ISO-8859-1 -t UTF-8 old_file.log > new_file.log
python doxygen_parser.py new_file.log --output output.csv
```

### `AssertionError` in pytest

```bash
# Run with full diff output to see exactly what failed
python -m pytest test_doxygen_parser.py -vv --tb=long

# Isolate a single failing test
python -m pytest test_doxygen_parser.py::TestParseWarnings::test_standard_warning_with_line_number -vv
```

### `Permission denied` on output CSV

```bash
chmod 644 warnings.csv
```

### Jenkins — Build Not Triggering from GitHub Push

1. Verify the webhook is set in GitHub: **Repo → Settings → Webhooks**
2. Payload URL: `http://<your-jenkins-host>:8080/github-webhook/`
3. Content type: `application/json`
4. Events: **Just the push event**
5. Check Jenkins **Manage Jenkins → System Log** for webhook delivery errors

### Jenkins — `doxygen: command not found`

```bash
# Install on the Jenkins agent
docker exec -u root jenkins apt-get install -y doxygen

# Verify
docker exec jenkins doxygen --version
```

---

## Final Validation Checklist

### RepoC (Python Parser)
- [ ] All 9 unit tests pass (`9 passed in X.XXs`)
- [ ] Coverage >= 95%
- [ ] No warnings or errors in test output
- [ ] Sample log parses correctly to CSV
- [ ] Empty log produces header-only CSV
- [ ] Missing file triggers clean `SystemExit`
- [ ] Performance: 1,000 warnings parsed in < 100ms

### PipelineB
- [ ] RepoA cloned successfully
- [ ] Doxygen config generated with `doxygen -g`
- [ ] `INPUT` set to `src`, only HTML output enabled
- [ ] `html/` directory created after Doxygen run
- [ ] `doc.tar.gz` archived in Jenkins artifacts
- [ ] Workspace cleaned after build (`cleanWs()`)

### PipelineC
- [ ] All PipelineB stages pass
- [ ] `WARN_FILE` configured in Doxygen config
- [ ] `doxygen_warnings.log` generated
- [ ] RepoC cloned and dependencies installed
- [ ] Python parser runs without errors
- [ ] `warnings.csv` archived in Jenkins artifacts

### GitHub
- [ ] 3 repositories public and accessible
- [ ] RepoA: fork of grpc
- [ ] RepoC: `doxygen-log-parser` with code and tests
- [ ] Pipeline repo: correct branch structure (`main`, `TaskB`, `TaskC`, `TaskD`)
- [ ] README.md present in each repo

---

## External References

- Jenkins: https://www.jenkins.io/doc/
- Doxygen configuration: https://www.doxygen.nl/manual/config.html
- GitHub Docs: https://docs.github.com/
- Python pytest: https://docs.pytest.org/
- Git LFS: https://git-lfs.github.com/
