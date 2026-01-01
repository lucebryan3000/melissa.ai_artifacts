# API Contract Playbook

> **Purpose**: Guide the design and documentation of API contracts that enable reliable integration between systems
> **Version**: 1.0
> **Last Updated**: 2024-12-31

---

## Mental Model

An API contract is a promise between provider and consumer. It says "Send me this, I'll give you that." Breaking this promise breaks integrations. The contract must be precise, complete, and versioned.

```
Consumer Need → Contract Design → Implementation → Documentation → Versioning
      ↓              ↓                  ↓               ↓             ↓
  "What they     "The promise"      "Fulfill"      "Communicate"  "Evolve"
   need to do"
```

**The API Contract Quality Ladder:**

| Level | State | Characteristics |
|-------|-------|-----------------|
| 0 | Tribal | "Ask the backend dev" |
| 1 | Listed | Endpoints exist, no details |
| 2 | Documented | Request/response shown, gaps remain |
| 3 | Specified | Full schemas, some errors missing |
| 4 | Complete | All cases covered, auth clear |
| 5 | Contractual | OpenAPI spec, versioned, tested |

**Target: Level 4-5 before implementation or consumer integration.**

---

## Inputs / Outputs

### Inputs
- **FRD/Use Cases**: What operations are needed
- **Data Models**: Entity structures from ERD
- **Consumer Needs**: Who calls this API and why
- **Security Requirements**: Auth, rate limits, data sensitivity
- **Performance Requirements**: Latency, throughput targets

### Outputs
- **API Specification**: OpenAPI 3.x document
- **Endpoint Documentation**: Complete operation details
- **Schema Definitions**: Request/response structures
- **Error Catalog**: All error responses
- **Authentication Guide**: How to authenticate
- **Versioning Strategy**: How API evolves

---

## Evaluation Dimensions

### Dimension 1: Endpoint Completeness
Every operation consumers need must be documented. Missing endpoints force workarounds or block integration.

**Probing Questions:**
- What operations do consumers need to perform?
- Are all CRUD operations present where needed?
- Are there batch/bulk operations?
- What about search, filter, pagination?
- Are there operations only for specific consumer types?

**Red Flags:**
- Missing DELETE when CREATE exists
- No search/filter on list endpoints
- No pagination on collections
- Admin operations undocumented

**Good Example:**
```yaml
# Orders API - Endpoint Inventory

## Customer-Facing
POST   /orders                    # Create order
GET    /orders                    # List user's orders (paginated)
GET    /orders/{id}               # Get order details
PATCH  /orders/{id}               # Update order (limited fields)
DELETE /orders/{id}               # Cancel order (if cancellable)

## Search & Filter
GET    /orders?status=pending     # Filter by status
GET    /orders?created_after=...  # Filter by date
GET    /orders?sort=created_at:desc # Sort results

## Admin-Only
GET    /admin/orders              # All orders (admin)
PATCH  /admin/orders/{id}/status  # Force status change
```

**Bad Example:**
```
POST /orders - Create order
GET /orders - Get orders
```

---

### Dimension 2: Request/Response Schemas
Every request and response must have a precise schema. Vague schemas cause integration bugs.

**Probing Questions:**
- What fields are in each request body?
- What fields are in each response?
- Which fields are required vs optional?
- What are the data types and constraints?
- Are there different response shapes for different scenarios?

**Red Flags:**
- "JSON body" without schema
- Missing field types
- No required/optional distinction
- Response schema differs from documentation

**Good Example:**
```yaml
# POST /orders - Create Order

## Request Schema
OrderCreateRequest:
  type: object
  required:
    - items
    - shipping_address_id
  properties:
    items:
      type: array
      minItems: 1
      items:
        type: object
        required: [product_id, quantity]
        properties:
          product_id:
            type: string
            format: uuid
          quantity:
            type: integer
            minimum: 1
            maximum: 99
    shipping_address_id:
      type: string
      format: uuid
    notes:
      type: string
      maxLength: 500

## Response Schema (201 Created)
OrderResponse:
  type: object
  properties:
    id:
      type: string
      format: uuid
    order_number:
      type: string
      example: "ORD-2024-00001"
    status:
      type: string
      enum: [draft, pending, paid, shipped, delivered]
    total:
      type: number
      format: decimal
    created_at:
      type: string
      format: date-time
```

