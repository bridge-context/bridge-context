---
layout: default
title: Multi-Repo Workflow
parent: Guides
nav_order: 1
---

# Multi-Repo Workflow Guide

Learn how to effectively use Context Bridge with microservices and multi-repository architectures.

---

## Setting Up a Microservices Project

### 1. Create the Project

Start by creating a project that groups your related services:

```bash
context-bridge project create my-platform "Microservices platform with user, notification, and payment services"
```

### 2. Add Your Repositories

Add each service as a separate repository:

```bash
# Core services
context-bridge repo add user-service ~/projects/user-service
context-bridge repo add notification-service ~/projects/notification-service
context-bridge repo add payment-service ~/projects/payment-service

# Shared libraries
context-bridge repo add shared-types ~/projects/shared-types
context-bridge repo add api-gateway ~/projects/api-gateway
```

### 3. Verify Your Setup

```bash
context-bridge project list
```

Output:
```
Projects:
---------
my-platform
  Description: Microservices platform with user, notification, and payment services
  Repositories:
    - user-service: ~/projects/user-service
    - notification-service: ~/projects/notification-service
    - payment-service: ~/projects/payment-service
    - shared-types: ~/projects/shared-types
    - api-gateway: ~/projects/api-gateway
```

---

## Workflow Patterns

### Cross-Repo Investigation

When you need to understand how systems interact across multiple repositories.

**Use Case:** Understanding how authentication flows across services.

```
"In my platform project, trace how a user login request flows from the
api-gateway through user-service and any other services involved."
```

Context Bridge will:
1. Query `api-gateway` for authentication routing
2. Query `user-service` for login implementation
3. Identify any downstream service calls
4. Synthesize a complete flow diagram

**Best For:**
- Debugging integration issues
- Onboarding new developers
- Documentation generation
- Audit and compliance reviews

---

### Coordinated Feature Implementation

When a new feature requires changes across multiple services.

**Use Case:** Adding a "forgot password" feature.

**Step 1: Query all repos to understand current patterns**

```
"Search all repos in my-platform for how password reset or recovery
is currently handled, and what email sending patterns exist."
```

**Step 2: Implement in user-service first**

```
"In user-service, implement a forgot password flow with:
- POST /auth/forgot-password endpoint accepting email
- Generate secure reset token with 1-hour expiry
- Store token in database
- Return success response (don't expose if email exists)"
```

**Step 3: Implement in notification-service with context**

```
"In notification-service, add a password reset email template and handler.
Context from user-service: The endpoint will call POST /notifications/send
with { type: 'password-reset', email: string, resetToken: string, userId: string }"
```

**Step 4: Verify both services**

```
"Run tests in user-service and notification-service to verify the new
password reset feature works correctly."
```

---

### Refactoring Across Services

When you need to rename or restructure shared concepts.

**Use Case:** Renaming `User` to `Account` across all services.

**Step 1: Survey the impact**

```
"Query all repos for where the User model, type, or interface is
defined and used. List all files that would need changes."
```

**Step 2: Update shared types first**

```
"In shared-types, rename the User interface to Account. Update all
exports and maintain backwards compatibility with a deprecated User alias."
```

**Step 3: Update each service sequentially**

```
"In user-service, update all references from User to Account, using
the new type from shared-types. Run tests after changes."
```

Repeat for each service, running tests after each change.

---

### Debugging Cross-Service Issues

When an issue spans multiple services.

**Use Case:** Investigating why email notifications aren't being sent.

**Step 1: Trace the data flow**

```
"Query all repos to find how notification sending is triggered.
Start from the API endpoint and trace through to the email dispatch."
```

**Step 2: Check each service for issues**

```
"In notification-service, examine the email sending code for
potential failure points, error handling, and logging."
```

**Step 3: Run diagnostic commands**

```bash
# Check logs in notification-service
context-bridge test-query --repo notification-service --query "Show me the error handling in the email sender"

# Check if API gateway is routing correctly
context-bridge test-query --repo api-gateway --query "How are /notifications routes configured?"
```

