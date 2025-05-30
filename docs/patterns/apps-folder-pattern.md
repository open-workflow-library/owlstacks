# 🦉 OWL “Apps” Folder Pattern — *Flat, Prefix‑Based, Side‑Car*

> The quick score: **one flat `apps/` directory, file names tell you everything, no sub‑folders, no embedded scripts/images.**  Works for humans *and* lets OWLKit (or another LLM) reason about Docker reuse, script injection, and CI automation.

---

## 1  Key Rules in One Glance

| 📏 Rule                                                                                                           | Why it matters                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Flat structure** — every CWL, Dockerfile, and wrapper script lives directly under `apps/`.                      | Simplest possible path logic: `apps/*.cwl` catches all tools; no nested globs.                                                        |
| **Prefix everything** with the software **package** (`tool`) and, if needed, a short **flavour**.                 | Makes it trivial to match files that belong together.                                                                                 |
| **Hyphen (`-`) separates the flavour**, underscore (`_`) separates the tool step.                                 | In ASCII sort, `-` (<code>0x2d</code>) comes *before* the dot in `.Dockerfile`, keeping all Dockerfiles on top of their family block. |
| **Dockerfile names:** `<tool>-<flavour>.Dockerfile` (e.g. `salmon-gpu.Dockerfile`, `salmon-standard.Dockerfile`). | OWLKit builds one image per flavour and deduplicates matching recipes.                                                                |
| **CWL + script names:** `<tool>-<flavour>_<step>.cwl`, `<tool>-<flavour>_<step>.<sh/py>`.                         | Sorting groups each tool family, then each step alphabetically.                                                                       |
| **Side‑car pointers** inside CWL:                                                                                 |                                                                                                                                       |

````yaml
x-owl:
  dockerfile: salmon-gpu.Dockerfile
  script:     salmon-gpu_quant.sh
``` | Standard CWL ignores these keys; OWLKit uses them to inject the script at run‑time and to trace Docker lineage. |
| **All sidecar scripts must support `--help`** and document their usage, parameters, and purpose.               | Enables self-documenting workflows and tool introspection for users and automated systems.                       |

---

## 2  Canonical Example Directory

```text
apps/
├─ fastqc.Dockerfile            # CPU‑only FastQC image
├─ fastqc.cwl
├─ fastqc.sh
├─ salmon-gpu.Dockerfile        # CUDA 12 image (GPU flavour)
├─ salmon-gpu_quant.cwl
├─ salmon-gpu_quant.sh
├─ salmon-standard.Dockerfile   # CPU‑only image (standard flavour)
├─ salmon-standard_index.cwl
├─ salmon-standard_index.sh
├─ salmon-standard_quant.cwl
└─ salmon-standard_quant.sh
````

*ASCII/byte sort ensures all `salmon-*` artefacts stay together, with Dockerfiles listed first in each family block.*

---

## 3  Inside a CWL (excerpt)

```yaml
cwlVersion: v1.2
class: CommandLineTool
baseCommand: [bash, salmon-gpu_quant.sh]

inputs:
  reads_1: File
  reads_2: File?
  index_dir: Directory

outputs:
  quant_dir:
    type: Directory
    outputBinding: {glob: "quant_out"}

requirements:
  DockerRequirement:
    dockerPull: ghcr.io/open-workflow-library/salmon-gpu:1.10.0-cuda12-owl1

x-owl:                          # side‑car metadata (ignored by cwltool)
  dockerfile: salmon-gpu.Dockerfile
  script:     salmon-gpu_quant.sh
```

* The wrapper script **is not** inside the image; OWLKit injects it via an `InitialWorkDirRequirement` at run‑time.
* When you promote the wrapper to a stable release, copy it into the Dockerfile and bump the tag. Older CWLs keep the previous tag.

### Docker Registry Convention
- **OWL Standard Registry**: `ghcr.io/open-workflow-library/<tool>:latest`
- **Use owlkit for publishing**: `owlkit docker build -f <tool>.Dockerfile -t <tool>:latest -c . --push -u open-workflow-library`
- **Access Required**: GitHub token with write permissions to open-workflow-library organization

