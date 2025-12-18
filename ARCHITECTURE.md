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

```mermaid
erDiagram
    USERS ||--o{ FUNCTIONS : creates
    USERS ||--o{ GUESTS : manages
    USERS ||--o{ BUDGET_ITEMS : tracks
    USERS ||--o{ PAYMENTS : records
    
    FUNCTIONS {
        int id PK
        string name
        datetime date
        time time
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
    GUESTS {
        int id PK
        string name
        boolean reception
        boolean lunch
        boolean return_gift
        boolean dinner
        boolean ceremony
        text notes
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
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
        string paid_by
        string payment_method
        string platform
        int user_id FK
        datetime created_at
        datetime updated_at
    }
    
    GUEST_FUNCTIONS {
        int id PK
        int guest_id FK
        int function_id FK
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
    Service->>Service: Update budget amount_paid
    Database-->>Service: Payment recorded
    Service-->>API: Success response
    API-->>Frontend: 201 Created
    Frontend-->>User: Payment recorded
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
        G4[Delete Guest]
        G5[Guest Filter/Search]
        G6[Export Guest Lists]
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
│   ├── PUT /:id (update)
│   ├── DELETE /:id (delete)
│   └── GET /export?filter=reception (export filtered)
├── /budget
│   ├── GET / (list all)
│   ├── POST / (create)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update)
│   ├── DELETE /:id (delete)
│   └── GET /summary (get summary)
├── /payments
│   ├── GET / (list all)
│   ├── POST / (create)
│   ├── GET /:id (get one)
│   ├── PUT /:id (update)
│   ├── DELETE /:id (delete)
│   └── GET /summary (get summary)
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
    Guests --> ExportGuests[Export Guest Lists]
    ExportGuests --> FilterReception[Filter by Reception]
    ExportGuests --> FilterLunch[Filter by Lunch]
    ExportGuests --> FilterReturnGift[Filter by Return Gift]
    
    Budget --> AddBudget[Add Budget Item]
    AddBudget --> BudgetForm[Enter Category, Amounts]
    BudgetForm --> CalculateRemaining[Calculate Remaining]
    CalculateRemaining --> SaveBudget[Save Budget]
    
    Payments --> AddPayment[Add Payment]
    AddPayment --> PaymentForm[Enter Payment Details]
    PaymentForm --> UpdateBudget[Update Budget Amount Paid]
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
5. **Real-time Calculations**: Budget remaining amount calculated automatically
6. **Flexible Guest Attributes**: Boolean flags for different functions allow easy filtering and export