---

## Best Practices

### 1. Explore Before Implementing

Always query repositories to understand existing patterns before making changes:

```
"In user-service, how are API endpoints structured? What validation
library is used? Show me an example controller."
```

This ensures your implementations follow established conventions.

---

### 2. Provide Cross-Service Context

When implementing in one service, provide context about other services:

```json
{
  "tool": "implement_in_repo",
  "arguments": {
    "repo": "notification-service",
    "task": "Add handler for password reset emails",
    "context": "user-service will call POST /notifications/send with payload: { type: 'password-reset', email: string, resetToken: string, expiresAt: ISO8601 }. The email should include a link to https://app.example.com/reset?token={resetToken}"
  }
}
```

---

### 3. Use Parallel Queries

When investigating across all repos, use `query_all_repos` instead of multiple individual queries:

```
"Search all repos for database connection patterns and environment
variable usage for database configuration."
```

This runs queries in parallel, significantly reducing total time.

---

### 4. Run Tests After Changes

Always verify changes with tests:

```
"Run tests in user-service to make sure the new password reset
endpoint works correctly and doesn't break existing functionality."
```

Or check specific test files:

```
"Run only the authentication-related tests in user-service."
```

---

## Example: Implementing "Forgot Password" Feature

This complete example shows how to implement a feature across two services.

### Project Setup

```
my-platform/
  - user-service (handles authentication)
  - notification-service (handles email sending)
```

### Step 1: Understand Current Patterns

```
"Query user-service and notification-service to understand:
1. How authentication endpoints are structured in user-service
2. How notification-service receives and processes email requests
3. What email templates exist"
```

### Step 2: Design the Integration

Based on the query results, you understand:
- user-service uses Express with Zod validation
- notification-service exposes POST /send with type-based routing
- Templates are stored in /templates as Handlebars files

### Step 3: Implement User Service Changes

```
"In user-service, implement forgot password feature:
- Add POST /auth/forgot-password endpoint
- Accept { email: string } body with Zod validation
- Generate crypto.randomBytes(32) reset token
- Store in password_resets table with user_id, token_hash, expires_at
- Call notification-service POST /send with type 'password-reset'
- Always return { success: true, message: 'If that email exists...' }
- Add corresponding tests"
```

### Step 4: Implement Notification Service Changes

```
"In notification-service, add password reset email support:
- Add 'password-reset' case to notification type handler
- Create templates/password-reset.hbs email template
- Include reset link: {{appUrl}}/reset-password?token={{resetToken}}
- Add expiry warning: 'This link expires in 1 hour'
- Add tests for the new email type

Context: user-service will send { type: 'password-reset', email: string,
resetToken: string, userId: string, expiresAt: string }"
```

### Step 5: Verify Both Services

```
"Run all tests in user-service"
```

```
"Run all tests in notification-service"
```

### Step 6: Check for Integration Issues

```
"Query both user-service and notification-service to verify the
password reset flow is correctly integrated. Check that the payload
structure matches between the services."
```

---

## Common Pitfalls

### Inconsistent Interfaces

**Problem:** Services evolve independently and interfaces drift.

**Solution:**
- Query shared-types first when implementing cross-service features
- Include explicit interface definitions in context
- Run type checks across affected services

### Missing Error Handling

**Problem:** Failures in downstream services not handled properly.

**Solution:**
- Always implement timeout handling for service calls
- Add retry logic for transient failures
- Log failures with correlation IDs for debugging

### Testing in Isolation

**Problem:** Tests pass individually but integration fails.

**Solution:**
- Use contract testing between services
- Query all services to verify interface compatibility
- Run integration tests after coordinated changes

---

## See Also

- [Quick Start](../getting-started/quick-start) - Initial setup guide
- [Query Tools](../mcp-tools/query-tools) - Deep dive on querying repos
- [Implementation Tools](../mcp-tools/implementation-tools) - Making changes across repos
- [Troubleshooting](./troubleshooting) - Common issues and solutions
