# LAB 06b: Production Repository Cleanup & Professional Refactoring

## Overview & Context

For this lab, we are shifting our operating format to simulate a production development environment. Your Karolina and I will act as **Senior Code Reviewers**. You will submit your cleanup branch, and we will review issuing explicit code changes or approval requirements based strictly on the checklist below.

---

## Part 1: Architectural Cleanup Guidelines

### Section A: Repository Architecture & File System Hygiene

#### 1. Adhere to the Requested Standard Repository Naming Schema

* **Rationale:** In corporate code ecosystems, discovery and **automated** internal tool indexing rely on naming conventions. Mismatched or erratic repository names break automated dependency tracking and project cataloging.
* **How to Fix:** * Check your repository name on GitHub. Ensure it exactly matches the format designated in your course registration guidelines (e.g., `2605_DS5111_<your_computing_id>`).
* If it is incorrect, navigate to **Settings** -> **Repository Name**, rename it, and update your local git remote tracking using:
```bash
git remote set-url origin git@github.com:USERNAME/CORRECT-REPO-NAME.git

```

#### 2. Enforce Strict `bin/` vs `lib/` Separation and Eliminate Chronological Folders

* **Rationale:** Organizing directories by "Week 1", "Week 2", or "Lab 3" is an anti-pattern. Code environments must be organized by structural role, not by the date they were assigned. Real-world platforms run code from predictable pathways like execution binaries (`bin/`) and shared modules (`lib/`).
* **How to Fix:**
* Move your latest executable scripts (e.g., extraction, enrichment, and analytical runners) directly into the `/bin` directory.
* Move any custom auxiliary modules, helper functions, or business-logic classes into `/lib`.
* Completely delete historical chronological folders (e.g., `week1/`, `lab2/`). Use Git tags if you ever need to reference a specific historical milestone snapshot.
* Verify that your code targets the new layout (e.g., `python bin/extract_transcripts_oop.py`).


#### 3. Eradicate Tracked Runtime Pollution (`__pycache__`, `*.pyc`, `*.log`)

* **Rationale:** Bytecode binaries (`.pyc`), local logging run statements (`.log`), and python optimization folders (`__pycache__`) are localized runtime artifacts. Committing them into source control pollutes code diffs, causes merge conflicts, and threatens system security if sensitive API payloads bleed into local text log files.
* **How to Fix:**
* Purge the files out of your Git tracking index without deleting them from your disk by executing:
```bash
git rm -r --cached $(find . -name "__pycache__" -o -name "*.pyc" -o -name "*.log")

```

* Commit this change immediately to clear your staging history.

#### 4. Check standards into the repo (`.gitignore`, `pylintrc`, `pytest.ini`)

* **Rationale:** Writing configuration files from scratch leads to human error and omission. Global engineering teams rely on verified, community-maintained boilerplates to capture subtle architecture edge cases. For instance, the official GitHub Python template automatically covers platform-specific metadata (`.DS_Store`), IDE settings (`.vscode/`), testing tracking caches (`.pytest_cache/`), and data science footprints (`.ipynb_checkpoints/`) that you would otherwise have to discover through painful experience.
* **How to Fix:**  We already covered how to generate or create the pylintrc and pytest.ini.  For the gitignore, there is much better way to go than creating one from scratch:
* 
1. Open your terminal, navigate to your repository root, and use `curl` to fetch the gold-standard Python template straight from GitHub's upstream engine:
`bash curl -sSL https://raw.githubusercontent.com/github/gitignore/main/Python.gitignore -o .gitignore `
2. Open your newly created `.gitignore` in Vim or your local text editor, scroll to the absolute bottom of the file, and append your pipeline's localized additions to a custom block. For example:
```
.venv/
**/*.log
```

---


### Section B: The Local Automation Layer (Makefile)

#### 5. Explicitly Activate Virtual Environments Inside the Makefile

* **Rationale:** Calling global system bin variables like `python3` or `pip` directly inside a Makefile is highly unstable. It risks running instructions against the host operating system's global environment instead of your localized sandboxed virtual environment.
* **How to Fix:**
* Do not rely on the user having run `source .venv/bin/activate` in their active terminal shell.
* Explicitly route your Makefile target commands directly through your local virtual environment folder variable path:
```makefile
ENV = env
PYTHON = $(VENV)/bin/python3
PIP = $(ENV)/bin/pip

test:
	$(PYTHON) -m pytest tests/
```


#### 6. Transition Linter Execution Targets from Single Files to Full Folders

