# Operational Expense ER Diagram

> **Database type:** MongoDB with Mongoose  
> In this document, the word **collection** is used instead of SQL table. Each main collection uses MongoDB's `_id` as its primary key. Report rows, fixed costs, report allocations, unmatched Tally rows, and location allocations are embedded subdocuments.

## Key Legend

- **PK**: Primary Key
- **FK**: Foreign Key / MongoDB ObjectId reference
- **UK**: Unique Key
- **CUK**: Compound Unique Key
- **EPK**: Embedded document key
- `||--o{`: One-to-many relationship
- `||--|{`: One-to-many embedded relationship

## ER Diagram

```mermaid
erDiagram
    PERMISSIONS ||--o{ ROLES : "permissionId"
    ROLES ||--o{ USERS : "roleId"

    USERS ||--o{ FINANCIAL_CATEGORIES : "createdBy / updatedBy"
    USERS ||--o{ COMPONENTS : "createdBy / updatedBy"
    USERS ||--o{ CLIENTS : "createdBy / updatedBy"
    USERS ||--o{ CLIENT_LOCATIONS : "createdBy / updatedBy"
    USERS ||--o{ OPERATIONAL_EXPENSE_REPORTS : "createdBy / updatedBy"
    USERS ||--o{ OPERATIONAL_EXPENSE_INVOICE_ALLOCATIONS : "createdBy / updatedBy"
    USERS ||--o{ OPERATIONAL_EXPENSE_INVOICES : "createdBy"

    FINANCIAL_CATEGORIES ||--o{ FINANCIAL_SUBCATEGORIES : "financialCategoryId"
    FINANCIAL_CATEGORIES ||--o{ COMPONENTS : "financialCategoryId"
    FINANCIAL_SUBCATEGORIES ||--o{ COMPONENTS : "financialSubCategoryId"

    COMPONENTS ||--o{ SUBCOMPONENTS : "componentId"
    COMPONENTS ||--o{ OPERATIONAL_EXPENSE_REPORT_ROWS : "componentId"

    CLIENTS ||--o{ CLIENT_LOCATIONS : "clientId"
    CLIENTS ||--o{ REPORT_CLIENT_ALLOCATIONS : "clientId"
    CLIENTS ||--o{ OPERATIONAL_EXPENSE_INVOICE_ALLOCATIONS : "clientId"
    CLIENTS ||--o{ OPERATIONAL_EXPENSE_INVOICES : "clientId"

    OPERATIONAL_EXPENSE_REPORTS ||--|{ OPERATIONAL_EXPENSE_REPORT_ROWS : "embeds rows"
    OPERATIONAL_EXPENSE_REPORTS ||--o{ REPORT_FIXED_COSTS : "embeds fixedCosts"
    OPERATIONAL_EXPENSE_REPORTS ||--o{ REPORT_CLIENT_ALLOCATIONS : "embeds allocations"
    OPERATIONAL_EXPENSE_REPORTS ||--o{ UNMATCHED_TALLY_ROWS : "embeds unmatchedTallyRows"
    OPERATIONAL_EXPENSE_REPORTS ||--o{ OPERATIONAL_EXPENSE_INVOICE_ALLOCATIONS : "reportId"
    OPERATIONAL_EXPENSE_REPORTS ||--o{ OPERATIONAL_EXPENSE_INVOICES : "reportId"

    OPERATIONAL_EXPENSE_INVOICE_ALLOCATIONS ||--o{ LOCATION_ALLOCATIONS : "embeds locationAllocations"
    CLIENT_LOCATIONS ||--o{ LOCATION_ALLOCATIONS : "locationId"
    CLIENT_LOCATIONS ||--o{ OPERATIONAL_EXPENSE_INVOICES : "locationId"

    PERMISSIONS {
        ObjectId _id PK
        String permissionName UK
        Boolean read
        Boolean write
        Boolean delete
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    ROLES {
        ObjectId _id PK
        String roleName UK
        ObjectId permissionId FK
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    USERS {
        ObjectId _id PK
        String username UK
        String password
        ObjectId roleId FK
        Boolean isActive
        Date createdAt
        Date updatedAt
    }

    FINANCIAL_CATEGORIES {
        ObjectId _id PK
        String categoryCode UK
        String categoryName
        String description
        Number displayOrder
        Boolean isActive
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
    }

    FINANCIAL_SUBCATEGORIES {
        ObjectId _id PK
        ObjectId financialCategoryId FK
        String subCategoryCode UK
        String subCategoryName
        String description
        Number displayOrder
        Boolean isActive
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
    }

    COMPONENTS {
        ObjectId _id PK
        String componentCode UK
        String componentName UK
        String description
        ObjectId financialCategoryId FK
        ObjectId financialSubCategoryId FK
        Boolean isPettyCash
        Boolean isCashFlowAnalysis
        Boolean isOperationalExpense
        Boolean isFullyLoadedCost
        Boolean requiresBill
        Number displayOrder
        Boolean isActive
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
        Number version
    }

    SUBCOMPONENTS {
        ObjectId _id PK
        ObjectId componentId FK
        String subComponentCode
        String subComponentName
        Boolean isPettyCash
        Boolean isCashFlowAnalysis
        Boolean isOperationalExpense
        Boolean isFullyLoadedCost
        Boolean requiresBill
        Number displayOrder
        Boolean isActive
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
        Number version
    }

    CLIENTS {
        ObjectId _id PK
        String clientCode UK
        String clientName UK
        String description
        Number displayOrder
        Boolean isActive
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
        Number version
    }

    CLIENT_LOCATIONS {
        ObjectId _id PK
        ObjectId clientId FK
        String locationCode CUK
        String locationName
        String billingName
        String billingAddress
        String stateName
        String stateCode
        String gstin
        Boolean isActive
        Number displayOrder
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
    }

    OPERATIONAL_EXPENSE_REPORTS {
        ObjectId _id PK
        String reportCode UK
        String reportName
        String reportMonth UK
        Date reportDate
        String currencyCode
        String sourceType
        String status
        Date generatedAt
        ObjectId generatedBy FK
        Date finalizedAt
        ObjectId finalizedBy FK
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
        Number version
    }

    OPERATIONAL_EXPENSE_REPORT_ROWS {
        ObjectId _id EPK
        ObjectId componentId FK
        String componentCode
        String componentName
        String rowSource
        ObjectId financialCategoryId FK
        String financialCategoryCode
        String financialCategoryName
        Number sourceAmount
        Number amount
        String matchStatus
        Boolean hasTallyExpense
        Boolean isManuallyAdded
        Boolean isEdited
        Number displayOrder
    }

    REPORT_FIXED_COSTS {
        ObjectId _id EPK
        String itemName
        String description
        Number amount
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
    }

    REPORT_CLIENT_ALLOCATIONS {
        ObjectId _id EPK
        ObjectId clientId FK
        String clientCode
        String clientName
        Number allocationPercentage
        Number amount
    }

    UNMATCHED_TALLY_ROWS {
        ObjectId _id EPK
        String tallyLedgerName
        Number amount
        Number excelRowNumber
        String reason
        String resolutionStatus
    }

    OPERATIONAL_EXPENSE_INVOICE_ALLOCATIONS {
        ObjectId _id PK
        ObjectId operationalExpenseReportId FK
        ObjectId clientId FK
        Number clientTotalAmount
        String allocationStatus
        Boolean isAutomatic
        ObjectId createdBy FK
        ObjectId updatedBy FK
        Date createdAt
        Date updatedAt
    }

    LOCATION_ALLOCATIONS {
        ObjectId _id EPK
        ObjectId locationId FK
        Number amount
    }

    OPERATIONAL_EXPENSE_INVOICES {
        ObjectId _id PK
        ObjectId operationalExpenseReportId FK
        ObjectId clientId FK
        ObjectId locationId FK
        String status
        Number gstRate
        Number taxableAmount
        Number gstAmount
        Number grandTotal
        ObjectId createdBy FK
        Date createdAt
        Date updatedAt
    }
```

