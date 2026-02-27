# Test Case Reviewer Agent - Technical Architecture Design
Version: 1.0  
Status: Approved design baseline from gated questionnaire  
Audience: Product, QA leadership, AI engineering, platform engineering, Copilot/agent implementation teams

---

## 1. Purpose

This document defines the target technical architecture for a **Test Case Reviewer Agent**. The agent's responsibility is to review, assess, and validate **individual test cases** against approved delivery artifacts and supporting evidence, then produce review outputs that are explicit, traceable, confidence-scored, and safe for human decision-making.

The design is intentionally grounded in the questionnaire decisions captured during the approval flow. Where implementation choices were not explicitly specified, this document uses **recommended architecture defaults** and labels them as such.

---

## 2. Confirmed Design Decisions from Questionnaire

The following choices were explicitly confirmed:

1. **System of record for valid test cases:** Test management tool
2. **Primary review unit:** Individual test case
3. **Primary review timing:** As soon as draft test cases are created
4. **Main review objective priority:** Completeness against requirements and acceptance criteria
5. **Requirement vs test conflict handling:** Flag for human resolution
6. **Review context recordings:** Mandatory evidence for every test case
7. **No explicit traceability link:** Mark low quality, but allow to remain in review
8. **Requirement covered but UI/error states missed:** Reduce completeness and quality score
9. **Duplicate/redundant test case handling:** Auto-merge into the strongest existing test case
10. **Coverage gap handling:** Flag the gap; human drafts the missing test case
11. **Defect intelligence:** Supporting context, not a strong scoring factor
12. **Release documentation:** Mandatory only for impacted features/releases
13. **Development change indicators:** Supporting evidence after requirement-based review
14. **Configuration/metadata references:** Optional supporting context
15. **Compliance/policy constraints:** Advisory guidance
16. **Dependency/integration map:** Mandatory only when integration/shared-service scope exists
17. **Historical review outcomes:** Ignore to avoid bias
18. **API contracts/examples:** Optional supporting context
19. **Confidence scoring:** Exposed per test case and per recommendation
20. **Final decision behavior:** Soft decision -> Approved / Needs Update / Needs Clarification

These decisions are treated as binding inputs to the architecture below.

---

## 3. Mission Statement

The Test Case Reviewer Agent should answer this core question:

> "Given the latest draft of an individual test case, and the best available approved evidence, is this test case complete, coherent, traceable, current, and fit for execution within the intended scope?"

It should not create hidden product assumptions, silently override conflicts, or fabricate traceability. When evidence is incomplete or contradictory, it must surface that condition explicitly.

---

## 4. Scope

### In Scope
- Review of **individual manual or automation-oriented test cases**
- Assessment against:
  - requirement baseline
  - UI specifications and flows
  - recordings
  - existing test assets
  - technical documentation
  - defect intelligence
  - release documentation
  - dependency and integration maps
  - review standards
  - data model/field definitions
  - compliance/policy constraints
  - API contracts/examples
  - traceability references
- Identification of:
  - coverage gaps
  - weak/defective test cases
  - impacted regression tests
  - existing test cases needing modification
  - recommendations to add/update/merge/retire/reprioritize/strengthen
- Scoring:
  - quality score
  - completeness score
  - traceability score
  - confidence score
- Soft decision output:
  - Approved
  - Needs Update
  - Needs Clarification

### Out of Scope
- Autonomous publication of new test cases into the system of record
- Autonomous closure of requirement ambiguity
- Full code-based semantic review of implementation behavior
- Automated legal/compliance sign-off
- Historical reviewer-bias learning from previous review comments
- Final human release approval

---

## 5. High-Level Operating Principles

1. **Evidence-first review**
   - The agent must review against approved evidence, not intuition.

2. **Requirement-led evaluation**
   - Requirements and acceptance criteria are the main completeness driver.

3. **Human escalation for conflicts**
   - If requirement and test case intent conflict, the agent must not self-resolve.

4. **No hallucinated traceability**
   - Missing traceability must be reported as missing, not inferred as fact.

5. **Early lifecycle review**
   - The review should happen at draft time, not just pre-execution.

6. **Individual test case granularity**
   - Every score, confidence signal, and recommendation is computed per test case.

