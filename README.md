# Ichthus Coding Conventions & Design Rationale

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

> **Purpose**  
> This document defines the engineering standards, coding conventions, architectural guidelines, and deliberate deviations from common industry “best practices” used across **all projects developed under Ichthus Development**.  
>
> Its purpose is to ensure clarity, consistency, and long-term maintainability across a mixed-language (.NET) ecosystem, while favoring explicit design and intentional tradeoffs over trend-driven convention.

---

## 1. Guiding Principles

These principles override tooling trends, framework fashion, and external style guides.

1. **Clarity over convention**  
   Readability and semantic correctness take precedence over popularity.

2. **Explicit beats implicit**  
   Hidden behavior (magic defaults, inferred namespaces, silent conversions) is avoided wherever possible.

3. **Consistency over popularity**  
   A consistently applied rule is preferred over an externally fashionable one.

4. **APIs are contracts**  
   Public APIs are designed to be stable, intentional, and unsurprising across languages.

5. **Tooling serves the developer—not the reverse**  
   IDEs, analyzers, and frameworks must not dictate architectural correctness.

---

## 2. Language Usage (VB.NET and C#)

Ichthus Development intentionally mixes **VB.NET** and **C#** within the same solution space.

### 2.1 Why VB.NET Exists Here

- Mature, expressive syntax for domain modeling
- Strong readability for business logic and data-heavy code
- Deep familiarity and long-term maintainability

### 2.2 Why C# Exists Here

- Broader ecosystem expectations
- Third-party library interoperability
- Modern tooling support

### 2.3 Cross-Language Rule

> **All public APIs must look natural, intentional, and idiomatic when consumed from both VB.NET and C#.**

Language-specific features must not leak into shared contracts.

---

## 3. Namespace and Project Structure

### 3.1 Root Namespace Policy (VB.NET)

- **VB.NET Root Namespace is always blank**
- All namespaces are declared explicitly in code

Rationale:
- Eliminates VB/C# impedance mismatch
- Prevents “hidden” namespace concatenation
- Makes public API shape explicit and predictable

---

### 3.2 Core Namespaces

Namespaces ending in `.Core` represent:

- Dependency-safe contracts
- Fundamental domain models
- Interfaces and abstractions intended for long-term stability

Rules:
- `*.Core` **must not** depend on non-Core namespaces
- Implementations may depend on Core
- Core namespaces should be safe to reference from any project

Example:

```
Ichthus.Text.JSON.Core
```

---

### 3.3 Namespace Stability and Versioning

Namespaces are treated as part of the public API surface.

Rules:
- Public namespaces must be considered stable once released
- Moving a public type to a different namespace is a breaking change
- Namespace refactors require explicit versioning or migration strategy

Rationale:
- Namespaces communicate domain ownership and responsibility
- Consumers bind to namespaces implicitly through imports/usings
- Treating namespaces as disposable leads to silent downstream breakage

### 3.4 Namespace Semantics

Certain namespaces within Ichthus carry **architectural meaning**, not just organizational grouping.

These namespaces communicate responsibility and intent and must be used consistently.

#### 3.4.1 `Core`
- Dependency-safe contracts
- Stable domain abstractions
- No external or implementation-specific dependencies

#### 3.4.2 `IO`
- Stream-based or persistence-oriented operations
- Files, sockets, buffers, or durable data sinks/sources
- Implies seek/read/write semantics

Examples:
- File readers/writers
- Network streams
- Memory-backed buffers

#### 3.4.3 `Console`
- Interactive, presentation-oriented output
- Human-facing input/output
- Not assumed to be durable or stream-seekable

Examples:
- Console writers
- Menu systems
- Interactive prompts

#### 3.4.4 `Diagnostics`
- Structured reporting of non-fatal conditions, validation issues, and system observations
- Not responsible for presentation or persistence
- May be consumed by logging, UI, telemetry, or test harnesses

Diagnostics represent facts; interpretation is the responsibility of the consumer.

#### 3.4.5 `Text`
- Textual data handling, parsing, tokenization, and transformation
- Format-aware but transport-agnostic
- Does not imply persistence, IO, or UI concerns

Examples:
- Parsers
- Tokenizers
- Format grammars

#### 3.4.6 `Policies`
- Behavioral rules and decision models
- No execution logic
- Used to influence how other components behave

Policies define *what should happen*, not *how it happens*.

