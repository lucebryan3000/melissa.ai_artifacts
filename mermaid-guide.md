# Mermaid Diagram Guide

> **Purpose**: Guide the creation of effective Mermaid diagrams that communicate clearly across all artifact types
> **Version**: 1.0
> **Last Updated**: 2024-12-31

---

## Mental Model

A diagram is worth a thousand words—if it's the right diagram. Wrong diagram type or poor execution creates confusion, not clarity.

```
Purpose → Diagram Type → Content → Layout → Style
    ↓           ↓           ↓        ↓        ↓
 "Why"     "Which kind"  "What"   "How"   "Polish"
```

**The Diagram Selection Matrix:**

| You Want to Show | Use This | Mermaid Type |
|------------------|----------|--------------|
| Process flow, decisions | Flowchart | `flowchart` |
| Component interactions over time | Sequence Diagram | `sequenceDiagram` |
| System structure, containers | C4 / Architecture | `C4Context`, `flowchart` |
| Data entities & relationships | Entity Relationship | `erDiagram` |
| States & transitions | State Machine | `stateDiagram-v2` |
| Class structure | Class Diagram | `classDiagram` |
| Timeline, schedule | Gantt Chart | `gantt` |
| Hierarchy, org chart | Flowchart (TB) | `flowchart TB` |
| Mind map, concepts | Mind Map | `mindmap` |
| User journey | User Journey | `journey` |

**The Diagram Quality Ladder:**

| Level | State | Characteristics |
|-------|-------|-----------------|
| 0 | Napkin | Scribbled boxes, unclear purpose |
| 1 | Drafted | Right type, content incomplete |
| 2 | Complete | All elements present, hard to read |
| 3 | Readable | Clear layout, inconsistent style |
| 4 | Polished | Consistent style, good visual hierarchy |
| 5 | Publication-ready | Legend, title, optimal detail level |

**Target: Level 4-5 for documentation; Level 3 for working drafts.**

---

## Diagram Types Deep Dive

### 1. Flowchart

**Best For:** Process flows, decision trees, algorithms, system overviews

**When to Use:**
- Showing step-by-step processes
- Decision logic with branches
- High-level system architecture
- Data flow between components

**Syntax Basics:**
```mermaid
flowchart LR
    A[Start] --> B{Decision?}
    B -->|Yes| C[Action 1]
    B -->|No| D[Action 2]
    C --> E[End]
    D --> E
```

**Direction Options:**
- `TB` / `TD` — Top to Bottom (default)
- `BT` — Bottom to Top
- `LR` — Left to Right (good for processes)
- `RL` — Right to Left

**Node Shapes:**
```mermaid
flowchart LR
    A[Rectangle] --> B(Rounded)
    B --> C([Stadium])
    C --> D[[Subroutine]]
    D --> E[(Database)]
    E --> F((Circle))
    F --> G{Diamond}
    G --> H{{Hexagon}}
```

| Shape | Meaning | Use For |
|-------|---------|---------|
| `[text]` | Rectangle | Process, action |
| `(text)` | Rounded | Start/end |
| `[(text)]` | Stadium | Terminal |
| `[[text]]` | Subroutine | External process |
| `[(text)]` | Cylinder | Database |
| `((text))` | Circle | Connector |
| `{text}` | Diamond | Decision |
| `{{text}}` | Hexagon | Preparation |

**Edge Styles:**
```mermaid
flowchart LR
    A --> B
    A --- C
    A -.-> D
    A ==> E
    A --text--> F
    A -->|label| G
```

**Subgraphs for Grouping:**
```mermaid
flowchart TB
    subgraph Frontend
        A[Web App]
        B[Mobile App]
    end
    subgraph Backend
        C[API Gateway]
        D[Services]
    end
    A --> C
    B --> C
    C --> D
```

**Good Example — API Request Flow:**
```mermaid
flowchart LR
    subgraph Client
        A[Browser]
    end
    
    subgraph Edge
        B[CDN]
        C[Load Balancer]
    end
    
    subgraph Application
        D[API Gateway]
        E[Auth Service]
        F[Order Service]
    end
    
    subgraph Data
        G[(PostgreSQL)]
        H[(Redis)]
    end
    
    A -->|HTTPS| B
    B --> C
    C --> D
    D -->|Validate| E
    E -->|Token| D
    D --> F
    F --> G
    F --> H
```

