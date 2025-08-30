# Field · Vector · Shadow (FVS)

**Idea**  
- **Field** = internal superposition of hypotheses (a belief state).  
- **Vector** = the chosen measurement frame (policy, cues, retrieval, style).  
- **Shadow** = the projected answer under that vector (with confidence); the field persists.

This reframes decisions: **maintain superposition, choose vectors deliberately, treat outputs as projections.**

## Axioms
- **A. Field First:** keep a weighted set of hypotheses \(H=\{h_k, w_k\}\).  
- **B. Vector is a choice:** compile \(O_q\) from **prompt + policy + landmarks + retrieval**.  
- **C. Shadow is provisional:** return *answer + confidence + top alternatives & why excluded*.  
- **D. Risk ↑ ⇒ Hold field:** delay collapse; ask for more evidence or show multiple shadows.

## Mapping to LLM stacks
- **Sky (explore):** proposes diverse hypotheses (builds the field).  
- **Ground (contain):** constrains the vector (safety/PII/policy).  
- **Judge (choose):** scores shadows + residual field; emits confidence.  
- **Landmarks (memory):** store short cues about good/bad vectors (no verbatim traces).  
- **Archive (optional):** on-demand facts when the field is too flat/ambiguous.

## Minimal algorithm
1) Build **FIELD** \(H\) from N diverse candidates.  
2) Compile **VECTOR** \(O_q\) from prompt + policy + landmarks (+ retrieved facts).  
3) Score candidates under \(O_q\) → normalized \(p_k\).  
4) Decide: low-risk → pick argmax; high-risk → hold field, add evidence or return bounded alternatives.  
5) Learn: promote landmarks tied to successful vectors; add avoid-cues for harmful ones; decay unused cues.

## Risk policy (defaults)

low: single shadow + 1 alternative; deep-dive if confidence < 0.70
medium: shadow + 2 alternatives; deep-dive if confidence < 0.75 or sources requested
high: hold field; deep-dive first; return bounded options; avoid single collapse until safe

## Output shape (example)
```json
{
  "answer": "...",
  "confidence": 0.82,
  "vector": {
    "policy": ["safety_v1","pii_redact"],
    "landmarks_used": ["Offer sensory-friendly options.","Avoid stereotypes."],
    "retrieval_ids": ["doc12","doc47"]
  },
  "alternatives": [
    {"hypothesis":"...", "weight":0.11, "why_excluded":"lower evidence"},
    {"hypothesis":"...", "weight":0.07, "why_excluded":"policy risk"}
  ]
}
```

@startuml
title FVS inside a reasoning loop

actor User
participant Orchestrator
participant Landmarks
participant Sky as "Field builder"
participant Ground as "Vector constraints"
participant Judge
participant Archive as "Cold facts"
database Audit as "Log"

User -> Orchestrator: Prompt
Orchestrator -> Sky: Build FIELD (N diverse hyps)
Orchestrator -> Landmarks: Get top-K cues
Orchestrator -> Ground: Compile VECTOR (policy+cues+(facts?))
alt confidence low or risk high
  Orchestrator -> Archive: Retrieve facts (minimal k)
  Orchestrator -> Ground: Update VECTOR with evidence
end
Judge -> Orchestrator: Score SHADOWS under VECTOR
Orchestrator -> User: Shadow (answer) + confidence + alternatives
Orchestrator -> Landmarks: Learn good/bad vectors (cues only)
Orchestrator -> Audit: Field summary, vector, shadow, decision
@enduml

Practical heuristics
	•	Name the vector: “Frame used: policy X + cues Y + retrieval Z.”
	•	Name the shadow: “Projection; confidence 0.82; excluded alternatives: …”
	•	Offer re-measurement: “Try stricter citations / cost-saver / novelty frames?”
	•	In crisis: “Multiple causes overlap; safest action is …; we are not collapsing to one label yet.”

Why it helps
	•	Keeps all standard predictions (unitarity/decoherence analogies) without metaphysical baggage.
	•	Gives AI a usable rule: superposition is truth; collapse is UI.
	•	Improves safety by avoiding premature single answers; improves clarity by exposing frames.
