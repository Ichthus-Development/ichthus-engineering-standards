<p align="right"><img width="120" height="96" alt="Ichthus Development logo" src="https://github.com/user-attachments/assets/acf27b44-5bb3-474c-ac0b-3d4ac58d9bbe" /></p>

# Ichthus Development Coding Conventions & Design Rationale

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

> **Purpose**  
> This document defines the engineering standards, coding conventions, architectural guidelines, and deliberate deviations from common industry "best practices" used across **all projects developed under Ichthus Development**.  
>
> Its purpose is to ensure clarity, consistency, and long-term maintainability across a mixed-language (.NET) ecosystem, while favoring explicit design and intentional tradeoffs over trend-driven convention.

## Non-Goals

This document intentionally does not attempt to:

- Serve as an exhaustive language reference or tutorial  
  These standards assume baseline professional competence in the languages and platforms being used.
- Replace official language, framework, or platform documentation  
  Vendor and standards-body documentation remains authoritative for syntax, runtime behavior, and compliance details.
- Prescribe stylistic preferences for every language or framework  
  Only conventions with architectural, maintainability, or cross-language impact are specified here.
- Dictate implementation details where multiple correct approaches exist  
  This document defines boundaries, contracts, and expectations—not micromanaged implementations.
- Enforce personal or aesthetic preferences unrelated to clarity, correctness, or maintainability  
  Rules exist to improve understanding and reduce ambiguity, not to satisfy stylistic taste.
- Retroactively rewrite legacy code to conform unless actively modified  
  These standards apply forward-looking unless otherwise documented.
- Guarantee performance, security, or correctness by their existence alone  
  Standards guide design and review; they do not replace engineering judgment, testing, or validation.
- Define organizational values or ethics  
  Organizational principles are defined separately (see the Babbagic Code).

## Relationship to Industry Best Practices

Ichthus Development standards are based primarily on established industry best practices across software engineering, data engineering, and system design.

Where this document defines deviations from commonly accepted practices, those deviations are intentional, experience-driven exceptions rather than wholesale rejections of industry guidance. Such exceptions are documented explicitly and exist to improve clarity, safety, cross-language interoperability, or long-term maintainability in real-world systems.

These standards apply to application code, libraries, APIs, data pipelines, and database artifacts produced under Ichthus Development.

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

6. **Tool-Independent Readability**  
   Code should be readable and understandable without reliance on IDE features such as syntax highlighting, IntelliSense, or visual designers.

   Naming conventions, formatting, and structure are chosen to preserve semantic clarity when viewed in plain text editors or minimal environments.

   Tooling may enhance productivity, but it MUST NOT be required to understand intent.

   This principle informs naming (§5), documentation (§7), and SQL standards (§13).

These standards are informed by the principles articulated in the Gold Fish Bowl Babbagic Code.

---

## 2. Terminology and Rule Severity

The following terms are used throughout this document:

- **MUST** — A mandatory requirement. Violations are considered defects.
- **SHOULD** — A strong recommendation. Deviations require justification.
- **MAY** — An optional guideline. Context-dependent.

These terms are used intentionally to enable future mechanical enforcement through tooling and to remove ambiguity during code review.

Lowercase must / should / may are descriptive prose only.  
Uppercase MUST / SHOULD / MAY define enforceable rules.

Usage of the terms "preferred" and "discouraged" indicate a strong default, not a prohibition.

For the purposes of this document, "public API" refers to any type, member, schema, or contract intended for consumption outside its defining assembly, project, or bounded context.

---

## 3. Language Usage (VB.NET and C#)

Ichthus Development intentionally mixes **VB.NET** and **C#** within the same solution space.

### 3.1 Why VB.NET Exists Here

- Mature, expressive syntax for domain modeling
- Strong readability for business logic and data-heavy code
- Deep familiarity and long-term maintainability

### 3.2 Why C# Exists Here

- Broader ecosystem expectations
- Third-party library interoperability
- Modern tooling support

### 3.3 Cross-Language Rule

