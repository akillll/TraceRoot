# Development and Evaluation Methodology

This document defines how AI Incident Investigator should be built, tested, and judged.

The system is not a generic Q&A model over ops data. It is an autonomous troubleshooting agent that must investigate incidents using:

- logs
- metrics
- commits
- deployments
- runbooks

The product goal is evidence-backed root cause analysis, not plausible-sounding summaries. The dataset in this repository is therefore not mock JSON for demos. It is the project’s evaluation framework, replay framework, and benchmark suite.

Medium and hard incidents are the real reasoning benchmark. Easy incidents are useful for smoke tests, tool wiring, and UX validation, but they are not enough to prove that the system can investigate like an engineer.

## 1. Development Principles

### Build for investigation, not answer generation

The product should expose the investigation process as a first-class artifact:

- what tools were called
- what evidence was collected
- which hypotheses were considered
- which hypotheses were eliminated
- why the final RCA was accepted

If the system can produce an answer but cannot explain the investigation path, it is not production-grade.

### Optimize for evidence correlation

The core capability is correlation across sources, not extraction from one source. Most development decisions should improve one or more of:

- evidence retrieval quality
- evidence normalization
- cross-source correlation
- contradiction detection
- hypothesis tracking
- trace replay

Features that make the system sound more fluent but do not improve evidence correlation should be lower priority.

### Preserve uncertainty

The agent must be able to:

- continue investigating when evidence is weak
- return low confidence when sources disagree
- fail safely when the incident cannot be resolved with available evidence

A system that always returns a confident RCA is less trustworthy than one that can explicitly say the evidence is insufficient.

### Keep tools narrow and composable

Tool boundaries should remain simple and inspectable. Prefer:

- `search_logs`
- `get_metrics`
- `get_recent_commits`
- `get_deployments`
- `search_runbooks`

Avoid black-box tools that collapse investigation into one opaque step. The evaluation framework needs visibility into tool choice and reasoning order.

### Treat traces as core product data

Tool traces, intermediate findings, timestamps, and hypothesis revisions are not debug leftovers. They are required for:

- replayability
- evaluation
- failure analysis
- regression diagnosis
- trust in the agent

## 2. Evaluation Philosophy

The project should evaluate investigation quality, not just final answers.

A correct RCA reached through poor reasoning is still a problem if the agent:

- ignored contrary evidence
- skipped key tools
- overfit to one signal
- hallucinated missing facts
- got lucky on the benchmark

Evaluation should therefore score both:

1. outcome quality
2. investigation process quality

The benchmark dataset exists to support this. Files such as `expected_rca.json`, `timeline.json`, and benchmark metadata are not synthetic decoration. They define the ground truth needed for reproducible agent evaluation.

The evaluation philosophy should assume:

- some incidents are underdetermined without careful investigation
- medium and hard incidents require multiple loops of hypothesis refinement
- timeline reasoning matters as much as symptom matching
- downstream saturation is often not the root cause

Success means the agent behaves like a careful junior SRE, not like a confident autocomplete system.

## 3. RCA Evaluation Metrics

RCA quality should be measured as a structured scoring problem, not a binary “correct/incorrect” decision.

### Root Cause Accuracy

Measure whether the reported root cause matches the benchmark ground truth at the right abstraction level.

Good:

- identifies the actual triggering change or failure mode
- distinguishes trigger from amplification
- separates primary cause from secondary symptoms

Bad:

- reports a downstream bottleneck as the root cause
- reports a broad class like “database issue” when the benchmark requires “cache stampede causing DB overload”
- confuses environmental context with the actual defect

### Evidence Coverage

Measure whether the RCA is supported by the evidence sources required by the incident.

Scoring should consider:

- number of required evidence sources used
- whether the cited evidence is actually relevant
- whether the agent connected the evidence causally rather than listing it

An RCA that cites only logs in a medium or hard incident should score poorly even if the final answer is close.

### Causal Coherence

Measure whether the explanation reconstructs the failure chain correctly:

- what changed
- what failed first
- what amplified next
- what secondary services degraded later
- why recovery occurred after the mitigation

