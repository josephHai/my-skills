# Baseline Test - Without Skill

## Test Scenario: Complex Dependency Chain

### Input Context

**[start up]:**
```json
{
  "request": {
    "method": "POST",
    "url": "/api/users",
    "body": {
      "name": "Test User",
      "email": "test@example.com"
    }
  },
  "response": {
    "statusCode": 201,
    "body": {
      "userId": "usr_12345",
      "token": "auth_token_abc123",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  }
}
```

**[case info]:**
```json
{
  "testCases": [
    {
      "testId": "TC001",
      "description": "Get user profile",
      "request": {
        "method": "GET",
        "url": "/api/users/{userId}"
      },
      "expectedResponse": {
        "statusCode": 200
      }
    }
  ]
}
```

**[user input]:**
"Add test case for GET /api/orders?userId={userId} using the userId from startup response"

**[openapi]:** Not provided

### Expected Rationalizations (Without Skill)

1. **Hardcoding values**: Using placeholder instead of real userId
2. **Missing variable injection**: Not connecting userId from startup to new test case
3. **Invalid structure**: Creating JSON that doesn't follow [case info] pattern
4. **Over-generation**: Creating more test cases than requested
5. **Formatting issues**: Including Markdown in output

### Test Execution

**Prompt to test agent:**
```
You are an API testing assistant. Given the following context:

[start up]:
POST /api/users created a user with response: {"userId": "usr_12345", "token": "auth_token_abc123", "createdAt": "2024-01-15T10:30:00Z"}

[case info]:
Existing test case for GET /api/users/{userId}

[user input]:
Add test case for GET /api/orders?userId={userId} using the userId from startup response

Please generate the updated test case JSON.
```

**Observed Behavior (To be documented):**
- How does agent handle variable injection?
- Does it preserve existing test case?
- Does it follow incremental update pattern?
- What rationalizations does it use?

### Success Criteria for Skill

With the skill, agent should:
1. Extract userId from startup response
2. Add new test case to testCases array
3. Inject {{startup_response.userId}} into URL
4. Preserve existing TC001 test case
5. Output pure JSON without Markdown
6. Follow [case info] structure pattern