All public APIs MUST look natural, intentional, and idiomatic when consumed from both VB.NET and C#.

Language-specific features MUST NOT leak into shared contracts or public abstractions.

---

## 4. Namespace and Project Structure

### 4.1 Root Namespace Policy (VB.NET)

- **VB.NET Root Namespace is always blank**
- All namespaces are declared explicitly in code

Rationale:
- Eliminates VB/C# impedance mismatch
- Prevents "hidden" namespace concatenation
- Makes public API shape explicit and predictable

---

### 4.2 Core Namespaces

Namespaces ending in `.Core` represent:

- Dependency-safe contracts
- Fundamental domain models
- Interfaces and abstractions intended for long-term stability

Rules:
- `*.Core` MUST NOT depend on non-Core namespaces
- Implementations MAY depend on Core
- Core namespaces SHOULD be safe to reference from any project

Example:

```
Ichthus.Text.JSON.Core
```

For the purposes of this document, `Core` refers to both a conceptual architectural boundary and, where applicable, a physical project or assembly boundary.

---

### 4.3 Namespace Stability and Versioning

Namespaces are treated as part of the public API surface.

Rules:
- Public namespaces MUST be considered stable once released
- Moving a public type to a different namespace is a breaking change
- Namespace refactors require explicit versioning or migration strategy

Rationale:
- Namespaces communicate domain ownership and responsibility
- Consumers bind to namespaces implicitly through imports/usings
- Treating namespaces as disposable leads to silent downstream breakage

### 4.4 Namespace Semantics

Certain namespaces within the Ichthus root namespace carry **architectural meaning**, not just organizational grouping.

These namespaces communicate responsibility and intent and must be used consistently.

#### 4.4.1 `Core`

- Dependency-safe contracts
- Stable domain abstractions
- No external or implementation-specific dependencies

#### 4.4.2 `IO`

- Stream-based or persistence-oriented operations
- Files, sockets, buffers, or durable data sinks/sources
- Implies seek/read/write semantics

Examples:
- File readers/writers
- Network streams
- Memory-backed buffers

#### 4.4.3 `Console`

- Interactive, presentation-oriented output
- Human-facing input/output
- Not assumed to be durable or stream-seekable

Examples:
- Console writers
- Menu systems
- Interactive prompts

#### 4.4.4 `Diagnostics`

- Structured reporting of non-fatal conditions, validation issues, and system observations
- Not responsible for presentation or persistence
- May be consumed by logging, UI, telemetry, or test harnesses

Diagnostics represent facts; interpretation is the responsibility of the consumer.

#### 4.4.5 `Text`

- Textual data handling, parsing, tokenization, and transformation
- Format-aware but transport-agnostic
- Does not imply persistence, IO, or UI concerns

Examples:
- Parsers
- Tokenizers
- Format grammars

#### 4.4.6 `Policies`

- Behavioral rules and decision models
- No execution logic
- Used to influence how other components behave

Policies define *what should happen*, not *how it happens*.

#### 4.4.7 `Security`

- Security-sensitive utilities and abstractions
- Explicit, opt-in usage
- No silent encryption, hashing, or obfuscation

All security behavior must be visible at the call site.

#### 4.4.8 `Cryptography`

- Cryptographic primitives and transformations
- Explicit encryption, decryption, signing, verification, and hashing operations
- No implicit key management, storage, or policy decisions

Cryptographic operations must be:
- Explicit at the call site
- Opt-in
- Transparent in intent

#### 4.4.9 Other Domain Namespaces

Additional namespaces (e.g., `EDI`, `JSON`, `HTTP`) must define a clear responsibility boundary and should not overlap in purpose.

Namespaces such as `Utilities`, `Helpers`, or `Common` are discouraged and should be treated as refactoring waypoints, not architectural destinations.

Poorly defined namespaces must be corrected rather than compensated for with verbose type names.

---

## 5. Naming Conventions

### 5.1 Acronyms and Initialisms

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

### 5.2 General Naming Rules

- Public members: **PascalCase**
- No camelCase for public APIs
- Private members may follow language-idiomatic conventions
- Names should prioritize meaning over brevity