7. **Exposed confidence**
   - Confidence must be user-visible per test case and per recommendation.

8. **Advisory, not destructive, except for duplicate merge logic**
   - The only explicitly aggressive behavior approved is duplicate consolidation into the strongest existing case. In implementation, this should still preserve audit history.

---

## 6. Source Model

## 6.1 Source Classes

### Vector-embedded sources (V)
These should be chunked, embedded, indexed, and retrievable semantically:
- Requirements baseline
- UI specifications and flows
- Review context recordings and transcripts
- Existing test assets
- Technical documentation
- Defect intelligence
- Release documentation
- Dependency and integration map
- Review standards and quality benchmarks
- Data model and field definitions
- Compliance and policy constraints
- API contracts and examples
- Traceability references

### Partial-vector sources (PV)
These should be indexed as metadata-rich retrieval sources, but not as full-code embeddings:
- Development change indicators
- Configuration and metadata references

## 6.2 System of Record
The **test management tool** is the authoritative source for:
- current version of a test case
- canonical test case ID
- status
- suite membership
- ownership
- execution references
- audit trail

Recommended placeholder:
- `<TestManagementSystem>` such as TestRail / Zephyr / qTest / ADO Test Plans

---

## 7. Recommended Reference Architecture

## 7.1 Logical Components

### A. Connectors and Ingestion Layer
Pulls or receives source data from:
- test management tool
- requirements repository
- design/UI repository
- meeting recording/transcript store
- defect systems
- documentation/wiki tools
- release/change management tools
- integration dependency registry
- API specification sources
- configuration/metadata exports

### B. Preprocessing and Normalization Layer
Normalizes heterogeneous artifacts into a review-ready representation:
- OCR/transcript cleanup where needed
- document segmentation
- requirement/AC extraction
- screen-state extraction
- field and validation rule normalization
- duplicate test case candidate detection
- relationship stitching across requirement/test/defect/release IDs

### C. Retrieval and Evidence Assembly Layer
Builds the evidence packet for a single test case review:
- fetch canonical test case from system of record
- retrieve relevant requirement chunks
- retrieve UI flow chunks
- retrieve recording transcript evidence
- retrieve similar/duplicate test cases
- retrieve traceability references
- conditionally retrieve release notes, dependency maps, API contracts, config references

### D. Review Reasoning Layer
Runs the phased review logic:
- structural quality assessment
- requirement completeness assessment
- UI and state coverage assessment
- traceability assessment
- duplication analysis
- regression impact analysis
- recommendation generation
- confidence calibration
- status recommendation

### E. Decision and Output Layer
Produces structured review outputs:
- reviewed test case record
- scores
- weak/defective flags
- modification-needed flags
- coverage gaps
- regression impact list
- recommendation list
- Approved / Needs Update / Needs Clarification

### F. Human Review Orchestration Layer
Routes unresolved items:
- requirement conflict
- unclear recording evidence
- incomplete source packet
- ambiguous duplicate merge
- low-confidence recommendations
- cross-team dependency uncertainty

### G. Observability and Audit Layer
Stores:
- evidence used
- chunk references
- score rationale
- conflict reasons
- recommendation rationale
- model/prompt version
- review timestamp
- reviewer override outcome

---

## 8. Suggested Physical Architecture

This section is a recommended implementation pattern, not a user-confirmed mandate.

### Core services
- **Connector services** for each source system
- **Normalization service**
- **Embedding/indexing pipeline**
- **Review orchestration service**
- **Scoring service**
- **Duplicate resolution service**
- **Recommendation service**
- **Review API**
- **Reviewer UI / Copilot surface**

### Data stores
- **Vector store** for semantically searchable evidence
- **Relational store** for review metadata, scores, IDs, status, traceability mappings, audit logs
- **Object/blob storage** for raw artifacts and transcripts
- **Cache** for low-latency review packets

### Execution model
- Event-driven when a new draft test case is created or updated
- Batch mode for backlog review or release review
- On-demand review invocation from reviewer UI

---

## 9. Canonical Review Object Model

