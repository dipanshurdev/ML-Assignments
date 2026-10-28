# FL-01: AI Workflow Audit & Tool Setup

- **Author:** Dipanshu Rawat
- **Track:** General AI Fluency (Week 1)
- **Role:** Machine Learning Engineering Intern (FlyRank)
- **Repository:** [https://github.com/dipanshurdev/ML-Assignments](https://github.com/dipanshurdev/ML-Assignments)

---

## 1. Executive Summary & Purpose

This workflow audit establishes an operational baseline for integrating artificial intelligence into my daily engineering, research, and academic routines. Based on Ethan Mollick's *"On-boarding your AI Intern"* framework, this audit maps cognitive tasks into four interaction tiers:
1. **Just Me:** Non-negotiable human ownership where AI use degrades learning, ethics, or authenticity.
2. **Delegate to AI with Review:** Structured tasks where AI generates first-pass drafts and human verifies accuracy.
3. **Collaborate with AI (Centaur/Cyborg):** Interactive pairing on complex, multi-modal problem solving.
4. **Fully Automate:** Deterministic, programmatic tasks governed by code and CI/CD without prompt overhead.

---

## 2. Weekly Workflow Audit Table (12 Real Recurring Tasks)

| # | Recurring Task Description | Context / Frequency | Interaction Tier | One-Line Rationale & Operational Boundary |
|---|---|---|---|---|
| **1** | **Initial Mathematical Concept Derivation** (e.g., deriving loss functions, backprop math, Gini impurity by hand) | Study / Weekly | **Just Me** | Relying on AI shortcuts prevents neural encoding of first-principles mathematical intuition; foundational learning must remain purely personal. |
| **2** | **High-Stakes Mentor & 1-on-1 Feedback Syncs** (e.g., career alignment meetings, performance reviews, retrospective syncs) | Professional / Weekly | **Just Me** | Authentic human connection, interpersonal empathy, and accountability cannot be outsourced; synthetic communication erodes team trust. |
| **3** | **Exploratory Data Analysis (EDA) Boilerplate** (e.g., checking null counts, dataframe shapes, computing baseline summary statistics) | Work / Daily | **Delegate to AI with Review** | Highly repetitive Pandas/SQL syntax where AI accelerates execution; human reviews distribution shapes and data type integrity. |
| **4** | **Drafting Technical Documentation & Docstrings** (e.g., writing docstrings for production pipeline functions, README setup steps) | Work / 3x week | **Delegate to AI with Review** | Standardized markdown/docstring formatting where LLMs excel; human verifies parameter contracts and edge cases before committing. |
| **5** | **Drafting Professional Status Updates & Emails** (e.g., weekly internship progress reports to coordinators) | Professional / Weekly | **Delegate to AI with Review** | AI organizes bullet points and enhances clarity; human validates factual veracity and injects personal voice. |
| **6** | **Debugging Complex Logic & Race Conditions** (e.g., memory exhaustion in DuckDB, multi-threading deadlocks) | Work / 2x week | **Collaborate with AI** | Human provides runtime traces and architecture constraints; AI acts as an interactive rubber-duck partner exploring root causes. |
| **7** | **System Architecture & Trade-Off Brainstorming** (e.g., batch inference vs. real-time scoring, DuckDB vs. Polars) | Work / Weekly | **Collaborate with AI** | Centaur-style exploration: AI outlines alternative trade-offs (cost, latency, complexity); human decides based on infrastructure bounds. |
| **8** | **Academic Paper Synthesis & Literature Triage** (e.g., reviewing new papers in Information Retrieval, KDD, NeurIPS) | Research / Weekly | **Collaborate with AI** | AI extracts core problem statements and experimental splits; human critically audits baseline fairness and leakage vulnerabilities. |
| **9** | **Refactoring & Code Style Harmonization** (e.g., converting procedural scripts into modular functions with type hints) | Work / 3x week | **Collaborate with AI** | Human specifies interface boundaries; AI generates clean, idiomatic Python patterns adhering to PEP 8 standards. |
| **10** | **Continuous Integration Smoke Tests & Linting** (e.g., flake8, black, CI dataset leak guards on pull requests) | DevOps / Daily | **Fully Automate** | Deterministic syntax and security rules enforced automatically on git push without manual prompt or human review overhead. |
| **11** | **Automated Query Benchmark Telemetry** (e.g., recording query latency and byte-scanned metrics to JSON receipts) | Work / Daily | **Fully Automate** | Programmatic pipeline telemetry that should be written automatically by scripts without manual intervention. |
| **12** | **Daily Git Commit & Artifact Syncing** (e.g., pushing build artifacts and generated figures to remote GitHub branches) | Work / Daily | **Fully Automate** | Mechanical pipeline step executed via background scripts once validation test suites pass. |

---

## 3. Three Target Tasks for In-Depth Focus (FL-02 Through FL-04)

The following three tasks are selected for systematic optimization and evaluation in subsequent modules:

### Target Task 1: Academic Research & Methodological Auditing (Tier: Collaborate)
- **Domain Context:** Evaluating published research papers and technical reports in machine learning, information retrieval, and search intelligence.
- **Measurable "Done Well" Definition:**
  1. Correctly identifies the exact dataset grain and whether the target label is an observed outcome or a defined heuristic rule.
  2. Identifies potential target/feature leakage (e.g., post-event click data leaking into pre-event predictions) within 3 minutes of initial paper ingestion.
  3. Formulates at least 3 concrete methodological audit questions evaluating baseline fairness and cross-domain validation splits.

### Target Task 2: Python / SQL Code Refactoring & Robustness Profiling (Tier: Collaborate)
- **Domain Context:** Translating exploratory notebook code into modular, production-ready pipeline components.
- **Measurable "Done Well" Definition:**
  1. Refactored code achieves 100% type hint coverage (`mypy` compliant) and PEP 8 conformity with zero functional regressions.
  2. Implements explicit defensive handling for all real-world edge cases (division-by-zero on zero impressions, missing parquet paths, empty partitions).
  3. Reduces execution time or memory footprint by at least 15% through vectorized Pandas/DuckDB operations.

### Target Task 3: Technical Executive Communication & Action Playbooks (Tier: Delegate w/ Review)
- **Domain Context:** Translating complex mathematical model predictions into actionable decision queues for non-technical content editors.
- **Measurable "Done Well" Definition:**
  1. Produces a 1-page executive summary under 300 words without machine learning jargon (focuses on decisions, actions, and ROI).
  2. Includes explicit "What NOT to Automate" operational guardrails to prevent damaging critical digital assets.
  3. Every recommended intervention is paired with a transparent reason code and an expected traffic recovery metric.

---

## 4. Claude Project Configuration & Custom Instructions

A dedicated Project has been created in Claude for engineering and research co-intelligence:

- **Project Name:** `ML Research & Engineering Co-Pilot`
- **Project Purpose:** Advanced pairing on machine learning pipelines, research auditing, and technical writing.
- **Custom Project Instructions (System Prompt):**

```text
You are an expert Machine Learning Staff Engineer and Technical Research Partner pairing with Dipanshu Rawat, an ML Engineering Intern at FlyRank working on search intelligence and production ranking pipelines.

USER CONTEXT & PROFILE:
- Name: Dipanshu Rawat
- Primary Tech Stack: Python 3.11, DuckDB, Pandas, Scikit-Learn, PyTorch, SQL, Hugging Face Parquet streaming, Git.
- Current Focus: Production ML, ranking anomaly detection, honest validation design (client-holdout splits), and translating model outputs into operational editorial playbooks.

TONE & COLLABORATION PREFERENCES:
1. Rigorous & Direct: Avoid sycophancy, excessive flattery, or generic boilerplate. Get straight to the technical insight or code.
2. First-Principles Thinking: Prioritize why a mathematical formulation or architecture fits the problem before writing code.
3. Code Quality: Write idiomatic, robust, production-safe Python with explicit type annotations, defensive null handling, and zero data leakage.
4. Honest Limitations: When reviewing claims or models, aggressively audit for target leakage, baseline fairness, and observational vs causal boundaries.
5. Formatting: Use structured markdown, GitHub-style tables, and concise code blocks.

GOAL:
Collaborate on accelerating research, auditing methodology, and building clean, reproducible ML pipelines that solve real enterprise search problems.
```

---

## 5. Free Tool Stack & Academy Enrollment Verification

| Tool / Platform | Account Status | Evidence & Configuration Details |
|---|---|---|
| **Claude (Anthropic)** | **Active & Configured** | Free account active; Project created (`ML Research & Engineering Co-Pilot`) with custom instructions configured in Section 4. |
| **ChatGPT (OpenAI)** | **Active** | Free tier active for comparative prompting, cross-validation, and multi-model consensus testing. |
| **Anthropic Academy** | **Enrolled** | Enrolled in *AI Fluency: Framework & Foundations* via Skilljar; completed Module 1 (*Introduction to AI Collaboration & Ethical Guardrails*). |

---