---

### 5.3 Constants

- Constants are declared using **SCREAMING_SNAKE_CASE**

Example:

**VB.NET**

```vbnet
Public Const MAX_RETRY_COUNT As Integer = 5
```

**C#**

```csharp
public const int MAX_RETRY_COUNT = 5;
```

Rationale:
- Visually distinguishes constants from variables
- Aligns with their immutable, global nature

---

### 5.4 Internal vs External Serialization Conventions

- Internal models should follow Ichthus Development naming conventions (PascalCase, ALL-CAPS acronyms).
- External serialization formats (e.g., JSON for public APIs) may adapt naming conventions as required for interoperability.
- Serialization naming differences must be explicit and intentional, not implicit or ad-hoc.

Rationale:
- Preserves internal clarity without sacrificing external compatibility.

---

### 5.5 Domain-Oriented Naming and Namespace Responsibility

Ichthus Development favors **domain-oriented namespaces** over redundant type prefixes.

#### 5.5.1 Namespace Carries Semantic Weight

When a type exists within a clearly defined domain namespace, **the namespace—not the type name—carries the primary semantic meaning**.

Redundant repetition of the domain name in type identifiers is discouraged.

Example:

**VB.NET**

```vbnet
Namespace Ichthus.EDI
    Public Interface IDelimiterDetector
End Namespace
```

**C#**

```csharp
namespace Ichthus.EDI
{
    public interface IDelimiterDetector
    {
    }
}
```

Preferred usage:

**VB.NET**

```vbnet
Dim detector As IDelimiterDetector
```

**C#**

```csharp
IDelimiterDetector detector;
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

#### 5.5.2 When Domain Prefixes Are Acceptable

Domain prefixes may be retained only when the concept itself is cross-domain or taxonomy-like, and may reasonably appear outside its defining namespace.

Example:

**VB.NET**

```vbnet
Public Enum EDIFormat
    Unknown
    X12
    EDIFACT
