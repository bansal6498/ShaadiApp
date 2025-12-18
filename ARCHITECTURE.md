# Wedding Management Web App - Architecture Diagram

## 1. System Architecture Overview

```mermaid
graph TB
    subgraph "Client Layer"
        UI[Web Browser]
        Mobile[Mobile Browser]
    end
    
    subgraph "Frontend Application"
        React[React/Next.js Frontend]
        Components[UI Components]
        State[State Management]
    end
    
    subgraph "API Layer"
        REST[REST API Server]
        Auth[Authentication Service]
        Validation[Input Validation]
    end
    
    subgraph "Business Logic Layer"
        FunctionService[Function Service]
        GuestService[Guest Service]
        BudgetService[Budget Service]
        PaymentService[Payment Service]
        ExportService[Export Service]
    end
    
    subgraph "Data Layer"
        DB[(PostgreSQL Database)]
        FileStorage[File Storage]
    end
    
    UI --> React
    Mobile --> React
    React --> Components
    Components --> State
    State --> REST
    REST --> Auth
    REST --> Validation
    Validation --> FunctionService
    Validation --> GuestService
    Validation --> BudgetService
    Validation --> PaymentService
    FunctionService --> DB
    GuestService --> DB
    BudgetService --> DB
    PaymentService --> DB
    ExportService --> DB
    ExportService --> FileStorage
```

## 2. Component Architecture

```mermaid
graph LR
    subgraph "Frontend Components"
        A[App Shell]
        B[Function List Module]
        C[Guest List Module]
        D[Budget Module]
        E[Payment Module]
        F[Export Module]
    end
    
    subgraph "Shared Components"
        G[Navigation]
        H[Data Table]
        I[Form Components]
        J[Modal/Dialog]
        K[Export Button]
    end
    
    A --> B
    A --> C
    A --> D
    A --> E
    A --> F
    A --> G
    
    B --> H
    B --> I
    B --> J
    
    C --> H
    C --> I
    C --> J
    C --> K
    
    D --> H
    D --> I
    
    E --> H
    E --> I
    
    F --> K
```

## 3. Database Schema Architecture

> **Note on `user_id`**: The `user_id` field in all tables (FUNCTIONS, GUESTS, BUDGET_ITEMS, PAYMENTS, GUEST_ATTRIBUTE_TYPES) is used for **multi-user support**. Each user can manage their own wedding independently. This ensures data isolation - when a user logs in, they only see and manage their own functions, guests, budget, payments, and custom attribute types. This is essential for a multi-tenant application where multiple users can use the same system for their separate weddings.

> **Note on Dynamic Guest Attributes**: The guest list now supports **dynamic custom columns**! Instead of fixed columns in the GUESTS table, we use a flexible system:
> - **GUEST_ATTRIBUTE_TYPES**: Stores custom attribute types (columns) that users can create (e.g., "Lunch", "Dinner", "Return Gift", "Reception", "Haldi", "Mehendi", etc.)
> - **GUEST_ATTRIBUTES**: Stores the actual values (checkboxes) for each guest-attribute combination
> - **Default Attributes**: System provides default attribute types (Lunch, Dinner, Return Gift, Ceremony) that users can customize
> - **Easy Management**: Users can add, edit, delete, and reorder custom attributes just like managing functions
> - **Unlimited Flexibility**: No schema changes needed - users can add as many custom columns as needed

