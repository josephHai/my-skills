# Skill Validation Test

## Test 1: Variable Injection

**Input:**
```
[start up]:
POST /api/items created item with response: {"itemId": "itm_789", "name": "Test Item"}

[case info]:
{
  "testCases": [
    {
      "testId": "TC001",
      "request": {
        "method": "GET", 
        "url": "/api/items/{itemId}"
      }
    }
  ]
}

[user input]:
"Update TC001 to use the itemId from startup response"

[openapi]: Not provided
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "request": {
        "method": "GET",
        "url": "/api/items/{itemId}",
        "pathParams": {
          "itemId": "{{startup_response.itemId}}"
        }
      }
    }
  ]
}
```

**Validation Points:**
- ✓ Variable injected from startup
- ✓ Template syntax used
- ✓ Original structure preserved
- ✓ No Markdown formatting

## Test 2: Incremental Update

**Input:**
```
[start up]: Not relevant for this update

[case info]:
{
  "testCases": [
    {
      "testId": "TC002",
      "description": "Create order",
      "request": {
        "method": "POST",
        "url": "/api/orders",
        "body": {
          "productId": "prod_123",
          "quantity": 1,
          "price": 100.0
        }
      }
    }
  ]
}

[user input]:
"Update TC002 price to be a random number between 50-200"

[openapi]: Available with price as number
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC002",
      "description": "Create order",
      "request": {
        "method": "POST",
        "url": "/api/orders",
        "body": {
          "productId": "prod_123",
          "quantity": 1,
          "price": 125.5
        }
      }
    }
  ]
}
```

**Validation Points:**
- ✓ Only price field updated
- ✓ Random value within 50-200 range
- ✓ All other fields unchanged
- ✓ Respects OpenAPI number type

## Test 3: New Test Case Generation

**Input:**
```
[start up]:
POST /api/users created user: {"userId": "usr_999", "email": "test@example.com"}

[case info]:
{
  "testCases": []
}

[user input]:
"Add test case for GET /api/users/{userId}/profile"

[openapi]: Available
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Get user profile",
      "request": {
        "method": "GET",
        "url": "/api/users/{userId}/profile",
        "pathParams": {
          "userId": "{{startup_response.userId}}"
        }
      },
      "expectedResponse": {
        "statusCode": 200
      }
    }
  ]
}
```

## Test 4: Headers Generation with Authentication

**Input:**
```
[start up]:
POST /api/auth/login returns {"token": "jwt_token_abc123", "userId": "usr_789"}

[case info]:
{
  "testCases": [
    {
      "testId": "TC001",
      "request": {
        "method": "POST",
        "url": "/api/orders",
        "body": {
          "productId": "prod_456",
          "quantity": 2
        }
      }
    }
  ]
}

[user input]:
"Update TC001 to include proper headers"

[openapi]: Available
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "request": {
        "method": "POST",
        "url": "/api/orders",
        "headers": {
          "Content-Type": "application/json",
          "Authorization": "Bearer {{startup_response.token}}"
        },
        "body": {
          "productId": "prod_456",
          "quantity": 2
        }
      }
    }
  ]
}
```

**Validation Points:**
- ✓ Content-Type header added automatically (body exists)
- ✓ Authorization header with Bearer token extracted from startup
- ✓ Template syntax used correctly: `{{startup_response.token}}`
- ✓ Original request body preserved unchanged

## Test 5: Test Case Type Detection - Positive Case

**Input:**
```
[start up]:
POST /api/products returns {"productId": "prod_999"}

[case info]:
{
  "testCases": []
}

[user input]:
"Add test case for successful user registration"

[openapi]: Available with user schema
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Register new user - success case",
      "request": {
        "method": "POST",
        "url": "/api/users",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "email": "user123@example.com",
          "name": "Test User",
          "password": "Password123!"
        }
      },
      "expectedResponse": {
        "statusCode": 201
      }
    }
  ]
}
```

**Validation Points:**
- ✓ Realistic email generated (contains @ and .)
- ✓ Descriptive name "Test User"
- ✓ Content-Type header added (body exists)
- ✓ Positive case inferred from "successful" keyword
- ✓ Status code 201 for creation

## Test 6: Test Case Type Detection - Negative Case

**Input:**
```
[start up]:
POST /api/products returns {"productId": "prod_888"}

[case info]:
{
  "testCases": []
}

[user input]:
"Add test case for invalid email validation error"

[openapi]: Available with email format validation
```

**Expected Output (with skill):**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Register with invalid email - error case",
      "request": {
        "method": "POST",
        "url": "/api/users",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "email": "invalid-email",
          "name": "",
          "password": "p"
        }
      },
      "expectedResponse": {
        "statusCode": 400
      }
    }
  ]
}
```

**Validation Points:**
- ✓ Invalid email format generated
- ✓ Empty string for name field
- ✓ Very short password for boundary testing
- ✓ Negative case inferred from "invalid" and "error" keywords
- ✓ Status code 400 for bad request

**Validation Points:**
- ✓ New test case added to array
- ✓ Variable injection from startup
- ✓ Follows OpenAPI constraints
- ✓ Proper testId generation

## Rationalization Table

| Potential Rationalization | Skill Countermeasure |
|---------------------------|----------------------|
| "I'll hardcode the ID" | Strict rule: Use only variables from startup |
| "Easier to rewrite everything" | Incremental update requirement |
| "User didn't specify format" | Follow [case info] structure pattern |
| "OpenAPI is too complex" | Contract mode with schema extraction |
| "Just one fictional field" | Zero hallucination rule |
| "Markdown makes it readable" | Pure output rule - no Markdown |