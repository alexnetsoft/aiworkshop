# ABL Business Entity Architecture Pattern

## Overview

The Business Entity pattern provides a standardized, maintainable approach to data access in OpenEdge ABL applications. It separates UI logic from database operations through a layered architecture that promotes reusability, testability, and consistency.

## Architecture Layers

### 1. UI Layer (Windows/Forms)
- User interaction and presentation
- Never directly accesses database tables
- Calls Business Entity methods with datasets

### 2. Business Entity Layer
- Data access, business rules, validation
- Extends `OpenEdge.BusinessLogic.BusinessEntity`
- Instantiated through EntityFactory (singleton pattern)

### 3. Database Layer
- Persistent storage
- Only through data-sources attached to business entities

## Key Components

### EntityFactory (Singleton Pattern)

```abl
CLASS business.EntityFactory:
    VAR PRIVATE STATIC EntityFactory objInstance.
    VAR PRIVATE CustomerEntity objCustomerEntityInstance.
    
    CONSTRUCTOR PRIVATE EntityFactory():
    END CONSTRUCTOR.
    
    METHOD PUBLIC STATIC EntityFactory GetInstance():
        IF objInstance = ? THEN
            objInstance = NEW EntityFactory().
        RETURN objInstance.
    END METHOD.
    
    METHOD PUBLIC CustomerEntity GetCustomerEntity():
        IF objCustomerEntityInstance = ? THEN
            objCustomerEntityInstance = NEW CustomerEntity().
        RETURN objCustomerEntityInstance.
    END METHOD.
END CLASS.
```

### Dataset Definition (.i Include Files)

```abl
DEFINE TEMP-TABLE ttCustomer BEFORE-TABLE bttCustomer
    FIELD CustNum AS INTEGER INITIAL "0" LABEL "Cust Num"
    FIELD Name AS CHARACTER LABEL "Name"
    FIELD Address AS CHARACTER LABEL "Address"
    INDEX CustNum IS PRIMARY UNIQUE CustNum ASCENDING.

DEFINE DATASET dsCustomer FOR ttCustomer.
```

### Business Entity Class

```abl
CLASS business.CustomerEntity INHERITS BusinessEntity USE-WIDGET-POOL:
    {business/CustomerDataset.i}
    DEFINE DATA-SOURCE srcCustomer FOR Customer.
    
    CONSTRUCTOR PUBLIC CustomerEntity():
        SUPER(DATASET dsCustomer:HANDLE).
        VAR HANDLE[1] hDataSourceArray = DATA-SOURCE srcCustomer:HANDLE.
        VAR CHARACTER[1] cSkipListArray = [""].
        THIS-OBJECT:ProDataSource = hDataSourceArray.
        THIS-OBJECT:SkipList = cSkipListArray.
    END CONSTRUCTOR.
END CLASS.
```

## Standard CRUD Operations

### Read (Query Operations)

```abl
METHOD PUBLIC LOGICAL GetCustomerByNumber(INPUT ipiCustNum AS INTEGER,
                                          OUTPUT DATASET dsCustomer):
    VAR CHARACTER cFilter.
    VAR LOGICAL lFound = FALSE.
    
    cFilter = "WHERE Customer.CustNum = " + STRING(ipiCustNum).
    THIS-OBJECT:ReadData(cFilter).
    lFound = CAN-FIND(FIRST ttCustomer).
    
    RETURN lFound.
END METHOD.
```

**Important**: Use `OUTPUT DATASET` without `BY-REFERENCE` for read operations.

### Create Operations

```abl
METHOD PUBLIC VOID CreateCustomer(INPUT-OUTPUT DATASET dsCustomer):
    THIS-OBJECT:CreateData(DATASET dsCustomer BY-REFERENCE).
END METHOD.
```

### Update Operations

```abl
METHOD PUBLIC VOID UpdateCustomer(INPUT-OUTPUT DATASET dsCustomer):
    THIS-OBJECT:UpdateData(DATASET dsCustomer BY-REFERENCE).
END METHOD.
```

**Usage from UI**:
```abl
TEMP-TABLE ttCustomer:TRACKING-CHANGES = TRUE.
FIND FIRST ttCustomer.
ttCustomer.Name = "Updated Name".
objCustomerEntity:UpdateCustomer(INPUT-OUTPUT DATASET dsCustomer BY-REFERENCE).
```

### Delete Operations

```abl
METHOD PUBLIC VOID DeleteCustomer(INPUT-OUTPUT DATASET dsCustomer):
    THIS-OBJECT:DeleteData(DATASET dsCustomer BY-REFERENCE).
END METHOD.
```

## Validation Pattern

```abl
METHOD PUBLIC LOGICAL ValidateCustomer(INPUT-OUTPUT DATASET dsCustomer,
                                     OUTPUT errorMessage AS CHARACTER):
    VAR LOGICAL isValid = TRUE.
    
    FIND FIRST ttCustomer NO-ERROR.
    IF AVAILABLE ttCustomer THEN DO:
        IF ttCustomer.Name = "" THEN DO:
            isValid = FALSE.
            errorMessage = "Customer name cannot be empty".
        END.
    END.
    
    RETURN isValid.
END METHOD.
```

## UI Integration Pattern

```abl
USING business.CustomerEntity FROM PROPATH.
USING business.EntityFactory FROM PROPATH.

{business/CustomerDataset.i}

ON CHOOSE OF GetCustomer IN FRAME DEFAULT-FRAME:
    VAR INTEGER iCustomerNumber = INTEGER(CustomerNumber:screen-value).
    VAR EntityFactory objFactory = EntityFactory:GetInstance().
    VAR CustomerEntity objCustomerEntity = objFactory:GetCustomerEntity().
    VAR LOGICAL lCustomerFound.
    
    lCustomerFound = objCustomerEntity:GetCustomerByNumber(
        iCustomerNumber, 
        OUTPUT DATASET dsCustomer
    ).
    
    IF lCustomerFound THEN DO:
        FIND FIRST ttCustomer.
        IF AVAILABLE ttCustomer THEN DO:
            CustomerName = ttCustomer.Name.
            DISPLAY CustomerName WITH FRAME {&frame-name}.
        END.
    END.
    ELSE 
        MESSAGE "Customer not found" VIEW-AS ALERT-BOX.
END.
```

## Common Pitfalls

1. **Using BY-REFERENCE on OUTPUT DATASET for Read**: Use `OUTPUT DATASET` without `BY-REFERENCE` for read operations.
2. **Forgetting Change Tracking**: Enable `TRACKING-CHANGES` on temp-table before updates.
3. **Missing Data-Source Assignment**: Assign data-source handles to `ProDataSource` in constructor.
4. **Direct Database Access from UI**: Always use business entities, never direct database access.
5. **Not Using Named Buffers**: Always use named buffers when accessing database tables.

## Refactoring Steps

1. Identify data access patterns in legacy code
2. Create dataset definition include file
3. Create business entity class inheriting from BusinessEntity
4. Add entity to EntityFactory
5. Refactor UI code to use entity methods
6. Extract validation logic to entity
7. Test incrementally

## Benefits

- **Maintainability**: Centralized data access logic
- **Reusability**: Business entities shared across UI components
- **Testability**: Business logic isolated from UI
- **Consistency**: Standard patterns across application
- **Scalability**: Easy to add new entities