End Enum
```

**C#**

```csharp
public enum EDIFormat
{
    Unknown,
    X12,
    EDIFACT
}
```

Rationale:
- `Format` alone is ambiguous across domains
- `EDIFormat` may appear in diagnostics, UI, metadata, or logging contexts
- Prefix improves clarity when consumed outside `Ichthus.EDI`

This exception is intentional and limited.

---

#### 5.5.3 Explicit Qualification Over Renaming

When name collisions occur across domains (e.g., `Writer`, `Reader`, `Parser`), explicit qualification or aliasing is preferred over renaming types.

Examples:

**VB.NET**

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

*Type aliasing is considered a first-class language feature and an intentional part of Ichthus Development API consumption patterns.*

---

#### 5.5.4 Design Implication

This convention places higher importance on namespace architecture.

As a result:
- Namespaces must be intentional
- Namespace boundaries define responsibility
- Type names should remain concise, descriptive, and domain-local

*Poorly designed namespaces are not compensated for with verbose type names.*

---

### 5.6 UI Control Naming Conventions (WinForms Only)

For Windows Forms applications, Ichthus Development adopts explicit control-prefix naming conventions.

This convention is **intentionally scoped to WinForms** and does not apply to WPF, MAUI, Blazor, or web-based UI frameworks.

#### Rationale

WinForms relies heavily on:
- Event-handler wiring
- Partial classes
- Designer-generated code

Prefix-based control naming:
- Improves discoverability of event handlers
- Makes control intent obvious when navigating code
- Reduces ambiguity in large forms with many controls
- Speeds up maintenance in legacy or mixed-era codebases

#### Convention

Controls SHOULD be prefixed according to their concrete type:

| Control Type | Prefix Example |
|-------------|----------------|
| `TextBox` | `txtUserName` |
| `Label` | `lblStatus` |
| `Button` | `cmdSubmit` |
| `CheckBox` | `chkIsEnabled` |
| `RadioButton` | `radOptionA` |
| `ComboBox` | `cboCountry` |
| `DataGridView` | `dgvResults` |
| `ListBox` | `lstItems` |
| `Panel` | `pnlMain` |
| `GroupBox` | `grpOptions` |

Prefixes MUST reflect the actual control type, not semantic intent.

> NOTE: This convention is retained for WinForms due to its event-driven, designer-generated architecture and is intentionally not extended to declarative or binding-based UI frameworks.

#### Scope and Limitations

- This convention MUST NOT be applied outside WinForms.
- It MUST NOT be emulated in WPF, MAUI, Blazor, or XAML-based UI frameworks.
- Semantic naming without prefixes is preferred in modern UI frameworks that support strong binding and declarative layouts.

This convention exists to improve maintainability in WinForms, not to impose legacy patterns on modern UI development.

---

## 6. Code Structure & Architectural Preferences

### 6.1 Variable Scope and Declaration Placement

- Local variables should be declared at the beginning of a method whenever practical.
- Variables scoped to blocks may be declared early and initialized to `Nothing` or an empty value when doing so improves readability without introducing unused state.
- Exceptions are permitted when early returns prevent unnecessary allocation.

Rationale:
- Improves scanability
- Reduces mid-method cognitive load

---

### 6.2 Type Safety and Explicit Typing

- Variables MUST be declared using the most specific, meaningful type available.
- Avoid using generic or ambiguous types (e.g., `Object`) when a concrete type exists.
- In C#, use of `var` SHOULD be limited to cases where the inferred type is immediately obvious and improves readability.
- In VB.NET, implicit typing that results in `Object` MUST be avoided.

Examples:

**VB.NET**

Avoid:

```vbnet
Dim result As Object = GetResult()
```

Prefer:

```vbnet
Dim result As ParseResult = GetResult()
```

**C#**

Avoid:

```csharp
var result = GetResult();
```

Prefer:

```csharp
ParseResult result = GetResult();
```

Exceptions are permitted only when:
- Required by reflection or late binding
- Imposed by external APIs or frameworks

Such exceptions MUST be documented inline with rationale.

---

### 6.3 Data Access and Object Design

- Centralized data access patterns are preferred.
- Lazy-loaded properties are acceptable when they improve performance or clarity.
- Attribute-based mapping and serialization control are preferred over convention-only approaches.

*Framework-specific abstractions must not leak into Core or domain contracts.*

---

### 6.4 Type Semantics

- Prefer rich types (e.g., `FileInfo`) over primitive representations (e.g., file paths as strings) when behavior matters.

---

### 6.5 Server-Side Technology Preference

- ASP.NET is the preferred platform for server-side logic and validation.
- Other technologies may be used only when constraints require it.

---

## 7. Code Documentation and Commenting

### 7.1 XML Documentation Comments

- XML documentation comments are required for all public types and members
- Internal members should be documented where intent is not obvious

All publicly visible members MUST be explicitly documented.

This includes:
- Public types
- Public methods
- Public properties (including read-only properties)
- Public fields and constants
- Enum types and individual enum values

Documentation is required even when behavior appears self-evident.

Public APIs are contracts, and contracts must describe intent, meaning, and usage—not just structure.

"Obvious" behavior is considered an implementation detail unless explicitly documented.

XML documentation comments SHOULD make use of:
- `<summary>` to describe intent
- `<param>` to explain parameter purpose
- `<returns>` where applicable
- `<remarks>` for constraints or edge cases
- `<seealso>` and `<cref>` for related types or specifications
- `<langword>` for language keywords

Documentation should explain *why a construct exists*, not merely restate syntax.

When documentation requires multiple conceptual paragraphs, `<para>` elements SHOULD be used instead of relying on line breaks or formatting conventions.

This ensures consistent rendering across IDEs and preserves semantic structure in generated documentation.

Examples:

**VB.NET (Without Documentation):**

```vbnet
Public Enum EDIFormat
    Unknown
    X12
    EDIFACT