## 9.1 Core entities
- `TestCase`
- `Requirement`
- `AcceptanceCriterion`
- `UIScreenState`
- `ReviewRecordingEvidence`
- `TestAssetReference`
- `DefectReference`
- `ReleaseImpactReference`
- `DependencyReference`
- `TraceabilityLink`
- `ReviewResult`
- `Recommendation`
- `CoverageGap`
- `RegressionImpact`
- `DuplicateCluster`

## 9.2 Example review result schema
```json
{
  "testCaseId": "<canonical-id>",
  "reviewTimestamp": "<iso-8601>",
  "status": "Approved | Needs Update | Needs Clarification",
  "scores": {
    "quality": 0,
    "completeness": 0,
    "traceability": 0,
    "confidence": 0
  },
  "summary": {
    "strengths": [],
    "issues": [],
    "conflicts": []
  },
  "coverageGaps": [],
  "weaknessFlags": [],
  "modificationNeeded": [],
  "affectedRegressionTests": [],
  "recommendations": [
    {
      "type": "add | update | merge | retire | reprioritize | strengthen",
      "target": "<test-case-id-or-new-gap-id>",
      "rationale": "",
      "confidence": 0
    }
  ],
  "evidenceRefs": [
    {
      "sourceType": "",
      "sourceId": "",
      "chunkId": ""
    }
  ]
}
```

---

## 10. Review Phases

The phases below should be enforced consistently. These phases are the backbone of both implementation and the Copilot prompt.

## Phase 1 - Intake and Canonical Test Case Resolution
Goal: Resolve the exact test case record to review.

Actions:
- Pull latest test case from the test management system
- Confirm canonical ID, version, owner, suite, linked requirement IDs, linked release if any
- Determine whether this is new, modified, or duplicate candidate
- Validate mandatory metadata presence

Outputs:
- Canonical test case packet
- Metadata completeness flags

Failure path:
- If the test case cannot be resolved uniquely, return **Needs Clarification**

## Phase 2 - Evidence Collection
Goal: Assemble the minimal complete evidence packet.

Mandatory evidence:
- requirement baseline
- review context recordings/transcripts
- existing test assets
- review standards/quality benchmark references
- data model/field definitions when relevant

Conditional evidence:
- release documentation for impacted feature/release
- dependency/integration map when integrations/shared services are involved
- API contracts/examples when useful
- development change indicators as supporting evidence
- config/metadata references as optional support
- defect intelligence as supporting context
- compliance/policy constraints as advisory context

Outputs:
- Evidence packet
- Evidence sufficiency score
- Missing evidence list

Failure path:
- If mandatory evidence is materially missing, downgrade confidence and likely return **Needs Clarification**

## Phase 3 - Structural Quality Review
Goal: Assess whether the test case is structurally usable.

Checks:
- title clarity
- preconditions clarity
- step clarity
- expected result specificity
- data references
- unambiguous actions
- environment references where needed
- clear pass/fail observability
- duplication against existing test cases

Outputs:
- structural quality findings
- weak/defective flags
- duplicate merge recommendation or action path

## Phase 4 - Requirement and Acceptance Criteria Completeness Review
Goal: Determine whether the test case sufficiently covers approved functional intent.

Checks:
- direct requirement coverage
- acceptance criteria coverage
- business rule references
- boundary conditions
- positive and negative expectation coverage
- validation rule awareness if referenced in source materials

Special rule from questionnaire:
- This phase is the **highest priority driver**

Outputs:
- completeness findings
- requirement coverage map
- missing scenario candidates

## Phase 5 - UI, User Journey, and Error-State Review
Goal: Determine whether UI state coverage is materially adequate.

Checks:
- screen-state representation
- navigation flow representation
- form/state transitions
- validation and inline error states
- permission or visibility behaviors when relevant
- empty states / failed states / retry states when evidence exists

Questionnaire rule:
- If requirement coverage exists but UI/error states are missed, reduce **quality** and **completeness**

Outputs:
- UI coverage findings
- missed state findings
- weak assertion findings

## Phase 6 - Traceability Review
Goal: Determine whether the test case is properly linked.

Checks:
- requirement link presence
- AC mapping presence
- defect-to-test link presence when relevant
- regression mapping presence if applicable
- release scope alignment if applicable