## Collection Structure and Keys

### 1. `permissions`

```text
permissions
├── _id                 ObjectId    PK
├── permissionName      String      Required, Unique
├── read                Boolean     Default: false
├── write               Boolean     Default: false
├── delete              Boolean     Default: false
├── isActive            Boolean     Default: true
├── createdAt           Date
└── updatedAt           Date
```

### 2. `roles`

```text
roles
├── _id                 ObjectId    PK
├── roleName            String      Required, Unique
├── permissionId        ObjectId    FK -> permissions._id
├── isActive            Boolean
├── createdAt           Date
└── updatedAt           Date
```

### 3. `users`

```text
users
├── _id                 ObjectId    PK
├── username            String      Required, Unique
├── password            String      Required, Hashed
├── roleId              ObjectId    FK -> roles._id
├── isActive            Boolean
├── createdAt           Date
└── updatedAt           Date
```

### 4. `financial_categories`

```text
financial_categories
├── _id                 ObjectId    PK
├── categoryCode        String      Required, Unique
├── categoryName        String      Required
├── description         String
├── displayOrder        Number
├── isActive            Boolean
├── createdBy           ObjectId    FK -> users._id
├── updatedBy           ObjectId    FK -> users._id
├── createdAt           Date
└── updatedAt           Date
```