This is especially important for hard incidents where the incident unfolds across multiple services.

### Actionability

Recommended actions should be judged on whether they logically address the identified failure mode. Good actions:

- reduce recurrence risk
- improve detection
- improve containment
- improve operational safety around rollout or failover

Weak actions are generic and not causally tied to the RCA.

### Severity of Error

Evaluation should distinguish between:

- minor wording mismatch
- missing contributing factor
- wrong primary cause
- hallucinated cause unsupported by the dataset

Not all wrong answers are equally wrong.

## 4. Tool Usage Evaluation

Tool use is a core evaluation dimension because the system is intended to investigate, not merely answer from prior assumptions.

### Tool Selection Quality

Measure whether the agent used the right classes of tools for the incident difficulty:

- easy incidents may need 2-3 sources
- medium incidents should usually require 3-4 evidence sources
- hard incidents should require 4 or more correlated signals plus timeline reconstruction

Failure patterns:

- using only logs for a hard incident
- skipping deployments when failure onset suggests a change event
- skipping commits when the RCA depends on a configuration or behavior change
- skipping runbooks when they should shape hypothesis generation

### Tool Ordering

The order of investigation matters. Good investigations often look like:

1. detect symptoms
2. form initial hypotheses
3. pull contradictory or supporting metrics
4. inspect recent changes
5. correlate deployment timing
6. refine or eliminate hypotheses
7. produce RCA

Tool traces should be evaluated for whether the sequence is defensible and efficient, not merely complete.

### Redundant or Wasteful Tool Use

The agent should not thrash across tools without refining the state of the investigation. Signals of poor investigation quality include:

- repeated equivalent searches without new hypotheses
- pulling irrelevant sources late in the incident
- gathering evidence after already asserting the conclusion

### Missing Negative Checks

Strong investigations often include elimination steps:

- checking whether a dependency probe stayed healthy
- confirming that a suspected producer duplicate metric stayed flat
- confirming that packet loss did not rise

The system should get credit for disproving plausible but wrong hypotheses.

## 5. Confidence Evaluation

Confidence scoring must be evidence-based.

The system should never produce arbitrary confidence labels. Confidence should reflect:

- source agreement
- causal coherence
- completeness of evidence
- presence or absence of conflicting signals
- whether timeline reconstruction is consistent

### High Confidence

Appropriate when:

- multiple required evidence sources agree
- ordering of events is clear
- competing hypotheses were tested and eliminated
- recovery after mitigation matches the proposed cause

### Medium Confidence

Appropriate when:

- the likely cause is supported
- one or more sources are incomplete or ambiguous
- there are unresolved alternative explanations

### Low Confidence

Appropriate when:

- evidence is fragmented
- key sources are missing
- multiple explanations remain plausible
- mitigation did not clearly validate the hypothesis

The agent should sometimes fail or return low confidence. That is a feature, not a defect. In production, false confidence is more dangerous than explicit uncertainty.

Evaluation should compare:

- reported confidence
- actual evidence strength
- benchmark expectation

This enables confidence calibration scoring rather than only answer scoring.

## 6. Hallucination Prevention

Hallucination control is a design requirement, not a prompt preference.

### Evidence Before Assertion

Every non-trivial claim in the RCA should be traceable to:

- a tool result
- a correlated benchmark source
- or a clearly labeled inference derived from multiple findings

The system must not invent:

- logs that were never retrieved
- metrics that were never observed
- deployments that were never present
- causal links unsupported by the timeline

### Distinguish Facts from Inferences

The agent should separate:

- observed evidence
- interpretation
- uncertainty

Example:

- observed: cache hit rate collapsed at 13:10
- observed: DB CPU peaked at 13:20
- inference: DB overload was likely downstream of the cache stampede

### Require Contradiction Handling

If logs imply one hypothesis but metrics weaken it, the system must say so. Ignoring conflicting evidence should count as a serious evaluation failure.

### Prefer Abstention to Fabrication

When the evidence does not justify a clean conclusion, the system should:

- return lower confidence
- present top hypotheses
- explain missing evidence

This behavior should be explicitly rewarded in evaluation.