```mermaid
erDiagram
    USERS ||--o{ FUNCTIONS : creates
    USERS ||--o{ GUESTS : manages
    USERS ||--o{ BUDGET_ITEMS : tracks
    USERS ||--o{ PAYMENTS : records
    USERS ||--o{ GUEST_ATTRIBUTE_TYPES : creates
    
    FUNCTIONS {
        int id PK
        string name
        datetime date
        time time
        int user_id FK
    }
    
    GUESTS {
        int id PK
        string name
        text notes
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
    GUEST_ATTRIBUTE_TYPES {
        int id PK
        string name
        string display_name
        boolean is_default
        int user_id FK
        int display_order
        datetime created_at
        datetime updated_at
    }
    
    GUEST_ATTRIBUTES {
        int id PK
        int guest_id FK
        int attribute_type_id FK
        boolean value
        datetime created_at
        datetime updated_at
    }
    
    USERS ||--o{ GUEST_ATTRIBUTE_TYPES : creates
    GUESTS ||--o{ GUEST_ATTRIBUTES : has
    GUEST_ATTRIBUTE_TYPES ||--o{ GUEST_ATTRIBUTES : defines
    
    BUDGET_ITEMS {
        int id PK
        string category
        decimal planned_amount
        decimal actual_amount
        decimal amount_paid
        decimal remaining_amount
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
    PAYMENTS {
        int id PK
        date payment_date
        decimal amount
        string category
        int budget_item_id FK
        string paid_by
        string payment_method
        string platform
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
    BUDGET_ITEMS ||--o{ PAYMENTS : "has payments"
    
    GUEST_FUNCTIONS {
        int id PK
        int guest_id FK
        int function_id FK
        int number_of_persons
        text notes
        datetime created_at
        datetime updated_at
    }
```

## 4. Data Flow Architecture

```mermaid
sequenceDiagram
    participant User
    participant Frontend
    participant API
    participant Service
    participant Database
    
    User->>Frontend: Add Function
    Frontend->>API: POST /api/functions
    API->>Service: FunctionService.create()
    Service->>Database: INSERT INTO functions
    Database-->>Service: Function created
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: Function added
    
    User->>Frontend: Add Guest with checkboxes
    Frontend->>API: POST /api/guests
    API->>Service: GuestService.create()
    Service->>Database: INSERT INTO guests
    Database-->>Service: Guest created
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: Guest added
    
    User->>Frontend: Export Guest List (Reception)
    Frontend->>API: GET /api/guests/export?filter=reception
    API->>Service: GuestService.export(filter)
    Service->>Database: SELECT * WHERE reception=true
    Database-->>Service: Guest list
    Service->>Service: Generate CSV/Excel
    Service-->>API: File data
    API-->>Frontend: File download
    Frontend-->>User: Download file
    
    User->>Frontend: Add Budget Item
    Frontend->>API: POST /api/budget
    API->>Service: BudgetService.create()
    Service->>Database: INSERT INTO budget_items
    Service->>Service: Calculate remaining_amount
    Database-->>Service: Budget item created
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: Budget item added
    
    User->>Frontend: Record Payment
    Frontend->>API: POST /api/payments
    API->>Service: PaymentService.create()
    Service->>Database: INSERT INTO payments
    Service->>Database: UPDATE budget_items SET amount_paid = amount_paid + payment.amount
    Service->>Database: UPDATE budget_items SET remaining_amount = planned_amount - amount_paid
    Database-->>Service: Payment recorded & Budget updated
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: Payment recorded & Budget updated
    
    User->>Frontend: Edit Guest Checkboxes
    Frontend->>API: PUT /api/guests/:id
    API->>Service: GuestService.update()
    Service->>Database: UPDATE guests SET reception, lunch, return_gift, etc.
    Database-->>Service: Guest updated
    Service-->>API: Success response
    API-->>Frontend: 200 OK
    Frontend-->>User: Guest updated
    
    User->>Frontend: Add Custom Attribute Type
    Frontend->>API: POST /api/guest-attributes/types
    API->>Service: GuestAttributeService.createType()
    Service->>Database: INSERT INTO guest_attribute_types
    Database-->>Service: Attribute type created
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: New column added to guest list
    
    User->>Frontend: Edit Guest with Custom Attributes
    Frontend->>API: GET /api/guest-attributes/types
    API->>Database: SELECT * FROM guest_attribute_types WHERE user_id=?
    Database-->>API: Attribute types list
    API-->>Frontend: Attribute types
    Frontend->>API: PATCH /api/guests/:id/attributes
    API->>Service: GuestService.updateAttributes()
    Service->>Database: UPSERT INTO guest_attributes
    Database-->>Service: Attributes updated
    Service-->>API: Success response
    API-->>Frontend: 200 OK
    Frontend-->>User: Guest attributes updated
```

