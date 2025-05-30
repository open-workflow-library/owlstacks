# 🦉 Open Workflow Library (OWL) – Public Summary for OWLstacks

## 📘 What Is OWL?

The **Open Workflow Library (OWL)** is a curated, community-driven collection of scientific workflow examples (called **Quests**), tools, and reusable components. It is built around transparency, reproducibility, and ease of reuse in computational biology and related scientific domains.

* **Website**: [owl.promptable.ai](https://owl.promptable.ai)
* **CLI & SDK**: [OWLkit](https://github.com/promptable-ai/owlkit) – the command-line and Python interface for interacting with the library
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

## 🧠 Intelligence Layer

* OWLkit integrates with AI-powered assistance (via **Sage**) for:

  * Suggesting tools
  * Debugging errors
  * Learning from past quests
* AI features are also partially available in air-gapped environments via Jupyter or CLI

## 🌐 Community

* OWL is community-first, with discussion threads, FAQs, and best practices available in OWLstacks
* Public contributions are welcome; each Quest contributes to a growing knowledge base of reusable workflows