Questionnaire rule:
- Missing explicit traceability -> mark **low quality**, but do not reject

Outputs:
- traceability score
- missing traceability warnings

## Phase 7 - Contextual Impact Review
Goal: Add bounded contextual intelligence without overpowering the core assessment.

Checks:
- defect intelligence as support only
- release notes for impacted scope
- development change indicators as support after requirement review
- dependency map if integration/shared service involved
- API contract evidence if relevant
- advisory compliance considerations

Outputs:
- impacted regression tests
- modification-needed flags
- cautionary notes

## Phase 8 - Conflict and Ambiguity Review
Goal: Identify issues that the agent must not resolve alone.

Checks:
- requirement contradicts test case
- recording contradicts written requirement
- release notes change behavior without updated requirement
- duplicate merge ambiguity
- insufficient evidence for reliable recommendation

Questionnaire rule:
- Requirement vs test conflict -> **human resolution**

Outputs:
- conflict list
- escalation reason list
- forced clarification flags

## Phase 9 - Scoring and Confidence Calibration
Goal: Compute stable and explainable scores.

Required scores:
- quality score
- completeness score
- traceability score
- confidence score

Calibration inputs:
- evidence sufficiency
- evidence consistency
- clarity of mapping
- number of unresolved ambiguities
- number of unsupported assumptions
- presence/absence of mandatory evidence

Outputs:
- numeric scores
- textual rationale
- confidence per recommendation

## Phase 10 - Recommendation and Decisioning
Goal: Produce the actionable review result.

Allowed recommendation types:
- add
- update
- merge
- retire
- reprioritize
- strengthen

Decision states:
- **Approved**
- **Needs Update**
- **Needs Clarification**

Decision guidance:
- **Approved** when evidence is sufficient and issues are minor/non-blocking
- **Needs Update** when test case changes are clearly required
- **Needs Clarification** when evidence is conflicting, insufficient, or the agent cannot safely conclude intent

---

## 11. Scoring Model

Recommended normalized scale: **0 to 100**

## 11.1 Quality Score
Measures structural correctness and usefulness.

Suggested dimensions:
- clarity of steps
- clarity of expected results
- observability
- ambiguity level
- duplicate/redundant status
- assertion strength

## 11.2 Completeness Score
Measures business and scenario coverage.

Suggested dimensions:
- requirement coverage
- AC coverage
- UI states
- error states
- negative scenarios
- edge conditions
- data validation coverage

## 11.3 Traceability Score
Measures explicit evidence linking.

Suggested dimensions:
- requirement linkage
- AC linkage
- release scope linkage
- regression mapping linkage
- defect linkage where applicable

## 11.4 Confidence Score
Measures trustworthiness of the agent's review output, not the test case itself.

Suggested dimensions:
- evidence sufficiency
- evidence consistency
- retrieval relevance confidence
- ambiguity count
- conflict count
- recommendation certainty

## 11.5 Suggested decision thresholds
These are recommended defaults and should remain configurable:
- **Approved**
  - confidence >= 75
  - completeness >= 80
  - quality >= 75
  - no unresolved critical conflict
- **Needs Update**
  - confidence >= 60
  - clear deficiencies exist
  - intent is sufficiently understood
- **Needs Clarification**
  - confidence < 60
  - or conflict exists
  - or mandatory evidence is incomplete/contradictory

---

## 12. Duplicate and Merge Logic

The questionnaire selected **auto-merge into the strongest existing test case**. Because this is operationally risky, implementation should use controlled merge logic.

Recommended execution policy:
1. Identify duplicate cluster using semantic similarity plus metadata overlap
2. Rank candidate strongest case by:
   - traceability richness
   - clearer expected results
   - broader coverage
   - most current active version
   - stronger linkage to requirement baseline
3. Preserve audit history of the merged-out test case
4. Record merge rationale
5. Never destroy historical references silently
6. If strongest-case selection is ambiguous, downgrade to **Needs Clarification**

---

## 13. Coverage Gap Handling

The questionnaire requires:
- the agent **flags** coverage gaps
- the agent does **not** fully draft missing test cases automatically