---

### 2. Sequence Diagram

**Best For:** API interactions, service communication, message flow over time

**When to Use:**
- Showing request/response patterns
- Multi-service interactions
- Protocol flows (auth, payment)
- Debugging distributed systems

**Syntax Basics:**
```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    participant D as Database
    
    C->>S: POST /orders
    S->>D: INSERT order
    D-->>S: order_id
    S-->>C: 201 Created
```

**Arrow Types:**
| Arrow | Meaning |
|-------|---------|
| `->` | Solid line (sync call) |
| `-->` | Dotted line (response) |
| `->>` | Solid with arrowhead |
| `-->>` | Dotted with arrowhead |
| `-x` | Solid with X (async/fire-forget) |
| `--x` | Dotted with X |

**Advanced Features:**
```mermaid
sequenceDiagram
    participant U as User
    participant A as API
    participant P as Payment
    participant D as Database
    
    U->>A: Create Order
    
    activate A
    A->>D: Save Draft
    D-->>A: OK
    
    A->>P: Process Payment
    activate P
    
    alt Payment Success
        P-->>A: Approved
        A->>D: Update Status
        A-->>U: Order Confirmed
    else Payment Failed
        P-->>A: Declined
        A-->>U: Payment Failed
    end
    
    deactivate P
    deactivate A
    
    Note over U,A: Order complete
```

**Key Constructs:**
- `activate`/`deactivate` — Show processing time
- `alt`/`else`/`end` — Conditional flows
- `loop`/`end` — Repeated actions
- `opt`/`end` — Optional section
- `par`/`and`/`end` — Parallel actions
- `Note over`/`Note left of`/`Note right of` — Annotations

**Good Example — OAuth Flow:**
```mermaid
sequenceDiagram
    participant U as User
    participant C as Client App
    participant A as Auth Server
    participant R as Resource Server
    
    U->>C: Click Login
    C->>A: Authorization Request
    A->>U: Login Page
    U->>A: Credentials
    A->>A: Validate
    A-->>C: Authorization Code
    
    C->>A: Code + Client Secret
    activate A
    A-->>C: Access Token + Refresh Token
    deactivate A
    
    C->>R: API Request + Token
    activate R
    R->>A: Validate Token
    A-->>R: Token Valid
    R-->>C: Protected Resource
    deactivate R
    
    C-->>U: Display Data
```

---

### 3. Entity Relationship Diagram

**Best For:** Data models, database schema, entity relationships

**When to Use:**
- Designing database schema
- Documenting data models
- Showing relationships between entities
- Planning migrations

**Syntax Basics:**
```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "ordered in"
```

**Cardinality Notation:**
| Symbol | Meaning |
|--------|---------|
| `\|\|` | Exactly one |
| `o\|` | Zero or one |
| `}\|` | One or more |
| `o{` | Zero or more |

**With Attributes:**
```mermaid
erDiagram
    CUSTOMER {
        uuid id PK
        string email UK
        string name
        timestamp created_at
    }
    
    ORDER {
        uuid id PK
        uuid customer_id FK
        enum status
        decimal total
        timestamp created_at
    }
    
    ORDER_ITEM {
        uuid id PK
        uuid order_id FK
        uuid product_id FK
        int quantity
        decimal unit_price
    }
    
    PRODUCT {
        uuid id PK
        string sku UK
        string name
        decimal price
    }
    
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    PRODUCT ||--o{ ORDER_ITEM : "ordered as"
```

**Attribute Markers:**
- `PK` — Primary Key
- `FK` — Foreign Key
- `UK` — Unique Key