### 5. `financial_sub_categories`

```text
financial_sub_categories
├── _id                         ObjectId    PK
├── financialCategoryId         ObjectId    FK -> financial_categories._id
├── subCategoryCode             String      Required, Unique
├── subCategoryName             String      Required
├── description                 String
├── displayOrder                Number
├── isActive                    Boolean
├── createdBy                   ObjectId    FK -> users._id
├── updatedBy                   ObjectId    FK -> users._id
├── createdAt                   Date
└── updatedAt                   Date
```

### 6. `components`

```text
components
├── _id                         ObjectId    PK
├── componentCode               String      Required, Unique, Immutable
├── componentName               String      Required, Unique
├── description                 String
├── financialCategoryId         ObjectId    FK -> financial_categories._id
├── financialSubCategoryId      ObjectId    FK -> financial_sub_categories._id
├── isPettyCash                 Boolean
├── isCashFlowAnalysis          Boolean
├── isOperationalExpense        Boolean
├── isFullyLoadedCost           Boolean
├── requiresBill                Boolean
├── displayOrder                Number
├── isActive                    Boolean
├── createdBy                   ObjectId    FK -> users._id
├── updatedBy                   ObjectId    FK -> users._id
├── createdAt                   Date
├── updatedAt                   Date
└── version                     Number
```

### 7. `subcomponents`

```text
subcomponents
├── _id                         ObjectId    PK
├── componentId                 ObjectId    FK -> components._id
├── subComponentCode            String      Required, Immutable
├── subComponentName            String      Required
├── isPettyCash                 Boolean
├── isCashFlowAnalysis          Boolean
├── isOperationalExpense        Boolean
├── isFullyLoadedCost           Boolean
├── requiresBill                Boolean
├── displayOrder                Number
├── isActive                    Boolean
├── createdBy                   ObjectId    FK -> users._id
├── updatedBy                   ObjectId    FK -> users._id
├── createdAt                   Date
├── updatedAt                   Date
└── version                     Number
```

### 8. `clients`

```text
clients
├── _id                 ObjectId    PK
├── clientCode          String      Required, Unique, Immutable
├── clientName          String      Required, Unique
├── description         String
├── displayOrder        Number
├── isActive            Boolean
├── createdBy           ObjectId    FK -> users._id
├── updatedBy           ObjectId    FK -> users._id
├── createdAt           Date
├── updatedAt           Date
└── version             Number
```

### 9. `client_locations`

```text
client_locations
├── _id                 ObjectId    PK
├── clientId            ObjectId    FK -> clients._id
├── locationCode        String      Required, Compound Unique
├── locationName        String      Required
├── billingName         String
├── billingAddress      String
├── stateName           String
├── stateCode           String
├── gstin               String
├── isActive            Boolean
├── displayOrder        Number
├── createdBy           ObjectId    FK -> users._id
├── updatedBy           ObjectId    FK -> users._id
├── createdAt           Date
└── updatedAt           Date
```

```text
UNIQUE (clientId, locationCode)
```

### 10. `operational_expense_reports`

```text
operational_expense_reports
├── _id                         ObjectId    PK
├── reportCode                  String      Required, Unique
├── reportName                  String      Required
├── reportMonth                 String      Required, Unique, YYYY-MM
├── reportDate                  Date        Required
├── currencyCode                String      Default: INR
├── sourceType                  String      TALLY_IMPORT | MANUAL | REGENERATED
├── status                      String      DRAFT | FINALIZED | INVOICED
├── rows[]                                  Embedded report rows
├── unmatchedTallyRows[]                    Embedded unmatched rows
├── fixedCosts[]                            Embedded fixed costs
├── markup                                  Embedded object
├── totals                                  Embedded object
├── allocations[]                           Embedded client allocations
├── generatedAt                 Date
├── generatedBy                 ObjectId    FK -> users._id
├── finalizedAt                 Date
├── finalizedBy                 ObjectId    FK -> users._id
├── createdBy                   ObjectId    FK -> users._id
├── updatedBy                   ObjectId    FK -> users._id
├── createdAt                   Date
├── updatedAt                   Date
└── version                     Number
```

