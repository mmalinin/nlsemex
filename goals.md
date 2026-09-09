# Explicit Semantic Knowledge Compiler

## Project premise

Large Language Models are exceptionally effective at recognizing patterns in natural language and using those patterns to produce useful answers and actions. However, a substantial part of the knowledge represented by an LLM is **implicit**.

The model does not normally contain an explicit, inspectable database of the facts it learned, nor an explicit representation of the procedures by which those facts can be manipulated. Instead, knowledge is distributed across a very large set of parameters and accessed through probabilistic computation.

This creates several fundamental problems.

* It is difficult to determine exactly which information a model has captured.
* Individual facts cannot generally be inspected, validated, updated, or removed independently.
* Adding a new body of knowledge normally requires expensive inference, fine-tuning, retrieval infrastructure, or repeated context injection.
* Similar concepts and operations are represented redundantly through many linguistic forms.
* Reasoning over knowledge requires repeatedly executing a large neural network.
* The computational cost is dominated not only by the amount of knowledge, but also by the high-dimensional representation in which that knowledge is encoded.
* Outputs are probabilistic and therefore difficult to make fully reproducible.
* A model can produce a plausible answer without providing a mechanically verifiable chain connecting the answer to the underlying knowledge.

The central premise of this project is that a large part of the useful pattern-recognition capability of an LLM can instead be used to **compile implicit linguistic knowledge into explicit symbolic representations**.

Rather than using the LLM as the permanent storage and reasoning engine, use it as a teacher that discovers and improves a compact semantic compiler.

The resulting system should transform natural-language knowledge into a small, explicit, canonical representation consisting of facts, relationships, states, procedures, rules, constraints, and operations.

The intended result is a transition from:

```
implicit probabilistic knowledge
distributed neural representation
repeated expensive inference
```

toward:

```
explicit symbolic knowledge
compact canonical representation
deterministic validation and execution
```

The LLM provides the initial pattern-recognition capability. The compiler progressively captures useful parts of that capability as explicit rules.

---

# 1. Core hypothesis

A sufficiently strong LLM can recognize semantic equivalences, relationships, procedures, and patterns in technical text that are difficult to encode manually.

For example, the following expressions can describe essentially the same operation:

```
Press and hold the reset button until the indicator turns green.

Keep the reset button pressed until the status light becomes green.

Continue pressing the reset control until the green LED illuminates.
```

A conventional rule-based parser may treat these as substantially different linguistic structures.

An LLM can recognize their common semantic pattern.

The project proposes using that capability to discover a canonical representation:

```
ACTION:
    HOLD(reset_button)

TERMINATION:
    indicator.color == GREEN
```

The LLM therefore acts as a **semantic pattern recognizer**, while the resulting rule system becomes the reusable implementation of that recognition.

The objective is not to reproduce the LLM internally.

The objective is to **extract reusable regularities from its behavior and turn them into explicit computational rules**.

---

# 2. Fundamental representation principle

Every piece of imported knowledge should exist in at least two representations.

## Canonical human-readable representation

A restricted technical language should express the meaning in a stable and easily inspectable form.

Example:

```
The reset button must remain pressed until the indicator is green.
```

or, where appropriate:

```
ACTION:
    hold reset_button

UNTIL:
    indicator.color = GREEN
```

This representation is intended for humans.

It should be concise, stable, unambiguous, and easy to compare across documents and versions.

## Internal semantic representation

The same information should be represented in a typed symbolic intermediate representation:

```
action hold(
    target = reset_button
)

termination:
    indicator.color == GREEN
```

The internal representation is intended for parsing, validation, storage, inference, planning, and execution.

The two representations must be generated from the same semantic structure rather than independently invented.

This provides a human verification path through the system.

---

# 3. Architecture

The system consists of five major components.

## 3.1 Semantic extractor

The semantic extractor transforms natural language into a constrained semantic representation.

Its inputs include:

* vocabulary;
* lexical relations;
* terminology;
* syntactic patterns;
* semantic patterns;
* canonicalization rules;
* domain-specific rules;
* contextual information.

The extractor should progressively learn to recognize different ways of expressing the same semantic operation.

Its output is not arbitrary natural language.

It is a structured set of semantic candidates.

The extractor should be deterministic whenever the applicable rules are deterministic.

---

## 3.2 Knowledge representation

Extracted information is stored in an explicit knowledge representation.

The representation should distinguish at least:

```
ENTITY
PROPERTY
STATE
FACT
RELATION
EVENT
ACTION
CONDITION
CONSTRAINT
PROCEDURE
PARAMETER
RESOURCE
EFFECT
```

Facts represent what is known.

Procedures represent what can be done with the known entities and states.

A procedure should explicitly represent:

```
preconditions
actions
parameters
state transitions
effects
temporal dependencies
alternatives
failure conditions
required resources
```

The system must not treat procedures as merely text snippets.

A procedure is executable knowledge.

---

# 4. Canonicalization

Natural language contains enormous representational variability.

The same concept may be expressed through synonyms, paraphrases, different grammatical constructions, abbreviations, implicit references, or domain-specific terminology.

The canonicalizer collapses this variability.

For example:

```
shut down
power off
switch off
turn the device off
```

may resolve to:

```
POWER_OFF(device)
```

provided the domain semantics establish that these expressions are equivalent.

Canonicalization therefore reduces the number of distinct representations that the downstream system must process.

This is a central compression mechanism of the project.

The system should not merely store every sentence that describes a fact.

It should store the fact once, together with provenance showing how it was derived.

---

# 5. Symbolic validation

Every canonical representation should be validated independently of the teacher LLM whenever possible.

Validation includes:

* schema validity;
* type consistency;
* argument validity;
* unit consistency;
* reference resolution;
* state consistency;
* precondition/effect consistency;
* procedure reachability;
* contradictory facts;
* impossible transitions;
* duplicate or equivalent knowledge;
* violations of domain constraints.

Where possible, execution itself becomes a validation mechanism.

For example, if a generated procedure claims to transform:

```
state A → state C
```

the system can attempt to execute or simulate it.

If the required preconditions cannot be satisfied, the representation is rejected or returned for correction.

The symbolic evaluator is therefore an external source of truth.

The LLM is not trusted merely because it is confident.

---

# 6. Planning and execution

A user query should ultimately be converted into a canonical goal.

Example:

```
"Calibrate the sensor."
```

becomes:

```
GOAL:
    sensor.calibrated = TRUE
```

The planner searches the accumulated procedures for a valid sequence of actions that achieves the goal.

Example:

```
REQUIRE:
    sensor.connected(controller)

ACTION:
    start_calibration(sensor)

WAIT:
    calibration.state = COMPLETE

VERIFY:
    calibration.result = SUCCESS
```

The planner operates over explicit states and operations rather than asking an LLM to invent an answer from its implicit knowledge.

Tool calls should similarly have explicit contracts:

```
INPUT TYPES
PRECONDITIONS
EFFECTS
FAILURE STATES
OUTPUT TYPES
```

This turns tool use into a symbolic planning problem.

---

# 7. LLM as meta-teacher

The LLM is not the primary runtime knowledge store.

Its primary role is to supervise the development of the semantic compiler.

When the compiler encounters novel or ambiguous language, the teacher analyzes the source and proposes:

* the intended semantic structure;
* canonical terminology;
* missing extraction rules;
* modified extraction rules;
* generalized patterns;
* domain-specific interpretations;
* canonical human-readable text;
* additional facts;
* procedural representations;
* validation cases;
* adversarial examples;
* regression tests.

The teacher should produce structured artifacts rather than merely a corrected answer.

For example:

```
SOURCE
    "Keep the switch depressed until the green indicator appears."

DISCOVERED PATTERN
    HOLD(X) UNTIL(Y)

CANONICAL FORM
    hold(switch)
    until indicator.color = GREEN

RULE
    "keep <X> depressed until <Y>"
    → hold(X) until Y

TEST CASES
    "hold X until Y"
    "continue pressing X until Y"
    "maintain pressure on X until Y"
```

The teacher thus contributes to the **compiler itself**.

---

# 8. Continuous learning loop

Every newly imported document follows the same general cycle:

```
DOCUMENT
    ↓
semantic extraction
    ↓
canonicalization
    ↓
symbolic validation
    ↓
knowledge integration
    ↓
regression testing
```

If validation fails:

```
failure
    ↓
teacher analysis
    ↓
classify failure
    ↓
modify rule / add rule / modify representation
    ↓
generate additional tests
    ↓
re-run validation
```

A successful correction becomes a permanent regression case.

The system therefore accumulates not only knowledge but also knowledge about **how language expresses knowledge**.

This is the key learning mechanism.

---

# 9. Explicit generalization

The desired form of learning is not merely memorization.

Suppose the teacher encounters:

```
Hold button A until LED B turns green.
```

