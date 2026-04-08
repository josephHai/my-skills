name: api-test-case-architect
description: Orchestrates sub-agents to generate/update API test case JSON with strict dependency injection and structure preservation.
---

# API Test Case Architect

## 🎯 Core Directive
Generate or update API test case JSON based on [start up] and [case info]. **Only modify explicitly requested fields. Preserve exact original structure.**

## 🗺️ Execution Workflow

### Step 1: Analyze Dependencies (Mandatory)
- Scan [start up] responses.
- Build mapping: `{ "variableName": "{{startup_response.path}}" }`.
- If a required ID is missing, use placeholder `{{variable_name}}` in URL.

### Step 2: Determine Intent & Delegate
| User Intent | Action | Sub-agent Trigger |
| :--- | :--- | :--- |
| **Full Generation** | Create new `testCases` array entry. | Required for Mock Data or OpenAPI extraction. |
| **Incremental Update** | Locate exact JSONPath. Update **only** that leaf node. | Required for complex nested array updates. |

**Sub-agent Input Protocol (Use JSON):**
```json
{
  "action": "Generate|Mock|Update",
  "target": "jsonpath_field",
  "context": "relevant_ids_or_openapi_ref",
  "requirement": "user_instruction"
}
```

### Step 3: Structured Assembly & Output
- Inject sub-agent results into [case info].
- **Output only the final raw JSON string.** No explanations, no Markdown.

## ⚠️ Mandatory Constraints Checklist

| # | Constraint | Violation Consequence |
| :--- | :--- | :--- |
| 1 | **Zero Hallucination**: Fields must exist in [start up] or [openapi]. | Reject generation. |
| 2 | **Exact Structure**: Do not add/remove fields not in [case info]. | Use `jsonpatch` style updates only. |
| 3 | **Derived Field Safety**: Updating `price` does **not** auto-update `total`. | Keep `total` unchanged unless requested. |
| 4 | **Empty Case Info Handling**: If `testCases` array missing/empty, return `{"error": "Empty case info", "message": "Provide base structure."}`. | Stop execution. |
| 5 | **Template Syntax**: Only use `{{startup_response.fieldName}}`. Never use `{{fieldName}}` or bracket notation. | Variable injection fails. |
| 6 | **Pure Output**: Output raw JSON only. No ```json markers. | Parser failure downstream. |

## 📚 Reference Patterns

### ✅ Correct Variable Injection
```json
{
  "request": {
    "url": "/api/users/{userId}",
    "pathParams": { "userId": "{{startup_response.data.id}}" }
  }
}
```

### ❌ Missing Dependency (Fallback)
*If `orderId` not found in startup:*
```json
{
  "request": {
    "url": "/api/orders/{orderId}",
    "pathParams": { "orderId": "{{orderId}}" }
  }
}
```

### ⚡ Incremental Update Example
*[case info] Original:* `{"total": 100, "price": 10}`
*User Request:* "Change price to 20"
*Correct Output:* `{"total": 100, "price": 20}`  *(Note: total remains 100)*
```
