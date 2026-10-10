Here is the GUM Algorithm specification in English, without pseudocode, code blocks, or tables.

# GUM Algorithm: A Versioned Reasoning Graph for Humanities

A formal, checkpointed reasoning engine for humanities-style arguments, combining graph versioning, generator/critic/arbiter agents, and rollback-based refinement.

## Overview

The GUM Algorithm models reasoning as a persistent versioned graph rather than a simple DAG. It supports branches, checkpoints, critiques, contradictions, and rollbacks. The system separates reasoning into three roles:

- Generator / Solver — proposes graph extensions and branches.
- Critic — formally validates or refutes each inference.
- Arbiter — resolves conflicts when branches remain consistent or iteration diverges.

The goal is not to reduce humanities problems to numbers, but to check them for logical correctness, consistency, and source justification.

## 1. Core: Versioned Reasoning Graph

The graph is a persistent versioned graph with branches and checkpoints.

### Node Types

- Concept — stores a concept, term, or theory.
- Claim — stores a proposition or statement.
- Source — stores an author, organization, or document.
- Decision — stores the main question or a candidate decision.
- Assumption — stores an assumption or hypothesis.
- Inference — stores an inference rule applied to premises.
- Checkpoint — stores a snapshot of branch state.
- Critique — stores a critic’s remark.
- Contradiction — stores a contradiction.
- Undecidable — stores an undecidable self-referential point.

### Edge Types

- supports — one node supports another.
- attacks — one node attacks another.
- undercuts — a node undermines an inference.
- infers — a conclusion follows from an inference.
- contradicts — logical contradiction between two nodes.
- equivalent — equivalence between two nodes.
- derives — formal derivation between two nodes.
- has_source — a claim has a source.
- assumes — a claim assumes an assumption.
- criticized_by — a claim is criticized by a critique.
- checkpoint_of — a checkpoint belongs to a branch.
- rollback_to — a branch is rolled back to a checkpoint.

### Node Attributes

Each node can carry an id, type, formula, logic type, status, confidence value, source, provenance, branch, checkpoint, critique list, and assumptions list. These attributes record not only what the node says, but also where it came from, how reliable it is, and which branch or checkpoint it belongs to.

### Statuses

- T — confirmed.
- F — refuted.
- U — unknown.
- Ø — undecidable.
- I — contradiction.

## 2. Agent Roles

### Generator / Solver

Builds the graph, applies heuristics, and proposes branches. It may use an LLM, but every proposal must be represented as a formal formula or a reference to an axiom or rule.

### Critic

Relies on strict formal logic:

- Propositional logic: truth tables, tautologies, DNF and CNF, De Morgan’s laws, contraposition, modus ponens, modus tollens, syllogisms, dilemmas.
- Predicate logic: quantifiers, prenex normal form, validity.
- Validation: premises entail a conclusion if and only if the conjunction of premises with the negation of the conclusion is unsatisfiable.
- Methods: resolution, proof by contradiction, search for a minimal unsatisfiable core.

The Critic does not “evaluate”; it proves or refutes each step.

### Arbiter

Triggered if the Generator and Critic diverge for more than N iterations, or if multiple consistent branches remain after rollback. The Arbiter decides by formal criteria:

1. Minimality of assumptions — Occam’s razor.
2. Reliance on sources with maximum reliability.
3. Robustness to counterfactuals.
4. Grounded extension in argumentation semantics.

## 3. Iteration Protocol with Rollback

Initialization:

- The graph is created from terms, claims, sources, and a decision.
- An initial checkpoint is created.
- The first branch begins.

The iteration proceeds as follows:

1. The Generator proposes an extension: new nodes, edges, inferences, and branches.
2. The Critic formally validates the extension:
   - For each inference, it checks whether the premises entail the conclusion.
   - It checks consistency: no claim and its negation may both hold.
   - It checks quantifiers, normal forms, and resolution.
   - It checks sources: no thesis substitution, no post hoc reasoning.
   - If an error is found, it creates a Critique node and attaches it to the guilty node.
3. If the Critic finds an error:
   - The Generator rolls back to the last checkpoint that does not contain the error.
   - The Critique node remains in the graph as a constraint: do not use inference X without proof Y.
   - The Generator builds a new branch considering the critique.
   - The process returns to step 1.
4. If the Critic confirms:
   - The branch is accepted.
   - A new checkpoint is created.
   - If unknown nodes remain, the Generator continues branching.
5. If divergence exceeds N iterations:
   - The Arbiter selects the final branch or decision.
   - The answer is fixed.

## 4. Formal Logic in the Critic

Each claim is translated into a predicate logic formula.

For example, consider three statements:

- Wage growth causes cost growth.
- Cost growth causes inflation.
- Therefore, wage growth causes inflation.

The Critic checks whether the conjunction of the first two statements logically entails the third. If the implication is a tautology, the inference is correct. If it is not, a Critique is created.

To check validity, the Critic may use:

- reduction to perfect CNF or perfect DNF;
- the resolution method;
- proof by contradiction: add the negation of the conclusion and search for the empty clause.

If the formula is not valid, the Critic produces a counterexample: an assignment of predicate values where the premises are true and the conclusion is false. This is how the error is exposed.

## 5. Example: Inflation

1. The Generator builds branch B1:
   - Source S_CentralBank says the cause is wage growth.
   - Claim P1: wage growth leads to cost growth, which leads to inflation.
2. The Critic checks:
   - Formally, the inference is correct.
   - But source S_CentralBank may be biased.
   - The Critic creates critique K1: S_CentralBank has high bias; independent confirmation is required.
3. The Generator rolls back to CP0, but K1 remains as a constraint.
   It builds branch B2: adds S_TradeUnions and claim P2: monopoly prices lead to cost growth, which leads to inflation.
4. The Critic checks B2:
   - Formally correct.
   - Sources conflict, but there is no logical contradiction.
5. The Arbiter:
   - Both branches are consistent.
   - Chooses a synthesis: inflation is multifactorial.
   - Final decision D: policy should account for both factors.

## 6. What This Gives You

- Reasoning history is preserved: every step, every critique, every rollback.
- The Critic is formal: it does not say “I think”; it says “the premises do not entail the conclusion, counterexample: ...”.
- The Generator learns from errors: critiques remain in the graph as constraints.
- The Arbiter avoids infinite iteration: it chooses by strict criteria.
- Humanities problems are not reduced to numbers, but they are checked for logical correctness, consistency, and source justification.

## Notes

This README is intended as a conceptual specification. Implementation details such as storage, serialization, LLM integration, and argumentation semantics are left open.
