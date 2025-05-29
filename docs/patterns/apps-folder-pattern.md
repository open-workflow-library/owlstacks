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
    dockerPull: ghcr.io/owl/salmon-gpu:1.10.0-cuda12-owl1

x-owl:                          # side‑car metadata (ignored by cwltool)
  dockerfile: salmon-gpu.Dockerfile
  script:     salmon-gpu_quant.sh
```

* The wrapper script **is not** inside the image; OWLKit injects it via an `InitialWorkDirRequirement` at run‑time.
* When you promote the wrapper to a stable release, copy it into the Dockerfile and bump the tag. Older CWLs keep the previous tag.

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
