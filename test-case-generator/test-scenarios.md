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

## Rationalization Patterns to Watch For

1. **"I'll just hardcode the values"** - Avoiding dependency analysis
2. **"It's easier to regenerate everything"** - Ignoring incremental update requirement
3. **"The user will fix it later"** - Creating incomplete variable mappings
4. **"OpenAPI is too complex to parse"** - Skipping contract mode when available
5. **"This field looks similar enough"** - Creating fictional fields not in startup