## 5. Module Breakdown

```mermaid
graph TD
    subgraph "Function List Module"
        F1[Function List View]
        F2[Add Function Form]
        F3[Edit Function Form]
        F4[Delete Function]
        F5[Function Calendar View]
    end
    
    subgraph "Guest List Module"
        G1[Guest List View]
        G2[Add Guest Form]
        G3[Edit Guest Form]
        G4[Edit Guest Checkboxes Inline]
        G5[Delete Guest]
        G6[Guest Filter/Search]
        G7[Export Guest Lists]
        G8[Manage Custom Attributes]
        G9[Add Custom Attribute Type]
        G10[Edit Custom Attribute Type]
        G11[Delete Custom Attribute Type]
        G12[Reorder Attributes]
    end
    
    subgraph "Budget Module"
        B1[Budget Overview]
        B2[Add Budget Item]
        B3[Edit Budget Item]
        B4[Delete Budget Item]
        B5[Budget Summary Dashboard]
        B6[Category-wise Breakdown]
    end
    
    subgraph "Payment Module"
        P1[Payment List View]
        P2[Add Payment Form]
        P3[Edit Payment Form]
        P4[Delete Payment]
        P5[Payment Filter]
        P6[Payment Summary]
        P7[Category Dropdown from Budget]
        P8[Auto-deduct from Budget]
    end
    
    subgraph "Export Module"
        E1[Export Guest List by Function]
        E2[Export Full Guest List]
        E3[Export Budget Report]
        E4[Export Payment Report]
        E5[Export Combined Report]
    end
```

## 6. Technology Stack Recommendation

### Frontend
- **Framework**: React.js / Next.js
- **State Management**: Redux Toolkit / Zustand
- **UI Library**: Material-UI / Ant Design / Tailwind CSS
- **Form Handling**: React Hook Form
- **Data Table**: TanStack Table / Material-UI DataGrid
- **Export**: SheetJS (xlsx) / CSV export

### Backend
- **Runtime**: Node.js
- **Framework**: Express.js / Nest.js
- **ORM**: Prisma / TypeORM / Sequelize
- **Validation**: Joi / Zod
- **Authentication**: JWT / Passport.js

### Database
- **Primary DB**: PostgreSQL
- **ORM**: Prisma (recommended)

### Deployment
- **Frontend**: Vercel / Netlify
- **Backend**: AWS / Railway / Render
- **Database**: AWS RDS / Supabase / Railway PostgreSQL

## 7. API Endpoints Structure

```
/api
├── /auth
│   ├── POST /register
│   ├── POST /login
│   └── POST /logout
├── /functions
│   ├── GET / (list all)
│   ├── POST / (create)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update)
│   └── DELETE /:id (delete)
├── /guests
│   ├── GET / (list all)
│   ├── POST / (create)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update - including attributes)
│   ├── PATCH /:id/attributes (update only attributes)
│   ├── DELETE /:id (delete)
│   └── GET /export?filter=attribute_name (export filtered)
├── /guest-attributes
│   ├── GET /types (list all attribute types)
│   ├── POST /types (create new attribute type)
│   ├── GET /types/:id (get one attribute type)
│   ├── PUT /types/:id (update attribute type)
│   ├── DELETE /types/:id (delete attribute type)
│   ├── PUT /types/reorder (reorder attribute types)
│   └── GET /types/defaults (get default attribute types)
├── /budget
│   ├── GET / (list all)
│   ├── POST / (create)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update)
│   ├── DELETE /:id (delete)
│   └── GET /summary (get summary)
├── /payments
│   ├── GET / (list all)
│   ├── POST / (create - auto-deducts from budget)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update - recalculates budget)
│   ├── DELETE /:id (delete - reverses budget deduction)
│   ├── GET /summary (get summary)
│   └── GET /categories (get categories from budget)
└── /export
    ├── GET /guests/:type (export guest list)
    ├── GET /budget (export budget)
    └── GET /payments (export payments)
```