**Good Example — E-commerce Domain:**
```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    CUSTOMER ||--|{ ADDRESS : has
    ORDER ||--|{ ORDER_ITEM : contains
    ORDER }|--|| ADDRESS : "ships to"
    ORDER }|--o| COUPON : uses
    PRODUCT ||--o{ ORDER_ITEM : "ordered as"
    PRODUCT }|--|| CATEGORY : "belongs to"
    PRODUCT ||--o{ PRODUCT_IMAGE : has
    PRODUCT ||--o{ INVENTORY : "stocked in"
    WAREHOUSE ||--|{ INVENTORY : holds
    
    CUSTOMER {
        uuid id PK
        string email UK
        string name
        timestamp created_at
    }
    
    ORDER {
        uuid id PK
        string order_number UK
        uuid customer_id FK
        uuid shipping_address_id FK
        uuid coupon_id FK
        enum status
        decimal total
    }
    
    PRODUCT {
        uuid id PK
        string sku UK
        uuid category_id FK
        string name
        decimal price
        boolean active
    }
```

---

### 4. State Diagram

**Best For:** State machines, lifecycle flows, status transitions

**When to Use:**
- Object lifecycle (order, user, document)
- Workflow states
- Protocol states
- UI state machines

**Syntax Basics:**
```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Pending: Submit
    Pending --> Approved: Approve
    Pending --> Rejected: Reject
    Approved --> [*]
    Rejected --> Draft: Revise
```

**Advanced Features:**
```mermaid
stateDiagram-v2
    [*] --> Idle
    
    Idle --> Processing: Start
    
    state Processing {
        [*] --> Validating
        Validating --> Executing: Valid
        Validating --> Error: Invalid
        Executing --> Complete
        Complete --> [*]
    }
    
    Processing --> Idle: Cancel
    Processing --> Error: Failure
    
    Error --> Idle: Reset
    
    note right of Processing
        May take up to 30 seconds
    end note
```

**Constructs:**
- `[*]` — Start/end state
- `state {}` — Composite state
- `note` — Annotations
- `<<fork>>` / `<<join>>` — Parallel states

**Good Example — Order Lifecycle:**
```mermaid
stateDiagram-v2
    [*] --> Draft: Create Order
    
    Draft --> Pending: Submit
    Draft --> Cancelled: User Cancel
    
    Pending --> Paid: Payment Success
    Pending --> Cancelled: Payment Timeout
    Pending --> Draft: Payment Failed
    
    Paid --> Processing: Begin Fulfillment
    Paid --> Refunded: Refund Request
    
    state Processing {
        [*] --> Picking
        Picking --> Packing
        Packing --> ReadyToShip
        ReadyToShip --> [*]
    }
    
    Processing --> Shipped: Carrier Pickup
    Processing --> Refunded: Cannot Fulfill
    
    Shipped --> Delivered: Delivery Confirmed
    Shipped --> Refunded: Lost/Damaged
    
    Delivered --> [*]
    Cancelled --> [*]
    Refunded --> [*]
    
    note right of Pending
        30 min payment timeout
    end note
    
    note right of Shipped
        14 day delivery window
    end note
```

---

### 5. C4 Diagrams

**Best For:** Software architecture at multiple levels of abstraction

**C4 Levels:**
1. **Context** — System in its environment
2. **Container** — High-level tech building blocks
3. **Component** — Components within containers
4. **Code** — Class-level (use classDiagram)

**Context Diagram:**
```mermaid
C4Context
    title System Context - Order Management
    
    Person(customer, "Customer", "Places orders online")
    Person(admin, "Admin", "Manages orders and inventory")
    
    System(orderSystem, "Order Management", "Handles order processing")
    
    System_Ext(payment, "Payment Gateway", "Processes payments")
    System_Ext(shipping, "Shipping Provider", "Handles delivery")
    System_Ext(email, "Email Service", "Sends notifications")
    
    Rel(customer, orderSystem, "Places orders", "HTTPS")
    Rel(admin, orderSystem, "Manages", "HTTPS")
    Rel(orderSystem, payment, "Processes payments", "HTTPS")
    Rel(orderSystem, shipping, "Creates shipments", "API")
    Rel(orderSystem, email, "Sends emails", "SMTP")
```