```

**VB.NET (With Documentation):**

```vbnet
''' <summary>
''' Identifies the EDI interchange format.
''' </summary>
Public Enum EDIFormat
    ''' <summary>
    ''' Format could not be determined from the input data.
    ''' This value indicates detection failure, not a valid interchange.
    ''' </summary>
    Unknown

    ''' <summary>
    ''' ANSI ASC X12 format.
    ''' </summary>
    X12

    ''' <summary>
    ''' UN/EDIFACT format.
    ''' </summary>
    EDIFACT
End Enum
```

**C# (Without Documentation):**

```csharp
public enum EDIFormat
{
    Unknown,
    X12,
    EDIFACT
}
```

**C# (With Documentation):**

```csharp
/// <summary>
/// Identifies the EDI interchange format.
/// </summary>
public enum EDIFormat
{
    /// <summary>
    /// Format could not be determined from the input data.
    /// This value indicates detection failure, not a valid interchange.
    /// </summary>
    Unknown,

    /// <summary>
    /// ANSI ASC X12 format.
    /// </summary>
    X12,

    /// <summary>
    /// UN/EDIFACT format.
    /// </summary>
    EDIFACT
}
```

Rationale:
- Improves IntelliSense across languages
- Serves as living documentation
- Forces clarity of intent at design time

---

### 7.2 Inline Comments

Inline comments are reserved for:
- Explaining non-obvious intent
- Documenting deliberate deviations
- Clarifying constraints imposed by external systems

Inline comments MUST NOT:
- Restate obvious code behavior
- Duplicate XML documentation
- Serve as a substitute for clear naming or structure

---

### 7.3 Block Comments and Comment Scope

VB.NET does not support block comments.

This is considered an intentional design constraint, not a deficiency.

Standards for expressing multi-line commentary are:
- XML documentation comments (`'''`) for public and protected members
- Consecutive line comments (`'`) for internal commentary
- `<para>` elements for multi-paragraph XML documentation
- `#Region` blocks for structural grouping and navigability

Block-comment style disabling or masking of executable code is discouraged.

If code must be conditionally excluded, it SHOULD be removed or gated explicitly rather than commented out.

Rationale:
- Prevents accidental execution ambiguity
- Preserves line-by-line clarity
- Ensures comments remain readable in plain-text editors
- Avoids tooling- or formatting-dependent interpretation

---

### 7.4 Comment Philosophy

- Comments should explain why, not what
- Redundant comments are discouraged
- Historical or rationale-based comments are acceptable and encouraged

---

### 7.5 External Standards and Business Rule Traceability

When a design is driven by an external standard, specification, or third-party business rules, the origin of those constraints MUST be documented at the point of implementation.

This includes, but is not limited to:
- Industry standards (e.g., data interchange formats, protocol specifications, security standards)
- Third-party APIs or data contracts
- Regulatory or compliance requirements
- Vendor- or client-defined business rules

Documentation SHOULD clearly identify:
- The name of the standard or authority
- The relevant section, version, or rule identifier when applicable
- The reason the constraint exists
- Any known tradeoffs or deviations

This documentation MAY appear in:
- XML documentation comments (`<remarks>`, `<para>`)
- Inline comments when tightly coupled to a specific line or decision
- Referenced specifications using `<seealso>` or `<cref>` where appropriate

Examples:

**Implementations**

```vbnet
''' <remarks>
''' Implements delimiter detection as defined in
''' ANSI X12 §2.3.1 (Element Separators).
''' </remarks>
```

**Business Rules**

```vbnet
''' <remarks>
''' Behavior is driven by third-party settlement rules
''' provided by <VendorName>, revision 2024-03.
''' </remarks>
```

The goal is to ensure that non-obvious design decisions are not mistaken for accidental complexity or poor design.

Code that implements external constraints without documenting their origin is considered incomplete.

Refer to §2 ("public API") and §12 (Concious Deviations) for further, detailed explanations.

---

## 8. Code Organization

### 8.1 `#Region` (VB.NET) and `#region` (C#) Usage

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

## 9. Error Handling and Diagnostics

