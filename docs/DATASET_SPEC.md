# Dataset Architecture Prompt — AI Incident Investigator

We are building the dataset layer for an AI agent project called **AI Incident Investigator**.

The system is an autonomous DevOps troubleshooting agent that investigates incidents using:

* logs
* metrics
* commit history
* deployment events
* operational runbooks

The agent follows a ReAct-style investigation workflow and generates evidence-backed Root Cause Analysis (RCA) reports.

The dataset is NOT just sample data.

It is an evaluation framework used to:

* benchmark RCA accuracy
* evaluate tool selection
* test evidence correlation
* validate confidence scoring
* replay investigations

The dataset structure should feel similar to a benchmark/evaluation suite used for AI agents.

---

# IMPORTANT CONCEPT

In real production systems:

* CloudWatch logs
* Datadog metrics
* GitHub commits
* deployments

exist naturally.

But evaluation-specific files like:

* expected_rca.json
* metadata.json
* timeline.json

do NOT exist naturally.

We are intentionally creating them because this project needs:

* evaluation
* benchmarking
* reproducibility
* replayability
* agent scoring

This is similar to how:

* RAG systems use benchmark datasets
* SWE-bench evaluates coding agents
* GAIA evaluates tool-using agents

The dataset acts as:

* historical incident replay
* ground truth benchmark
* agent evaluation framework

---

# DATASET GOALS

The dataset should:

* simulate realistic production incidents
* require multi-source reasoning
* contain noisy evidence
* avoid trivial root causes
* support agent evaluation

Root causes should NOT be obvious from logs alone.

The agent should need to correlate:

* logs
* metrics
* commits
* deployments
* runbooks

to identify the RCA.

---

# DATASET DIFFICULTY LEVELS

The incidents are divided into:

## Easy

Single-service failures.

Examples:

* DB connection pool exhaustion
* failed deployment

Usually requires:

* 2–3 tools

---

## Medium

Cross-system reasoning.

Examples:

* memory leak
* RDS saturation
* ECS scaling issue

Usually requires:

* 3–4 evidence sources

---

## Hard

Distributed or cascading failures.

Examples:

* reconnect storm
* retry amplification
* cascading voice outage

Requires:

* multiple hypotheses
* conflicting evidence
* timeline reasoning
* distributed system understanding

---

# REQUIRED FOLDER STRUCTURE

data/

├── incidents/
│
│   ├── easy/
│   │
│   │   ├── db_pool_exhaustion/
│   │   │
│   │   ├── metadata.json
│   │   ├── logs.json
│   │   ├── metrics.json
│   │   ├── commits.json
│   │   ├── deployments.json
│   │   ├── expected_rca.json
│   │   └── timeline.json
│   │
│   │   └── failed_deployment/
│   │
│   ├── medium/
│   │
│   │   ├── memory_leak/
│   │   └── rds_saturation/
│   │
│   └── hard/
│       │
│       ├── cascading_voice_outage/
│       └── retry_storm/
│
├── runbooks/
│
│   ├── database_timeout.md
│   ├── memory_leak.md
│   ├── deployment_failure.md
│   ├── ecs_crash.md
│   └── dependency_outage.md
│
└── evaluations/
│
├── benchmark_easy.json
├── benchmark_medium.json
└── benchmark_hard.json

---

# FILE PURPOSES

## metadata.json

Defines incident metadata.

Contains:

* id
* title
* difficulty
* service
* severity
* summary
* tags

Example purpose:

* dataset organization
* filtering
* benchmarking
* UI rendering

---

## logs.json

Represents runtime application/system logs.

Should contain:

* timestamps
* levels
* service names
* realistic errors
* noise

The logs should:

* contain useful clues
* contain irrelevant logs/noise
* NOT explicitly reveal the root cause

---

## metrics.json

Represents infrastructure/system metrics.

Can contain:

* CPU
* memory
* latency
* error rates
* DB connections
* queue depth

Metrics should:

* support or contradict hypotheses
* correlate with logs/timeline

---

## commits.json

Represents recent code/configuration changes.

Should contain:

* commit IDs
* timestamps
* commit messages
* changed files

Commit messages should:

* hint at issues
* NOT explicitly state the root cause

---

## deployments.json

Represents operational deployment events.

Should contain:

* deployment timestamp
* service name
* version
* deployment status

This allows the agent to correlate:
deployment → incident timing

---

## expected_rca.json

MOST IMPORTANT FILE.

This is the evaluation ground truth.

Should contain:

* expected root cause
* expected confidence
* evidence sources
* recommended actions
* expected investigation path

This file enables:

* RCA accuracy scoring
* investigation quality evaluation
* tool usage evaluation

---

## timeline.json

Chronological reconstruction of the incident.

Used for:

* replay mode
* UI timeline
* debugging
* temporal reasoning

Should contain:

* ordered incident events
* timestamps
* escalation sequence

---

# RUNBOOK REQUIREMENTS

The runbooks folder contains operational documentation for RAG retrieval.

Each runbook should include:

* symptoms
* diagnostics
* common causes
* recommended actions

Runbooks should feel like internal engineering documentation.

NOT prompts.

Example topics:

* database timeouts
* memory leaks
* ECS crashes
* deployment failures
* dependency outages

---

# EVALUATION REQUIREMENTS

The evaluations folder stores benchmark definitions.

benchmark_easy.json
benchmark_medium.json
benchmark_hard.json

These should define:

* which incidents belong to each benchmark
* evaluation groups
* difficulty buckets

Later this will be extended to store:

* RCA accuracy
* tool usage
* investigation duration
* confidence quality

---

# IMPORTANT DATASET RULES

## Rule 1

Do NOT make root causes obvious from a single source.

Bad:
ERROR ROOT CAUSE IS DATABASE FAILURE

Good:
Database timeout errors
+
DB connections at 100%
+
pool-size commit

---

## Rule 2

Add realistic noise.

Not every log should matter.

Real incidents are noisy.

---

## Rule 3

The incidents should feel inspired by real engineering postmortems.

Sources of inspiration:

* Discord outages
* Cloudflare postmortems
* GitHub incidents
* Slack engineering incidents
* AWS post-event summaries

But the dataset itself should be synthetic and structured.

---

# INITIAL INCIDENTS

Easy:

* db_pool_exhaustion
* failed_deployment

Medium:

* memory_leak
* rds_saturation

Hard:

* cascading_voice_outage
* retry_storm

---

# GOAL

The final dataset should feel like:

* a production-inspired incident benchmark
* an agent evaluation suite
* a replayable investigation framework

The dataset is one of the most important parts of the project.
