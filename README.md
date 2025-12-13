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
