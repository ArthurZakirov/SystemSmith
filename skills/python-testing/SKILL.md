---
name: python-testing
description: Design, extend, and review Python tests across the existing repository's test architecture, with emphasis on observable behavior, clear subject-under-test boundaries, source-aligned test organization, explicit Arrange–Act–Assert structure, and readable parameterization. Use whenever implementing or modifying Python behavior, creating or updating tests, or reviewing test coverage.
---

# Python testing

## Objective

Maintain confidence in **observable behavior at the appropriate architectural boundary**. Treat testing as part of every behavior change, not as a final step that consists of adding a few unit tests. Respect the repository's established test architecture while correcting missing coverage when necessary.

## 1. Discover the test architecture before writing tests

Before implementing or changing tests:

1. Inspect the repository structure and existing test conventions. Search beyond directories named `tests`: include `.feature` files, BDD step definitions, integration/system/end-to-end suites, contract tests, fixtures, test utilities, and relevant CI/test configuration.
2. Identify the test levels actually present, what each considers its **system under test (SUT)**, where execution enters that SUT, and how its outcome is observed.
3. Trace which existing behaviors and contracts the proposed code change affects. Locate existing scenarios, parameter tables, and regression tests that should be extended.
4. Identify missing coverage even when the changed module/class/function has **no existing test file**. Absence of a precedent is not a reason to omit a test.
5. Identify the owning production module/class for each unit-level behavior to be tested, and locate its corresponding test module (or the appropriate place to create one). Do not organize tests by the feature/change that prompted them.
6. Use existing conventions and frameworks where practical; don't introduce a competing structure by default.

Do not equate the test architecture with only the test files adjacent to the changed code.

## 2. Cover the change at every relevant test level

A change is not adequately tested merely because a new unit test passes. Consider **all test levels present in the repository** and update the ones whose observable contracts or scenarios are affected: unit, component/integration, contract, BDD, system, and end-to-end tests.

- Extend existing tests and `.feature` scenarios when they already cover the behavior being changed.
- Create a new test or test file when a relevant behavioral contract lacks coverage, even if no tests existed for the changed component.
- When a feature changes end-user or system-level behavior, account for that behavior at the appropriate higher-level test boundary, not only in isolated unit tests.
- Do not mechanically duplicate every case at every level. Each level should verify a meaningful contract at **its own** boundary, with detail and breadth appropriate to its purpose.
- If a relevant level cannot feasibly be tested, explain the specific limitation rather than silently skipping it.

Before finishing, report which levels were inspected, which were changed or covered already, and which remain unverified.

## 3. Define the boundary before choosing what to call or assert

**Test behavior, not implementation** is an operational rule, not merely a slogan.

**Universal rule: define and preserve the SUT boundary at every test scope.** This applies regardless of technology, repository, architectural layer, or test framework: a single function, a class, an orchestrating service, an event-driven workflow, and an end-to-end data pipeline are all possible systems under test. "Internal" and "external" are relative to the selected SUT, not fixed properties of a component. A component can be an implementation detail of a larger SUT and the public entry point of its own smaller SUT.

For each test suite or scenario, determine:

- **SUT:** What component, service, pipeline, or complete system is this test intended to verify?
- **Entry boundary:** What public interface or externally available input starts the behavior?
- **Observation boundary:** Through what public interface, persisted artifact, or externally visible effect can the outcome be verified?
- **Inside the black box:** Which implementation steps are deliberately *not* part of this test's contract?

**Arrange** preconditions, **act** through the SUT's intended entry boundary, and **assert** the resulting behavior through the SUT's intended observation boundary. The production entry point selected as the SUT must be recognizable directly in the test's Act step (see section 8). Keep those boundaries consistent wherever tests are grouped as one coherent suite, and in particular across scenarios in the same BDD feature file. Different test scopes are legitimate, but give them their own appropriately scoped tests or suites instead of treating them as interchangeable cases of one SUT.

Do not call internal components directly or assert internal intermediate results merely because they are convenient to access. Such tests may be valid, but belong in a suite whose SUT is that smaller component.

### Example: event-driven service

If a system's public input is an API Gateway request and the promised output is data persisted by a downstream Java service:

- **Act:** Send the request to the API Gateway.
- **Assert:** Retrieve the persisted data through the supported read interface and check its business-relevant content.
- **Do not substitute:** Direct invocation of an internal Lambda, a Lambda's success response, or an intermediate EventBridge payload as proof of the full system's behavior.

An integration test **of that Lambda** may properly invoke the Lambda and assert its public result. The scope of the SUT determines what is an implementation detail.

### Example: Databricks medallion pipeline

If the SUT begins with input ingested into Bronze and its observable contract ends in Gold:

- **Act:** Submit realistic input through the ingestion boundary and run the supported pipeline workflow.
- **Assert:** Read Gold output and check the resulting rows, columns, values, and required invariants.
- **Do not substitute:** Assertions on the generated `INSERT` SQL text or intermediate Silver transformations as proof of the overall pipeline contract.

Silver-level behavior can still deserve its own tests **when Silver is the SUT of those tests**, such as transformation/component tests.

### Boundary nuances

- Prefer the public interface **of the selected SUT**, not necessarily an interface defined directly in the repository.
- For asynchronous systems, await eventual externally observable outcomes with bounded polling/timeouts rather than asserting an internal acknowledgement as proof of completion.
- Inspecting internal execution details for *diagnostics* is different from making them the test's *success criterion*.
- An internal representation can itself be a contract in a lower-level test (e.g., a serializer's generated SQL); do not ban such tests globally. The relevant question is whether that representation is observable behavior **at that test's boundary**.
- Do not mix different SUT scopes within one BDD feature. Group scenarios by the **same system boundary**, not merely by shared business terminology, changed feature, or common pipeline components. A feature must have a coherent entry boundary and observation boundary; scenarios may vary their data and outcomes without changing that scope.

### Apply the same boundary principle when grouping BDD scenarios

The general rule above also constrains test organization, not only individual assertions. A feature file is not a catch-all for tests that mention the same business feature: it specifies behavior of one selected SUT at one coherent architectural boundary.

**Define the SUT before grouping scenarios.** In a `.feature` file, all scenarios (including those added later) must exercise the same *kind* of SUT via its designated entry boundary and verify behavior at its designated observation boundary. Different inputs, business cases, and expected outcomes are variations of one feature **only if they preserve that scope**. Do not smuggle component or intermediate-stage tests into a larger-system feature merely because they are part of the same end-to-end workflow.

For example, in a Bronze → Silver → Gold pipeline, these are **three distinct SUT scopes**, not three scenarios to mix in one feature. This is an illustration of the universal boundary rule, **not a Databricks- or data-pipeline-specific convention**:

| Scope | Act / entry | Assert / observation | Suitable owning feature |
| --- | --- | --- | --- |
| Bronze → Silver | Submit or stage Bronze input through the selected component boundary | Read and validate Silver output | `bronze_to_silver.feature` |
| Silver → Gold | Supply Silver input through the selected component boundary | Read and validate Gold output | `silver_to_gold.feature` |
| Bronze → Gold | Submit data at the Bronze ingestion boundary and execute the full pipeline | Read and validate Gold output | `bronze_to_gold.feature` |

The names are illustrative; follow repository naming and test-level conventions. Bronze → Gold tests must **not** start by directly injecting data into Silver, or assert Silver as if it were the final promised outcome. Silver is an implementation detail *of the Bronze → Gold SUT*, while Silver is a legitimate public test boundary *of a narrower component SUT*.

Similarly, a system feature that enters through API Gateway and reads final results through a Java service must not contain another scenario that directly invokes an intermediate Lambda and asserts its response. Keep that Lambda test, but place it in the appropriate component/integration suite (or owning repository) with its own defined SUT and feature. **Different boundaries are not additional parameter values or extra scenarios of the same feature.**

Before creating or extending a BDD feature, explicitly record its SUT, entry, and observation boundaries, and verify every scenario uses those boundaries. If a new behavior needs a different boundary, find or create its own properly scoped feature/suite instead of appending it to the existing file. Do not over-split files merely because cases exercise different values or success/error paths at the *same* boundary.

## 4. Test public behavior even when implemented by private hooks

Private helpers and framework callbacks often implement behavior exposed through a different interface. Test the **behavior through that interface**, not the privacy of the implementation method.

**Pydantic example:** If a field/model validator is added or changed, exercise the model through `Model.model_validate(input_data)` (or the repository's actual public model API). Assert the normalized/validated model values for valid inputs, and the relevant validation error for invalid inputs. Do this even when:

- The changed validator method is private or framework-invoked.
- `model_validate` is inherited from Pydantic rather than defined by the repository.
- The model did not previously have a unit-test module.

A rule such as “test only public methods” must **not** be interpreted as “test only methods implemented explicitly by this repository.”

## 5. Prefer readable, data-driven test cases when the behavior is shared

Before adding another test function or another BDD scenario, ask whether the cases verify the **same behavioral rule** with different inputs and expected outcomes.

If they do, prefer a **decision table** / parameterized test:

- For pytest, use `@pytest.mark.parametrize` with meaningful case IDs when they improve readability.
- For Gherkin BDD, use `Scenario Outline` and `Examples` to express distinct executions of the *same scenario*.
- Use Gherkin `Data Tables` for structured multi-row/multi-field input **within one scenario**; they do not, by themselves, create multiple scenario executions. Choose between `Examples` and a `Data Table` according to the behavior being modeled.
- Extend existing decision tables and scenario outlines instead of duplicating their control flow.
- Keep input conditions and expected outcomes visible together wherever that improves understanding.

**Do not over-parameterize.** Prefer separate tests or scenarios when the cases express different behavioral contracts, require meaningfully different setups/actions, have distinct ownership or failure semantics, or when a parameter table would introduce branching and obscure the intent. The goal is clarity, not the smallest test-function count.

### Human-readable test representations

Tests document expected behavior for a human reader. Optimize for immediate comprehension, not merely syntactic validity, brevity, parameterization, or formatter compliance.

- **Visual clarity:** Make relationships between conditions, inputs, and expected results apparent at a glance. A decision table is useful only when readers can readily associate each value with its meaning and compare cases without counting tuple positions or reconstructing mappings.
- **Low cognitive load:** Avoid opaque indirection, positional decoding, unexplained abstractions, or unnecessary navigation. Keep relevant inputs and expected outcomes close together.
- **Choose the representation for the information:** Parameterize shared behavior when that makes comparison clearer; use named cases, descriptive IDs, separate tests, or other structures when they communicate better. Do not force all cases into one syntactic shape.
- **Judge the formatted artifact:** Assess what a developer encountering the final code without implementation context will actually see. Formatter compliance does not establish readability; if the final representation is difficult to scan, redesign it rather than adding exceptions to a formatting rule.

The objective is human comprehension across all tests, not compliance with any one example layout. The same standard applies to test descriptions, fixtures, mocks, BDD steps, and organization.

## 6. Test composed behavior; isolate external effects, not internal logic by default

Choose a **unit of behavior** and its public contract, rather than assuming that one function, method, or architectural layer must be isolated from every other one. A higher-level service or orchestrator can be the SUT even when it executes many real lower-level functions and domain objects.

### Default: sociable tests of the selected behavior

- Exercise the selected SUT through its public entry point, with its real, deterministic in-process collaborators (helpers, value objects, domain logic, transformations) wherever practical.
- Assert the SUT's **resulting observable behavior**: returned values, raised errors, resulting state, produced artifacts, or a meaningful outgoing effect at a defined boundary.
- Do **not** mock an internal helper merely to assert that the orchestrator delegated to it with particular arguments. Such call assertions ordinarily encode an implementation choice and break when code is refactored without changing behavior.
- Add focused tests for lower-level components when they have independently meaningful contracts, complex edge cases, or are independently reused. A higher-level test does not eliminate the value of focused tests, and focused tests alone do not replace coverage of the composed behavior.

This approach is often called **sociable unit testing**. It is valid to test multiple real cooperating functions in a unit test; a unit need not be one function or one class.

### Replace undesirable effects at an architectural seam

- Do not perform real network/API requests, cloud operations, uncontrolled database writes, or other slow, costly, nondeterministic, or dangerous effects in fast isolated tests.
- Replace the **external dependency or infrastructure adapter at a stable port/boundary** with a suitable fake, stub, or other test double. Keep the actual application workflow and its internal domain logic running.
- Prefer a **behavioral fake** (for example, an in-memory repository) when it enables assertions about resulting state. Use a stub when only controlled responses are needed. A fake need not replicate every production feature, but must implement the relevant semantics faithfully.
- After running the service, assert the resulting business state or output. Do not automatically assert every `get`, `post`, or internal method call just because a mock framework makes that easy.

Illustrative example (names and contracts depend on the repository):

```python
def test_shipping_an_order_updates_its_state():
    orders = InMemoryOrderRepository.with_order("order-1", status="pending")
    service = OrderService(repository=orders)

    service.ship_order("order-1")

    assert orders.get("order-1").status == "shipped"
```

Here the real service workflow executes, while the persistence dependency is replaced. The assertion observes its effect through the repository contract rather than asserting a specific internal call such as `repository.save.assert_called_once_with(...)`.

### Interaction assertions are a narrow, legitimate exception

Sometimes a SUT's **observable output is an outgoing interaction**: issuing a payment command, publishing an event, sending an email, or invoking a documented outbound port. If no suitably trustworthy state/output is observable within the isolated test, a spy/mock can verify that **contractual outgoing effect**, including its destination and semantically important payload. That is different from verifying incidental interactions with private helper functions.

- Prefer observing resulting state via a fake when possible; otherwise verify the outgoing **command** at the SUT's chosen boundary.
- Verify only behavior-relevant aspects of the interaction. Avoid asserting the exact internal method chain, read-only collaborator calls, arbitrary call counts, or execution order unless they are part of the requirement (for example, at-most-once submission).
- Do not classify every call assertion as forbidden, nor every mocked call as part of the public contract. Ask: *Would an implementation refactor that preserves the SUT's observable behavior be allowed to change this interaction?* If yes, the assertion is likely too coupled.

### Type class-based mocks for IDE navigation and interface fidelity

When replacing a class-based external dependency (for example, an HTTP/API adapter), **type the mock as the class whose contract it stands in for**. Prefer `typing.cast(RealAdapter, ...)` where a mock framework's inferred type prevents IDE navigation: Ctrl/Cmd-click on `adapter.some_method` should lead to the real `RealAdapter.some_method` declaration. Use the same rule for other class-based test doubles when a cast meaningfully improves code navigation and typing.

```python
from typing import cast
from unittest.mock import Mock, create_autospec

# Arrange: replace the whole network-owning adapter at the service boundary.
adapter = cast(ApiAdapter, create_autospec(ApiAdapter, instance=True, spec_set=True))
cast(Mock, adapter.fetch_record).return_value = {"id": "123", "status": "ready"}
service = RecordService(adapter=adapter)

# Act: execute the real service logic, not an adapter method.
result = service.load_record("123")

# Assert: verify business behavior through the service's public result.
assert result.status == "ready"
```

- Prefer `create_autospec(RealAdapter, instance=True, spec_set=True)` (or the framework's equivalent) to a free-form `Mock()`: autospec constrains available methods and signatures; `spec_set` prevents assigning non-existent attributes. Cast **the resulting object** to the real class/port type for static navigation.
- `typing.cast` **does not turn the mock into a real instance or validate it at runtime**. It only informs static typing and the IDE. Keep the runtime spec/autospec and targeted tests of the real adapter; never treat the cast as proof of API compatibility.
- A class-typed method will no longer expose mock-specific attributes in static analysis. When setup or a justified boundary-interaction assertion needs `.return_value` or `.assert_called_once_with`, cast **that method expression** to `Mock` locally rather than typing the entire adapter as `Mock`.
- Do not mock deterministic internal collaborators merely to use this pattern. Prefer a real, typed in-memory fake when it yields clearer state-based assertions. Do not add redundant casts when a test double is already accurately typed and navigable.

### Do not confuse isolation with integration coverage

- A passing service test against a fake **does not prove** that the production API adapter correctly serializes requests, authenticates, handles HTTP errors, or matches the provider's contract. Cover that separately through adapter/contract/integration tests at the adapter's boundary.
- Conversely, integration tests do not require real network traffic: tests may integrate multiple real modules or an actual local database. **The terms unit and integration vary across teams.** Prefer documenting the SUT, which collaborators are real versus replaced, and which contracts are verified over debating labels.
- As a working convention, call fast tests of one cohesive behavior with real in-process domain collaborators and fake infrastructure **unit tests (sociable)**. If a test verifies the interaction between independently owned components, treat it as a component/integration test, even if everything runs locally. Respect established repository naming when it differs.

**Decision rule:** Execute as much deterministic application logic as serves the chosen SUT's behavioral contract. Isolate external effects at stable boundaries. Observe results wherever possible; verify calls only when they represent a meaningful boundary contract rather than internal wiring.

## 7. Organize tests by their SUT, not by the feature that caused the change

A feature is a **reason to update several tests**, not a good default basis for grouping unrelated unit tests in one file. The test layout should make it straightforward to locate tests for a production module or class without searching through feature-specific grab bags.

- **Default unit-test file ownership:** Mirror the source-package structure where practical, with one test module per production module. For example, `app/models/payload.py` → `tests/unit/app/models/test_payload.py`, and `app/services/payload_parser.py` → `tests/unit/app/services/test_payload_parser.py`. Follow established repository naming and layout rather than imposing a second parallel convention.
- **Class grouping:** If the production module has one main class, its matching test module will naturally focus on that class. When it has several related classes or module-level functions, group their tests within the same corresponding test module, using clear test classes/sections only when helpful. One test file per class is also reasonable when scale or distinct ownership warrants it; neither mapping is absolute.
- **No feature-named catch-all unit tests:** Do not create `test_<new_feature>.py` and mix tests for several unrelated production modules or abstraction layers merely because one change touched them all. Add each behavioral case to the test module that owns its SUT; create that module in the **correct test-level directory** if it does not exist.
- **Name by behavior and owner:** Test names should identify the contract and expected observable outcome, e.g. `test_payload_model_rejects_missing_identifier` or `test_parse_payload_normalizes_status`. Prefer precise names/parameter IDs over generic names like `test_new_feature` or `test_validation`.
- **Different levels have different organization:** Integration, contract, BDD, and system tests can be organized around an **interface or business capability** rather than a single source module. Their ownership must still be explicit and their SUT boundaries consistent; a single BDD feature must retain one coherent SUT/entry/observation scope (section 3). Do not mingle internal component tests with whole-system scenarios simply because they share a feature label.

For a change affecting `PayloadModel` and `PayloadParser.parse_payload()`, the model's validation cases belong with the **model tests**, and the parser's observable behavior belongs with the **parser tests**. Consider higher-level scenarios independently under section 2. Do not combine them in one unit-test file just because both are part of the same feature.

## Behavioral descriptions must match the tested boundary

Test names, parameter IDs, BDD scenario titles, and Given–When–Then wording are part of the behavioral specification. They must describe the **same externally observable contract, at the same SUT boundary**, that the test executes and asserts.

- **Boundary alignment:** Name the input conditions and observable result of the SUT, not an internal mechanism used to produce it. A test of a model's public validation should describe what input it accepts or rejects and what value/error the caller observes, rather than focusing on a private validator or trimming implementation.
- **Behavioral specificity:** Prefer concrete domain entities, relevant conditions, and expected outcomes over vague groupings such as “correct systems,” “handles validation,” or “works as expected.” Name the actual system or effect when it is relevant to the contract.
- **Self-contained meaning:** A developer should understand the specified behavior from the name and immediate enclosing context without opening the test body, step definitions, fixtures, or production code merely to discover what is meant.
- **Semantic consistency:** Description, Act/When, and Assert/Then must refer to one coherent contract. Do not describe an implementation detail when the test asserts a public outcome, or promise an outcome the assertions do not verify.
- **Information density:** Use the enclosing test module, class, or BDD feature to avoid redundant wording, but never at the expense of identifying the behavior. Prefer precise domain language to mechanically verbose names.

For example, a public model-validation test might be named `test_accepts_serial_number_with_whitespace_and_returns_normalized_value` rather than `test_trims_whitespace`; a system BDD Then step should identify the observable record or service outcome rather than claiming that unspecified downstream systems were called. These are illustrations, not prescribed naming templates.

## 8. Make the SUT and the Act step unmistakable

Every test should communicate, at a glance, **which production interface is exercised**, **what input/precondition is supplied**, and **what observable outcome is expected**. Use a readily identifiable **Arrange–Act–Assert (AAA)** structure (or **Given–When–Then** for BDD):

- **Arrange / Given:** Construct inputs, fixtures, SUT instances, fakes, and required state. Factories and helpers are welcome here if they simplify preparation without hiding the relevant conditions.
- **Act / When:** **Directly call the actual production entry point of the selected SUT, visibly in the test body.** If the SUT is `PayloadModel`, show `PayloadModel.model_validate(...)`; if it is `PayloadParser`, show `parser.parse_payload(...)`. For error cases, put that direct call inside `pytest.raises(...)`. Do not put a locally defined test helper, opaque factory/workflow wrapper, or unrelated higher-layer entry point here and force readers to trace multiple calls to discover what is really under test.
- **Assert / Then:** Check the SUT's expected observable result/error/contract at that test's boundary. Do not substitute assertions on internal collaborators or on a more distant layer.

**Prefer an explicit single SUT operation in Act** for a focused behavioral test. A scenario can legitimately involve multiple actions when that sequence is *itself* the public workflow being specified, but keep the entry point(s) and intended boundary clear; do not hide them behind generic helpers. Other production calls used only to arrange state or read a result may occur in Arrange/Assert without becoming additional SUTs.

Use a blank-line-separated AAA layout; add `# Arrange`, `# Act`, and `# Assert` comments when they make the phases faster to recognize (especially in nontrivial tests). In short, self-evident tests, avoid comment noise; visible separation is the requirement. Gherkin's `Given`, `When`, and `Then` already serve as phase markers.

Example: two SUT boundaries, **two owning test modules**:

```python
# tests/unit/app/models/test_payload.py

def test_payload_model_normalizes_status():
    # Arrange
    payload = {"status": " ready "}

    # Act
    model = PayloadModel.model_validate(payload)

    # Assert
    assert model.status == "ready"
```

```python
# tests/unit/app/services/test_payload_parser.py

def test_parse_payload_returns_normalized_status():
    # Arrange
    parser = PayloadParser()
    payload = {"status": " ready "}

    # Act
    result = parser.parse_payload(payload)

    # Assert
    assert result.status == "ready"
```

The types and field values are illustrative. The second test may execute `PayloadModel.model_validate` *internally*, which is correct for a sociable test of the parser. It must still **call `parse_payload` directly as its Act**, because that is its chosen boundary. Do not replace either visible Act with `run_feature_case(payload)` or a multi-layer builder that eventually reaches the relevant production method.

The same principle applies at higher levels: for a system scenario, the `When` step invokes the **system's public entry point**, not an internal Lambda or test-defined shortcut. In BDD, step definitions may wrap the actual adapter call as required by the framework, but must preserve a transparent, named mapping from the `When` step to that public boundary.

### Use `pytest.fixture(autouse=True)` for genuinely universal setup

Pytest's `@pytest.fixture(autouse=True)` automatically applies a fixture to every test within its visibility/scope, without requiring the fixture as a test-function argument. **Use it when a precondition or cleanup is deliberately shared by every test in that scope**, such as resetting common mutable state or enforcing a no-network rule in a unit-test directory. This removes repetitive boilerplate while keeping the SUT's Act step prominent.

```python
@pytest.fixture(autouse=True)
def clear_shared_registry():
    # Applies automatically to every test in this module.
    shared_registry.clear()
    yield
    shared_registry.clear()
```

- Choose the **smallest appropriate scope** (test class, test module, or a dedicated `conftest.py` subtree) and normal `function` fixture lifetime when per-test isolation is needed. Avoid unintentional global effects across integration/system suites.
- **Do not make all fixtures autouse.** Inputs, per-scenario responses, SUT construction, and scenario-specific state that a reader needs to understand should normally remain **explicit** in Arrange or named fixture arguments. Otherwise implicit setup hides test preconditions and undermines the visible-SUT/AAA principle.
- Make side effects and cleanup of autouse fixtures obvious from their names and definitions. Use `yield` teardown, `monkeypatch`, or other reliable cleanup to avoid state leakage. Do not add an autouse fixture merely to avoid one clear fixture argument.