#### Embedded `rows[]`

```text
rows[]
├── _id                         ObjectId    EPK
├── componentId                 ObjectId    FK -> components._id, Nullable
├── componentCode               String
├── componentName               String
├── rowSource                   String
├── financialCategoryId         ObjectId    FK -> financial_categories._id
├── financialCategoryCode       String
├── financialCategoryName       String
├── sourceAmount                Number
├── amount                      Number
├── matchStatus                 String
├── hasTallyExpense             Boolean
├── matchedTallyLedgers[]                   Embedded array
├── isManuallyAdded             Boolean
├── isEdited                    Boolean
└── displayOrder                Number
```

#### Embedded `fixedCosts[]`

```text
fixedCosts[]
├── _id                 ObjectId    EPK
├── itemName            String      Required
├── description         String
├── amount              Number
├── createdBy           ObjectId    FK -> users._id
├── updatedBy           ObjectId    FK -> users._id
├── createdAt           Date
└── updatedAt           Date
```

#### Embedded `allocations[]`

```text
allocations[]
├── _id                     ObjectId    EPK
├── clientId                ObjectId    FK -> clients._id
├── clientCode              String
├── clientName              String
├── allocationPercentage    Number      0 to 100
└── amount                  Number
```

#### Embedded `markup`

```text
markup
├── enabled             Boolean
├── percentage          Number
├── baseType            String
├── baseAmount          Number
├── amount              Number
└── notes               String
```

#### Embedded `totals`

```text
totals
├── totalDirectCost              Number
├── totalIndirectCost            Number
├── subtotal                     Number
├── markupAmount                 Number
├── directorRemunerationTotal    Number
├── fixedCostTotal               Number
├── operationalExpenseTotal      Number
├── allocatedTotal               Number
└── unallocatedBalance           Number
```

### 11. `operational_expense_invoice_allocations`

```text
operational_expense_invoice_allocations
├── _id                             ObjectId    PK
├── operationalExpenseReportId      ObjectId    FK -> operational_expense_reports._id
├── clientId                        ObjectId    FK -> clients._id
├── clientTotalAmount               Number
├── locationAllocations[]                       Embedded array
├── allocationStatus                String      DRAFT | CONFIRMED
├── isAutomatic                     Boolean
├── createdBy                       ObjectId    FK -> users._id
├── updatedBy                       ObjectId    FK -> users._id
├── createdAt                       Date
└── updatedAt                       Date
```

```text
UNIQUE (operationalExpenseReportId, clientId)
```

#### Embedded `locationAllocations[]`

```text
locationAllocations[]
├── _id             ObjectId    EPK
├── locationId      ObjectId    FK -> client_locations._id
└── amount          Number
```

### 12. `operational_expense_invoices`

```text
operational_expense_invoices
├── _id                             ObjectId    PK
├── operationalExpenseReportId      ObjectId    FK -> operational_expense_reports._id
├── clientId                        ObjectId    FK -> clients._id
├── locationId                      ObjectId    FK -> client_locations._id, Nullable
├── status                          String      DRAFT | GENERATED
├── gstRate                         Number
├── taxableAmount                   Number
├── gstAmount                       Number
├── grandTotal                      Number
├── createdBy                       ObjectId    FK -> users._id
├── createdAt                       Date
└── updatedAt                       Date
```

```text
UNIQUE (operationalExpenseReportId, clientId, locationId)
```

## Relationship Summary

```text
Permission
    1 ───────< Role
                   1 ───────< User

Financial Category
    1 ───────< Financial Subcategory
    1 ───────< Component
                   1 ───────< Subcomponent

Client
    1 ───────< Client Location
    1 ───────< Report Client Allocation
    1 ───────< Invoice Allocation
    1 ───────< Invoice

Operational Expense Report
    1 ───────< Embedded Report Rows
    1 ───────< Embedded Fixed Costs
    1 ───────< Embedded Client Allocations
    1 ───────< Invoice Allocations
    1 ───────< Invoices

Invoice Allocation
    1 ───────< Embedded Location Allocations

Client Location
    1 ───────< Location Allocations
    1 ───────< Invoices
```

## Important Design Note

The system contains two client-allocation structures:

1. `operational_expense_reports.allocations[]` stores the report-level percentage and amount split.
2. `operational_expense_invoice_allocations` stores the final client amount and location-wise invoice allocation.

These structures serve different workflow stages. The backend should keep both values synchronized to prevent report allocation totals from differing from invoice allocation totals.