## 7. Investigation Replay & Traceability

Replayability is a hard requirement for engineering quality.

Every benchmark run should produce enough trace information to answer:

- what the agent saw
- what it did next
- what it believed at each step
- why it changed its mind
- why it ended the investigation

### Required Trace Artifacts

At minimum, the system should preserve:

- tool calls
- tool outputs or normalized findings
- timestamps
- intermediate hypotheses
- confidence updates
- final RCA

### Why Traceability Matters

Traceability enables:

- debugging weak benchmark performance
- understanding regressions between model versions
- detecting hallucination patterns
- measuring investigation efficiency
- replaying incidents in demos and internal reviews

Without traces, benchmark failures become hard to diagnose and the system becomes difficult to improve systematically.

## 8. Benchmarking Strategy

The benchmark should be used progressively, not as one flat pass/fail test.

### Easy Incidents

Use for:

- schema validation
- tool integration testing
- UI workflow checks
- smoke tests for basic evidence retrieval

Easy incidents are not sufficient proof of reasoning ability.

### Medium Incidents

Use for:

- hypothesis refinement testing
- cross-system reasoning validation
- measuring whether the agent correlates sources instead of anchoring on one

Medium incidents are the minimum acceptable benchmark for reasoning quality.

### Hard Incidents

Use for:

- distributed reasoning
- timeline reconstruction
- competing-hypothesis elimination
- robustness against misleading evidence

Hard incidents are the true benchmark of system quality. If the agent performs well on easy and medium but fails systematically on hard incidents, the architecture is not yet solving the intended problem.

### Regression Strategy

Run benchmarks:

- after retrieval changes
- after tool contract changes
- after reasoning or orchestration changes
- after model upgrades
- after dataset additions

Track not just final RCA accuracy, but also:

- source coverage
- tool trace quality
- confidence calibration
- time to first correct hypothesis
- number of false conclusions entertained

## 9. Dataset Design Philosophy

The dataset should be treated as a benchmark corpus, not as sample fixture data.

### What the Dataset Is

The dataset is:

- a historical incident replay framework
- a ground truth RCA library
- an evaluation suite for tool-using agents
- a source of traceable, multi-step reasoning tasks

### What the Dataset Is Not

The dataset is not:

- mock demo JSON
- a static prompt context blob
- a one-shot answer key
- a replacement for real operational thinking

### Design Implications

Incident data should continue to emphasize:

- realistic noise
- misleading local symptoms
- indirect deployment correlation
- causal ambiguity
- multi-service effects in medium and hard cases

Expected RCA files should define:

- the real cause
- the evidence needed
- the expected investigation path

Timelines should be treated as evaluation ground truth for sequence reconstruction, not merely as UI content.

Runbooks should remain operational documents, not hidden prompts that leak the answer.

## 10. Future Evaluation Expansion

The current benchmark is a strong foundation, but the methodology should expand over time.

### Additional Evaluation Dimensions

Future work should measure:

- investigation duration
- tool efficiency
- recovery recommendation quality
- hypothesis branching behavior
- ability to detect when an incident spans multiple overlapping faults

### Adversarial and Missing-Data Scenarios

The benchmark should expand to include:

- incomplete telemetry
- stale or contradictory runbooks
- delayed metrics
- partial deployment metadata
- incidents where the correct answer is “insufficient evidence”

### Calibration and Human Review

Longer term, evaluation should compare:

- model confidence versus benchmark truth
- model confidence versus human reviewer judgment
- investigation traces across model versions

### Online and Production Evaluation

Once the system is used beyond static benchmarks, evaluation should include:

- replay of real historical incidents
- shadow-mode comparison against human investigators
- post-incident trace review
- false-confidence audits

## Closing Position

AI Incident Investigator should be developed as an evidence-correlating investigation system, not as an RCA text generator.

The benchmark dataset is the backbone of that discipline. Development should improve traceability, evidence correlation, and calibrated reasoning. Evaluation should reward correct investigation behavior, not just correct-looking answers. Medium and hard incidents should be the main bar for quality, because they are where the system either demonstrates real troubleshooting ability or reveals that it is only pattern matching.
