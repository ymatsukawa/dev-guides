# Clean Architecture
This document describes the essence of Clean Architecture and the recommendations and deprecations that support it.

## Essence of Clean Architecture

In one sentence, Clean Architecture is the art of drawing boundaries to separate policies from details, enabling the minimization of human resources required to build and maintain software systems. When using this, follow these philosophies:

- [ ] **Prioritize architecture and design quality**: The only way to go fast is to go well. Clean code is always faster than messy code in both short and long term
- [ ] **Control source code dependencies absolutely**: Use dependency inversion to ensure source code dependencies point only inward toward higher-level policies, never outward toward details
- [ ] **Separate business rules from details completely**: Database, Web, and Framework are details that should never contaminate the core business logic

## Recommendations

**Apply SOLID principles rigorously**:

The SOLID principles create mid-level software structures that are resilient to change, easy to understand, and form the basis of reusable components. Each principle addresses a specific architectural concern:
- Single Responsibility Principle (SRP): Ensure modules have responsibility to only one actor to avoid merge conflicts and unexpected coupling
- Open-Closed Principle (OCP): Design software components to be open for extension but closed for modification using polymorphism
- Liskov Substitution Principle (LSP): Ensure derived types are substitutable for their base types without breaking system behavior
- Interface Segregation Principle (ISP): Prevent clients from depending on methods they don't use to avoid unnecessary recompilation
- Dependency Inversion Principle (DIP): Depend on abstractions, not concretions, creating plugin architectures with stable interfaces

**Draw architectural boundaries carefully**:

Software architecture is the art of drawing boundaries that separate elements and restrict their knowledge of each other. This allows deferring decisions about frameworks, databases, and other details:
- Identify boundaries between business rules and technical details early
- Use the Dependency Rule: source code dependencies must point only inward
- Create boundaries that allow independent development and deployment of components
- Apply the Humble Object pattern at boundaries to separate testable from hard-to-test code

**Organize components using cohesion and coupling principles**:

Component structure should evolve with the system's logical design, not be predetermined:
- Apply REP (Reuse/Release Equivalence): Components must be releasable units with cohesive themes
- Use CCP (Common Closure): Group classes that change for the same reasons at the same times
- Follow CRP (Common Reuse): Don't force users to depend on things they don't need
- Enforce ADP (Acyclic Dependencies): Eliminate circular dependencies to avoid "morning after syndrome"
- Apply SDP (Stable Dependencies): Depend in the direction of stability
- Use SAP (Stable Abstractions): Stable components should be abstract to remain extensible

**Design for testability from the start**:

Test design is integral to system architecture and impacts long-term maintainability:
- Create a Test API that allows testing business rules without the GUI
- Use Humble Object pattern to separate testable business logic from UI details
- Ensure tests don't depend on volatile things like databases or web servers
- Structure tests to hide application structure, preventing structural coupling

**Make architecture scream about use cases**:

Good architectures are centered on use cases, allowing the system to be described without mentioning frameworks:
- Top-level directory structure should reveal the system's intent (healthcare, accounting, etc.)
- Business rules should be the most visible and central elements
- Framework names should not dominate the architecture
- The system should clearly communicate what it does, not how it's built

**Defer decisions about details**:

Keep options open by deferring decisions about details that don't affect business rules:
- Database selection can often be delayed for months or years
- Web server and UI framework choices are details, not architectural drivers
- Use interfaces to isolate business rules from I/O devices
- Create systems that can be tested without databases, web servers, or frameworks

**Control framework coupling strictly**:

Frameworks are tools, not ways of life, and should be kept at arm's length:
- Keep frameworks in the outermost circle of the architecture
- Don't derive business objects from framework base classes
- Use dependency injection only in Main or configuration components
- Create proxies or adapters when framework coupling is unavoidable
- Be prepared to protect your architecture from framework evolution

## Deprecations

**Avoid prioritizing behavior over structure**:

Focusing only on making software work while ignoring structure leads to escalating costs and eventual system failure:
- Avoid the trap of "we can clean it up later" - later never comes
- Don't let urgent feature requests override important architectural work
- Recognize that messy code slows down development immediately, not just in the future
- Never sacrifice design quality for perceived speed of delivery

**Avoid creating circular dependencies**:

Circular dependencies between components create tight coupling that makes systems fragile and hard to change:
- Avoid the "morning after syndrome" where one team's changes break another's build
- Don't allow components to have mutual dependencies
- Break cycles immediately using DIP or by extracting common elements
- Never accept "it works" as justification for circular dependencies

**Avoid coupling to database schemas**:

Allowing database structure to permeate the system creates widespread coupling to a technical detail:
- Don't pass database rows or tables as business objects
- Avoid ORMs that encourage thinking in tables rather than business concepts
- Don't let SQL or database-specific features leak into business rules
- Never treat the database as the center of the application

**Avoid letting frameworks dictate architecture**:

Frameworks should serve the application, not dominate it:
- Don't organize code around framework conventions (controllers/, models/, views/)
- Avoid framework annotations throughout business objects
- Don't inherit business objects from framework base classes
- Never let marketing decisions ("we need to be more web-like") drive architectural changes

**Avoid confusing services with architecture**:

Services are deployment and scalability mechanisms, not architectural elements:
- Don't assume services automatically create good architecture
- Avoid creating services that share data schemas or database tables
- Don't let service boundaries replace proper component boundaries
- Never use services as an excuse to ignore SOLID principles

**Avoid testing through volatile interfaces**:

Tests that depend on GUIs, databases, or web services become fragile and impede change:
- Don't write tests that break when UI layouts change
- Avoid tests that require specific database states or schemas
- Don't create test suites that take hours to run due to external dependencies
- Never let test fragility discourage refactoring

**Avoid ignoring embedded/firmware architecture**:

Embedded code needs clean architecture just as much as enterprise systems:
- Don't let hardware details pollute entire codebase (firmware)
- Avoid making all code dependent on specific processor or OS
- Don't write code that can only be tested on target hardware
- Never assume embedded code doesn't need to evolve

**Avoid over-engineering boundaries**:

While boundaries are important, creating too many too early can cripple productivity:
- Don't create full architectural boundaries for every possible future change
- Avoid YAGNI violations by implementing unused flexibility
- Don't add layers of abstraction without clear current benefit
- Balance the cost of boundaries against their actual value