Therefore the agent should produce:
- gap title
- impacted requirement/AC reference
- missing scenario description
- why current suite is insufficient
- execution risk if not addressed
- recommendation priority
- confidence

But it should not automatically create a full replacement-ready test case unless a future design change authorizes that behavior.

---

## 14. Impacted Regression Review

Affected regression tests should be produced when:
- feature behavior changed
- release notes introduce impact
- dependency or integration path changed
- metadata or access behavior changed
- reopened defect indicates regression weakness

Recommended method:
- map reviewed test case to requirement/component/module
- intersect with changed feature/module metadata
- retrieve related regression tests from test management tool
- label as:
  - impacted - update likely
  - impacted - execution priority increase
  - impacted - traceability check needed

---

## 15. Human-in-the-Loop Design

Human review is mandatory in the following cases:
- requirement/test case conflict
- conflicting recording vs written requirement intent
- low-confidence merge decision
- insufficient mandatory evidence
- unresolved scope ambiguity
- source inconsistency across approved artifacts
- reviewer override requested

Reviewer actions:
- approve agent recommendation
- override scores/status
- reclassify duplicate merge
- defer pending requirement clarification
- route to BA/PO/QA lead

Audit requirements:
- capture override reason
- preserve original agent rationale
- preserve evidence references

---

## 16. Non-Functional Requirements

### Accuracy
- No fabricated traceability
- No unsupported requirement inference presented as fact

### Explainability
- Every score and recommendation must be explainable with evidence references

### Auditability
- Preserve review packet, evidence packet version, prompt version, model version, timestamps

### Security
- Enforce least-privilege access to requirements, defects, and recordings
- Support source-level access control inheritance where possible

### Privacy
- If recordings/transcripts contain sensitive information, redact before vectorization where required

### Freshness
- Review should use the latest approved sources, not stale drafts, unless draft comparison is intentionally enabled

### Configurability
- Thresholds, scoring weights, evidence requirements, and recommendation policies should be configurable per program

### Reliability
- Partial evidence failure should degrade gracefully into lower confidence, not silent omission

---

## 17. Recommended Workflow

```text
Draft test case created or updated
    ->
Resolve canonical test case from test management system
    ->
Assemble mandatory + conditional evidence packet
    ->
Run phased review
    ->
Compute scores and confidence
    ->
Check conflict rules
    ->
Generate findings, gaps, regression impacts, recommendations
    ->
Assign status: Approved / Needs Update / Needs Clarification
    ->
Persist review record and audit trail
    ->
Route to reviewer queue if clarification/override is needed
```

---

## 18. Recommended Metrics

Operational metrics:
- review throughput per day/week
- average review latency
- percent auto-approved
- percent needs update
- percent needs clarification
- average confidence by module/team
- evidence completeness failure rate
- duplicate detection rate
- regression impact detection rate

Quality metrics:
- post-review defect escape rate
- test case rework rate
- requirement coverage rate
- traceability completeness rate
- reviewer override rate
- false positive recommendation rate
- merge reversal rate

---

## 19. Implementation Risks and Controls

### Risk 1 - Mandatory recording evidence may create retrieval overhead
Control:
- pre-transcribe and chunk recordings
- create story/session metadata index

### Risk 2 - Auto-merge can create accidental information loss
Control:
- preserve source case audit trail
- require ambiguity checks before merge finalization

### Risk 3 - Mixed-quality requirements may reduce accuracy
Control:
- confidence scoring must respond to requirement ambiguity
- unresolved intent routes to Needs Clarification

### Risk 4 - Missing traceability may be common in legacy suites
Control:
- do not reject automatically
- lower quality/traceability score and surface remediation path

### Risk 5 - Source inconsistency across tools
Control:
- evidence packet should label source conflicts explicitly
- no silent precedence except canonical test case source of record

---

## 20. Recommended Rollout Phases

## Rollout Phase A - Foundation
- Connect test management tool
- Connect requirements repository
- Connect recording transcript pipeline
- Connect existing test asset repository
- Define canonical review schema
- Stand up vector + metadata stores

## Rollout Phase B - Core Review Engine
- Requirement completeness review
- UI/error-state review
- structural quality review
- traceability review
- soft-decision output

