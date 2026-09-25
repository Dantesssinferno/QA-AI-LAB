# QA-AI-LAB

AI-assisted QA copilot for requirements analysis and test-design planning.

Repository: https://github.com/Dantesssinferno/QA-AI-LAB

## What the project does

QA-AI-LAB is a local proof-of-concept built with n8n, PostgreSQL and OpenRouter LLMs.

The current workflow accepts a software task or requirement description and processes it through several QA-oriented stages:

1. requirement extraction
2. requirement normalization and deterministic metrics
3. requirement review report
4. HTML dashboard generation
5. test-design strategy selection

The project focuses on requirements quality and test-design assistance. It is not presented as a replacement for a QA Engineer.

## Current capabilities

- Requirement analysis and structured extraction
- Separation of business, functional, non-functional and implicit requirements
- Extraction of acceptance criteria, business rules, dependencies and constraints
- Detection of assumptions, ambiguities, missing information and risks
- Role-specific clarification questions for Analyst, Product Owner, Developer, Designer and DevOps
- Requirement quality metrics:
  - requirements maturity
  - confidence
  - completeness
  - testability
- Recommendation for exploratory testing and manual review
- HTML requirement-review dashboard
- Test Design Advisor for selecting applicable test-design techniques
- Requirement coverage matrix for the selected strategy
- Estimated testing effort/check volume
- Structured JSON between AI stages

## Planned / not connected in the current workflow

The workflow report already contains placeholders for future stages such as:

- Checklist Generator
- Test Case Generator
- API Test Generator
- SQL Test Generator
- Security Test Generator
- Performance Test Generator

These are not connected execution nodes in the current workflow. The project is a working proof-of-concept for a larger QA automation pipeline.

---

## Architecture

```text
Manual Trigger
      |
      v
   User Input
      |
      v
Requirement Prompt Builder
      |
      v
Requirement Extractor <--- Main LLM
      |                    <--- Conversation Memory
      |                    <--- Structured Output Parser
      v
Requirement Normalizer
      |
      v
Requirement Review Report
      |
      +--------------------------+
      |                          |
      v                          v
 HTML Dashboard            Test Design Advisor
                                   ^
                                   |
                          OpenRouter Chat Model
                          Structured Output Parser
```

Workflow file:

`n8n/workflows/requirement-analysis-workflow.json`

---

## Tech stack

- n8n - workflow orchestration
- PostgreSQL 16 - n8n database
- Docker / Docker Compose - local deployment
- OpenRouter - LLM provider
- GPT-4.1-mini - model shown in `.env.example`

No local Node.js or Python installation is required for the core workflow because n8n and PostgreSQL run inside Docker.

---

# Installation

## Prerequisites

Install:

1. Docker Desktop with Docker Compose support
2. Git
3. An OpenRouter API key

## 1. Clone the repository

```bash
git clone https://github.com/Dantesssinferno/QA-AI-LAB.git
cd QA-AI-LAB
```

## 2. Create .env

Linux/macOS:

```bash
cp .env.example .env
```

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Configure at least:

```env
OPENROUTER_API_KEY=YOUR_OPENROUTER_API_KEY
OPENROUTER_MODEL=openai/gpt-4.1-mini

POSTGRES_USER=n8n
POSTGRES_PASSWORD=change_me
POSTGRES_DB=n8n

N8N_PORT=5678
N8N_HOST=localhost
TZ=Europe/Chisinau
```

The current `docker-compose.yml` references `N8N_HOST` and `TZ`, so include them in a clean deployment.

Never publish a real API key.

## 3. Start the stack

```bash
docker compose up -d
```

Check the services:

```bash
docker compose ps
```

Expected containers:

- qa-ai-postgres
- qa-ai-n8n

## 4. Open n8n

Open:

http://localhost:5678

On first run, complete the n8n owner-account setup.

---

# Import the workflow

The workflow is stored at:

```text
n8n/workflows/requirement-analysis-workflow.json
```

In n8n:

1. Open Workflows.
2. Select Import from File.
3. Choose `n8n/workflows/requirement-analysis-workflow.json`.
4. Open the imported workflow.
5. Configure the OpenRouter credential in both OpenRouter nodes if it is not resolved automatically.

Credential IDs are instance-specific. Do not copy credentials from another n8n installation.

---

# Configure OpenRouter

Two nodes use OpenRouter:

- `Main LLM`
- `OpenRouter Chat Model`

Create an OpenRouter credential in n8n and assign it to both nodes.

The repository example configuration uses:

```text
openai/gpt-4.1-mini
```

