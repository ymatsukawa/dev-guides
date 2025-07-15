# Software Architecture
This document describes the essence of Software Architecture and the recommendations and deprecations that support it.

## Essence of Software Architecture

In one sentence, Software Architecture is the art of managing complexity while creating systems that serve their intended purpose effectively over time. When practicing software architecture, follow these philosophies:

- [ ] **Minimize complexity and pursue clarity**: Software complexity manifests as change amplification, unknown unknowns, and obscurity. Clear code can be understood quickly with minimal cognitive load, ensuring long-term development efficiency and quality. Complex code is often slower, while simple, clean code typically makes systems faster.

- [ ] **Practice thorough information hiding and modularization**: Deep modules hide powerful functionality behind simple interfaces, providing excellent abstractions. This encapsulates design decisions within modules, localizing the impact of changes and reducing overall system complexity. The principle of separation of concerns remains foundational regardless of architectural style.

- [ ] **Maintain a strategic investment mindset**: Tactical programming prioritizing short-term speed accumulates complexity as "technical debt," severely reducing productivity over time. Investing time in design and documentation early yields returns within months, improving long-term development velocity and software quality.

## Recommendations

**Design for long-term system health through strategic approach**:

Most code extends existing codebases, making it crucial to facilitate future extensions. While short-term investments temporarily slow development, they accelerate progress long-term. Technical debt incurred through tactical programming costs more than initially borrowed. Allocate 10-20% of development time to continuous small design improvements rather than large upfront investments. When modifying existing code, maintain the strategic approach by constantly seeking ways to improve system design.

**Create deep modules with simple interfaces**:

Deep modules hide substantial functionality behind simple interfaces, preventing users from seeing internal complexity. This provides superior abstractions that reduce overall system complexity. Always aim for modules with powerful capabilities but simple interfaces. For example, consolidate numerous low-level functions into one simple, general-purpose API.

**Write documentation as part of the design process**:

Comments written during the design process produce higher quality documentation and better designs. Comments serve as design decision records and test design quality, functioning as complexity's "canary in the coal mine." Write interface comments during class or method design, not after coding completion. When comments become too long or complex, it signals potential design problems requiring review.

**Choose meaningful and precise names**:

Well-chosen names for variables, methods, and entities serve as documentation, making code easier to understand. This reduces the need for other documentation and facilitates error detection. Use accurate, consistent names that allow readers to correctly infer meaning at first glance. Name boolean variables as predicates (e.g., `cursorVisible`) and describe variables as nouns representing what they are, not how they're manipulated.

**Make code obvious through clarity**:

Obvious code allows readers to understand behavior and meaning quickly without deep thought, with correct first impressions. This dramatically reduces time and effort for comprehension while minimizing misunderstandings and bugs. Use appropriate whitespace for formatting, insert blank lines between parameter documentation and code blocks for readability, and supplement non-obvious sections with effective comments.

**Consider multiple design alternatives and analyze trade-offs**:

The first design idea is rarely optimal for complex software problems. Evaluate multiple options for each major design decision, objectively assessing pros and cons. This process teaches what makes designs better or worse, improving your ability to identify excellent designs over time. All architectural choices involve trade-offs with no absolute right or wrong answers.

**Ensure organizational and team effectiveness**:

Software design encompasses not just technical challenges but organizational and human aspects. Great software comes from great people - focusing only on technical aspects overlooks more than half the picture. Conway's Law shows organizational communication structure directly impacts system design. Architects should work closely with development teams, ideally writing code to understand the impact of their decisions.

**Design for continuous improvement and evolution**:

Software systems constantly evolve, requiring continuous learning and adaptation rather than perfect initial designs. Development is iterative and incremental, progressing through evolutionary stages. Systems must continue evolving after deployment as usage patterns change. Focus on creating frameworks where the right systems can emerge and grow, rather than perfect end products.

## Deprecations

**Avoid tactical programming approaches**:

Tactical programming appears to accelerate initial projects (10-20% faster) but accumulates complexity over time, reducing productivity (10-20% slower). This manifests as "technical debt," often requiring perpetual repayment exceeding the original time borrowed. Instead of fixating on short-term results, invest additional time upfront to build clean software structures enabling efficient future work.

**Avoid accepting code change amplification**:

When seemingly simple changes require modifications across many different system locations, it indicates complexity that makes development tasks difficult. One design goal should be reducing the amount of code affected by each design decision. For example, centrally managing website banner colors enables site-wide changes with single modifications.

**Avoid creating overly complex code**:

Code with excessive cyclomatic complexity undermines nearly all desirable codebase characteristics including modularity, testability, and deployability. While developers often gravitate toward complexity, unnecessary complexity undermines architectural leadership. Quantitatively measure and continuously monitor code complexity. Implement fitness functions to automatically verify architectural principles and design characteristics.

**Avoid repeating code in comments**:

Comments that duplicate information already present in adjacent code provide no value for understanding. Particularly avoid reusing words from method or variable names directly in comments. Instead, provide additional information not obvious from code, such as units, purpose, design rationale, or special case behavior. Choose different words than the code uses, explaining at more abstract or detailed levels.

**Avoid using vague or ambiguous names**:

Names that are too broad or could refer to many different things convey minimal useful information to developers and increase the likelihood of misuse. This ambiguity often causes serious bugs. Avoid generic coordinate names like `x` and `y`, unclear status names like `blinkStatus`, or meaningless variables like `result`. Use specific, precise names like `charIndex` or `cursorVisible` that convey the entity's nature.

**Avoid mixing implementation details in interface comments**:

Interface documentation that describes implementation details unnecessary for interface users pollutes information and confuses users. This represents one of the most common interface comment mistakes. Interface comments should provide all information needed for method invocation but not describe internal implementation methods. Place implementation documentation inside methods, clearly separated from interface documentation.

**Avoid designs overly dependent on unknown unknowns**:

Software systems often fail due to "unknown unknowns" - problems no one anticipated. This makes "big design up front" approaches risky as designs may fail mid-development. Design systems iteratively and incrementally, maintaining agility to handle uncertainty. This flexibility enables early discovery and correction of unexpected problems.

**Avoid excessive reliance on test-driven development**:

TDD primarily focuses on making specific features work rather than finding the best design. Attention concentrates on adding functionality rather than deep design consideration. This exemplifies tactical programming without clear opportunities for design reflection, potentially resulting in confused design and long-term complexity accumulation. Adopt strategic programming approaches prioritizing design quality, investing in overall system structure and abstraction, not just functional operation.
