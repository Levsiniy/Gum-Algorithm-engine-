GUM Algorithm: A Versioned Reasoning Graph for Humanities
A formal, checkpointed reasoning engine for humanities-style arguments, combining graph versioning, generator/critic/arbiter agents, and rollback-based refinement.
________________________________________
Overview
The GUM Algorithm models reasoning as a persistent versioned graph rather than a simple DAG. It supports branches, checkpoints, critiques, contradictions, and rollbacks. The system separates reasoning into three roles:
•	Generator / Solver — proposes graph extensions and branches.
•	Critic — formally validates or refutes each inference.
•	Arbiter — resolves conflicts when branches remain consistent or iteration diverges.
The goal is not to reduce humanities problems to numbers, but to check them for logical correctness, consistency, and source justification.
________________________________________
1. Core: Versioned Reasoning Graph
The graph is a persistent versioned graph with branches and checkpoints.
Node Types
Type	Symbol	Stores
Concept	C	concept, term, theory
Claim	P	proposition, statement
Source	S	author, organization, document
Decision	D	main question / candidate decision
Assumption	U	assumption, hypothesis
Inference	I	inference rule applied to premises
Checkpoint	CP	snapshot of branch state
Critique	K	critic's remark
Contradiction	↓	contradiction
Undecidable	Ø	undecidable self-referential point
Edge Types
Edge	Meaning
supports(P1, P2)	P1 supports P2
attacks(P1, P2)	P1 attacks P2
undercuts(P1, I)	P1 undermines inference I
infers(I, P)	P follows from I
contradicts(P1, P2)	logical contradiction
equivalent(P1, P2)	equivalence
derives(P1, P2)	formal derivation
has_source(P, S)	P has source S
assumes(P, U)	P assumes U
criticized_by(P, K)	P is criticized by K
checkpoint_of(CP, Branch)	checkpoint belongs to branch
rollback_to(Branch, CP)	rollback branch to checkpoint
Node Attributes
json
{
  "id": "P17",
  "type": "Claim",
  "formula": "Inflation -> MoneySupplyGrowth",
  "logic": "predicate",
  "status": "U",
  "confidence": 0.42,
  "source": "S_CentralBank",
  "provenance": ["doc1:p3", "expert:N"],
  "branch": "B2",
  "checkpoint": "CP4",
  "critique": ["K3"],
  "assumptions": ["U1", "U2"]
}
Statuses
Status	Meaning
T	confirmed
F	refuted
U	unknown
Ø	undecidable
I	contradiction
________________________________________
2. Agent Roles
Generator / Solver
Builds the graph, applies heuristics, and proposes branches. It may use an LLM, but every proposal must be represented as a formal formula or a reference to an axiom/rule.
Critic
Relies on strict formal logic:
•	Propositional logic: truth tables, tautologies, DNF/CNF, De Morgan's laws, contraposition, modus ponens, modus tollens, syllogisms, dilemmas.
•	Predicate logic: quantifiers, prenex normal form, validity.
•	Validation: premises |= conclusion iff premises ∧ ¬conclusion is unsatisfiable.
•	Methods: resolution, proof by contradiction, search for a minimal unsatisfiable core.
The Critic does not "evaluate"; it proves or refutes each step.
Arbiter
Triggered if the Generator and Critic diverge for more than N iterations, or if multiple consistent branches remain after rollback. The Arbiter decides by formal criteria:
1.	Minimality of assumptions (Occam's razor).
2.	Reliance on sources with maximum reliability.
3.	Robustness to counterfactuals.
4.	Grounded extension in argumentation semantics.
________________________________________
3. Iteration Protocol with Rollback
Initialization:
•	G = graph(terms, claims, sources, D)
•	CP0 = checkpoint(G)
•	branch = B0
1.	Generator proposes an extension ΔG: new nodes, edges, inferences, branches.
2.	Critic formally validates ΔG:
o	For each inference I: check that premises |= conclusion.
o	Check consistency: no P and ¬P.
o	Check quantifiers, normal forms, resolution.
o	Check sources: no thesis substitution, no post hoc.
o	If an error is found: create a Critique K node and attach it to the guilty node.
3.	If the Critic finds an error:
o	The Generator rolls back to the last checkpoint CP_k that does not contain the error.
o	But the Critique K node remains in the graph as a constraint:
"Do not use inference X without proof Y."
o	The Generator builds a new branch considering K.
o	Go to step 1.
4.	If the Critic confirms:
o	The branch is accepted.
o	A new checkpoint CP_{k+1} is created.
o	If U nodes remain, the Generator continues branching.
5.	If divergence exceeds N iterations:
o	The Arbiter selects the final branch/decision.
o	The answer is fixed.
________________________________________
4. Formal Logic in the Critic
Each claim is translated into a predicate logic formula.
Example:
•	P1: "Wage growth causes cost growth"
∀x (WageGrowth(x) → CostGrowth(x))
•	P2: "Cost growth causes inflation"
∀x (CostGrowth(x) → Inflation(x))
•	Conclusion:
WageGrowth → Inflation
The Critic checks:
(P1 ∧ P2) → (WageGrowth → Inflation) — tautology?
If yes, the inference is correct. If not, a Critique is created.
To check validity, the Critic may use:
•	reduction to perfect CNF/perfect DNF (PCNF/PDNF);
•	the resolution method;
•	proof by contradiction: add ¬conclusion and search for the empty clause.
If the formula is not valid, the Critic produces a counterexample: an assignment of predicate values where the premises are true and the conclusion is false. This is how the error is exposed.
________________________________________
5. Example: Inflation
1.	Generator builds branch B1:
o	S_CentralBank: "the cause is wage growth";
o	P1: WageGrowth → CostGrowth → Inflation.
2.	Critic checks:
o	formally, the inference is correct;
o	but source S_CentralBank may be biased;
o	Critic creates K1: "S_CentralBank has bias=high; independent confirmation required."
3.	Generator rolls back to CP0, but K1 remains as a constraint.
It builds B2: adds S_TradeUnions, P2: MonopolyPrices → CostGrowth → Inflation.
4.	Critic checks B2:
o	formally correct;
o	sources conflict, but there is no logical contradiction.
5.	Arbiter:
o	both branches are consistent;
o	chooses a synthesis: "inflation is multifactorial";
o	final decision D: policy should account for both factors.
________________________________________
6. What This Gives You
•	Reasoning history is preserved: every step, every critique, every rollback.
•	The Critic is formal: it does not say "I think"; it says "the premises do not entail the conclusion, counterexample: ...".
•	The Generator learns from errors: critiques remain in the graph as constraints.
•	The Arbiter avoids infinite iteration: it chooses by strict criteria.
•	Humanities problems are not reduced to numbers, but they are checked for logical correctness, consistency, and source justification.
________________________________________
Notes
This README is intended as a conceptual specification. Implementation details such as serialization, LLM integration, and 