**Bad Example:**
```
Request: Send order data
Response: Returns the created order
```

---

### Dimension 3: Error Responses
Every error case must be documented. Undocumented errors surprise consumers and cause poor error handling.

**Probing Questions:**
- What errors can each endpoint return?
- What's the error response format?
- Are error codes consistent across the API?
- What information helps consumers fix the problem?
- Are there different errors for different consumers?

**Red Flags:**
- Only 200 and 500 documented
- Inconsistent error format
- Generic error messages
- Sensitive info in errors

**Good Example:**
```yaml
# Standard Error Format
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "items[0].quantity",
        "code": "INVALID_RANGE",
        "message": "Quantity must be between 1 and 99"
      }
    ],
    "request_id": "req_abc123"
  }
}

# POST /orders - Error Responses

## 400 Bad Request
- VALIDATION_ERROR: Request body validation failed
- INVALID_PRODUCT: Product ID does not exist
- INSUFFICIENT_STOCK: Requested quantity unavailable

## 401 Unauthorized
- UNAUTHORIZED: Missing or invalid authentication
- TOKEN_EXPIRED: Access token has expired

## 403 Forbidden
- FORBIDDEN: User lacks permission
- ORDER_NOT_OWNED: Order belongs to different user

## 404 Not Found
- ORDER_NOT_FOUND: Order with given ID does not exist

## 409 Conflict
- ORDER_NOT_MODIFIABLE: Order status prevents modification

## 429 Too Many Requests
- RATE_LIMITED: Retry after {retry_after} seconds
```

**Bad Example:**
```
Returns error if something goes wrong
```

---

### Dimension 4: Authentication/Authorization
Who can call what? Security must be designed into the API, not bolted on.

**Probing Questions:**
- How do consumers authenticate?
- What auth methods are supported?
- Which endpoints require authentication?
- What permissions/scopes exist?
- How are different user types handled?

**Red Flags:**
- Auth not documented
- Inconsistent auth requirements
- No scope/permission model
- Secrets in URLs

**Good Example:**
```yaml
# Authentication

## Methods Supported
- Bearer Token (JWT): Primary method
- API Key: Server-to-server integrations

## Token Format
Authorization: Bearer <jwt_token>

## Scopes
| Scope | Description |
|-------|-------------|
| orders:read | Read own orders |
| orders:write | Create/modify orders |
| admin:orders | Full order access |

## Endpoint Authorization
| Endpoint | Public | Customer | Admin |
|----------|--------|----------|-------|
| GET /products | ✅ | ✅ | ✅ |
| POST /orders | ❌ | ✅ | ✅ |
| GET /orders | ❌ | Own only | All |
| PATCH /admin/orders | ❌ | ❌ | ✅ |
```

**Bad Example:**
```
API requires authentication
```

---

### Dimension 5: Versioning Strategy
APIs evolve. How do you change without breaking consumers?

**Probing Questions:**
- How are versions indicated?
- What's the current version?
- What's the deprecation policy?
- How are breaking changes communicated?
- How long are old versions supported?

**Red Flags:**
- No versioning strategy
- Breaking changes without version bump
- No deprecation timeline

**Good Example:**
```yaml
# Versioning Strategy

## Approach: URL Path Versioning
Base URL: https://api.example.com/v1

## Breaking vs Non-Breaking Changes

### Non-Breaking (no version bump)
- Adding new endpoints
- Adding optional request fields
- Adding response fields

### Breaking (requires version bump)
- Removing endpoints
- Removing or renaming fields
- Changing field types
- Changing auth requirements

## Deprecation Policy
1. Announce 6 months before EOL
2. Add Deprecation header
3. Document migration path
4. Disable after EOL date
```

