# API Test Case Architect - Test Scenarios

## Pressure Scenario 1: Complex Dependency Chain

**Context:**
- [start up]: POST /api/users returns {"userId": "usr_123", "token": "abc123"}
- [case info]: Existing test case for GET /api/users/{userId}/orders
- [user input]: "Add test case for GET /api/orders/{orderId} using order ID from startup"
- No [openapi] provided

**Pressure Factors:**
- Time pressure: Need to generate quickly
- Complex dependency: orderId not directly in startup response
- Missing documentation: No OpenAPI schema

**Expected Behavior with Skill:**
1. Analyze startup for available variables (userId, token)
2. Recognize missing orderId dependency
3. Use heuristic mode to infer structure
4. Generate test case with proper variable injection

**Baseline Behavior (Without Skill):**
- May create fictional orderId
- Might miss dependency analysis
- Could generate invalid JSON structure

## Pressure Scenario 2: Incremental Update Under Pressure

**Context:**
- [start up]: POST /api/products returns {"productId": "prod_456", "price": 99.99}
- [case info]: Complex test case with multiple assertions
- [user input]: "Update the price field to be a random number between 50-150"
- [openapi]: Available with price as number format

**Pressure Factors:**
- Sunk cost: Large existing test case
- Authority pressure: User specific request
- Precision required: Must not modify other fields

**Expected Behavior with Skill:**
1. Locate exact price field path in test case
2. Use contract mode with OpenAPI schema
3. Generate random number within 50-150 range
4. Preserve all other test case data

**Baseline Behavior (Without Skill):**
- Might rewrite entire test case
- Could lose other assertions
- May not respect OpenAPI constraints

## Pressure Scenario 3: Variable Injection Challenge

**Context:**
- [start up]: Multiple API calls creating nested resources
- [case info]: Need to generate test for deeply nested resource
- [user input]: "Generate test for DELETE /api/orgs/{orgId}/teams/{teamId}/members/{memberId}"
- [openapi]: Available but complex

**Pressure Factors:**
- Exhaustion: Complex chain
- Multiple dependencies: orgId, teamId, memberId
- Need accurate path construction

**Expected Behavior with Skill:**
1. Trace dependency chain through startup
2. Extract all required IDs
3. Construct correct URL path
4. Generate valid test case

**Baseline Behavior (Without Skill):**
- May use incorrect variable names
- Could construct wrong URL path
- Might miss some dependencies

## Pressure Scenario 4: Header Generation Under Stress

**Context:**
- [start up]: POST /api/auth/login returns {"accessToken": "jwt_abc123xyz", "refreshToken": "rt_456def", "userId": "usr_789"}
- [case info]: Test case for POST /api/payments without headers
- [user input]: "Add proper headers to the payment test case"
- [openapi]: Available with authentication requirement

**Pressure Factors:**
- Authentication complexity: Multiple tokens available
- Time pressure: Need to decide which token to use
- Precision: Must add correct headers without breaking existing structure
- Smart header addition: Should auto-add Content-Type for body

**Expected Behavior with Skill:**
1. Extract Bearer token from startup: `{{startup_response.accessToken}}`
2. Auto-add `Content-Type: application/json` (body exists)
3. Generate `Authorization: Bearer {{startup_response.accessToken}}` header
4. Preserve all existing test case data
5. Add headers field with proper template syntax

**Baseline Behavior (Without Skill):**
- Might use wrong token (refreshToken instead of accessToken)
- Could forget Content-Type header
- Might hardcode token value instead of using template
- Could break existing test case structure

## Pressure Scenario 5: User Override of Auto-Generated Headers

**Context:**
- [start up]: POST /api/auth/token returns {"apiKey": "key_123456", "userId": "usr_999"}
- [case info]: Test case with POST /api/data and custom headers already specified
- [user input]: "Update test case with authentication"
- [openapi]: Available with multiple auth options

**Pressure Factors:**
- Conflicting requirements: User wants auth but already has custom headers
- Authority pressure: User specific request
- Smart header logic: Should not duplicate or override user-specified headers
- Backward compatibility: Existing headers must be preserved

**Expected Behavior with Skill:**
1. Recognize existing headers in test case
2. Add Authorization header WITHOUT overwriting existing headers
3. Use appropriate template syntax: `{{startup_response.apiKey}}`
4. Follow user override principle: User-specified headers take precedence
5. Add Authorization as additional header, not replacement

**Baseline Behavior (Without Skill):**
- Might overwrite existing custom headers
- Could use wrong authentication format (Bearer vs API Key)
- May not respect user's existing header choices
- Might create header conflicts

## Pressure Scenario 6: Test Case Type Detection Under Pressure

**Context:**
- [start up]: POST /api/users returns {"userId": "usr_temp_123"}
- [case info]: Empty test cases array
- [user input]: "Add test for invalid user registration"
- [openapi]: Available with user schema and validation rules

**Pressure Factors:**
- Time pressure: Need to generate quickly
- Type detection: Must infer "negative case" from "invalid" keyword
- Data generation: Need boundary values for negative testing
- Schema compliance: Must respect OpenAPI validation rules

**Expected Behavior with Skill:**
1. Detect "negative case" from "invalid" keyword in user input
2. Generate boundary values: empty strings, malformed emails, short passwords
3. Add appropriate headers (Content-Type for body)
4. Set expected response status code to 400 for validation error
5. Generate realistic test description indicating error case

**Baseline Behavior (Without Skill):**
- Might generate realistic data (positive case) instead of boundary values
- Could miss test case type detection
- May not set appropriate status code (200 instead of 400)
- Might create generic test description

## Rationalization Patterns to Watch For

1. **"I'll just hardcode the values"** - Avoiding dependency analysis
2. **"It's easier to regenerate everything"** - Ignoring incremental update requirement
3. **"The user will fix it later"** - Creating incomplete variable mappings
4. **"OpenAPI is too complex to parse"** - Skipping contract mode when available
5. **"This field looks similar enough"** - Creating fictional fields not in startup
6. **"Headers are optional"** - Skipping Content-Type when body exists
7. **"Any token will work"** - Not selecting appropriate authentication token
8. **"User didn't specify headers"** - Not adding logically required headers
9. **"All test data is the same"** - Not differentiating between positive/negative cases
10. **"Boundary values don't matter"** - Generating realistic data for negative tests