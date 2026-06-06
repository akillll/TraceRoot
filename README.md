# AI Incident Investigator

AI Incident Investigator is an autonomous DevOps troubleshooting system for evidence-based root cause analysis.

The goal is not log summarization. The goal is to investigate incidents the way a junior SRE would:

1. gather evidence
2. form hypotheses
3. validate hypotheses
4. correlate findings
5. produce an explainable RCA

The core inputs are:

- logs
- metrics
- commits
- deployments
- runbooks

## Project Focus

This repository is centered on the investigation workflow, not just final answers.

Key product principles:

- show the investigation, not only the conclusion
- require evidence before conclusions
- prefer multi-source correlation over single-source pattern matching
- make confidence explainable and evidence-based
- preserve tool traces for replayability and debugging

The benchmark dataset in this repo is a real evaluation framework, not mock JSON.

## Repository Structure

```text
.
├── AGENTS.md
├── README.md
├── backend/
│   └── main.py
├── frontend/
│   ├── package.json
│   └── ...
├── data/
│   ├── incidents/
│   │   ├── easy/
│   │   ├── medium/
│   │   └── hard/
│   ├── evaluations/
│   └── runbooks/
└── docs/
    ├── DATASET_SPEC.md
    └── DEVELOPMENT_EVALUATION_METHODOLOGY.md
```

## Dataset

The dataset is the backbone of the project.

It currently includes:

- `easy` incidents for smoke tests and basic tool validation
- `medium` incidents for cross-system reasoning
- `hard` incidents for distributed failure analysis, cascading outages, and competing hypotheses
- `expected_rca.json` files as ground truth
- `timeline.json` files for reconstruction and replay
- benchmark metadata in `data/evaluations/`
- operational runbooks in `data/runbooks/`

The medium and hard sets are the real reasoning benchmark.

## Current State

Current repo status:

- dataset tiers exist across `easy`, `medium`, and `hard`
- runbooks and benchmark manifests exist
- methodology and dataset design docs exist
- backend is a minimal FastAPI stub
- frontend is a Vite + React app scaffold

This means the repo already has the evaluation asset layer, while the investigation runtime and product UI are still early.

## Running the Project

### Backend

The backend currently contains a minimal FastAPI app in `backend/main.py`.

If you have FastAPI and an ASGI server installed, run:

```bash
cd backend
uvicorn main:app --reload
```

### Frontend

The frontend is a Vite React app.

```bash
cd frontend
npm install
npm run dev
```

Other frontend commands:

```bash
npm run build
npm run lint
npm run preview
```

## Documentation

- [docs/DATASET_SPEC.md] explains the benchmark dataset structure and purpose.
- [docs/DEVELOPMENT_EVALUATION_METHODOLOGY.md] defines how the agent should be built and evaluated.
- [AGENTS.md] captures the product mission and operating principles for the agent.

## What Good Looks Like

A successful system should:

- investigate autonomously
- use the right tools in the right order
- gather evidence before concluding
- correlate multiple sources
- show its work through traces and timelines
- produce a structured RCA with calibrated confidence
- return low confidence when evidence is weak

## Near-Term Priorities

Practical next steps for the project:

- implement provider-agnostic investigation tools
- build the investigation timeline and trace view
- wire the agent to the benchmark dataset for replay
- score investigation quality, not only RCA correctness
- evaluate first on medium incidents, then on hard incidents

## Guiding Constraint

If a feature improves polish but does not improve investigation transparency, evidence correlation, replayability, or RCA quality, it should be lower priority than work that does.