#### 3.4.7 `Security`
- Security-sensitive utilities and abstractions
- Explicit, opt-in usage
- No silent encryption, hashing, or obfuscation

All security behavior must be visible at the call site.

#### 3.4.8 `Cryptography`
- Cryptographic primitives and transformations
- Explicit encryption, decryption, signing, verification, and hashing operations
- No implicit key management, storage, or policy decisions

Cryptographic operations must be:
- Explicit at the call site
- Opt-in
- Transparent in intent

#### 3.4.9 Other Domain Namespaces
Additional namespaces (e.g., `EDI`, `JSON`, `HTTP`) must define a clear responsibility boundary and should not overlap in purpose.

Namespaces such as Utilities, Helpers, or Common are discouraged and should be treated as refactoring waypoints, not architectural destinations.

Poorly defined namespaces must be corrected rather than compensated for with verbose type names.

---

## 4. Naming Conventions

### 4.1 Acronyms and Initialisms

**All acronyms are written in ALL CAPS.**

Examples:
- EDI
- XML
- JSON
- ASCII
- HL7

Rationale:
- Acronyms represent discrete semantic concepts
- Pascal-casing acronyms obscures meaning

Correct:
- `EDIParser`
- `XMLSerializer`

Incorrect:
- `EdiParser`
- `XmlSerializer`

---

### 4.2 General Naming Rules

- Public members: **PascalCase**
- No camelCase for public APIs
- Private members may follow language-idiomatic conventions
- Names should prioritize meaning over brevity

---

### 4.3 Constants

- Constants are declared using **SCREAMING_SNAKE_CASE**

Example:

```vbnet
Public Const MAX_RETRY_COUNT As Integer = 5
```

Rationale:
- Visually distinguishes constants from variables
- Aligns with their immutable, global nature

---

### 4.4 Internal vs External Serialization Conventions

- Internal models should follow Ichthus naming conventions (PascalCase, ALL-CAPS acronyms).
- External serialization formats (e.g., JSON for public APIs) may adapt naming conventions as required for interoperability.
- Serialization naming differences must be explicit and intentional, not implicit or ad-hoc.

Rationale:
- Preserves internal clarity without sacrificing external compatibility.

---

### 4.5 Domain-Oriented Naming and Namespace Responsibility

Ichthus Development favors **domain-oriented namespaces** over redundant type prefixes.

#### 4.5.1 Namespace Carries Semantic Weight

When a type exists within a clearly defined domain namespace, **the namespace—not the type name—carries the primary semantic meaning**.

Redundant repetition of the domain name in type identifiers is discouraged.

Example:

```vbnet
Namespace Ichthus.EDI
    Public Interface IDelimiterDetector
End Namespace
```

Preferred usage:

```vbnet
Dim detector As IDelimiterDetector
```

Avoid:

```vbnet
IEDIDelimiterDetector
```

Rationale:
- Reduces visual noise
- Improves readability
- Encourages meaningful namespace design
- Prevents "stuttering" identifiers (`EDI.EDIDelimiterDetector`)

---

#### 4.5.2 When Domain Prefixes Are Acceptable

Domain prefixes may be retained only when the concept itself is cross-domain or taxonomy-like, and may reasonably appear outside its defining namespace.

Example:

```vbnet
Public Enum EDIFormat
    Unknown
    X12
    EDIFACT
End Enum
```

Rationale:
- `Format` alone is ambiguous across domains
- `EDIFormat` may appear in diagnostics, UI, metadata, or logging contexts
- Prefix improves clarity when consumed outside `Ichthus.EDI`

This exception is intentional and limited.

---

#### 4.5.3 Explicit Qualification Over Renaming

When name collisions occur across domains (e.g., `Writer`, `Reader`, `Parser`), explicit qualification or aliasing is preferred over renaming types.

Examples:

**VB.NET:**

```vbnet
Imports EDIWriter = Ichthus.EDI.IO.Writer
Imports ConsoleWriter = Ichthus.Console.Writer
```

**C#**

```csharp
using EDIWriter = Ichthus.EDI.IO.Writer;
using ConsoleWriter = Ichthus.Console.Writer;
```

Rationale:
- Preserves clean, intention-revealing type names
- Avoids artificial suffixes (`EDIWriter`, `ConsoleWriter`)
- Keeps APIs natural and idiomatic
- Leverages language features instead of encoding context into names

