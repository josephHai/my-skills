# Multiple Fields Ambiguity Test Scenario

## Test Context

**Objective:** Test the Multiple Fields Ambiguity Resolution Protocol when user requests to update fields without specifying which ones.

### Input Context:

```
[start up]:
POST /api/products returns {"productId": "prod_999", "price": 100.00, "stock": 50}

[case info]:
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Update product information",
      "request": {
        "method": "PUT",
        "url": "/api/products/{productId}",
        "body": {
          "productId": "prod_999",
          "name": "Test Product",
          "price": 100.00,
          "stock": 50,
          "category": "electronics"
        }
      },
      "expectedResponse": {
        "statusCode": 200,
        "body": {
          "updated": true,
          "timestamp": "2024-01-15T10:30:00Z"
        }
      }
    }
  ]
}

[user input]:
"update fields"

[openapi]: Available with full product schema
```

### Expected Output:

According to the Ambiguity Resolution Protocol in SKILL.md, when multiple fields could be updated and user doesn't specify which ones, the skill should output:

```json
{"query": "multiple_fields", "possible_targets": ["productId", "name", "price", "stock", "category"], "message": "Which field(s) to update? Specify in request."}
```

OR a subset of fields that could reasonably be updated (excluding immutable fields like productId if appropriate).

### Test Rationale:

1. **Multiple fields exist**: The request body contains multiple fields: productId, name, price, stock, category
2. **Unclear intent**: User says "update fields" but doesn't specify which fields to update
3. **Protocol should trigger**: According to SKILL.md section "Ambiguity Resolution Protocol", for multiple fields ambiguity, output clarification JSON
4. **Possible targets**: Should list fields that could be updated (may exclude immutable fields like productId if that's understood from context)

### Success Criteria:

1. ✅ Outputs pure JSON (no ```json markers)
2. ✅ Follows query format: `{"query": "multiple_fields", "possible_targets": [...], "message": "..."}`
3. ✅ Message asks which field(s) to update
4. ✅ Does NOT modify the original [case info] structure
5. ✅ Lists appropriate fields from the request body