* **Rationale:** Linting an isolated file leaves gaps in your quality gate. If a developer introduces an unvetted script into a subdirectory, a file-specific linter target will miss it entirely. We lint structural boundaries, not specific individual scripts.
* **How to Fix:**
* Modify your `make lint` target instruction inside your `Makefile` to scan whole directory targets rather than hardcoded file links:
```makefile
lint:
	$(PYTHON) -m pylint bin/ lib/ tests/

```



#### 7. Implement a Zero-Silencing Policy for Quality Gates

* **Rationale:** Appending tricks like `|| true`, `|| exit 0`, or prefixing Makefile instructions with `-` simply to trick a build into showing a green checkmark is a catastrophic practice. It masks active structural code failures, breaking pipeline tracing and code reliability.
* **How to Fix:**
* Audit your `Makefile`. Remove any command manipulation strings that intentionally catch or silence execution return exit codes.
* If a linter rule fails on a structural component that you purposefully designed, cleanly configure that exclusion inside your root `.pylintrc` file, or explicitly use inline python annotations (`# pylint: disable=broad-except`), rather than blind-spot silencing at the shell execution layer.



#### 8. Standardize Your Project's Target Management Contract

* **Rationale:** Every production data platform repository must expose an identical interface for onboarding developers. A new engineer should be able to clone any repository across an enterprise and interact with it using a predictable set of commands.
* **How to Fix:**
* Ensure your `Makefile` exposes at least the following targets:
* `make env` — Bootstraps the local virtual directory environment layout.
* `make update` — Updates project tracking dependencies safely via pip requirements specifications.
* `make lint` — Evaluates style, quality gates, and syntax conventions.
* `make test` — Executes the localized pytest verification harness suite.
* `make run` (or specific sub-script runners) — Executes pipeline steps end-to-end.





---

### Section C: Continuous Integration (GitHub Actions)

#### 9. Delegate All CI/CD Workflow Workloads Direct to the Makefile

* **Rationale:** Hardcoding multi-line shell command sequences directly inside a GitHub Actions configuration YAML creates an automation silo. If a test routine changes, you must modify it across multiple configuration screens. By delegating all execution steps directly to your `Makefile`, your local manual commands match your automated environment identically.
* **How to Fix:**
* Open `.github/workflows/ci.yml`.
* Refactor your execution steps to pass through your Makefile contract rather than directly invoking individual python scripts:
```yaml
- name: Run Code Quality Checks
  run: make lint

- name: Run Complete Test Engine Suite
  run: make test

```





#### 10. Configure Multi-Version Matrix Verification

* **Rationale:** Enterprise data pipelines must maintain stability through upstream runtime migrations. Running checks against only a single specific local Python patch leaves your system exposed to runtime failures when cloud compute providers upgrade their default system images.
* **How to Fix:**
* Update your `ci.yml` build structure to include a `matrix` definition testing across multiple Python versions (e.g., `3.11`, `3.12`, `3.13`):
```yaml
strategy:
  matrix:
    python-version: ["3.11", "3.12", "3.13"]

```


* Ensure your environment step accurately references the active matrix value token: `python-version: ${{ matrix.python-version }}`.



#### 11. Run Linting and Testing Quality Gates in Parallel Workflows

* **Rationale:** In production software lifecycles, engineer compute hours are expensive. Forcing a developer to wait for a 10-minute integration test suite to run before finding out they missed a trailing whitespace lint error slows down velocity. Linting checking and unit execution models are decoupled jobs that should execute in parallel.
* **How to Fix:**
* Deconstruct your singular GitHub Actions job sequence into distinct, isolated named jobs inside your YAML layout: `lint` and `test`.
* Ensure they do not have a sequential `needs:` lock on each other if they do not share operational artifacts, allowing the runner instances to spin up concurrently.

Use this example to parallelize your lint and test jobs in your ci.yml
```yaml
name: Continuous Integration Quality Pipeline

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  # JOB 1: Code Style & Syntax Auditing running in parallel
  lint:
    name: Code Quality Inspection (Linter)
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Runtime Dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      # Pass control directly to the local Makefile interface contract
      - name: Execute Linter Quality Gate
        run: make lint

  # JOB 2: Functional Test Executions running in parallel
  test:
    name: Unit Testing Engine (Pytest)
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12", "3.13"]

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Runtime Dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      # Pass control directly to the local Makefile interface contract
      - name: Execute Automated Test Suite
        run: make test
```



#### 12. Enforce absolute Continuous Integration Integrity (No Hidden Comments)