## Rollout Phase C - Contextual Intelligence
- defect support context
- release impact analysis
- dependency/integration review
- duplicate/merge engine
- regression impact analysis

## Rollout Phase D - Governance and Tuning
- scoring threshold tuning
- reviewer override dashboard
- audit reporting
- prompt/model version governance
- program-specific weighting

---

## 21. Copilot Agent Prompt - Production Draft

The following prompt is the `.md` prompt block intended to seed a Copilot-style agent implementation. It is written to reflect the approved design choices.

---

# System Prompt: Test Case Reviewer Agent

You are the **Test Case Reviewer Agent**, operating as a formal quality-engineering reviewer for individual test cases.

Your primary responsibility is to **review, assess, and validate an individual test case** using approved evidence. You must remain grounded in the available source material. Do not hallucinate. Do not invent traceability, requirements, or user intent.

## Core Objective
Determine whether the current test case is:
- complete against requirements and acceptance criteria
- structurally sound and executable
- sufficiently aligned to UI flows and error states when applicable
- traceable to approved artifacts
- current with respect to release and dependency context when applicable

## Authoritative Rules
1. The **test management tool** is the source of truth for the current valid version of the test case.
2. The **primary review unit** is the **individual test case**.
3. Review should happen **as soon as draft test cases are created**.
4. Prioritize **completeness against requirements and acceptance criteria** above all other dimensions.
5. If the requirement baseline and the test case conflict, do **not** resolve the conflict yourself. Return **Needs Clarification** and explain the conflict.
6. Review context recordings are **mandatory evidence** for every review.
7. If traceability is missing, do not reject automatically. Lower the relevant score and explain the traceability gap.
8. If requirement coverage exists but UI states or error states are materially missed, reduce **quality** and **completeness**.
9. If a test case is duplicate or redundant, consolidate it into the strongest existing case only if the strongest case can be identified confidently. Preserve rationale.
10. If you find a missing scenario, flag it as a **coverage gap**, but do not fully author a new test case.
11. Defect intelligence is supporting context only. It must not dominate the review.
12. Release documentation is mandatory only when the reviewed test case belongs to an impacted feature or release.
13. Development change indicators are supporting evidence after requirement-based review.
14. Configuration and metadata references are optional supporting context.
15. Compliance and policy constraints are advisory.
16. Dependency and integration mapping is mandatory only if the scenario touches integrations, shared services, or external dependencies.
17. Ignore historical review outcomes to avoid bias.
18. API contracts/examples are optional supporting context.
19. Expose **confidence** per test case and per recommendation.
20. Final status must be one of:
   - Approved
   - Needs Update
   - Needs Clarification

## Required Inputs
You may receive any combination of the following:
- requirement baseline
- UI specifications and flows
- review context recordings or transcripts
- existing test assets
- development change indicators
- technical documentation
- defect intelligence
- release documentation
- configuration and metadata references
- dependency and integration map
- review standards and quality benchmarks
- data model and field definitions
- compliance and policy constraints
- API contracts and examples
- traceability references

## Review Phases
Always execute the review in this order:

### Phase 1 - Resolve Test Case
Resolve the canonical test case from the test management source of truth.

### Phase 2 - Assemble Evidence
Collect mandatory evidence first:
- requirements baseline
- review context recordings/transcripts
- existing test assets
- review standards
- relevant data model/field definitions when applicable

Then collect conditional evidence only if relevant:
- release notes
- dependency map
- API contracts
- development change indicators
- config/metadata references
- defect intelligence
- compliance guidance

### Phase 3 - Structural Quality Review
Check:
- clarity
- preconditions
- step precision
- expected result precision
- observability
- ambiguity
- duplicates/redundancy

### Phase 4 - Requirement Completeness Review
Check:
- requirement coverage
- acceptance criteria coverage
- business rules
- validations
- negative and boundary coverage where applicable

### Phase 5 - UI and Error-State Review
Check:
- screen states
- navigation paths
- error states
- validations
- visibility/access states when relevant

### Phase 6 - Traceability Review
Check:
- requirement linkage
- AC linkage
- defect/release/regression linkage when relevant

### Phase 7 - Contextual Impact Review
Use contextual evidence in a bounded way:
- defect intelligence
- release notes
- development change indicators
- dependency map
- API examples
- compliance guidance

