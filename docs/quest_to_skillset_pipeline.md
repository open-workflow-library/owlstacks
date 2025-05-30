# 📚 From Quest to Skill Set: The OWLStacks Pipeline

**Location:** `owlstacks/docs/quest_to_skillset_pipeline.md`

---

## 🧭 Overview

This document outlines how OWLStacks transforms individual, generative AI-assisted quests into structured skill sets. These skill sets form the foundation for adaptive learning, runtime augmentation, and community recognition within the OWL ecosystem.

Quests may be completed entirely by AI agents such as ClaudeCode, or may be guided and assisted by human experts. The framework is designed to support both autonomous and collaborative workflows.

This document outlines how OWLStacks transforms individual, generative AI-assisted quests into structured skill sets. These skill sets form the foundation for adaptive learning, runtime augmentation, and community recognition within the OWL ecosystem.

---

## 🧱 1. Quest Structure and Metadata

Each quest is its own GitHub repository and follows a standardized structure optimized for generative agents (e.g., ClaudeCode) and human collaborators:

```bash
quest-example-upload-gdc/
├── .quest/
│   ├── meta.yaml               # Core metadata and guidance for the agent
│   ├── goals.md                # Natural-language mission description
│   ├── state.yaml              # Tracks current step, last change, agent notes
├── CWL/                        # Required CWL tool(s) and/or workflow(s)
│   ├── upload.cwl
│   └── inputs.json             # Optional example inputs
├── docs/
│   ├── prompt_seed.md         # Starting prompt and assumptions
│   ├── FAQ.md                 # Clarifications for agent and human users
│   └── notes.md               # Inline observations during generation
├── tasks/
│   ├── outputs/               # Logs, generated results, evaluations
│   ├── tests/                 # CWL conformance tests or validations
│   └── report.md              # Summary of progress or test coverage
├── README.md                  # Dynamic guide for quest execution
└── .gitattributes             # Optional: lock branch, normalize diffs
```

### Example `.quest/meta.yaml`

```yaml
title: "Automate GDC Uploads"
quest_type: "workflow_creation"
status: "in_progress"
tagged_domains: ["data upload", "genomics"]
llm_guidance: "Start by reading goals.md and prompt_seed.md. Work through CWL creation. Log status in state.yaml and results in tasks/outputs."
```

### Example `.quest/state.yaml`

```yaml
current_step: "Define CWL inputs/outputs"
agent_notes: "Considering separating metadata fetch from upload step."
last_updated: 2025-07-30T14:12:00Z
```

---

## 🧠 2. Quest Clustering into Skill Sets

### Methods:

* Use natural language embeddings (e.g., OpenAI, Cohere)
* Cluster by tag, embedding, or graph similarity (e.g., Neo4j)

### Skill Set Registry Entry (YAML)

```yaml
skill_set: "genomic_upload_automation"
quests:
  - quest-gdc-upload
  - quest-dbGaP-sync
  - quest-s3-harmonize
tags: ["API", "manifest", "GDC"]
modules:
  - gdc_uploader
  - metadata_validator
```

---

## 🧪 3. Training Data Generation

From the clustered quests:

* Extract prompt-response pairs
* Harvest successful CWL patterns
* Log examples and test outcomes

Stored in:

```bash
skill_sets/genomic_upload_automation/examples.md
```

Example entry:

```yaml
- input: "Create a CWL tool to validate metadata TSV for GDC"
  output: "tool: metadata_validator.cwl..."
```

---

## 🤖 4. Building More Agency into the Quest

A core driver of success in generative quests is the strength of the initial prompt. The `docs/prompt_seed.md` file plays a critical role in this process — it is the **first document the agent reads and the starting point for action**. It should:

* Translate the high-level goal into an agent-friendly instruction
* Anchor the agent's initial reasoning, assumptions, and decisions
* Set expectations for autonomy, pacing, and self-evaluation

### ✨ Prompt Seed Importance

The `prompt_seed.md` is not just a convenience — it is **mission-critical**. A high-quality seed prompt:

* Enables the agent to take initiative with clarity
* Frames the problem in domain-specific language
* Encourages modular, testable work patterns

### 📌 Example `prompt_seed.md`

```markdown
You are an expert in bioinformatics workflow engineering.

Your quest is to define a CWL tool that validates GDC metadata TSV files before upload. Your goal is to:
- Inspect the structure of the TSV
- Validate required fields
- Emit a report in JSON format

Start by defining the CWL inputs/outputs, then work step-by-step. Log your progress in `state.yaml`. Validate with a test case in `tasks/tests/`.
```

To encourage ClaudeCode or similar agents to work through quests autonomously over 1–2 hours:

### a. **State Tracking**

Use `.quest/state.yaml` to:

* Record progress step-by-step
* Add thoughts or forks in logic
* Persist across long sessions or timeouts

### b. **Prompt Chaining**

* `docs/prompt_seed.md` initializes the first agent prompt
* Future prompts can be triggered based on `state.yaml` or feedback loops

### c. **Agent Roles and Behavior**

Define in `meta.yaml`:

```yaml
agent_role: "workflow engineer"
prompt_style: "iterative and cautious, test frequently"
time_budget_minutes: 120
```

### d. **Encouraged Behaviors**

* Use `tasks/tests/` to validate outputs
* Log decisions in `docs/notes.md`
* Self-review in `tasks/report.md`
* Use checkpoints in `state.yaml` every 15–30 minutes

---

## 🚀 5. Runtime Skill Set Activation

During live inference or quest generation, Sage can:

* Detect relevant skill set from user prompt
* Load matching CWL tools, test templates, and examples
* Apply tuned prompt templates from `docs/`

---

## 🏅 6. Feedback and Advancement

As quests are completed and reviewed:

* Reputation scores increase per skill set
* Badges, medals, and relics are awarded
* Contributors and AI agents gain persistent skill attribution

### Feedback signals:

* `tasks/report.md` summarizing outcomes
* Manual PR reviews or votes
* Conformance testing

---

## 📂 Where to Store This Document

Location within OWLStacks:

```bash
owlstacks/docs/quest_to_skillset_pipeline.md
```

This document guides both humans and AI agents through how OWL operationalizes structured knowledge accumulation, using reproducible CWL-only quests, dynamic feedback loops, and a modular, generative template.

---

## ✅ Next Steps

* Create a GitHub template repo with the new structure
* Add `.quest/` scaffolding generator to `owlKit`
* Enable ClaudeCode and other agents to read `state.yaml` and adapt behavior
* Auto-classify new quests into skill sets based on embeddings