---

## 4  CI / Validation Snippet

```yaml
# .github/workflows/ci.yml
name: Validate CWL + miniature tests
on: [push, pull_request]

jobs:
  cwl:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install cwltool cwltest schema-salad
      - name: Validate all CWL files
        run: |
          for cwl in apps/*.cwl; do
            cwltool --validate "$cwl";
          done
      - name: Run tests
        run: cwltest --tool cwltool --test tests/tasks/*.yml
```

Trigger heavyweight image rebuilds only on demand:

```bash
owlkit docker build --update   # builds any Dockerfile not yet in GHCR
```

---

## 5  Guidelines & Gotchas

* **Pick a flavour naming convention once** (`standard`, `gpu`, `dev`, `slim`) and document it in `README.md`.
* Never use `latest` tags in `DockerRequirement`—pin by hash or semantic tag so builds are reproducible.
* Keep test data < 1 MiB in `tests/data/`; large references should be downloaded at runtime.
* If two Dockerfiles hash to the same digest, OWLKit will PR a switch to a shared `dockerPull` tag.
* For ad‑hoc debugging you can still run:

  ```bash
  cwltool --singularity apps/salmon-gpu_quant.cwl salmon_gpu_quant_job.yml
  ```

  because the side‑car script is injected automatically.

---

## 5  Best‑Practice Checklist

| ✅ Do                                                                                    | 🚫 Don’t                                     |
| --------------------------------------------------------------------------------------- | -------------------------------------------- |
| **Pin images by semantic tag or digest** (`ghcr.io/owl/salmon-gpu:1.10.0-cuda12-owl1`). | Use `latest` — breaks reproducibility.       |
| Keep *one* Dockerfile per flavour; reuse it across many CWLs.                           | Rebuild a new image for every CWL step.      |
| Keep test data ≤ 1 MiB in `tests/data/`; fetch big references at runtime.               | Commit 100 MB FASTQs or BAMs into the repo.  |
| Run `owlkit docker build --update` nightly or manually.                                 | Trigger heavy image builds on every push.    |
| Document flavour meaning in `README.md` for contributors.                               | Assume newcomers know which variant to pick. |
| Use hyphen (`-`) for flavour, underscore (`_`) for step.                                | Mix delimiters or change naming mid‑project. |

---

## 6  FAQ

| Question                                        | Quick answer                                                                                          |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Why do Dockerfiles sort to the top?**         | Capital **D** sorts before lowercase; hyphen sorts before the dot, so `*-*.Dockerfile` appears first. |
| **Can a single CWL reference two Dockerfiles?** | No—one `DockerRequirement` per tool. Split into separate CWL tools if needed.                         |
| **Where do common helper scripts live?**        | Store once (e.g., `salmon_common.sh`) and reference via `$include` or `x-owl.script`.                 |
| **How does OWLKit deduplicate images?**         | It hashes each Dockerfile; identical hashes get one build and unified `dockerPull` PRs.               |
| **Is embedding still allowed?**                 | Yes, but the side‑car pattern keeps files smaller and CI faster.                                      |
| **What about Singularity/Apptainer?**           | Call `cwltool --singularity`; the side‑car script is injected the same way.                           |

---

### TL;DR for an LLM

1. **Detect family** via filename prefix up to first dot: `fastqc`, `salmon-gpu`, `salmon-standard`.
2. **Map** each CWL to its `x-owl.dockerfile` and `x-owl.script`.
3. **Infer reuse**: all CWLs referencing the *same* Dockerfile form a build unit.
4. **Deduplicate** identical Dockerfile hashes across families.
5. **Inject** scripts via `InitialWorkDirRequirement` before container start.

This pattern gives maximum clarity for humans, deterministic grouping for tooling, and zero friction for rapid workflow development in OWL.

---

## Sidecar Script Standards

### Help Documentation Requirement

**Every sidecar script (`.sh`, `.py`, etc.) MUST implement `--help` support** that provides:

- **Purpose**: What the script does
- **Usage**: Command syntax and examples  
- **Parameters**: All options, flags, and arguments
- **Input/Output**: Expected file formats and locations
- **Exit codes**: Success (0) and error conditions

