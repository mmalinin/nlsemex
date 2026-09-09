For this project, I would define the compiler scope **narrowly**. The compiler should not attempt to become a general-purpose replacement for an LLM. Its job is to convert a bounded class of natural-language knowledge into an explicit, executable, and verifiable representation.

### Compiler scope

**The compiler accepts:**

1. **Controlled technical language**

   * facts
   * definitions
   * relationships
   * properties and states
   * procedures
   * constraints
   * conditions
   * causal/dependency statements
   * quantities, units, and ranges
   * tool/API descriptions
   * structured operational documentation

2. **Natural-language variants of known semantics**

   * synonyms
   * paraphrases
   * common grammatical variations
   * domain-specific terminology
   * references and anaphora where resolvable

3. **Previously compiled knowledge**

   * terminology
   * semantic primitives
   * extraction rules
   * canonicalization rules
   * schemas
   * procedures
   * validation rules
   * regression tests

The output is:

```text
Natural language
      ↓
Linguistic analysis
      ↓
Semantic extraction
      ↓
Canonicalization
      ↓
Semantic IR
      ↓
Validation
      ↓
Knowledge / procedure graph
      ↓
Executable planning primitives
```

The compiler therefore has **two outputs**:

* **domain knowledge** — facts, procedures, constraints, terminology;
* **compiler knowledge** — rules describing how future language should be interpreted and compiled.

---

## What is inside the compiler

### 1. Linguistic frontend

Responsible for turning text into a structural representation.

It handles:

* tokenization
* morphology
* syntax/dependencies
* entity references
* terminology
* semantic roles
* controlled-language grammar
* known linguistic constructions

It does **not** need to understand arbitrary human language equally well.

---

### 2. Semantic compiler

Its central responsibility is:

> Map different linguistic expressions that have the same operational meaning into the same semantic representation.

For example:

```text
"Switch the pump off."
"Turn off the pump."
"Stop the pump."
```

may compile to the same representation **only if the domain establishes that these operations are semantically equivalent**.

The compiler must not infer equivalence merely because expressions appear similar.

---

### 3. Semantic IR

This is the compiler's central contract.

Something approximately like:

```text
ENTITY
PROPERTY
STATE
RELATION
FACT
ACTION
CONDITION
CONSTRAINT
PROCEDURE
PARAMETER
RESOURCE
EFFECT
FAILURE
```

The IR must be substantially smaller and more rigid than natural language.

The critical design goal is:

> **Many linguistic forms → one canonical semantic form.**

---

### 4. Knowledge compiler

The IR is compiled into persistent structures:

```text
entities
relations
facts
states
procedures
constraints
dependencies
terminology
provenance
```

The resulting knowledge should be independently usable without reconstructing the original prose.

---

### 5. Procedure compiler

Procedures become executable state transitions rather than textual instructions.

For example:

```text
PRECONDITION:
    pump.state == stopped

ACTION:
    open(valve)

EFFECT:
    valve.state == open
```

This enables:

* applicability checking
* planning
* simulation
* contradiction detection
* execution
* regression testing

---

### 6. Validator

The validator is deliberately **not the LLM**.

It checks things such as:

* type correctness
* schema validity
* missing references
* impossible states
* violated constraints
* inconsistent units
* invalid procedure transitions
* unreachable states
* contradictory facts
* duplicate/equivalent knowledge
* invalid tool arguments

This is what turns the system from a text transformation pipeline into a compiler.

---

### 7. Planner/executor

The compiler may expose compiled procedures as executable operations.

Its scope is:

```text
goal
 ↓
known state
 ↓
applicable procedures
 ↓
candidate plan
 ↓
precondition validation
 ↓
execution
 ↓
effect validation
```

The planner should reason over **explicit state**, rather than asking an LLM to invent a plan from prose.

---

### 8. Meta-teacher interface

The LLM is outside the trusted compiler core.

It is allowed to propose:

```text
new extraction rule
new canonicalization rule
new semantic primitive
new fact
new procedure
new ambiguity
new test
```

But it does not directly modify trusted knowledge.

Conceptually:

```text
                 ┌──────────────┐
                 │   LLM teacher│
                 └──────┬───────┘
                        │ proposals
                        ↓
              ┌────────────────────┐
              │ compiler validation │
              └─────────┬──────────┘
                        │
                  accepted rules
                        ↓
                 ┌──────────────┐
                 │    compiler  │
                 └──────────────┘
```

This distinction is fundamental.

---

# Compiler limits

The most important part is explicitly defining what the compiler **does not promise**.

### 1. It is not a general natural-language interpreter

It should not promise:

> "Any English sentence can be converted into correct semantic IR."

Instead:

> "The compiler supports a defined semantic domain and a progressively expanding language subset."

Unsupported language is a normal compiler outcome.

```text
UNSUPPORTED_CONSTRUCTION
AMBIGUOUS
INSUFFICIENT_KNOWLEDGE
CONFLICTING_KNOWLEDGE
UNVERIFIABLE
```

are legitimate results.

---

### 2. It cannot recover information that isn't present

If:

> "The valve was adjusted."

doesn't specify who adjusted it, when, or to what setting, the compiler cannot legitimately manufacture those facts.

It must preserve the incompleteness.

---

### 3. It cannot reliably resolve unrestricted ambiguity

For example:

> "Open the door with the key."

could encode different relationships depending on context.

If the knowledge base cannot disambiguate it, compilation should stop or produce multiple explicitly marked interpretations.

**Ambiguity must not silently become a fact.**

---

### 4. It does not replace world knowledge

The compiler can know:

```text
human(X) → mammal(X)
```

if that rule exists.

It cannot automatically acquire arbitrary common-sense knowledge merely because something sounds obvious.

The project should distinguish:

```text
KNOWN
DERIVED
ASSUMED
UNKNOWN
```

very strictly.

---

### 5. It does not guarantee semantic truth

A compiler can establish:

> "This document consistently compiles to representation X."

It cannot establish:

> "The document describes reality correctly."

That requires external evidence.

Therefore provenance should remain attached to compiled knowledge.

```text
FACT
 ├─ source
 ├─ source version
 ├─ location
 ├─ extraction rule
 └─ verification status
```

---

### 6. It does not need to reproduce arbitrary LLM reasoning

This is perhaps the most important boundary.

The objective is **not**:

```text
LLM
   ↓
smaller LLM
```

nor:

```text
LLM
   ↓
reimplementation of everything the LLM knows
```

The objective is:

```text
LLM
   ↓
useful recurring semantic behavior
   ↓
explicit rules
```

Anything that cannot be profitably expressed in the compiler's representation can remain outside it.

---

### 7. Probabilistic interpretation remains at the boundary

Initially, the teacher may be necessary for difficult cases:

```text
                 deterministic core
                       ↑
                       │
natural language → LLM-assisted boundary
```

The goal is to progressively move the boundary outward:

```text
Phase 1
LLM ─────────────── compiler

Phase 2
LLM ─────── compiler

Phase 3
LLM ── compiler

Phase 4
LLM ─ compiler
    ↑
exception handling only
```

The compiler does **not** need to eliminate probabilistic computation completely. It needs to eliminate it from places where deterministic representation and verification provide a better solution.

---

# The hard boundary

I would make this an explicit project principle:

> **The compiler is responsible for semantics that can be represented, validated, and executed within its defined semantic model. It is not responsible for arbitrary linguistic interpretation, unstated world knowledge, truth of external information, or reasoning that cannot be reduced to a verifiable representation.**

This gives the project a very useful failure contract:

```text
                INPUT
                  │
          Can it be parsed?
             /          \
           no            yes
           │              │
       unsupported     Can semantics
                        be resolved?
                       /          \
                     no            yes
                     │              │
                 ambiguous      Can IR validate?
                                /          \
                              no            yes
                              │              │
                           rejected       COMPILED
```

That is a much healthier target than "understand natural language."

### In one sentence

**The compiler's scope is to transform a bounded, extensible subset of natural-language technical knowledge into a canonical semantic IR and then into validated persistent facts and executable procedures; its limit is everything that cannot be represented, disambiguated, validated, or externally verified within that semantic model.**

That boundary also makes the self-extension mechanism meaningful: **the compiler can expand its own language coverage, but every expansion must itself become an explicit, testable compiler artifact.**