**Bad Example:**
```
The API may change in the future
```

---

### Dimension 6: Idempotency
Can operations be safely retried? Network failures happen.

**Probing Questions:**
- Which operations are naturally idempotent?
- Which operations need idempotency keys?
- How are duplicate requests detected?
- What's the idempotency window?

**Red Flags:**
- POST operations with no idempotency strategy
- Duplicate payments possible
- No retry guidance

**Good Example:**
```yaml
# Idempotency

## Idempotency Key Header
Idempotency-Key: <uuid>

## Requirements
- Required for: POST /orders, POST /payments
- Window: 24 hours
- Scope: Per user/API key

## Behavior
- First request: Process normally, cache response
- Duplicate request (same key + body): Return cached response
- Conflict (same key, different body): Return 409
```

**Bad Example:**
```
Avoid submitting the same request twice
```

---

## Extended Question Bank

### Endpoint Design
1. What are ALL the operations consumers need?
2. Are CRUD operations complete for each resource?
3. What batch/bulk operations are needed?
4. What search/filter/sort operations exist?
5. Are there sub-resources (nested endpoints)?

### Schemas
6. What's every field in each request body?
7. What's every field in each response?
8. Which fields are required vs optional?
9. What are the validation rules?
10. Are there different response shapes for success vs error?

### Errors
11. What's every error each endpoint can return?
12. Is the error format consistent?
13. What information helps consumers debug?
14. Are there rate limiting errors?
15. How do you handle unexpected errors?

### Security
16. How does authentication work?
17. What scopes/permissions exist?
18. Which endpoints are public vs protected?
19. How do different user roles see different data?
20. Are there rate limits?

### Evolution
21. What's the versioning strategy?
22. What constitutes a breaking change?
23. How are deprecations communicated?
24. How long are old versions supported?
25. How do consumers migrate between versions?

### Reliability
26. Which operations need idempotency keys?
27. What's the retry strategy recommendation?
28. What are the rate limits?
29. What's the expected latency?
30. What timeout should consumers use?

---

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---------|---------|-----|
| **Inconsistent naming** | `/getOrders` vs `/order/create` | Use nouns, consistent patterns |
| **Missing errors** | Undocumented 4xx/5xx | Catalog all error cases |
| **Vague schemas** | Integration bugs | Full schemas with examples |
| **No versioning** | Can't evolve API | Version from day one |
| **Auth afterthought** | Security gaps | Design auth with API |
| **No pagination** | Timeouts | Paginate all collections |
| **No idempotency** | Duplicate transactions | Idempotency keys |

---

## API Contract Template (OpenAPI 3.0)

```yaml
openapi: 3.0.3
info:
  title: [API Name]
  version: 1.0.0
  description: [Description]

servers:
  - url: https://api.example.com/v1

security:
  - bearerAuth: []

paths:
  /resources:
    get:
      summary: List resources
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ResourceList'
        '401':
          $ref: '#/components/responses/Unauthorized'
    
    post:
      summary: Create resource
      parameters:
        - name: Idempotency-Key
          in: header
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ResourceCreate'
      responses:
        '201':
          description: Created
        '400':
          $ref: '#/components/responses/ValidationError'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  
  schemas:
    ResourceCreate:
      type: object
      required: [name]
      properties:
        name:
          type: string
    
    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            request_id:
              type: string
  
  responses:
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    ValidationError:
      description: Validation failed
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

---

## Invariants

1. **Every endpoint MUST have documented request/response schemas** — no "sends JSON"
2. **Every error case MUST be documented** — consumers need to handle all cases
3. **Authentication MUST be explicit per endpoint** — public vs protected clear
4. **Versioning strategy MUST be defined** — from day one
5. **Breaking changes MUST increment version** — never break consumers
6. **Mutation endpoints SHOULD support idempotency** — networks fail
7. **Collections MUST be paginated** — unbounded lists timeout
8. **Error format MUST be consistent** — same structure everywhere

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Initial playbook |

---

*Integration Specialist — Designing contracts that consumers can trust.*