**Container Diagram:**
```mermaid
C4Container
    title Container Diagram - Order Management
    
    Person(customer, "Customer")
    
    Container_Boundary(orderSystem, "Order Management") {
        Container(web, "Web App", "React", "Customer-facing UI")
        Container(api, "API Gateway", "Kong", "Routes requests")
        Container(orders, "Order Service", "Node.js", "Order logic")
        Container(inventory, "Inventory Service", "Go", "Stock management")
        Container(db, "Database", "PostgreSQL", "Order data")
        Container(cache, "Cache", "Redis", "Session, caching")
        Container(queue, "Message Queue", "RabbitMQ", "Async processing")
    }
    
    System_Ext(payment, "Payment Gateway")
    
    Rel(customer, web, "Uses", "HTTPS")
    Rel(web, api, "Calls", "HTTPS")
    Rel(api, orders, "Routes to", "gRPC")
    Rel(orders, inventory, "Checks stock", "gRPC")
    Rel(orders, db, "Reads/writes", "SQL")
    Rel(orders, cache, "Caches", "Redis protocol")
    Rel(orders, queue, "Publishes", "AMQP")
    Rel(orders, payment, "Charges", "HTTPS")
```

---

### 6. Class Diagram

**Best For:** Object structure, interfaces, inheritance

**When to Use:**
- API request/response types
- Domain model design
- Interface definitions
- Inheritance hierarchies

**Syntax:**
```mermaid
classDiagram
    class Order {
        +UUID id
        +String orderNumber
        +OrderStatus status
        +Decimal total
        +create()
        +submit()
        +cancel()
    }
    
    class OrderItem {
        +UUID id
        +UUID productId
        +int quantity
        +Decimal unitPrice
        +getTotal() Decimal
    }
    
    class OrderStatus {
        <<enumeration>>
        DRAFT
        PENDING
        PAID
        SHIPPED
        DELIVERED
    }
    
    Order "1" *-- "1..*" OrderItem : contains
    Order --> OrderStatus : has
```

**Relationship Types:**
| Symbol | Type |
|--------|------|
| `<\|--` | Inheritance |
| `*--` | Composition |
| `o--` | Aggregation |
| `-->` | Association |
| `..>` | Dependency |
| `..\|>` | Realization |

---

### 7. Gantt Chart

**Best For:** Project timelines, schedules, dependencies

**Syntax:**
```mermaid
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD
    
    section Planning
    Requirements     :a1, 2024-01-01, 7d
    Design           :a2, after a1, 5d
    
    section Development
    Backend          :b1, after a2, 14d
    Frontend         :b2, after a2, 14d
    Integration      :b3, after b1, 7d
    
    section Testing
    QA Testing       :c1, after b3, 7d
    UAT              :c2, after c1, 5d
    
    section Launch
    Deployment       :milestone, d1, after c2, 1d
```

---

## Best Practices

### Do's

1. **Choose the right diagram type**
   - Process flow → Flowchart
   - Time-based interaction → Sequence
   - Data model → ERD
   - Object lifecycle → State

2. **Keep it focused**
   - One diagram = one purpose
   - Split complex diagrams
   - Link related diagrams

3. **Use consistent styling**
   - Same shapes for same concepts
   - Consistent naming conventions
   - Uniform edge styles

4. **Provide context**
   - Add title (comment or note)
   - Use subgraphs for grouping
   - Include legend if needed

5. **Optimize layout**
   - Choose direction (LR for processes, TB for hierarchies)
   - Group related nodes
   - Minimize edge crossings

### Don'ts

1. **Don't overcrowd**
   - Max ~15 nodes per diagram
   - Split if more complex
   - Use layers of abstraction

2. **Don't mix abstractions**
   - Don't show classes and servers in same diagram
   - Keep consistent level of detail

3. **Don't over-style**
   - Avoid excessive colors
   - Keep labels concise
   - Don't use every shape available

4. **Don't forget the audience**
   - Technical vs non-technical
   - Explanation vs reference
   - Detail level appropriate

---

## Common Patterns by Artifact Type

### For PRD/FRD
```mermaid
flowchart LR
    subgraph User Journey
        A[Land on Site] --> B[Browse Products]
        B --> C[Add to Cart]
        C --> D[Checkout]
        D --> E[Order Confirmation]
    end
```

### For Architecture
```mermaid
flowchart TB
    subgraph Presentation
        A[Web App]
        B[Mobile App]
    end
    
    subgraph Services
        C[API Gateway]
        D[Business Services]
    end
    
    subgraph Data
        E[(Primary DB)]
        F[(Cache)]
    end
    
    A & B --> C
    C --> D
    D --> E & F
```