### Phase 8 - Conflict Review
Stop and escalate if:
- sources conflict materially
- requirement intent is unclear
- mandatory evidence is insufficient
- duplicate merge is not safe

### Phase 9 - Score and Calibrate Confidence
Produce:
- quality score
- completeness score
- traceability score
- confidence score

### Phase 10 - Recommend and Decide
Return:
- reviewed test case summary
- coverage gaps
- weak/defective issues
- modification-needed assessment
- affected regression tests
- recommendations
- final status

## Output Contract
Return output in structured markdown using the following sections:

1. **Reviewed Test Case**
   - test case ID
   - title
   - final status
   - quality score
   - completeness score
   - traceability score
   - confidence score

2. **Assessment Summary**
   - strengths
   - main weaknesses
   - evidence sufficiency summary

3. **Coverage Gaps**
   - missing scenarios
   - missed validations
   - weak assertions
   - missing edge cases
   - uncovered negative scenarios

4. **Defective / Weak Test Cases**
   - ambiguous
   - duplicate
   - redundant
   - outdated
   - low-value
   - incorrectly scoped items

5. **Existing Test Cases Needing Modification**
   - impacted by requirement changes
   - impacted by UI changes
   - impacted by defect fixes
   - impacted by release or dependency changes

6. **Affected Regression Tests**
   - impacted regression cases with reason

7. **Review Recommendations**
   - add
   - update
   - merge
   - retire
   - reprioritize
   - strengthen
   Include confidence per recommendation.

8. **Conflicts / Clarifications Needed**
   - any unresolved conflict
   - any missing evidence
   - any human decision required

## Behavioral Constraints
- Never fabricate evidence.
- Never fabricate traceability.
- Never silently resolve requirement conflicts.
- Do not use historical reviewer outcomes to shape scoring.
- Be formal, precise, and audit-friendly.
- If evidence is incomplete, say so explicitly and lower confidence.
- If you cannot safely conclude, return **Needs Clarification**.

---

## 22. Prompt Input Template for Runtime

```md
# Review Request

## Test Case Metadata
- Test Case ID:
- Title:
- Suite:
- Owner:
- Version:
- Linked Requirements:
- Linked Release:
- Linked Defects:
- Linked Regression Pack:

## Source Artifacts
### Requirements Baseline
<paste or attach references>

### UI Specifications and Flows
<paste or attach references>

### Review Context Recordings / Transcripts
<paste or attach references>

### Existing Test Assets
<paste or attach references>

### Development Change Indicators
<paste or attach references>

### Technical Documentation
<paste or attach references>

### Defect Intelligence
<paste or attach references>

### Release Documentation
<paste or attach references>

### Configuration / Metadata References
<paste or attach references>

### Dependency / Integration Map
<paste or attach references>

### Review Standards / Quality Benchmarks
<paste or attach references>

### Data Model / Field Definitions
<paste or attach references>

### Compliance / Policy Constraints
<paste or attach references>

### API Contracts / Examples
<paste or attach references>

### Traceability References
<paste or attach references>

## Required Output
Review this individual test case and return:
- Reviewed Test Case
- Coverage Gaps
- Defective / Weak Test Cases
- Existing Test Cases Needing Modification
- Affected Regression Tests
- Review Recommendations
- Final Status
```

---

## 23. Open Configuration Items for Implementation Team

These were not explicitly decided in the questionnaire and should be finalized during engineering design:
- exact connector list and source systems
- vector database selection
- model selection and routing strategy
- transcript generation and redaction policy
- score weighting values
- threshold tuning by project/program
- duplicate merge execution safeguards
- reviewer UI integration pattern
- SLA and throughput target
- role-based access model

---

## 24. Final Recommendation

Proceed with a **phased, evidence-first, human-safe Test Case Reviewer Agent** that:
- reviews at individual test case level
- starts at draft creation time
- prioritizes requirement completeness
- treats recordings as mandatory evidence
- flags rather than fabricates
- uses soft status decisions
- exposes confidence transparently
- preserves human resolution where intent conflicts exist

This is the most defensible architecture based on the approved answers and the constraints provided.
