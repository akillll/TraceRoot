# AGENTS.md

# AI Incident Investigator

## Mission

AI Incident Investigator is an autonomous DevOps troubleshooting system that performs evidence-based root cause analysis.

The goal is not log summarization.

The goal is to investigate incidents the same way a junior SRE would:

1. Gather evidence
2. Form hypotheses
3. Validate hypotheses
4. Correlate findings
5. Produce an explainable RCA

Every feature added to the project should strengthen at least one of these capabilities.

---

# Product Principles

## 1. Show The Investigation

The investigation process is the product.

Users should be able to observe:

* Tool execution
* Evidence collection
* Investigation progress
* Root cause formation

Avoid black-box experiences.

The investigation timeline is a first-class feature.

---

## 2. Evidence Before Conclusions

The system should never generate a root cause without supporting evidence.

Every RCA must reference findings collected during the investigation.

Bad:

"Database issue detected."

Good:

"Database connection pool exhaustion detected because:

* timeout errors found in logs
* DB connections reached 100%
* recent deployment modified pool configuration"

---

## 3. Multi-Source Correlation

Strong conclusions require multiple evidence sources.

Preferred:

Logs + Metrics + Commits

Over:

Logs only

The system should demonstrate correlation, not summarization.

---

## 4. Explainable Confidence

Confidence must be derived from evidence agreement.

Confidence should never be arbitrary.

The user should always understand why a confidence score was assigned.

---

# Product Priorities

Priority 1

Investigation transparency

Priority 2

Root cause accuracy

Priority 3

Evidence quality

Priority 4

RAG quality

Priority 5

UI polish

---

# Agent Behavior Rules

The agent must:

* Gather evidence before concluding
* Prefer multiple evidence sources
* Cite supporting findings
* Avoid unsupported assumptions
* Continue investigating when evidence is weak

The agent must not:

* Jump directly to conclusions
* Invent evidence
* Produce arbitrary confidence scores
* Ignore conflicting evidence

---

# Tool Design Rules

Every tool should:

* Have one responsibility
* Return structured output
* Be independently testable
* Be provider agnostic

Good:

search_logs()

get_metrics()

search_runbooks()

Bad:

investigate_incident()

---

# Dataset Philosophy

The incident dataset is a core product asset.

Each incident should require evidence correlation.

Root causes should not be obvious from a single source.

Every incident should contain:

* Logs
* Metrics
* Commit history
* Deployment events
* Ground truth RCA

---

# Runbook Philosophy

Runbooks are operational knowledge.

Runbooks should be written like internal engineering documentation.

Runbooks are not prompts.

Every runbook should include:

* Symptoms
* Diagnostics
* Common Causes
* Recommended Actions

---

# UI Philosophy

The dashboard consists of three core areas:

1. Incident Input
2. Investigation Timeline
3. RCA Report

The Investigation Timeline is the primary demo experience.

When making design decisions, prioritize investigation visibility over visual complexity.

---

# Evaluation Philosophy

Every major feature should be measurable.

Key metrics:

* Root Cause Accuracy
* Evidence Coverage
* Confidence Accuracy
* Investigation Duration

Avoid subjective evaluation whenever possible.

---

# Future Integrations

Planned integrations:

* GitHub
* AWS CloudWatch Logs
* CloudWatch Metrics
* ECS Events
* Prometheus
* Datadog

The agent should remain independent of specific providers.

Use abstraction layers.

Prefer:

get_logs()

Over:

get_cloudwatch_logs()

---

# Definition of Success

A user submits an incident.

The system:

1. Investigates autonomously
2. Collects evidence
3. Streams the investigation live
4. Produces a structured RCA
5. Explains why it reached the conclusion

The final experience should feel like a real engineer performed the investigation.