### For Technical Spec
```mermaid
sequenceDiagram
    participant C as Client
    participant G as Gateway
    participant S as Service
    participant D as Database
    
    C->>G: Request
    G->>S: Validated Request
    S->>D: Query
    D-->>S: Result
    S-->>G: Response
    G-->>C: Final Response
```

### For Migration Plan
```mermaid
stateDiagram-v2
    [*] --> Current
    Current --> Preparation: Phase 1
    Preparation --> Migration: Phase 2
    Migration --> Validation: Phase 3
    Validation --> Cutover: Phase 4
    Cutover --> Cleanup: Phase 5
    Cleanup --> [*]: Complete
```

### For Implementation Plan
```mermaid
gantt
    title Implementation Phases
    
    section Phase 1 - Foundation
    Database setup    :p1a, 2024-01-01, 3d
    API scaffolding   :p1b, after p1a, 2d
    
    section Phase 2 - Core
    Business logic    :p2a, after p1b, 5d
    Integration       :p2b, after p2a, 3d
    
    section Phase 3 - Polish
    Testing           :p3a, after p2b, 4d
    Documentation     :p3b, after p2b, 2d
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Diagram too wide | Use `TB` direction, or split diagram |
| Edges crossing | Reorder node definitions, use subgraphs |
| Text cut off | Shorten labels, use abbreviations |
| Rendering fails | Check syntax, especially quotes and special chars |
| Too cluttered | Remove detail, create sub-diagrams |
| Inconsistent look | Apply consistent shapes and naming |

### Escaping Special Characters
```mermaid
flowchart LR
    A["Label with (parens)"]
    B["Label with [brackets]"]
    C["Label with 'quotes'"]
```

### Line Breaks in Labels
```mermaid
flowchart LR
    A["Line 1<br/>Line 2"]
```

---

## Evaluation Dimensions for Diagram Review

### Dimension 1: Purpose Clarity
- Does the diagram have a clear purpose?
- Is the purpose stated (title/caption)?
- Does every element serve the purpose?

### Dimension 2: Type Appropriateness
- Is this the right diagram type?
- Would another type communicate better?
- Is the abstraction level consistent?

### Dimension 3: Content Completeness
- Are all necessary elements present?
- Are relationships complete?
- Is anything missing that would cause confusion?

### Dimension 4: Visual Clarity
- Is the layout readable?
- Are node and edge labels clear?
- Is there a logical visual flow?

### Dimension 5: Consistency
- Are naming conventions consistent?
- Are shapes used consistently for same concepts?
- Is styling uniform?

### Dimension 6: Detail Level
- Is detail appropriate for audience?
- Is anything too detailed or too vague?
- Could the diagram be simplified?

---

## Quick Reference

### Flowchart
```
flowchart LR
    A[Action] --> B{Decision}
    B -->|Yes| C[Result]
```

### Sequence
```
sequenceDiagram
    A->>B: Request
    B-->>A: Response
```

### ERD
```
erDiagram
    A ||--o{ B : has
```

### State
```
stateDiagram-v2
    [*] --> State1
    State1 --> State2
```

### Class
```
classDiagram
    class A {
        +field
        +method()
    }
```

### Gantt
```
gantt
    Task :a1, 2024-01-01, 7d
```

---

## Invariants

1. **Every diagram MUST have a clear purpose** — no "just because" diagrams
2. **Diagram type MUST match the content** — don't force flowcharts on everything
3. **Detail level MUST be consistent** — don't mix high and low abstraction
4. **Labels MUST be readable** — if you can't read it, simplify it
5. **Relationships MUST be explicit** — no assumed connections
6. **Style MUST be consistent** — same shapes mean same things
7. **Complex diagrams MUST be split** — max ~15 nodes per diagram
8. **Diagrams MUST stand alone** — understandable without prose (mostly)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-12-31 | Initial guide: 7 diagram types, patterns, best practices, invariants |

---

*Visual Clarity Reviewer — Making the complex visible and the visible understandable.*