Check the imported node configuration for the actual model selection. n8n stores node and credential settings inside the workflow/runtime.

---

# How to enter a task

The current workflow starts with a Manual Trigger.

The task text is stored in the `task` field of:

`User Input`

## Step-by-step

1. Open the `User Input` node.
2. Find the `task` field.
3. Replace its content with your requirement or task description.
4. Save the workflow.
5. Click Execute Workflow.

You can paste either a structured specification or plain product text.

A full example is included in:

`docs/examples/task-01.md`

---

# Node-by-node explanation

## Start

**Type:** Manual Trigger

Workflow entry point. Starts execution manually from the n8n editor.

## User Input

**Type:** Set

Stores the input task in the `task` field.

This is the main place to paste a new requirement.

## Requirement Prompt Builder

**Type:** Set

Builds the instruction passed to the first AI stage.

The prompt tells the model to act as a senior QA analyst, extract information without inventing facts, identify ambiguities and missing information, and generate clarification questions for relevant roles.

## Main LLM

**Type:** OpenRouter Chat Model

Provides the language model for requirement analysis.

## Conversation Memory

**Type:** Buffer Window Memory

Provides recent conversational context to the Requirement Extractor.

The current workflow uses a limited memory window.

## Structured Output Parser

**Type:** Structured Output Parser

Defines the JSON structure expected from the Requirement Extractor.

This allows downstream nodes to consume structured fields instead of parsing free-form prose.

## Requirement Extractor

**Type:** AI Agent

Primary requirements-analysis stage.

It extracts:

- business requirements
- functional requirements
- non-functional requirements
- implicit requirements
- acceptance criteria
- business rules
- dependencies
- constraints
- assumptions
- ambiguities
- missing information
- role-specific questions
- risks
- test focus

## Requirement Normalizer (Метрики, Статистика)

**Type:** Code

Transforms the extractor output into normalized data and calculates deterministic metrics.

Current calculations include:

- counts by requirement category
- total requirement count
- confidence score
- completeness score
- testability score
- requirements maturity
- recommendation for exploratory testing
- recommendation for manual review

The formulas are implemented in JavaScript in this node.

## Requirement Review Report (Агрегация, Рекомендации)

**Type:** Code

Aggregates the normalized output into one QA-oriented report.

It groups:

- quality metrics
- requirements
- findings
- assumptions
- ambiguities
- missing information
- risks
- role-specific questions
- test focus
- pipeline status
- recommendations

The report is passed to both the dashboard and the Test Design Advisor.

## HTML Dashboard

**Type:** Code

Generates an HTML review dashboard from the structured report.

The current dashboard renders:

- quality metrics
- requirement categories
- assumptions
- ambiguities
- missing information
- risks
- questions by role
- testing focus
- pipeline status
- recommendations

It is generated inside the workflow and is not a separate frontend application.

## Test Design Advisor

**Type:** AI Agent

Does not generate test cases.

Its task is to choose a practical test-design strategy based on the reviewed requirements.

The prompt limits recommendations to a defined set of techniques, including:

- EP
- BVA
- Decision Table
- State Transition
- Use Case Testing
- Cause-Effect Graphing
- Error Guessing
- Pairwise
- Orthogonal Array
- CRUD
- Workflow
- Role-Based
- Permission Matrix
- API Contract
- Database Integrity
- Negative
- Exploratory
- Risk-Based

For each recommended technique, the prompt requests rationale, priority, confidence, linked requirement IDs and estimated checks. It also requires a coverage matrix.

## OpenRouter Chat Model

**Type:** OpenRouter Chat Model

Provides the model for the Test Design Advisor.

## Structured Output Parser1

**Type:** Structured Output Parser

Defines the JSON structure expected from the Test Design Advisor.

---

# What the pipeline does

The workflow is intentionally staged instead of using one large prompt.

### 1. Understand the requirement

The first agent extracts what is explicitly specified and separates it from:

- assumptions
- ambiguities
- missing information
- risks

It also creates clarification questions for different project roles.

### 2. Measure requirement quality

The Code node calculates normalized metrics from the structured result.

This stage is deterministic rather than asking the LLM to invent a quality score.

### 3. Build a QA review

The review-report node aggregates the findings into one report that can be inspected before test design.

The goal is to answer both:

> What should QA test?

and:

> Is the requirement complete and testable enough to design tests?

### 4. Choose a test-design strategy

The Test Design Advisor maps requirements to suitable test-design techniques and explains the reason for each choice.