* **Rationale:** Commenting out flaky tests or linter checks inside your workflow YAML files to force a pull request to merge is a direct violation of continuous integration best practices. If a component is unstable under automation, it must be addressed via code architecture or formal testing flags, never masked in silence.
* **How to Fix:**
* Verify that no steps inside your active structural workflow `.yml` definitions are commented out or configured with parameters like `continue-on-error: true`.
* The GitHub Actions summary page must explicitly evaluate and display green across all active jobs using unmanipulated exit states.

**NB:** This point specifically addresses a lot of commented out steps in some ci.yml files I've seen.  If yours has no commented steps you're good.



---

### Section D: Testing Rigor & Documentation

#### 13. Maintain and Document a Comprehensive, Multi-Type Testing Suite

* **Rationale:** Complex production data engines encounter various external edge cases, such as operating system differences, dependency version shifts, and unpredictable external infrastructure conditions. Your test layout must explicitly prove it can handle these variations using native testing decorators rather than manual intervention.
* **How to Fix:**
* Verify that your `tests/` directory contains all historical pipeline validations.
* Ensure your test suite leverages the following advanced `pytest` features across your test scripts:
* **Parametrization (`@pytest.mark.parametrize`):** To execute a single structural test loop against multiple input data signatures without writing redundant functions.
* **OS/Environment Skips (`@pytest.mark.skipif`):** Cleanly bypass execution steps that require specific local environment configurations (like a Windows vs Linux path).
* **Expected Failures (`@pytest.mark.xfail`):** Mark known edge-case failures or unimplemented components without breaking the build pipeline status.





#### 14. Document Infrastructure Resilience Within Your README

* **Rationale:** A project's `README.md` is an operational runbook, not just a casual introduction. A gold-standard documentation test is **The Disaster Recovery Standard**: *If an AWS EC2 compute instance is completely terminated right now, how long would it take a new engineer using only this documentation to reconstruct the entire stack from scratch and make it fully operational?*
* **How to Fix:**
* Audit your `README.md`. Ensure it cleanly clearly documents:
1. **Project Core Objective:** A concise paragraph explaining exactly what data the pipeline processes and where it lands.
2. **Explicit Bootstrapping Instructions:** Step-by-step terminal instructions to build the local virtual architecture (`make env`, `make update`).
3. **Environment Configuration Variables:** A markdown table defining all required environment configurations, proxy parameters, and keys required to prevent structural crashes.
4. **Verification Steps:** Explicit terminal statements showing a developer how to verify their environment is properly configured via tests and lint execution workflows.





---

## Part 2: Evaluation Rubric

## Part 2: Evaluation Rubric

Every single clean repository element maps directly to 1 point. Your TA will evaluate your PR against this flat rubric.

| Item | Evaluation Criterion | Target Component | Points |
| --- | --- | --- | --- |
| **1** | Repository explicitly conforms to standard course naming convention (`2605_DS5111_<id>`). | Repository Identity | 1 pt |
| **2** | All chronological directories removed; codebase structured purely into `/bin` and `/lib`. | File System Structure | 1 pt |
| **3** | Repository completely purged of tracked runtime pollution (`__pycache__`, `.pyc`, `.log`). | Index Hygiene | 1 pt |
| **4** | Project root contains valid, functional instances of `.gitignore`, `.pylintrc`, and `pytest.ini`. | System Configuration | 1 pt |
| **5** | Makefile abstracts and activates Python virtual components explicitly via explicit environment variables. | Local Automation | 1 pt |
| **6** | Linter rules target entire directory sweeps instead of individual isolated code files. | Local Automation | 1 pt |
| **7** | Zero use of command-silencing mechanisms (like `\|\| true` or `-`) to bypass check errors. | Local Automation | 1 pt |
| **8** | Complete lifecycle command contract exposed (`env`, `update`, `lint`, `test`, `run`). | Local Automation | 1 pt |
| **9** | GitHub Action execution workflow delegates all major operational tasks directly to `make`. | Continuous Integration | 1 pt |
| **10** | Workflow configuration executes matrix runs across multiple explicit versions of Python. | Continuous Integration | 1 pt |
| **11** | Quality gates (`lint` and `test`) are split to run in parallel pipelines. | Continuous Integration | 1 pt |
| **12** | CI workflow contains zero commented-out code steps or failure avoidance flags. | Continuous Integration | 1 pt |
| **13** | Test suite incorporates diverse native test types (`parametrize`, `skipif`, `xfail`). | Validation Rigor | 1 pt |
| **14** | Operational README contains structural project descriptions, environment tables, and setup runbooks. | Project Documentation | 1 pt |
| **Total** | **Minimum Passing Baseline Target: 14 / 14 Points** |  | **14 pts** |
