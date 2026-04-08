# Ambiguity Resolution Test Scenario

## Test Context

**Objective:** Test the Ambiguity Resolution Protocol when a field already exists in [case info] and user intent is unclear.

### Input Context:

```
[start up]:
POST /api/products returns {"productId": "prod_789", "price": 75.50}

[case info]:
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Create order with existing product",
      "request": {
        "method": "POST",
        "url": "/api/orders",
        "body": {
          "productId": "prod_789",
          "quantity": 3,
          "price": 75.50
        }
      },
      "expectedResponse": {
        "statusCode": 201,
        "body": {
          "orderId": "order_456",
          "total": 226.50
        }
      }
    }
  ]
}

[user input]:
"add price field"

[openapi]: Available with price as number format
```

### Expected Output:

According to the Ambiguity Resolution Protocol in SKILL.md, when field ambiguity exists (field already exists but user intent is unclear), the skill should output:

```json
{"query": "field_ambiguity", "field": "price", "message": "Field 'price' already exists in request body. Specify 'update existing' or 'add new field' in request."}
```

### Test Rationale:

1. **Field exists**: The "price" field already exists in the request body at `testCases[0].request.body.price`
2. **Unclear intent**: User says "add price field" but price already exists - unclear if they want to update existing or add new field
3. **Protocol should trigger**: According to SKILL.md section "Ambiguity Resolution Protocol", this should output a clarification JSON
4. **Pure JSON output**: No Markdown formatting, just the JSON query

### Success Criteria:

1. ✅ Outputs pure JSON (no ```json markers)
2. ✅ Follows exact query format: `{"query": "field_ambiguity", "field": "price", "message": "..."}`
3. ✅ Message mentions field already exists and asks for clarification
4. ✅ Does NOT modify the original [case info] structure
5. ✅ Does NOT add or update any fields in the test case