The prompt explicitly tells the model to avoid recommending techniques merely for the sake of completeness.

---

# Example

The repository contains a complete example in:

`docs/examples/task-01.md`

It describes:

- registration
- password recovery
- validation rules
- password policy
- REST API endpoints
- password reset token lifecycle
- database requirements
- security requirements
- logging
- acceptance criteria

This example is useful for demonstrating the end-to-end requirement-analysis pipeline.

---

# Example of the kind of questions the agent can surface

For a password-recovery specification, the pipeline can surface questions such as:

- What happens when more than one reset token exists?
- What is the exact behavior after token expiration?
- Can a used token be submitted again?
- Are all active sessions terminated after a password change?
- What response is returned for an existing versus non-existing email?
- What is the exact rate-limit behavior?
- Which component sends the email?
- What information must be logged?

These questions are generated from the requirement supplied to the workflow rather than being a fixed hard-coded questionnaire.

---

# Why structured output matters

LLMs naturally return free-form text.

QA-AI-LAB forces the major AI stages into schemas so that the workflow can pass a predictable object between nodes:

```text
Requirement Extractor
        |
        v
Structured JSON
        |
        v
Deterministic normalization
        |
        v
Requirement Review Report
        |
        v
Test Design Advisor
```

This is useful for a larger QA automation pipeline because later stages can consume the same structured data instead of parsing another prose response.

---

# Current limitations

## Not yet a full test-generation platform

The connected workflow currently ends with requirement review and test-design strategy.

Future generators are represented in reports/prompts, but are not wired into the current execution graph.

## AI output still needs QA review

The system assists a QA Engineer. Generated assumptions, risks and strategy recommendations should be checked against the actual product, architecture and business rules.

## Local proof-of-concept deployment

The Docker Compose stack is intended for local development and experimentation.

A production deployment would need additional work around security, secrets management, access control, observability, persistence and model/provider reliability.

---

# Repository structure

```text
QA-AI-LAB/
├── .env.example
├── docker-compose.yml
├── README.md
├── docs/
│   ├── examples/
│   │   └── task-01.md
│   └── prompts/
│       ├── bug-generator.md
│       ├── checklist-generator.md
│       ├── requirement-analyzer.md
│       ├── risk-analyzer.md
│       └── testcase-generator.md
├── n8n/
│   └── workflows/
│       └── requirement-analysis-workflow.json
└── backup/
    └── ...
```

The `docs/prompts` files are drafts for future extensions. A prompt file does not mean that the corresponding generator is currently connected to the n8n workflow.

---

# Troubleshooting

## AI stage fails

Check:

- OpenRouter credential exists in n8n
- API key is valid
- selected model is available
- account quota/credits are sufficient
- requested output fits the model/node limits

## PostgreSQL connection problems

Run:

```bash
docker compose ps
docker compose logs postgres
docker compose logs n8n
```

Then verify the PostgreSQL values in `.env`.

## Port 5678 is busy

Change:

```env
N8N_PORT=5678
```

to another host port, for example:

```env
N8N_PORT=5679
```

Then open:

http://localhost:5679

## Existing local data causes startup problems

The repository can contain local development runtime data.

For a reproducible fresh deployment, use a clean project copy and treat n8n/PostgreSQL runtime directories as environment-specific state.

---

# Security notes

- Never commit `.env` with a real API key.
- Never publish API keys in screenshots or documentation.
- Do not reuse credential IDs from another n8n installation.
- Treat runtime data and logs as local environment state.
- For public distribution, prefer a clean deployment layout over sharing personal runtime artifacts.

---

# Roadmap

The current architecture is ready for downstream QA stages:

```text
Requirement Review
        |
        v
Test Design Advisor
        |
        +--> Checklist Generator
        +--> Test Case Generator
        +--> API Test Generator
        +--> SQL Test Generator
        +--> Security Test Generator
        +--> Performance Test Generator
```

The repository already contains prompt drafts for several of these directions.

---

# Project idea

QA-AI-LAB explores a practical question:

**How can AI reduce repetitive requirement-analysis work for QA without removing the QA Engineer from the decision-making loop?**

The current answer is a staged pipeline:

**requirements -> quality review -> risks and questions -> test-design strategy**

The purpose is to move repetitive analysis into a reusable workflow while keeping test decisions, risk assessment and product context with the QA Engineer.

---

## Author

**Maxim Starostenko**

QA Engineer · Manual QA · API / Backend Testing · iGaming

GitHub: https://github.com/Dantesssinferno

Project: https://github.com/Dantesssinferno/QA-AI-LAB