The system should not only record that sentence.

It should discover the semantic abstraction:

```
HOLD(X) UNTIL(Y)
```

and add it to the canonicalization rules.

Later, previously unseen expressions should be recognized using the generalized rule.

The learning process therefore has two outputs:

```
knowledge:
    facts and procedures

meta-knowledge:
    rules for recognizing and extracting facts and procedures
```

This distinction is fundamental.

The first extends the knowledge database.

The second extends the compiler.

---

# 10. Synthetic training and self-testing

The teacher should continuously generate synthetic examples of the discovered patterns.

For every rule:

```
natural-language variants
    ↓
semantic representation
    ↓
symbolic validation
```

The dataset should contain both positive and adversarial examples.

Examples should deliberately vary:

* synonyms;
* grammatical structure;
* word order;
* implicit references;
* nested conditions;
* temporal relationships;
* quantities;
* units;
* exceptions;
* negation;
* alternatives;
* domain terminology.

The objective is not maximum linguistic diversity for its own sake.

The objective is to discover the boundary of each semantic rule and make that boundary explicit.

---

# 11. Knowledge integration

Each documentation source becomes a compilable knowledge package.

A package should contain:

```
source metadata
vocabulary
canonical terminology
semantic rules
facts
procedures
constraints
validation rules
canonical documentation
provenance
regression tests
```

The package should be independently loadable into the global knowledge database.

Existing knowledge should be checked when new knowledge is added.

This allows the system to detect:

* contradictions;
* superseded facts;
* incompatible procedures;
* terminology conflicts;
* duplicate knowledge;
* changed tool behavior;
* newly introduced dependencies.

A document should therefore not simply be "added to context."

It should be **compiled into the accumulated knowledge state**.

---

# 12. Read-once memorization

A major project goal is read-once memorization.

Traditional LLM usage repeatedly pays the computational cost of interpreting source text.

This project instead aims for:

```
document
    ↓
one-time semantic compilation
    ↓
compact explicit representation
    ↓
persistent storage
```

After compilation, the original prose should not normally be required for routine queries.

The system should be able to answer from the compiled representation.

This creates a fundamentally different memory model:

```
RAG:
    store text
    retrieve text
    repeatedly interpret text

semantic compiler:
    interpret once
    store meaning
    query meaning
```

This is particularly attractive for large collections of technical documentation.

---

# 13. Compression objective

The project hypothesizes that natural-language knowledge contains substantial redundancy.

Many linguistic expressions correspond to the same semantic operation.

Instead of representing:

```
thousands of linguistic variations
```

the system attempts to represent:

```
one semantic concept
+ a small number of recognition rules
```

Likewise, instead of repeatedly encoding a procedure as natural-language text, it stores the procedure once as a symbolic state transition system.

This potentially reduces:

* storage;
* memory bandwidth;
* inference compute;
* context requirements;
* repeated parsing;
* retrieval overhead.

The important measurement is not merely bytes of storage.

The system should measure:

```
information represented
/ computational resources required
```

and compare this against equivalent LLM-based approaches.

---

# 14. Verification as a first-class property

The final representation must be human-verifiable.

A person should be able to inspect:

```
SOURCE
    ↓
CANONICAL FORM
    ↓
INTERNAL REPRESENTATION
    ↓
DERIVATION / PROVENANCE
```

and determine whether the system captured the source correctly.

A canonical representation should therefore favor explicitness over stylistic naturalness.

For example:

```
"The unit should normally be powered down before maintenance."
```

should not silently become:

```
maintenance.requires(powered_off)
```

unless "normally" has been resolved according to explicit domain semantics.

Uncertainty must remain explicit.

Possible representations include:

```
REQUIRED
RECOMMENDED
OPTIONAL
CONDITIONAL
UNKNOWN
```

The compiler must never convert uncertainty into certainty merely because the latter is easier to process.

---

# 15. Stable generation

Human-readable output should be generated from the semantic representation.

The same semantic structure should produce a stable canonical description.

This provides a useful round-trip invariant:

```
SOURCE
  ↓
PARSE
  ↓
IR
  ↓
CANONICAL ENGLISH
  ↓
PARSE
  ↓
equivalent IR
```

The second parse should be semantically equivalent to the first.

This creates a powerful automated consistency test.

---

# 16. Toolbench

A dedicated toolbench should be used to evaluate the system's operational capabilities.

Each tool should expose an explicit semantic contract.

The benchmark should test:

* tool identification;
* argument extraction;
* type conversion;
* precondition recognition;
* state tracking;
* multi-step planning;
* failure recovery;
* procedure composition;
* goal satisfaction;
* result interpretation.

The teacher can generate increasingly difficult tasks.

A task is accepted only when the symbolic executor confirms that the produced plan satisfies the goal.

The benchmark therefore becomes both:

```
training data
```

and:

```
compiler specification
```

---

# 17. Long-term objective: autonomous semantic compiler

The initial system may rely heavily on a strong LLM.

The desired progression is:

```
LLM-dependent parser
    ↓
LLM-assisted parser
    ↓
mostly symbolic parser
    ↓
symbolic parser + learned matching
    ↓
autonomous semantic compiler
```

The LLM eventually becomes a supervisor rather than an operational dependency.

Ideally, routine documentation parsing, querying, planning, and execution require no external LLM.

The external teacher is invoked only when:

* novel language is encountered;
* a rule fails;
* knowledge conflicts;
* a new semantic construct must be learned;
* the system needs to extend its own capabilities.

This is the stretch goal: a system capable of acting on its accumulated explicit knowledge and procedures while using the LLM only for **meta-learning and exceptional semantic discovery**.

---

# 18. Core research questions

The project should experimentally investigate:

1. How much useful technical knowledge can be represented in a small semantic vocabulary?

2. How aggressively can linguistic variability be collapsed without losing meaning?

3. How many explicit rules are required to cover a given documentation domain?

4. Can LLM-generated rules generalize to unseen linguistic constructions?

5. Can a rule system progressively reproduce useful portions of an LLM's semantic pattern recognition?

6. How much storage reduction is possible compared with raw documents or embeddings?

7. How much inference compute is saved when queries operate on explicit knowledge?

8. How reliably can symbolic validation detect semantic extraction errors?

9. Can failed executions automatically identify missing semantic rules?

10. Can regression-driven teacher feedback produce monotonic improvement?

11. Can newly learned rules remain isolated enough to avoid catastrophic regressions?

12. Can a compiled knowledge package be transferred between independent systems without requiring the original teacher?

13. Can procedures extracted from documentation be executed safely using symbolic precondition/effect validation?

14. At what point does the symbolic system become capable of operating without the teacher?

---

# 19. Success criteria

The project is successful if it demonstrates that a limited-domain technical corpus can be transformed into:

```
compact explicit knowledge
+
explicit executable procedures
+
explicit semantic extraction rules
+
human-readable canonical documentation
```

such that:

* knowledge can be imported in approximately one semantic pass;
* facts can be independently inspected;
* procedures can be mechanically validated;
* queries can be answered from the compiled representation;
* actions can be planned from explicit procedures;
* tool calls can be validated before execution;
* new linguistic forms can cause the rule system to generalize;
* improvements become permanent regression-tested capabilities;
* the same knowledge can be loaded into another instance of the system;
* routine operation does not require the original LLM.

---

# 20. Fundamental thesis

The project is based on a simple proposition:

> **LLMs are extremely good at discovering semantic patterns, but they are unnecessarily expensive and difficult to verify as permanent representations of those patterns.**

Instead of treating the LLM's implicit knowledge as the final product, use its pattern-recognition ability as a **compiler construction mechanism**.

The LLM observes natural language and identifies equivalence classes, facts, relationships, procedures, and transformations.

The compiler turns those discoveries into explicit rules.

The symbolic system validates the resulting representation.

The knowledge database stores the result.

The planner operates on that explicit representation.

The generator converts the result back into stable human-readable language.

The teacher observes failures and extends the compiler.

The resulting loop is:

```
implicit neural pattern recognition
                ↓
        semantic discovery
                ↓
          explicit rules
                ↓
        canonical knowledge
                ↓
      symbolic validation
                ↓
      executable reasoning
                ↓
          observed failure
                ↓
         teacher feedback
                ↓
      improved semantic rules
                ↓
             repeat
```

The fundamental goal is therefore not to build a smaller LLM.

It is to **convert useful portions of neural pattern recognition into an explicit, compact, inspectable and executable computational system**.

If successful, each new document increases not only the amount of knowledge available to the system, but also its ability to recognize and compile future knowledge.

The long-term result is a self-extending knowledge compiler: a system in which linguistic variability is progressively converted into stable semantic primitives, implicit knowledge becomes explicit data and procedures, and probabilistic interpretation is replaced wherever possible by deterministic, reproducible, human-verifiable computation.