Diagnostics are defined as structured facts while logging is a consumer concern. As such, logging SHOULD be implemented according to the environment and business rules appropriate to the domain and SHOULD be implemented outside of `Core` or reusable libraries.

### 9.1 Diagnostics over Exceptions

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

Examples:

Avoid: throwing for expected, recoverable conditions

**VB.NET**

```vbnet
Throw New InvalidOperationException("Invalid record format")
```

**C#**

```csharp
throw new InvalidOperationException("Invalid record format");
```

Prefer: emitting diagnostics and continuing

**VB.NET**

```vbnet
diagnostics.Add(Diagnostic.Error(
    code: "INVALID_FORMAT",
    message: "Record format does not match expected schema"
))
```

**C#**

```csharp
diagnostics.Add(Diagnostic.Error(
    code: "INVALID_FORMAT",
    message: "Record format does not match expected schema"
));
```

---

### 9.2 Severity Model

Diagnostics use a shared severity model:
- Information
- Warning
- Error
- Critical
- Fatal

Severity models should be consistent across all Ichthus Development projects.

---

## 10. UI Separation

Rules:
- No UI code in Core or shared libraries
- No `MessageBox`, dialogs, or UI callbacks in reusable logic
- Libraries must be safe for headless and server-side execution

Rationale:
- Enables batch processing
- Enables pipeline execution
- Prevents hidden coupling

---

## 11. Data Handling Philosophy

- Prefer immutable or read-only data structures where practical
- Preserve raw input alongside parsed or transformed representations
- Never silently normalize or coerce data without documentation

---

## 12. Conscious Deviations from Common Best Practices

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

## 13. SQL and Database Development Standards

SQL is treated as a first-class programming language and is subject to the same clarity and maintainability standards as application code.

### 13.1 Formatting and Readability

- SQL keywords MUST be written in UPPERCASE.
- Major clauses (`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`) MUST begin on new lines.
- Each selected column SHOULD appear on its own line.
- Logical conditions SHOULD be vertically aligned for readability.

---

### 13.2 Identifier Naming and Quoting

- Object names MUST be descriptive and domain-relevant.
- Avoid cryptic abbreviations unless they are domain-standard.
- Object names MUST conform to the same naming conventions as described in §5 when the SQL environment allows. Specifically:
  - (§5.1) Acronyms and initialisms MUST be written in ALL CAPS
  - (§5.2) Object names (tables, columns, schemas, etc.) MUST be written in PascalCase
  - (§5.5) Schemas (when supported by the SQL environment) MUST be used as a part of the database design structure to define the domain in which the data resides similarly to how namespaces carry the primary semantic meaning in executable code
- Structural SQL object names represent domain truth and, as such, should read cleanly, descriptively, and without decoration. They MUST NOT contain prefixes or suffixes (`tbl_Customers`, `sch_Inventory`, `col_Address`, etc.)
- Functional SQL object names MUST include prefixes explicitly defining their usage and intent:
  - View: `vw_`
  - Stored Procedure: `usp_` (user-defined)
  - Scalar Function: `fn_`
  - Table-Valued Function: `tvf_`
  - Trigger: `tr_`
- Mixed-case or reserved identifiers MUST be quoted according to the target dialect:
  - ANSI SQL Standard: `"Identifier"`
  - PostgreSQL: `"Identifier"` (Double quotes preserve case; Unquoted names are automatically converted to lowercase)
  - SQL Server (T-SQL): `[Identifier]` (Double quotes work if `QUOTED_IDENTIFIER` is `ON`)
  - MySQL: `` `Identifier` ``
  - SQLite: `"Identifier"` or `` `Identifier` ``

Do not rely on implicit case folding or engine-specific quirks.

---

### 13.3 Schema Design and Evolution

- Database schema is considered part of the public contract.
- Prefer additive, non-destructive schema changes.
- Breaking changes MUST be intentional and documented.

---

### 13.4 Query Intent

- Prefer clarity over cleverness.
- Use Common Table Expressions (CTEs) when they improve readability.
- Avoid deeply nested queries that obscure intent.

---

### 13.5 SQL Location and Ownership