*Type aliasing is considered a first-class language feature and an intentional part of Ichthus API consumption patterns.*

---

#### 4.5.4 Design Implication

This convention places higher importance on namespace architecture.

As a result:
- Namespaces must be intentional
- Namespace boundaries define responsibility
- Type names should remain concise, descriptive, and domain-local

*Poorly designed namespaces are not compensated for with verbose type names.*

---

## 5. Code Structure & Architectural Preferences

### 5.1 Variable Declaration Style

- Local variables should be declared at the beginning of a method whenever practical.
- Variables scoped to blocks may be declared early and initialized to `Nothing` or an empty value.
- Exceptions are permitted when early returns prevent unnecessary allocation.

Rationale:
- Improves scanability
- Reduces mid-method cognitive load

---

### 5.2 Data Access and Object Design

- Centralized data access patterns are preferred.
- Lazy-loaded properties are acceptable when they improve performance or clarity.
- Attribute-based mapping and serialization control are preferred over convention-only approaches.

*Framework-specific abstractions must not leak into Core or domain contracts.*

---

### 5.3 Type Semantics

- Prefer rich types (e.g., `FileInfo`) over primitive representations (e.g., file paths as strings) when behavior matters.

---

### 5.4 Server-Side Technology Preference

- ASP.NET is the preferred platform for server-side logic and validation.
- Other technologies may be used only when constraints require it.

---

## 6. Code Documentation

### 6.1 XML Documentation Comments

- XML documentation comments are required for all public types and members
- Internal members should be documented where intent is not obvious

Rationale:
- Improves IntelliSense across languages
- Serves as living documentation
- Forces clarity of intent at design time

---

### 6.2 Comment Philosophy

- Comments should explain why, not what
- Redundant comments are discouraged
- Historical or rationale-based comments are acceptable and encouraged

---

## 7. Code Organization

### 7.1 `#Region` (VB.NET) and `#region` (C#) Usage

- `#Region` blocks are used intentionally to organize:
  - Public properties
  - Private fields
  - Constructors
  - Public methods
  - Private methods
  - Event handlers

Rationale:
- Improves navigability in large files (code folding in IDE)
- Encourages logical grouping
- Aids long-term maintainability

---

## 8. Error Handling and Diagnostics

### 8.1 Diagnostics over Exceptions

Libraries should:
- Prefer structured diagnostics over throwing exceptions
- Reserve exceptions for unrecoverable or truly exceptional conditions

Decision Boundary:
- Libraries emit diagnostics and continue execution where possible
- Consumers are responsible for interpreting diagnostics
- Escalation (e.g., throwing exceptions) is a policy decision, not a library default

This allows:
- Batch processing without hard failure
- Partial success scenarios
- Environment-specific strictness (development vs production)

---

### 8.2 Severity Model

Diagnostics use a shared severity model:
- Information
- Warning
- Error
- Critical
- Fatal

Severity models should be consistent across all Ichthus projects.

---

## 9. UI Separation

Rules:
- No UI code in Core or shared libraries
- No `MessageBox`, dialogs, or UI callbacks in reusable logic
- Libraries must be safe for headless and server-side execution

Rationale:
- Enables batch processing
- Enables pipeline execution
- Prevents hidden coupling

---

## 10. Data Handling Philosophy

- Prefer immutable or read-only data structures where practical
- Preserve raw input alongside parsed or transformed representations
- Never silently normalize or coerce data without documentation

---

## 11. Conscious Deviations from Common Best Practices

As defined above, Ichthus Development intentionally deviates from some mainstream .NET conventions.

These deviations are:
- Deliberate
- Documented
- Consistently enforced

Examples include:
- ALL-CAPS acronyms instead of PascalCase acronyms
- Explicit namespace declarations
- SCREAMING_SNAKE_CASE constants
- Heavy use of diagnostics instead of exception-driven control flow
- Resistance to "magic" frameworks and opaque behavior

These choices exist to improve:
- Long-term maintainability
- Cross-language clarity
- Debuggability
- Architectural transparency

---

## 12. Living Document

This document is expected to evolve.

Changes must:
- Be intentional
- Include rationale
- Be applied consistently across all Ichthus Development projects

---

## License

This documentation is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

---

_Ichthus Engineering Standards exist to serve understanding, not fashion._

© Ichthus Development
