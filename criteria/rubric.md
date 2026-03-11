# Clean Architecture Evaluation Rubric

> Based on *Clean Architecture: A Craftsman's Guide to Software Structure and Design* by Robert C. Martin (2017).

## Overview

This rubric defines the evaluation criteria used to score open source ERP/CRM projects for Clean Architecture compliance. Each criterion is scored from **0 to 3**; the total possible score is **18**.

---

## The Four Layers

Clean Architecture organises code into four concentric layers, from innermost (most stable) to outermost (most volatile):

| # | Layer | Contains |
|---|-------|----------|
| 1 | **Entities** | Enterprise business rules; pure domain objects |
| 2 | **Use Cases** | Application-specific business rules; orchestration logic |
| 3 | **Interface Adapters** | Controllers, Presenters, Gateways; data translation |
| 4 | **Frameworks & Drivers** | Web framework, database, UI, third-party services |

**The Dependency Rule**: Source code dependencies may only point *inward*. An inner layer must never know anything about an outer layer.

---

## Evaluation Criteria

### 1. Layer Separation (0–3)

Measures how clearly the four layers are separated in the project's directory and module structure.

| Score | Description |
|-------|-------------|
| **0** | No recognizable layer separation. Business logic, persistence, and presentation are mixed throughout. |
| **1** | Basic separation (e.g., MVC) that does not map to Clean Architecture layers. Major concerns are still mixed. |
| **2** | Three or more distinct layers are identifiable (e.g., Entity, Service, Repository, Controller). Some cross-layer leakage exists. |
| **3** | All four Clean Architecture layers are clearly separated, named, and consistently applied throughout the codebase. |

**What to look for**: Separate namespaces/packages/directories for entities, services/use-cases, controllers/adapters, and infrastructure. No SQL in domain classes. No HTTP imports in use cases.

---

### 2. Dependency Rule (0–3)

Measures whether source code imports always point inward (inner layers never import from outer layers).

| Score | Description |
|-------|-------------|
| **0** | Dependencies point in all directions; inner layers import directly from outer layers (e.g., domain imports from ORM). |
| **1** | Some effort to control dependencies, but inner layers still frequently import outer-layer specifics. |
| **2** | Most dependencies point inward. A few violations exist (e.g., framework annotations on entities). |
| **3** | All dependencies strictly point inward. Outer layers depend on interfaces defined by inner layers (DIP applied). |

**What to look for**: Domain/entity classes that import from the web framework, ORM, or HTTP libraries are a red flag. Use cases should import only entities and abstract interfaces.

---

### 3. Entity Independence (0–3)

Measures whether domain entities are free from framework, ORM, and infrastructure dependencies.

| Score | Description |
|-------|-------------|
| **0** | Entities extend or import from ORM/framework base classes. They cannot be instantiated without infrastructure. |
| **1** | Entities have some framework annotations or utilities but could theoretically function without infrastructure with effort. |
| **2** | Entities are mostly plain objects. Minor framework dependencies (e.g., serialisation annotations) do not affect behaviour. |
| **3** | Entities are pure domain objects with zero framework dependencies. They represent business concepts independently. |

**What to look for**: `extends Model`, `extends ActiveRecord`, `extends Document`, `extends SugarBean` are all red flags. Ideal entities are plain classes with only business logic and no `import framework.*` statements.

---

### 4. Use Case / Application Service Layer (0–3)

Measures whether there is a clearly identifiable application service or use case layer that orchestrates domain logic independently of infrastructure.

| Score | Description |
|-------|-------------|
| **0** | No use case or application service layer. All workflow logic is in controllers, models, or scripts. |
| **1** | Some service classes exist but they mix application logic with framework or infrastructure concerns. |
| **2** | Service/use case classes are present and reasonably separated, but some infrastructure coupling remains. |
| **3** | A clear use case layer exists. Use cases are named after domain operations, are framework-free, and depend only on domain entities and repository interfaces. |

**What to look for**: Classes named after business operations (e.g., `PlaceOrderUseCase`, `RegisterCustomerService`). Use cases should not contain SQL, HTTP calls, or framework imports.

---

### 5. Testability (0–3)

Measures whether the core domain and application logic can be unit-tested in isolation, without requiring a database, web server, or other infrastructure.

| Score | Description |
|-------|-------------|
| **0** | Unit testing domain logic is effectively impossible without a live database or running framework. |
| **1** | Domain logic can be tested with significant mocking effort, but the architecture does not facilitate it. |
| **2** | Domain and service logic can be unit-tested with moderate mocking. Repository interfaces or DI make isolation feasible. |
| **3** | Domain logic can be easily tested in pure unit tests. DI, interface abstractions, and clean boundaries make mocking straightforward. |

**What to look for**: Unit tests that do not set up a database or web server. Use of mock/stub objects for repositories. Test class structure that mirrors the layer structure.

---

### 6. Abstraction and Interface Usage (0–3)

Measures whether interfaces (ports) are defined for external dependencies, allowing the Dependency Inversion Principle to be applied.

| Score | Description |
|-------|-------------|
| **0** | No interfaces used for external dependencies. All code depends on concrete implementations. |
| **1** | Some interfaces exist but are inconsistently applied. Most external dependencies are still concrete. |
| **2** | Repository or service interfaces are used consistently for major external dependencies. Some concrete classes remain. |
| **3** | All external dependencies are accessed through interfaces (ports). Concrete adapters are in the outermost layer. Full DIP. |

**What to look for**: Repository interfaces (`IOrderRepository`, `ContactRepositoryInterface`). Service contracts. Port/Adapter naming conventions. The number of `new ConcreteClass()` calls inside use cases.

---

## Total Score Interpretation

| Range | Level | Interpretation |
|-------|-------|----------------|
| 0 – 5 | ❌ **None** | Does not follow Clean Architecture. Layers are collapsed, dependencies uncontrolled, and testing domain logic requires infrastructure. |
| 6 – 11 | ⚠️ **Partial** | Some Clean Architecture elements present (service layers, repository patterns), but the dependency rule is not consistently enforced. |
| 12 – 15 | ✅ **Mostly** | Most principles applied. Clear layering and good testability, but minor violations or framework coupling exist. |
| 16 – 18 | 🏆 **Full** | Consistent application of all principles: strict layer separation, the dependency rule, entity independence, clear use cases, excellent testability, and full use of abstractions. |

---

## Dataset Score Distribution

| Project | Score | Level |
|---------|-------|-------|
| metasfresh | 16 | 🏆 Full |
| OroCRM | 14 | ✅ Mostly |
| EspoCRM | 11 | ⚠️ Partial |
| Tryton | 9 | ⚠️ Partial |
| iDempiere | 9 | ⚠️ Partial |
| Odoo | 5 | ❌ None |
| ERPNext | 5 | ❌ None |
| Vtiger | 4 | ❌ None |
| SuiteCRM | 3 | ❌ None |
| Dolibarr | 1 | ❌ None |