- Schema and migration SQL is authoritative and versioned
- Ad-hoc SQL in application code SHOULD be minimized
- Complex queries SHOULD be named, documented, or externalized

Rationale:  
SQL is code and deserves the same review, ownership, and traceability.

---

## 14. Enforceability and Tooling Alignment

These standards are written with the expectation that they can be enforced through tooling where feasible (e.g., static analyzers, linters, CI validation).

Manual enforcement alone is considered insufficient for long-term consistency.

Where mechanical enforcement is not yet available, standards remain normative and are enforced through review.

Tooling enforcement does not replace human judgment but is intended to support consistency and early feedback.

Tooling enforcement supports these standards but does not override documented design intent or architectural judgment.

---

### 14.1 Language-Specific Standards

Language-specific standards are defined only for languages actively used and maintained within Ichthus Development.

For other languages or ecosystems, this document defines architectural principles and expectations rather than prescriptive syntax rules.

---

### 14.2 Framework-Specific Guidance

Framework- or platform-specific guidance (e.g., Blazor, MAUI, UI frameworks) is documented in project- or domain-specific companion specifications rather than in this core standards document.

---

### 14.3 Disabling and Suppressing Compiler Messages

Compiler warning suppression via `#Disable Warning` (VB.NET) or `#pragma warning disable` (C#) is disallowed.

Blanket or scope-based suppression obscures intent, hides future regressions, and undermines tooling effectiveness.

When a warning must be suppressed intentionally, targeted suppression via language-supported attributes (e.g., `<SuppressMessage>` / `[SuppressMessage]`) MAY be used only when:

- The specific rule being suppressed is named
- The suppression scope is limited to the smallest applicable symbol
- A clear justification is provided explaining why the rule does not apply

Suppressions without documented rationale are considered defects.

This follows the same principle as explicit typing rules (§6.2): tooling must not be silenced to compensate for unclear design.

---

### 14.4 Debugger and IntelliSense Visibility Attributes

Attributes that influence debugger stepping or IntelliSense visibility (e.g., `<DebuggerStepThrough>`, `<EditorBrowsable>`, `<DebuggerBrowsable>`) MAY be used when they improve developer ergonomics without obscuring intent or correctness.

Such attributes are permitted only when:

- The underlying code is correct, well-tested, and intentional
- The attribute reduces noise rather than hiding complexity
- The member remains inspectable through source or reflection when necessary

> **Note:** In this context, "well-tested" implies the behavior has been verified through unit tests, integration tests, or long-standing production stability.

These attributes MUST NOT be used to:

- Conceal poorly designed APIs
- Hide complex or non-obvious logic
- Avoid addressing legitimate tooling warnings or design issues

When applied, the rationale for altering debugger or IntelliSense visibility SHOULD be evident from the surrounding context or documented inline if non-obvious.

---

### 14.5 Languages with Source Transformation or Minification

Some languages and toolchains (e.g., JavaScript, TypeScript, CSS preprocessors) involve source-level transformation, bundling, or minification as part of their build or deployment process.

In such environments:
- Comments MUST NOT be relied upon to control runtime behavior.
- Comment-based masking or disabling of executable code is discouraged.
- Executable intent MUST be expressed through explicit language constructs, configuration, or build-time rules.

When minification or transformation is used:
- Source code MUST remain readable and intention-revealing prior to transformation.
- Tooling steps MUST NOT change program semantics beyond what is explicitly configured.
- Generated or transformed artifacts MUST NOT be treated as authoritative source.

Rationale:
- Comments are non-semantic and may be removed or altered by tooling.
- Readability and correctness must be preserved independently of build pipelines.
- Behavior should be explicit, reviewable, and enforceable at the source level.

---

## 15. Living Document

This document is expected to evolve.

Changes must:
- Be intentional
- Include rationale
- Be applied consistently across all Ichthus Development projects

---

## License

This documentation is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

---

*Ichthus Development Engineering and Coding Standards exist to serve understanding, not fashion.*

© Gold Fish Bowl, LLC, DBA Ichthus Development
