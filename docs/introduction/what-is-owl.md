# 🦉 Open Workflow Library (OWL) – Public Summary for OWLstacks

## 📘 What Is OWL?

The **Open Workflow Library (OWL)** is a curated, community-driven collection of scientific workflow examples (called **Quests**), tools, and reusable components. It is built around transparency, reproducibility, and ease of reuse in computational biology and related scientific domains.

* **Website**: [owl.promptable.ai](https://owl.promptable.ai)
* **CLI & SDK**: [OWLkit](#owlkit-developer-toolkit) – the command-line and Python interface for interacting with the library
* **Contact**: [hoot@promptable.ai](mailto:hoot@promptable.ai) – OWL team contact for questions, contributions, or support

## 📂 OWL Repository Structure

Each OWL Quest or project repository follows a simple, minimalistic layout:

```
/README.md         <- Human-facing documentation
/meta.owl          <- YAML metadata file (machine + human readable)
/CWL/              <- Contains .cwl files
/DOCS/             <- Supplementary documentation
/TESTS/            <- Organized testing structure:
                      - /data
                      - /scripts
                      - /utilities
                      - /tasks
```

Additionally, optional `Dockerfile`s and wrapper scripts (`.sh` or `.py`) live alongside their corresponding CWL files, named using a `component-subcomponent` pattern.

## 🧱 Project Types: Quests

OWL uses the concept of **Quests** instead of generic projects. Each Quest is:

* A real-world **case study** or **use case snapshot**
* Structured like a hero’s journey: a mission with clear goals, tools, and outcomes
* Given a mission-style name (e.g., `Mission 3: Variant Voyage`)
* Anonymous but well-documented, with metadata such as:

  * Start/end date
  * Tools used
  * Relevant cloud platforms
  * Contributors (anonymized if requested)
  * Outcome or lessons learned

## 🛠 Tools and Best Practices

* **OWLkit** handles running workflows, inspecting tool metadata, and validating components
* Testing is organized by type to promote modular debug workflows
* Flat `cwl/` directories use consistent naming for easy discovery and sorting

### Naming Conventions

* CWL: `component-subcomponent.cwl`
* Script: `component-subcomponent-script.sh` or `.py`
* Docker: `component.Dockerfile`

## 🧰 OWLkit Developer Toolkit

**OWLkit** is the essential developer toolkit that bridges the gap between complex workflow technologies and practical implementation. It provides a unified interface for managing the entire workflow development lifecycle.

### Core Value Proposition

* **Unified Interface**: Single CLI for Docker, CWL, and Seven Bridges operations
* **Secure Credential Management**: Keyring-based storage with encrypted fallback
* **GitHub Integration**: Native support for GitHub Container Registry with Codespaces authentication
* **Developer Experience**: Rich console output, progress indicators, and intuitive commands
* **Testing Framework**: Integrated CWL validation and workflow execution
* **Cross-Platform**: Works in local development, CI/CD, and cloud environments

### Key Features

#### Docker Operations (`owlkit docker`)
```bash
owlkit docker login ghcr.io                    # Auto-detects GitHub credentials
owlkit docker build -t myimage:latest .        # Build with progress indicators
owlkit docker push ghcr.io/org/image:latest   # Secure push to registries
```

#### CWL Workflow Management (`owlkit cwl`)
```bash
owlkit cwl validate workflow.cwl               # Comprehensive validation
owlkit cwl run workflow.cwl --input file.json  # Execute with rich output
owlkit cwl list outputs/                       # Browse workflow results
```

#### Seven Bridges Integration (`owlkit sbpack`)
*Coming soon* - Direct integration with Seven Bridges platform tooling

### Why OWLkit Matters

1. **Reduces Complexity**: Eliminates the need to learn multiple tool interfaces (docker, cwltool, gdc-client, etc.)
2. **Improves Security**: Handles credentials safely without exposing secrets in shell history
3. **Enhances Productivity**: Provides better UX than raw command-line tools with progress indicators and clear error messages
4. **Ensures Consistency**: Standardizes workflow execution across different environments
5. **Simplifies CI/CD**: Works seamlessly in GitHub Actions, GitLab CI, and other automation platforms

### Real-World Example

Instead of managing complex tool chains manually:
```bash
# Traditional approach - error-prone and verbose
docker login ghcr.io -u $USER -p $TOKEN
docker build -t ghcr.io/org/tool:latest .
docker push ghcr.io/org/tool:latest
cwltool --outdir ./results workflow.cwl inputs.json
```

With OWLkit:
```bash
# Unified, secure, and intuitive
owlkit docker login ghcr.io  # Auto-detects credentials
owlkit docker build -t ghcr.io/org/tool:latest .
owlkit docker push ghcr.io/org/tool:latest
owlkit cwl run workflow.cwl --input inputs.json --output-dir ./results
```

## 🧠 Intelligence Layer

* OWLkit integrates with AI-powered assistance (via **Sage**) for:

  * Suggesting tools
  * Debugging errors
  * Learning from past quests
* AI features are also partially available in air-gapped environments via Jupyter or CLI

## 🌐 Community

* OWL is community-first, with discussion threads, FAQs, and best practices available in OWLstacks
* Public contributions are welcome; each Quest contributes to a growing knowledge base of reusable workflows