#### Example Implementation

**Bash Script (`gdc_upload.sh`)**:
```bash
#!/bin/bash

show_help() {
    cat << EOF
GDC Uploader - Upload genomic files to NIH Genomic Data Commons

USAGE:
    gdc_upload.sh [OPTIONS] FILES_DIRECTORY

DESCRIPTION:
    Uploads genomic sequence files to the GDC using parallel processing.
    Requires GDC metadata JSON and authentication token.

OPTIONS:
    -m, --metadata FILE     Path to GDC metadata JSON file (required)
    -t, --token FILE        Path to GDC authentication token file (required)
    -j, --threads N         Number of parallel upload threads (default: 4)
    -r, --retries N         Number of retry attempts for failed uploads (default: 3)
    -h, --help              Show this help message

EXAMPLES:
    gdc_upload.sh -m metadata.json -t token.txt -j 8 /data/files/
    gdc_upload.sh --metadata meta.json --token auth.txt --retries 5 ./sequence-files/

EXIT CODES:
    0    Success - all files uploaded
    1    Error - missing required parameters or files
    2    Error - upload failures after retries
    3    Error - invalid metadata or authentication

DEPENDENCIES:
    - gdc-client (GDC Data Transfer Tool)
    - GNU parallel
    - jq (JSON processor)
EOF
}

# Parse arguments
case "$1" in
    -h|--help)
        show_help
        exit 0
        ;;
esac
```

**Python Script (`yaml2json.py`)**:
```python
#!/usr/bin/env python3

import argparse
import sys

def main():
    parser = argparse.ArgumentParser(
        description="Convert YAML metadata to JSON format for GDC uploads",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
EXAMPLES:
    yaml2json.py input.yaml output.json
    yaml2json.py --validate metadata.yaml result.json
    cat input.yaml | yaml2json.py - output.json

EXIT CODES:
    0    Success - conversion completed
    1    Error - invalid YAML format  
    2    Error - file I/O error
    3    Error - validation failed
        """
    )
    
    parser.add_argument('input', help='Input YAML file (use "-" for stdin)')
    parser.add_argument('output', help='Output JSON file (use "-" for stdout)')
    parser.add_argument('--validate', action='store_true', 
                       help='Validate converted JSON against GDC schema')
    parser.add_argument('--pretty', action='store_true',
                       help='Pretty-print JSON output with indentation')
    
    args = parser.parse_args()
    # ... implementation
```

### Documentation Integration

The help output should be **automatically extracted and included in tool documentation**:

1. **CWL `doc` field**: Include a summary that references `--help` for full details
2. **README sections**: Extract key usage examples from script help
3. **API documentation**: Generate parameter tables from help output

#### CWL Documentation Example

```yaml
cwlVersion: v1.2
class: CommandLineTool
label: "GDC Uploader"
doc: |
  Upload genomic files to NIH Genomic Data Commons with parallel processing.
  
  This tool manages batch uploads of BAM/FASTQ files using the GDC Data Transfer Tool.
  Supports retry logic, parallel execution, and comprehensive error reporting.
  
  For detailed usage information, run: gdc_upload.sh --help
  
  Key features:
  - Parallel upload processing (configurable thread count)
  - Automatic retry for failed uploads  
  - MD5 checksum validation
  - Comprehensive upload reporting

# NOTE: x-owl metadata will be added once namespace support is implemented
# x-owl:
#   script: gdc_upload.sh
#   help-command: "gdc_upload.sh --help"
```

### Best Practice Checklist for Help Documentation

| ✅ Do | 🚫 Don't |
|--------|----------|
| **Always implement `--help` and `-h`** for every script | Assume users will read source code for usage |
| **Include real usage examples** with actual parameter values | Provide only generic syntax without examples |
| **Document all exit codes** and their meanings | Exit with random codes without explanation |
| **Reference dependencies** and version requirements | Assume all dependencies are always available |
| **Use consistent format** across all scripts in a project | Mix different help formats between scripts |
| **Test help output** as part of CI validation | Let help text become outdated or broken |
