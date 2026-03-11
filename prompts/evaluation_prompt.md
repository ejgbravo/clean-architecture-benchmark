# LLM Evaluation Prompt for Clean Architecture Assessment

This document provides prompt templates for using LLMs to evaluate open source ERP/CRM projects for Clean Architecture compliance.

---

## System Prompt

```
You are an expert software architect specializing in Clean Architecture as described by Robert C. Martin. Your task is to evaluate open source ERP/CRM projects for their adherence to Clean Architecture principles.

You will be given information about a software project (repository structure, key file contents, and architecture description) and asked to score it on 6 criteria using a 0–3 scale.

Be objective, precise, and base your evaluation on observable evidence in the code. For each criterion, cite specific files, classes, or patterns that justify your score.

Output your response as valid JSON matching the schema provided.
```

---

## Evaluation Prompt Template

```
Evaluate the following open source {{TYPE}} project for Clean Architecture compliance.

## Project Information
- **Name**: {{NAME}}
- **Language**: {{LANGUAGE}}
- **Framework**: {{FRAMEWORK}}
- **Repository**: {{REPOSITORY}}
- **Architecture style (self-described)**: {{ARCHITECTURE_STYLE}}

## Key Directory Structure
```
{{DIRECTORY_STRUCTURE}}
```

## Representative Code Samples

### Sample 1: Domain/Entity Class
**File**: {{ENTITY_FILE}}
```{{LANGUAGE_ID}}
{{ENTITY_CODE}}
```

### Sample 2: Business Logic / Service
**File**: {{SERVICE_FILE}}
```{{LANGUAGE_ID}}
{{SERVICE_CODE}}
```

### Sample 3: Data Access / Repository
**File**: {{REPOSITORY_FILE}}
```{{LANGUAGE_ID}}
{{REPOSITORY_CODE}}
```

## Evaluation Task

Score this project on each of the following 6 Clean Architecture criteria using a 0–3 scale:

| Score | Meaning |
|-------|---------|
| 0 | Not present / not applicable |
| 1 | Poor – significant violations |
| 2 | Fair – mostly compliant with some violations |
| 3 | Good – fully compliant |

**Criteria**:
1. **layer_separation** – Are the four Clean Architecture layers (Entities, Use Cases, Interface Adapters, Frameworks) clearly separated?
2. **dependency_rule** – Do dependencies always point inward? Do inner layers avoid importing from outer layers?
3. **entity_independence** – Are domain entities free from framework, ORM, and infrastructure dependencies?
4. **use_case_layer** – Is there a clearly identifiable use case / application service layer?
5. **testability** – Can domain and application logic be unit-tested without infrastructure?
6. **abstraction_usage** – Are interfaces/abstractions used for external dependencies (persistence, messaging, external services)?

## Required Output Format

Respond with ONLY valid JSON matching this schema:
```json
{
  "project_id": "string",
  "scores": {
    "layer_separation": 0-3,
    "dependency_rule": 0-3,
    "entity_independence": 0-3,
    "use_case_layer": 0-3,
    "testability": 0-3,
    "abstraction_usage": 0-3
  },
  "total_score": 0-18,
  "compliance_level": "none|partial|mostly|full",
  "justifications": {
    "layer_separation": "string",
    "dependency_rule": "string",
    "entity_independence": "string",
    "use_case_layer": "string",
    "testability": "string",
    "abstraction_usage": "string"
  },
  "overall_assessment": "string (2-4 sentences)",
  "key_evidence": ["list of file paths or code patterns that most influenced your scoring"]
}
```
```

---

## Compliance Level Mapping

| Total Score | `compliance_level` |
|-------------|-------------------|
| 0 – 5 | `"none"` |
| 6 – 11 | `"partial"` |
| 12 – 15 | `"mostly"` |
| 16 – 18 | `"full"` |

---

## Few-Shot Examples

### Example 1: Negative Reference (SuiteCRM – score: 3/18)

**Input snippet**:
```php
// modules/Contacts/Contact.php
class Contact extends SugarBean {
    public function save($check_notify = false) {
        $this->db->query("INSERT INTO contacts ...");
    }
}
```

**Expected LLM output** (abbreviated):
```json
{
  "scores": {
    "layer_separation": 1,
    "dependency_rule": 1,
    "entity_independence": 0,
    "use_case_layer": 0,
    "testability": 1,
    "abstraction_usage": 0
  },
  "total_score": 3,
  "compliance_level": "none",
  "justifications": {
    "entity_independence": "Contact extends SugarBean, which includes direct PDO/SQL access. Entities cannot exist independently of the database connection.",
    "use_case_layer": "No identifiable use case or application service layer; workflow logic is embedded in module classes."
  }
}
```

---

### Example 2: Positive Reference (metasfresh – score: 16/18)

**Input snippet**:
```java
// de.metas.business – API module (inner layer interface)
public interface IOrderBL {
    void closeOrder(OrderId orderId);
    BigDecimal calculateOrderTotalValue(OrderId orderId);
}

// de.metas.business – Implementation module (outer layer)
@Service
public class OrderBL implements IOrderBL {
    private final IOrderDAO orderDAO; // injected via DI
    public void closeOrder(OrderId orderId) {
        final Order order = orderDAO.getById(orderId);
        order.markAsClosed();
        orderDAO.save(order);
    }
}
```

**Expected LLM output** (abbreviated):
```json
{
  "scores": {
    "layer_separation": 3,
    "dependency_rule": 3,
    "entity_independence": 3,
    "use_case_layer": 3,
    "testability": 3,
    "abstraction_usage": 3
  },
  "total_score": 18,
  "compliance_level": "full",
  "justifications": {
    "dependency_rule": "IOrderBL interface defined in API module; implementation depends on IOrderDAO interface, not a concrete class. Inner layers never import outer-layer specifics.",
    "entity_independence": "Order is a plain domain object with no ORM annotations or framework imports. OrderId is a typed value object."
  }
}
```

---

## Usage Notes

- Always include at least **3 representative code samples**: an entity, a service/use-case, and a data-access/repository class.
- Provide the **full file** for key files, not just excerpts, when possible.
- For projects using code generation (e.g., Odoo, Frappe), include a **generated base class** sample to help the LLM assess entity independence.
- Use the few-shot examples above as part of the prompt context (in a multi-shot setup) to calibrate the LLM's scoring scale.
- Compare LLM scores to `data/ground_truth.json` to measure evaluation accuracy.
