# Software Testing Design and Operations
This document describes the essence of Software Testing Design and Operations and the recommendations and deprecations that support it.

## Essence of Software Testing Design and Operations

In one sentence, Software Testing Design and Operations is the practice of building quality into the development process from the beginning, establishing rapid feedback loops through automation, and continuously experimenting to understand complex systems. When implementing this, follow these philosophies:

- [ ] Build quality into the development process from the start - Quality is not something to be "tested in" at the end, but rather "built in" from the design phase through practices like TDD, BDD/ATDD, and continuous integration
- [ ] Establish test automation as the foundation with rapid feedback - Manual testing alone cannot keep pace with modern software development; automated tests providing fast, frequent feedback are essential for teams to deliver value in short iterations
- [ ] Understand complex systems through continuous experimentation - Modern software systems are inherently complex and unpredictable; actively experiment through approaches like chaos engineering to discover unknown failure modes and improve resilience

## Recommendations

**Prioritize test automation above all else**:

Test automation is critical to agile project success because it provides rapid feedback, reduces technical debt, and eliminates fear of change. As applications grow in size, maintaining quality and speed becomes impossible with manual testing alone.

Implement automation for unit tests, functional tests, load tests, and data creation as primary candidates. Ensure all regression tests are automated without exception. Focus on API-level testing for the highest return on investment, as these tests are easiest to automate and most stable. Design UI tests carefully as they tend to be fragile. Utilize tools like FitNesse, Selenium, and Ruby with Watir for effective test automation.

**Emphasize continuous feedback throughout development**:

Rapid feedback enables early problem detection and correction, guides development direction, and promotes continuous learning and adaptation. Short iterations are essential for experimenting with various automation approaches and quickly adjusting course.

Run automated tests frequently through continuous integration to detect behavioral changes early. Help product owners articulate requirements through concrete examples and tests. Use chaos engineering experiments to provide feedback about system behavior. Ensure testers facilitate continuous feedback by clarifying requirements in testable forms.

**Keep tests and code simple at all times**:

Simplicity facilitates understanding, maintenance, and debugging while maintaining code clarity and focus. Unnecessary complexity undermines reliability and should be actively avoided.

Implement "just enough" testing with the lightest-weight tools and techniques available. Start with simple happy-path tests before progressing to complex scenarios. Apply simple coding standards and avoid duplication. Design automated tests to be short and verify only one thing at a time.

**Make quality a whole-team responsibility**:

When quality is recognized as a shared responsibility across the entire team rather than just testers, it introduces broader skill sets and perspectives, leading to better product design and solutions. This eliminates the "quality police" stereotype.

Ensure business stakeholders, programmers, testers, and analysts collaborate to improve the product. Have testers facilitate communication between customers and developers. Require programmers to write tests and design code with testability in mind. Use Acceptance Test-Driven Development (ATDD) where teams work with product owners to define acceptance criteria collaboratively.

**Design for testability from the beginning**:

Code designed with testability in mind makes testing easier, faster, and more reliable. It naturally promotes good design principles like modularity, separation of concerns, high cohesion, information hiding, and loose coupling.

Separate layers (UI, business logic, data access) to facilitate testing below the GUI. Use dependency injection to insert measurement points during testing. Apply Test-Driven Development (TDD) to ensure programmers consider testability while developing. When refactoring legacy code, prioritize establishing testability.

**Design tests as hypothesis-based experiments**:

Treating tests as experiments establishes clear objectives, validates assumptions, provides deep understanding of system behavior, and helps discover unexpected results.

Start chaos engineering experiments with hypotheses about steady-state behavior. Use TDD with tests as "executable specifications" where test failures are predicted experimental outcomes. In requirements discussions, use "what if" scenarios to promote hypothesis-driven exploration. Think of each test case as a scientific claim that can be either true or false.

**Continuously improve testing practices**:

Software systems constantly evolve, so tests must evolve accordingly. Retrospectives provide crucial opportunities to identify improvements and fix unstable tests.

Participate in retrospectives to evaluate what works well and what needs improvement. Thoroughly investigate and fix flaky or unreliable tests through better isolation or splitting complex tests. Adjust metrics to organizational needs. Use feedback from programs like DiRT to iteratively improve operations year over year.

**Isolate tests for reliability and consistency**:

Isolation prevents test interference, ensures reliability and consistency, enables parallel execution, and provides clearer results.

Each test should generate its own data and clean up after completion. Avoid shared databases that can cause inconsistencies between tests. Use dedicated accounts for automation systems to eliminate conflicts from concurrent execution. Utilize simulators for external systems to help isolate tests. Remember that pure unit tests can safely run in parallel, limited only by CPU and memory.

**Focus on business value and customer needs**:

Ensure teams build what customers actually need and prioritize work based on business value. This prevents building technically correct but valueless software.

Direct agile testing focus toward business value and customer-required quality. Help product owners clarify requirements through examples and tests. Prioritize stories based on value. Use ATDD/BDD to deeply understand customer requirements. Design chaos engineering experiments considering user experience and customer impact.

## Deprecations

**Avoid trying to test quality in after development**:

Quality cannot be "tested in" after product construction but must be "built in" from the earliest development stages. Attempting to ensure quality after the fact delays problem discovery and exponentially increases fixing costs.

Traditional waterfall approaches that test products after construction to assess quality are fundamentally flawed. Instead, consider quality from the code design stage and adopt approaches that prioritize testability. Implement continuous integration to maintain known quality states throughout development.

**Avoid separating testing from programming**:

Independent test teams working separately from programmers delay feedback and increase bug-fixing costs. This separation can inhibit direct communication between programmers and testers.

Defect tracking systems may discourage direct dialogue between team members. Instead, promote the "whole team" agile approach with shared responsibility for quality and testing. Build systems where programmers write tests and collaborate with testers to ensure quality. Prioritize direct dialogue and rapid fixes over excessive reliance on defect tracking systems.

**Avoid neglecting legacy code**:

Legacy code causes continuous technical debt accumulation, making system changes and maintenance extremely difficult. Adding new features to old, poorly designed legacy code creates complex interactions between new and old code, increasing error likelihood.

Actively consider refactoring legacy code to ensure testability. Adopt strategic approaches like the Strangler Application pattern to gradually eliminate legacy code. Use test doubles and mock objects to isolate external dependencies in legacy code and make it testable.

**Avoid relying on fixed delays in tests**:

Operation completion times always vary, so fixed delays reduce test reliability and cause unnecessary waiting. This creates flaky tests that undermine confidence in the test suite.

Even if an operation needs 10 seconds to succeed 99% of the time, you waste 7+ seconds in 50% of cases. Instead, use dynamic wait mechanisms that pause until specific conditions are met. Tools like Selenium WebDriver provide smart methods that wait until elements are found. Utilize methods like Wait.Until or Wait.While with timeouts to wait for conditions.

**Avoid unnecessary coupling between components**:

Strong dependencies between components increase change costs, heighten system fragility, and impede scaling. Directly embedding third-party code creates tight coupling, while mixing UI and business logic makes testing difficult and causes changes to ripple throughout the system.

Incorporate principles of separation of concerns, information hiding, and loose coupling into design. Isolate third-party code behind your own abstractions or adapters. Use dependency injection and asynchronous communication as tools to manage coupling. Apply the DRY principle carefully to avoid creating unnecessary coupling for reusability.

**Avoid over-reliance on prediction and planning**:

Software development's inherent unpredictability makes accurate future prediction impossible; adaptability is more important than prediction. Defined process control models like waterfall require complete understanding of all work, which is unrealistic.

Execute tasks "just in time" and focus on preparation at each iteration's start. Deliver small business value increments in short release cycles and adjust plans flexibly as needed. Introduce continuous verification systems and deepen system behavior understanding through experimentation.

**Avoid misunderstanding or ignoring the test automation pyramid**:

The test automation pyramid guides efficient, cost-effective test automation strategy. Ignoring its principles leads to difficult-to-maintain, unstable test suites.

Having few unit/component tests at the pyramid's base but many GUI-level tests creates fragile tests requiring extensive maintenance. Top-layer GUI tests are most expensive to automate and tend to be most fragile. Instead, concentrate most test automation on low-level, fast technical tests like unit and component tests. Automate business-facing tests for rapid feedback. Minimize GUI tests and test business logic directly when possible.

**Avoid hardcoded strings and locators in tests**:

Directly hardcoded strings and UI locators in test code are vulnerable to UI and environment changes, creating maintenance nightmares. Embedding WebDriver locators directly in test method bodies makes tests fragile to UI changes.

Use the Page Object pattern to encapsulate UI element locators within classes. Generate dynamic values like GUIDs and environment-specific values uniquely for each test execution. Retrieve values from configuration files or external sources. Add prefixes or postfixes to changeable values to maintain uniqueness. Use domain-specific languages based on business rules to make tests more readable and change-resistant.

**Avoid ignoring test data and snapshot degradation**:

Snapshots conveniently restore test environment initial states quickly but can degrade over time. When verifying snapshot consistency between Raft Leaders and Followers, some sizes may be zero, indicating snapshot corruption.

Regularly verify snapshot integrity. Implement cleanup mechanisms to delete or rollback entities created by tests after completion. Restore databases from clean backups for each test and populate only with necessary data. Maintain test data strategies that ensure reproducible, independent tests.