## 8. Feature Flow Diagram

```mermaid
flowchart TD
    Start([User Login]) --> Dashboard[Dashboard]
    Dashboard --> Functions[Function List]
    Dashboard --> Guests[Guest List]
    Dashboard --> Budget[Budget Management]
    Dashboard --> Payments[Payment Management]
    
    Functions --> AddFunction[Add Function]
    AddFunction --> FunctionForm[Enter Name, Date, Time]
    FunctionForm --> SaveFunction[Save Function]
    
    Guests --> AddGuest[Add Guest]
    AddGuest --> GuestForm[Enter Name + Checkboxes]
    GuestForm --> SaveGuest[Save Guest]
    Guests --> EditGuest[Edit Guest]
    EditGuest --> EditCheckboxes[Edit Checkboxes Inline]
    EditCheckboxes --> SaveGuestChanges[Save Changes]
    Guests --> ManageAttributes[Manage Custom Attributes]
    ManageAttributes --> AddAttributeType[Add New Attribute Type]
    ManageAttributes --> EditAttributeType[Edit Attribute Type]
    ManageAttributes --> DeleteAttributeType[Delete Attribute Type]
    ManageAttributes --> ReorderAttributes[Reorder Attributes]
    Guests --> ExportGuests[Export Guest Lists]
    ExportGuests --> FilterByAttribute[Filter by Any Attribute]
    
    Budget --> AddBudget[Add Budget Item]
    AddBudget --> BudgetForm[Enter Category, Amounts]
    BudgetForm --> CalculateRemaining[Calculate Remaining]
    CalculateRemaining --> SaveBudget[Save Budget]
    
    Payments --> AddPayment[Add Payment]
    AddPayment --> LoadCategories[Load Categories from Budget]
    LoadCategories --> PaymentForm[Enter Payment Details]
    PaymentForm --> SelectCategory[Select Category from Budget]
    SelectCategory --> UpdateBudget[Auto-deduct from Budget]
    UpdateBudget --> SavePayment[Save Payment]
```

## 9. Security Architecture

```mermaid
graph TB
    subgraph "Security Layers"
        A[HTTPS/TLS]
        B[JWT Authentication]
        C[Role-Based Access]
        D[Input Validation]
        E[SQL Injection Prevention]
        F[CSRF Protection]
    end
    
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
```

## 10. Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        CDN[CDN - Static Assets]
        Frontend[Frontend Hosting]
        API[API Server]
        DB[(Database)]
        Storage[File Storage]
    end
    
    User[Users] --> CDN
    User --> Frontend
    Frontend --> API
    API --> DB
    API --> Storage
```

---

## Key Design Decisions

1. **Modular Architecture**: Each feature (Functions, Guests, Budget, Payments) is a separate module for maintainability
2. **RESTful API**: Standard REST endpoints for easy integration and scalability
3. **Relational Database**: PostgreSQL for data integrity and complex queries
4. **Export Functionality**: Server-side export generation for better performance
5. **Real-time Calculations**: Budget remaining amount calculated automatically when payments are made
6. **Dynamic Guest Attributes**: Users can create, edit, and delete custom attribute types (columns) for guests. Default attributes (Lunch, Dinner, Return Gift, Ceremony) are provided, but users can add custom ones like "Reception", "Haldi", "Mehendi", etc.
7. **Flexible Guest Attributes**: Guest attributes are stored in a flexible many-to-many relationship, allowing unlimited custom columns without schema changes
8. **Payment-Budget Integration**: Payments automatically deduct from budget items, and payment categories are populated from existing budget categories
9. **Guest Management**: Guests can be edited after creation, including updating attributes inline. Custom attributes can be reordered and managed easily
10. **User Isolation**: `user_id` in all tables ensures multi-user support - each user manages their own wedding data independently, including their custom attribute types
11. **Guest-Function Details**: `GUEST_FUNCTIONS` table stores additional information like number of persons and notes for each guest-function relationship
12. **Attribute Type Management**: Users can add/edit/delete/reorder custom attribute types, making the guest list fully customizable to their wedding needs

