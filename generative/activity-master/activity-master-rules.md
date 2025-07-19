# ActivityMaster Integration Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Interface Hierarchies](#interface-hierarchies)
4. [Service Usage Patterns](#service-usage-patterns)
5. [Type Safety Requirements](#type-safety-requirements)
6. [Transaction Management](#transaction-management)
7. [Sequence Diagrams for Key Workflows](#sequence-diagrams-for-key-workflows)
8. [Use Cases with Code Examples](#use-cases-with-code-examples)
9. [Best Practices](#best-practices)
10. [Common Issues and Solutions](#common-issues-and-solutions)
11. [References](#references)

## Introduction

This guide provides comprehensive information for AI agents and developers to integrate smoothly with the ActivityMaster and CerialMaster libraries. It covers the architecture, interface hierarchies, service usage patterns, transaction management, and includes sequence diagrams and code examples for common use cases.

The ActivityMaster system is a Java implementation of a Financial Services Data Model (FSDM) utilizing Guicedee for dependency injection. It provides a robust framework for managing enterprises, classifications, security, and various domain-specific entities like addresses, events, involved parties, products, and resource items.

## Architecture Overview

### System Architecture

The ActivityMaster FSDM implements a service-oriented architecture with the following key components:

1. **Core Modules**:
   - **ActivityMasterCore**: Contains the core FSDM implementation, including base entity structures, core services, data model definitions, and database access layer.
   - **ActivityMasterClient**: Client-side implementation for consuming the FSDM services.

2. **Domain-Specific Modules**:
   - **GeographyMaster**: Handles geographical data and operations.
   - **CerialMaster**: Manages communication with serial ports.
   - **SessionMaster**: Manages user sessions and authentication.
   - **ProfileMaster**: Manages user profiles.
   - **ImageMaster**: Handles image storage and processing.
   - **DocumentMaster**: Manages document storage and processing.
   - And many others for specific domains.

3. **Service Layer**:
   - Services are accessed through Guice bound interfaces, not REST endpoints.
   - Key services include EnterpriseService, ClassificationService, SecurityTokenService, SystemsService, and domain-specific services.
   - Services implement reactive programming patterns using Mutiny (Uni<T> and Multi<T>).

4. **Data Access Layer**:
   - Uses JPA with custom extensions for database access.
   - Implements a query builder pattern for type-safe queries.
   - Supports temporal data management with Slowly Changing Dimension (SCD) pattern.

5. **Security Layer**:
   - Implements a token-based security model.
   - Each entity has an associated security token table.
   - Security tokens control CRUD permissions.

### Technology Stack

1. **Dependency Injection**: Google Guice (via Guicedee)
2. **Reactive Programming**: Mutiny (Uni<T> and Multi<T>)
3. **Persistence**: JPA with Hibernate Reactive
4. **Database**: SQL-based with comprehensive indexing strategy
5. **Security**: Custom token-based security model

### Entity Structure

The FSDM follows a sophisticated entity model with the following key components:

1. **Base Entity Structure**:
   - All entities inherit from a hierarchy of base classes:
     - SCDEntity → WarehouseBaseTable → WarehouseCoreTable → WarehouseTable → WarehouseSCDTable
   - Each level in the hierarchy adds specific columns and functionality.

2. **Core Entities**:
   - Enterprise: Represents a business entity
   - Classification: Categorizes other entities
   - Systems: Represents a system that interacts with the FSDM
   - SecurityToken: Manages access control

3. **Domain Entities**:
   - Address: Stores location information
   - Event: Represents activities or occurrences
   - Geography: Represents geographical locations
   - InvolvedParty: Represents individuals or organizations
   - Product: Represents products or services
   - ResourceItem: Represents resources
   - Rules: Represents business rules
   - Arrangement: Represents financial arrangements

4. **Relationship Structure**:
   - Entities are related through cross-reference (X) tables
   - All relationships include classification information
   - Relationships support temporal data management

5. **Temporal Data Management**:
   - Slowly Changing Dimension (SCD) pattern for tracking changes
   - All entities have effectiveFromDate and effectiveToDate
   - Historical records are preserved when data is updated

## Interface Hierarchies

The ActivityMaster system uses complex interface hierarchies to define the capabilities of entities and query builders. Understanding these hierarchies is essential for effective integration.

### Entity Interface Hierarchy

```
ISCDEntity (from com.guicedee.activitymaster.fsdm.client.services.builders)
└── IWarehouseSCDTable (from com.guicedee.activitymaster.fsdm.client.services.builders.warehouse.base)
    ├── IWarehouseSecurityTable
    ├── IWarehouseNameAndDescriptionTable
    └── IWarehouseCoreTable
        └── IWarehouseTable
            ├── IWarehouseRelationshipTable
            │   └── IWarehouseRelationshipClassificationTable
            │       └── IWarehouseRelationshipClassificationTypeTable
            └── IResourceData
```

Key interfaces in this hierarchy:

1. **ISCDEntity**: The base interface for entities with Slowly Changing Dimension functionality.
2. **IWarehouseSCDTable**: Extends ISCDEntity to add warehouse-specific functionality like expiring entities and getting table names.
3. **IWarehouseSecurityTable**: Extends IWarehouseSCDTable to add security-related functionality, including CRUD permissions and security tokens.
4. **IWarehouseCoreTable**: Extends IWarehouseSCDTable to add security-related functionality, including a method for creating default security.
5. **IWarehouseTable**: Extends IWarehouseCoreTable, UUID, and IContainsRowRecordInformation. This interface represents a standard warehouse table with security capabilities, SCD functionality, and original source system tracking.
6. **IWarehouseRelationshipTable**: Extends IWarehouseTable to represent relationships between two warehouse tables.
7. **IWarehouseRelationshipClassificationTable**: Extends IWarehouseRelationshipTable to add classification functionality.
8. **IResourceData**: Extends IWarehouseTable to represent resource items with data.

### Query Builder Interface Hierarchy

```
IQueryBuilder (from com.entityassist.services.querybuilders)
└── IQueryBuilderSCD (from com.guicedee.activitymaster.fsdm.client.services.builders)
    ├── IQueryBuilderWarehouseBaseTable (from com.guicedee.activitymaster.fsdm.client.services.builders) [Deprecated]
    │   └── IQueryBuilderWarehouseCoreTable
    │       └── IQueryBuilderEnterprise
    │           └── IQueryBuilderFlags
    │               └── IQueryBuilderValues
    │                   └── IQueryBuilderRelationships
    │                       └── IQueryBuilderClassifications
    └── IQueryBuilderSecurity
```

Key interfaces in this hierarchy:

1. **IQueryBuilder**: The base interface for all query builders.
2. **IQueryBuilderSCD**: Extends IQueryBuilder to add Slowly Changing Dimension (SCD) functionality, including date range queries and active range functionality.
3. **IQueryBuilderWarehouseCoreTable**: Extends IQueryBuilderWarehouseBaseTable to work with IWarehouseCoreTable entities.
4. **IQueryBuilderSecurity**: Extends IQueryBuilderSCD to add security-related functionality.
5. **IQueryBuilderEnterprise**: Extends IQueryBuilderWarehouseCoreTable to add enterprise-specific filtering.
6. **IQueryBuilderFlags**: Extends IQueryBuilderEnterprise to add flag-related functionality.
7. **IQueryBuilderValues**: Extends IQueryBuilderFlags to add value-related functionality.
8. **IQueryBuilderRelationships**: Extends multiple interfaces to provide comprehensive query building capabilities.
9. **IQueryBuilderClassifications**: Extends multiple interfaces to provide classification-related functionality.

### Service Interface Hierarchy

The service interfaces in ActivityMaster follow a consistent pattern with generic type parameters:

```
IService<J extends IService<J>>
└── IEnterpriseService<J extends IEnterpriseService<J>>
└── IClassificationService<J extends IClassificationService<J>>
└── ISecurityTokenService<J extends ISecurityTokenService<J>>
└── ISystemsService<J extends ISystemsService<J>>
└── Domain-specific services (IAddressService, IEventService, etc.)
```

Each service interface defines methods for creating, retrieving, updating, and managing entities in its domain. All methods return reactive types (Uni<T> or Multi<T>) to support non-blocking operations.

### IManage Interface Hierarchy

The IManage interfaces are part of the capability system that manages domain joins between entities:

```
IManageSomething<J extends IWarehouseBaseTable<J, ?, ? extends Serializable>>
└── IManageClassifications<J extends IWarehouseBaseTable<J, ?, ? extends Serializable>>
└── IManageAddresses<J extends IWarehouseBaseTable<J, ?, ? extends Serializable>>
└── IManageEvents<J extends IWarehouseBaseTable<J, ?, ? extends Serializable>>
└── Other domain-specific IManage interfaces
```

These interfaces provide methods for managing relationships between entities, such as adding, finding, and counting related entities.

## Service Usage Patterns

### Accessing Services

Services in ActivityMaster are accessed through Guice bound interfaces. Always use the IGuiceContext to get service instances:

```java
// CORRECT: Using IGuiceContext to get a service with proper generic type parameter
IEnterpriseService<?> enterpriseService = com.guicedee.client.IGuiceContext.get(IEnterpriseService.class);

// INCORRECT: Missing generic type parameter
IEnterpriseService enterpriseService = com.guicedee.client.IGuiceContext.get(IEnterpriseService.class);
```

### Service Method Invocation

Service methods return reactive types (Uni<T> or Multi<T>) and should be invoked using reactive programming patterns:

```java
// CORRECT: Using reactive programming patterns
enterpriseService.getEnterprise(name)
    .chain(enterprise -> {
        // Process the enterprise
        return doSomethingElse(enterprise);
    })
    .subscribe().with(
        result -> {
            // Handle success
        },
        error -> {
            // Handle error
        }
    );

// INCORRECT: Blocking the event loop
Enterprise enterprise = enterpriseService.getEnterprise(name).await().indefinitely();
```

### Service Composition

When composing multiple service calls, use reactive composition to avoid callback hell:

```java
// CORRECT: Using reactive composition
enterpriseService.getEnterprise(name)
    .chain(enterprise -> systemsService.getSystem(systemName, enterprise))
    .chain(system -> classificationService.find(classificationName, system))
    .subscribe().with(
        classification -> {
            // Process the classification
        },
        error -> {
            // Handle error
        }
    );

// INCORRECT: Callback hell
enterpriseService.getEnterprise(name)
    .subscribe().with(enterprise -> {
        systemsService.getSystem(systemName, enterprise)
            .subscribe().with(system -> {
                classificationService.find(classificationName, system)
                    .subscribe().with(classification -> {
                        // Process the classification
                    });
            });
    });
```

### Error Handling

Always handle errors in reactive chains:

```java
// CORRECT: Proper error handling
enterpriseService.getEnterprise(name)
    .onFailure().invoke(error -> log.error("Error getting enterprise", error))
    .onFailure().recoverWithItem(defaultEnterprise)
    .subscribe().with(
        enterprise -> {
            // Process the enterprise
        },
        error -> {
            // Handle other types of errors
        }
    );

// INCORRECT: Missing error handling
enterpriseService.getEnterprise(name)
    .subscribe().with(
        enterprise -> {
            // Process the enterprise
        }
    );
```

## Type Safety Requirements

### Generic Type Parameters

When referencing a service interface that has generic type parameters, always include the generic type parameters in the reference. If you're not sure about the specific type, use the wildcard `?` as a placeholder.

```java
// CORRECT: Using wildcard generic type parameter
ISystemsService<?> systemsService = com.guicedee.client.IGuiceContext.get(ISystemsService.class);

// INCORRECT: Missing generic type parameter
ISystemsService systemsService = com.guicedee.client.IGuiceContext.get(ISystemsService.class);
```

### Common Service Interfaces That Require Generic Type Parameters

The following service interfaces require generic type parameters:

#### Core Service Interfaces
- `ISystemsService<T>`
- `IEnterpriseService<T>`
- `IClassificationService<T>`
- `IInvolvedPartyService<T>`
- `IPasswordsService<T>`
- `IActiveFlagService<T>`
- `IAddressService<T>`
- `IArrangementsService<T>`
- `IClassificationDataConceptService<T>`
- `IEventService<T>`
- `IProductService<T>`
- `IResourceItemService<T>`
- `IRulesService<T>`
- `ISecurityTokenService<T>`
- `ICerialMasterService<T>`
- `IImageService<T>`
- `IProfileService<T>`
- `IRolesService<T>`
- `ISessionLoginService<T>`
- `IUserSessionService<T>`

#### IManage Interfaces
- `IManageAddresses<T>`
- `IManageArrangementTypes<T>`
- `IManageArrangements<T>`
- `IManageClassifications<T>`
- `IManageEventTypes<T>`
- `IManageEvents<T>`
- `IManageGeographies<T>`
- `IManageInvolvedParties<T>`
- `IManagePartyIdentificationTypes<T>`
- `IManagePartyNameTypes<T>`
- `IManagePartyTypes<T>`
- `IManageProductTypes<T>`
- `IManageProducts<T>`
- `IManageResourceItemTypes<T>`
- `IManageResourceItems<T>`
- `IManageRuleTypes<T>`
- `IManageRules<T>`

### Why Type Safety Is Important

Using proper generic type parameters ensures:

1. **Type Safety**: The compiler can verify that you're using the service correctly.
2. **Method Resolution**: Methods like `findClassifications` can be properly resolved.
3. **Lambda Expression Type Inference**: Lambda expressions like `emitter::fail` can properly infer types.
4. **Reactive Chain Integrity**: Ensures that the types flowing through your reactive chains are consistent and properly typed.

### Examples

#### Example 1: Referencing a Service in Reactive Code

```java
ISystemsService<?> systemsService = com.guicedee.client.IGuiceContext.get(ISystemsService.class);
return systemsService.getActivityMaster(enterprise)
    .chain(system -> {
        // Now we can use system in a reactive chain
        return doSomethingWithSystem(system);
    });
```

#### Example 2: Using a Service in a Reactive Method Chain

```java
public Uni<Boolean> doesEnterpriseExist(String name) {
    IEnterpriseService<?> enterpriseService = IGuiceContext.get(IEnterpriseService.class);
    return enterpriseService.findEnterpriseByName(name)
        .onItem().transform(enterprise -> enterprise != null)
        .onFailure().recoverWithItem(false);
}
```

## Transaction Management

### Transaction Patterns

The ActivityMaster system uses ReactiveTransactionUtil for transaction management. This utility provides methods for executing code within a transaction boundary:

```java
// CORRECT: Using ReactiveTransactionUtil for transaction management
return ReactiveTransactionUtil.withTransaction(session -> {
    // Code that needs to be executed within a transaction
    return entity.persist();
});

// INCORRECT: Not using transaction management
return entity.persist();
```

### Transaction Usage Per Layer

#### Service Layer

Services should use ReactiveTransactionUtil.withTransaction() for operations that modify data:

```java
public Uni<IResourceItem<?, ?>> create(String type, String value, ISystems<?, ?> system, UUID... token) {
    return ReactiveTransactionUtil.withTransaction(session -> {
        // Implementation that creates a resource item
        return resourceItem.persist();
    });
}
```

#### Entity Layer

Entities should not manage transactions directly. Instead, they should expose methods that can be called within a transaction:

```java
// CORRECT: Entity method that can be called within a transaction
public Uni<J> persist() {
    return builder().persist((J) this).map(_ -> (J) this);
}

// INCORRECT: Entity method that manages transactions
public Uni<J> persistWithTransaction() {
    return ReactiveTransactionUtil.withTransaction(session -> {
        return builder().persist((J) this).map(_ -> (J) this);
    });
}
```

#### Query Builder Layer

Query builders should not manage transactions directly. Instead, they should expose methods that can be called within a transaction:

```java
// CORRECT: Query builder method that can be called within a transaction
public Uni<E> get() {
    Mutiny.SelectionQuery<E> query = getQuery();
    return query.getSingleResultOrNull()
        .onItem().ifNull().failWith(() -> new NoSuchElementException("Entity not found"));
}

// INCORRECT: Query builder method that manages transactions
public Uni<E> getWithTransaction() {
    return ReactiveTransactionUtil.withTransaction(session -> {
        Mutiny.SelectionQuery<E> query = getQuery();
        return query.getSingleResultOrNull()
            .onItem().ifNull().failWith(() -> new NoSuchElementException("Entity not found"));
    });
}
```

### Transaction Best Practices

1. **Use ReactiveTransactionUtil**: Always use ReactiveTransactionUtil.withTransaction() for operations that modify data.
2. **Transaction Boundaries**: Keep transaction boundaries as narrow as possible to minimize lock contention.
3. **Error Handling**: Always handle errors properly in transactions to ensure proper rollback.
4. **Avoid Nested Transactions**: Avoid nesting transactions as it can lead to unexpected behavior.
5. **Reactive Composition**: Use reactive composition to chain operations within a transaction.

## Sequence Diagrams for Key Workflows

### Creating a Resource Item

```
┌─────────┐          ┌───────────────────┐          ┌─────────────────┐          ┌──────────────┐
│ Client  │          │ResourceItemService│          │ ResourceItem    │          │ Transaction  │
└────┬────┘          └─────────┬─────────┘          └────────┬────────┘          └──────┬───────┘
     │                         │                             │                          │
     │ create(type, value, system)                           │                          │
     │────────────────────────>│                             │                          │
     │                         │                             │                          │
     │                         │ withTransaction(session)    │                          │
     │                         │─────────────────────────────────────────────────────────>│
     │                         │                             │                          │
     │                         │ findResourceItemType(type)  │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ create new ResourceItem     │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ set properties              │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ persist()                   │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ createDefaultSecurity()     │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ addResourceItemType()       │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ transaction complete        │                          │
     │                         │<─────────────────────────────────────────────────────────│
     │                         │                             │                          │
     │ return ResourceItem     │                             │                          │
     │<────────────────────────│                             │                          │
     │                         │                             │                          │
```

### Finding a Resource Item by UUID

```
┌─────────┐          ┌───────────────────┐          ┌─────────────────┐          ┌──────────────┐
│ Client  │          │ResourceItemService│          │ ResourceItem    │          │ Transaction  │
└────┬────┘          └─────────┬─────────┘          └────────┬────────┘          └──────┬───────┘
     │                         │                             │                          │
     │ findByUUID(uuid)        │                             │                          │
     │────────────────────────>│                             │                          │
     │                         │                             │                          │
     │                         │ withTransaction(session)    │                          │
     │                         │─────────────────────────────────────────────────────────>│
     │                         │                             │                          │
     │                         │ builder().where(id, uuid)   │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ inActiveRange()             │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ inDateRange()               │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ get()                       │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ transaction complete        │                          │
     │                         │<─────────────────────────────────────────────────────────│
     │                         │                             │                          │
     │ return ResourceItem     │                             │                          │
     │<────────────────────────│                             │                          │
     │                         │                             │                          │
```

### Adding a Classification to an Entity

```
┌─────────┐          ┌───────────────────┐          ┌─────────────────┐          ┌──────────────┐
│ Client  │          │ Entity            │          │ EntityXClassification │     │ Transaction  │
└────┬────┘          └─────────┬─────────┘          └────────┬────────┘          └──────┬───────┘
     │                         │                             │                          │
     │ addClassification(classification, value, system)      │                          │
     │────────────────────────>│                             │                          │
     │                         │                             │                          │
     │                         │ withTransaction(session)    │                          │
     │                         │─────────────────────────────────────────────────────────>│
     │                         │                             │                          │
     │                         │ create new relationship     │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ set properties              │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ persist()                   │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ createDefaultSecurity()     │                          │
     │                         │────────────────────────────>│                          │
     │                         │                             │                          │
     │                         │ transaction complete        │                          │
     │                         │<─────────────────────────────────────────────────────────│
     │                         │                             │                          │
     │ return relationship     │                             │                          │
     │<────────────────────────│                             │                          │
     │                         │                             │                          │
```

## Use Cases with Code Examples

### Use Case 1: Creating and Retrieving an Enterprise

```java
// Get the enterprise service
IEnterpriseService<?> enterpriseService = IGuiceContext.get(IEnterpriseService.class);

// Create a new enterprise
enterpriseService.create("MyEnterprise", "My Enterprise Description")
    .chain(enterprise -> {
        // Add a classification to the enterprise
        return enterprise.addClassification("EnterpriseType", "Corporate", system);
    })
    .subscribe().with(
        result -> {
            System.out.println("Enterprise created successfully");
        },
        error -> {
            System.err.println("Error creating enterprise: " + error.getMessage());
        }
    );

// Retrieve an enterprise by name
enterpriseService.getEnterprise("MyEnterprise")
    .subscribe().with(
        enterprise -> {
            System.out.println("Found enterprise: " + enterprise.getName());
        },
        error -> {
            System.err.println("Error retrieving enterprise: " + error.getMessage());
        }
    );
```

### Use Case 2: Creating and Retrieving a Resource Item

```java
// Get the resource item service
IResourceItemService<?> resourceItemService = IGuiceContext.get(IResourceItemService.class);

// Create a new resource item
resourceItemService.create("Document", "MyDocument", system)
    .chain(resourceItem -> {
        // Add a classification to the resource item
        return resourceItem.addClassification("DocumentType", "PDF", system);
    })
    .subscribe().with(
        result -> {
            System.out.println("Resource item created successfully");
        },
        error -> {
            System.err.println("Error creating resource item: " + error.getMessage());
        }
    );

// Retrieve a resource item by UUID
resourceItemService.findByUUID(uuid)
    .subscribe().with(
        resourceItem -> {
            System.out.println("Found resource item: " + resourceItem.getName());
        },
        error -> {
            System.err.println("Error retrieving resource item: " + error.getMessage());
        }
    );
```

### Use Case 3: Finding Entities by Classification

```java
// Get the classification service
IClassificationService<?> classificationService = IGuiceContext.get(IClassificationService.class);

// Find a classification
classificationService.find("DocumentType", system)
    .chain(classification -> {
        // Find all resource items with this classification
        return resourceItemService.findByClassificationAll("Document", "DocumentType", "PDF", system);
    })
    .subscribe().with(
        relationships -> {
            System.out.println("Found " + relationships.size() + " resource items");
            relationships.forEach(relationship -> {
                System.out.println("Resource item: " + relationship.getPrimary().getName());
            });
        },
        error -> {
            System.err.println("Error finding resource items: " + error.getMessage());
        }
    );
```

### Use Case 4: Using CerialMaster to Communicate with Serial Ports

```java
// Get the cerial master service
ICerialMasterService<?> cerialMasterService = IGuiceContext.get(ICerialMasterService.class);

// Create a new serial port connection
cerialMasterService.createSerialPortConnection("COM1", "9600", "8", "1", "None", system)
    .chain(connection -> {
        // Send a message to the serial port
        return cerialMasterService.sendMessage(connection, "Hello, world!");
    })
    .subscribe().with(
        result -> {
            System.out.println("Message sent successfully");
        },
        error -> {
            System.err.println("Error sending message: " + error.getMessage());
        }
    );

// Find all serial port connections
cerialMasterService.findAllSerialPortConnections(system)
    .subscribe().with(
        connections -> {
            System.out.println("Found " + connections.size() + " connections");
            connections.forEach(connection -> {
                System.out.println("Connection: " + connection.getName());
            });
        },
        error -> {
            System.err.println("Error finding connections: " + error.getMessage());
        }
    );
```

## Best Practices

### 1. Type Safety

- Always use generic type parameters with service references.
- If you're not sure about the specific type, use the wildcard `?` as a placeholder.
- Follow the type safety guidelines to ensure proper method resolution and lambda expression type inference.

### 2. Transaction Management

- Always use ReactiveTransactionUtil.withTransaction() for operations that modify data.
- Keep transaction boundaries as narrow as possible to minimize lock contention.
- Always handle errors properly in transactions to ensure proper rollback.
- Avoid nesting transactions as it can lead to unexpected behavior.

### 3. Reactive Programming

- Never block the event loop with synchronous operations.
- Use reactive composition to chain operations.
- Always handle errors in reactive chains.
- Use timeouts with await() if you must use it.

### 4. Query Building

- Always use the builder pattern to create queries.
- Apply appropriate temporal constraints with inDateRange() and inActiveRange().
- Use column selection for performance when only specific columns are needed.
- Use joins for complex queries instead of multiple separate queries.

### 5. Entity Management

- Use the persist() method to save entities.
- Use the builder() method to create queries for entities.
- Use the addClassification() method to add classifications to entities.
- Use the createDefaultSecurity() method to create default security for entities.

## Common Issues and Solutions

### 1. Missing Generic Type Parameters

**Issue**: Methods like `enterprise.findClassifications` may not be resolved if the `system` parameter is not properly typed.

**Solution**: Always use generic type parameters with service references:

```java
// CORRECT
ISystemsService<?> systemsService = IGuiceContext.get(ISystemsService.class);

// INCORRECT
ISystemsService systemsService = IGuiceContext.get(ISystemsService.class);
```

### 2. Blocking the Event Loop

**Issue**: Blocking the event loop with synchronous operations can cause performance issues.

**Solution**: Use reactive programming patterns instead of blocking operations:

```java
// CORRECT
enterpriseService.getEnterprise(name)
    .subscribe().with(enterprise -> {
        // Process the enterprise
    });

// INCORRECT
Enterprise enterprise = enterpriseService.getEnterprise(name).await().indefinitely();
```

### 3. Missing Transaction Management

**Issue**: Operations that modify data may fail without proper transaction management.

**Solution**: Use ReactiveTransactionUtil.withTransaction() for operations that modify data:

```java
// CORRECT
return ReactiveTransactionUtil.withTransaction(session -> {
    return entity.persist();
});

// INCORRECT
return entity.persist();
```

### 4. Callback Hell

**Issue**: Nested callbacks in reactive programming can lead to "callback hell" and make code hard to read.

**Solution**: Use reactive composition with chain, map, and flatMap:

```java
// CORRECT
enterpriseService.getEnterprise(name)
    .chain(enterprise -> systemsService.getSystem(systemName, enterprise))
    .chain(system -> classificationService.find(classificationName, system))
    .subscribe().with(
        classification -> {
            // Process the classification
        },
        error -> {
            // Handle error
        }
    );

// INCORRECT
enterpriseService.getEnterprise(name)
    .subscribe().with(enterprise -> {
        systemsService.getSystem(systemName, enterprise)
            .subscribe().with(system -> {
                classificationService.find(classificationName, system)
                    .subscribe().with(classification -> {
                        // Process the classification
                    });
            });
    });
```

### 5. Missing Error Handling

**Issue**: Missing error handling in reactive chains can lead to unhandled exceptions.

**Solution**: Always handle errors in reactive chains:

```java
// CORRECT
enterpriseService.getEnterprise(name)
    .onFailure().invoke(error -> log.error("Error getting enterprise", error))
    .onFailure().recoverWithItem(defaultEnterprise)
    .subscribe().with(
        enterprise -> {
            // Process the enterprise
        },
        error -> {
            // Handle other types of errors
        }
    );

// INCORRECT
enterpriseService.getEnterprise(name)
    .subscribe().with(
        enterprise -> {
            // Process the enterprise
        }
    );
```

## Domain Services and Capabilities

This section provides a detailed breakdown of the available domains in the ActivityMaster system, their services, capabilities, and relationships.

### Core Domains

#### Enterprise Domain

The Enterprise domain is foundational to the ActivityMaster system, representing business entities and their organizational structure.

**Key Service**: `IEnterpriseService<J extends IEnterpriseService<J>>`

**Capabilities**:
- Enterprise lifecycle management (creation, retrieval)
- Enterprise existence verification
- System updates management
- Post-startup operations

**Key Methods**:
- `getEnterprise(String name)`: Retrieves an enterprise by name
- `getEnterprise(UUID uuid)`: Retrieves an enterprise by UUID
- `doesEnterpriseExist(String name)`: Checks if an enterprise exists
- `startNewEnterprise(String enterpriseName, String adminUserName, String adminPassword)`: Creates a new enterprise with an admin user
- `loadUpdates(IEnterprise<?,?> enterprise)`: Loads updates for an enterprise

**Relationships**:
- Used by most other services as enterprises are the top-level organizational units
- Works closely with SystemsService to manage system installations

#### Classification Domain

The Classification domain provides categorization capabilities across the entire system, allowing entities to be classified and organized.

**Key Services**: 
- `IClassificationService<J extends IClassificationService<J>>`
- `IClassificationDataConceptService<J extends IClassificationDataConceptService<J>>`

**Capabilities**:
- Classification creation and management
- Classification hierarchy management
- Classification data concept management

**Key Methods**:
- `create(String name, String description, EnterpriseClassificationDataConcepts concept, ISystems<?,?> system, Integer sequenceOrder, String parentName, UUID... identityToken)`: Creates a classification
- `find(String name, ISystems<?,?> system, UUID... identityToken)`: Finds a classification by name
- `getHierarchyType(ISystems<?,?> system, UUID... identityToken)`: Gets the hierarchy type classification
- `getNoClassification(ISystems<?,?> system, UUID... identityToken)`: Gets the "no classification" classification

**Relationships**:
- Used by virtually all other domains for categorization
- Provides the foundation for the classification-based relationships in the system

#### Security Domain

The Security domain manages access control and permissions throughout the system.

**Key Service**: `ISecurityTokenService<J extends ISecurityTokenService<J>>`

**Capabilities**:
- Security token creation and management
- Permission granting and checking
- Predefined security group management

**Key Methods**:
- `create(String classificationValue, String name, String description, ISystems<?,?> system)`: Creates a security token
- `grantAccessToToken(ISecurityToken<?,?> fromToken, ISecurityToken<?,?> toToken, boolean create, boolean update, boolean delete, boolean read, ISystems<?,?> system)`: Grants permissions from one token to another
- `getEveryoneGroup(ISystems<?,?> system, UUID... identityToken)`: Gets the "Everyone" group token
- `getAdministratorsFolder(ISystems<?,?> system, UUID... identityToken)`: Gets the "Administrators" folder token

**Relationships**:
- Used by all other domains for access control
- Works closely with the Enterprise domain for organizational security

#### Systems Domain

The Systems domain manages system registrations and configurations within the ActivityMaster framework.

**Key Service**: `ISystemsService<J extends ISystemsService<J>>`

**Capabilities**:
- System registration and management
- System existence verification
- Security identity token management

**Key Methods**:
- `getActivityMaster(IEnterprise<?, ?> requestingSystem, UUID... identityToken)`: Gets the Activity Master system
- `findSystem(IEnterprise<?, ?> enterprise, String systemName, UUID... identityToken)`: Finds a system by enterprise and name
- `create(IEnterprise<?, ?> enterprise, String systemName, String systemDesc, UUID... identityToken)`: Creates a new system
- `getSecurityIdentityToken(ISystems<?, ?> system, UUID... identityToken)`: Gets the security identity token for a system

**Relationships**:
- Used by all other domains as systems are the operational contexts
- Works closely with the Enterprise domain as systems belong to enterprises

### Domain-Specific Services

#### ResourceItem Domain

The ResourceItem domain manages generic resources that can store various types of data.

**Key Service**: `IResourceItemService<J extends IResourceItemService<J>>`

**Capabilities**:
- Resource item creation and management
- Resource item type management
- Binary data storage and retrieval

**Key Methods**:
- `create(String identityResourceType, String resourceItemDataValue, ISystems<?,?> system, UUID... identityToken)`: Creates a resource item
- `createType(String value, String description, ISystems<?,?> system, UUID... identityToken)`: Creates a resource item type
- `findByUUID(UUID uuid)`: Finds a resource item by UUID
- `getDataForResourceItemValue(IRelationshipValue<IResourceItem<?,?>, IResourceData<?,?,?>, ?> data)`: Gets data for a resource item value

**Relationships**:
- Used by many other domains for storing binary data
- Used by CerialMaster for storing connection details

#### Address Domain

The Address domain manages various types of addresses, including physical, electronic, and communication addresses.

**Key Service**: `IAddressService<J extends IAddressService<J>>`

**Capabilities**:
- Address creation and management
- Specialized address type handling (IP, web, email, phone, street, postal)

**Key Methods**:
- `create(String addressClassification, ISystems<?,?> system, String value, UUID... identifyingToken)`: Creates an address
- `addOrFindIPAddress(String ipAddress, ISystems<?,?> system, UUID... identityToken)`: Adds or finds an IP address
- `addOrFindEmailContact(String emailAddressString, ISystems<?, ?> system, UUID... identityToken)`: Adds or finds an email contact
- `addOrFindStreetAddress(String number, String street, String streetType, ISystems<?,?> system, UUID... identityToken)`: Adds or finds a street address

**Relationships**:
- Used by InvolvedParty domain for contact information
- Used by Geography domain for location information

#### Event Domain

The Event domain manages activities and occurrences in the system.

**Key Service**: `IEventService<J extends IEventService<J>>`

**Capabilities**:
- Event creation and management
- Event type management

**Key Methods**:
- `createEvent(String eventType, ISystems<?,?> system, UUID... identityToken)`: Creates an event
- `createEventType(String eventType, ISystems<?,?> system, UUID... identityToken)`: Creates an event type
- `findEventType(String eventType, ISystems<?,?> system, UUID... identityToken)`: Finds an event type by name

**Relationships**:
- Used by many domains for tracking activities
- Used by CerialMaster for recording messages sent to and received from COM ports

#### Geography Domain

The Geography domain manages geographical data at various levels.

**Key Service**: `IGeographyService<J extends IGeographyService<J>>`

**Capabilities**:
- Geographical entity management (planets, continents, countries, etc.)
- Geographical data loading and retrieval

**Key Methods**:
- `createPlanet(String value, String originalUniqueID, ISystems<?,?> system, UUID... identityToken)`: Creates a planet entity
- `findCountry(GeographyCountry country, ISystems<?,?> system, UUID... identityToken)`: Finds a country
- `loadCountryInfo(ISystems<?,?> system)`: Loads country information
- `findPostalCodeSuburb(String code, String description, ISystems<?,?> system, UUID... identityToken)`: Finds a postal code by code and description

**Relationships**:
- Used by Address domain for location information
- Provides geographical context for many other domains

#### InvolvedParty Domain

The InvolvedParty domain manages individuals and organizations in the system.

**Key Service**: `IInvolvedPartyService<J extends IInvolvedPartyService<J>>`

**Capabilities**:
- Involved party creation and management
- Party type, name type, and identification type management

**Key Methods**:
- `create(ISystems<?,?> system, Pair<String, String> idTypes, boolean isOrganic, UUID... identityToken)`: Creates an involved party
- `createType(ISystems<?,?> system, String name, String description, UUID... identityToken)`: Creates an involved party type
- `createNameType(String name, String description, ISystems<?,?> system, UUID... identityToken)`: Creates an involved party name type
- `createIdentificationType(ISystems<?,?> system, String name, String description, UUID... identityToken)`: Creates an involved party identification type

**Relationships**:
- Used by SessionMaster for user representation
- Used by many domains for representing parties involved in various activities

#### Product Domain

The Product domain manages products and services in the system.

**Key Service**: `IProductService<J extends IProductService<J>>`

**Capabilities**:
- Product creation and management
- Product type management

**Key Methods**:
- `createProduct(String productType, String name, String description, String code, ISystems<?,?> system, UUID... identityToken)`: Creates a product
- `createProductType(String productType, String description, ISystems<?,?> system, UUID... identityToken)`: Creates a product type
- `findProduct(String name, ISystems<?,?> system, UUID... identityToken)`: Finds a product by name
- `findByProductTypes(String type, ISystems<?,?> system, UUID... identityToken)`: Finds products by product type name

**Relationships**:
- Used by Arrangement domain for product-related arrangements
- Used by InvolvedParty domain for product associations

#### Rules Domain

The Rules domain manages business rules and constraints in the system.

**Key Service**: `IRulesService<J extends IRulesService<J>>`

**Capabilities**:
- Rule creation and management
- Rule type management

**Key Methods**:
- `createRules(String rulesType, String name, String description, ISystems<?,?> system, UUID... identityToken)`: Creates a rule
- `createRulesType(String rulesType, String description, ISystems<?,?> system, UUID... identityToken)`: Creates a rule type
- `findRules(String name, IEnterprise<?,?> enterprise, UUID... identityToken)`: Finds a rule by name and enterprise
- `findRulesByProduct(IProduct<?,?> product, String classificationName, String value, ISystems<?,?> system, UUID... identityToken)`: Finds rules by product

**Relationships**:
- Used by many domains for defining business rules
- Used by Arrangement domain for rule-based arrangements

#### Arrangement Domain

The Arrangement domain manages agreements or contracts between involved parties.

**Key Service**: `IArrangementsService<J extends IArrangementsService<J>>`

**Capabilities**:
- Arrangement creation and management
- Arrangement type management
- Arrangement relationship management

**Key Methods**:
- `create(String type, String arrangementTypeClassification, String arrangementTypeValue, ISystems<?,?> system, UUID... identityToken)`: Creates an arrangement
- `createArrangementType(String type, ISystems<?,?> system, UUID... identityToken)`: Creates an arrangement type
- `findArrangementsByInvolvedParty(IInvolvedParty<?,?> involvedParty, String classificationName, String value, ISystems<?,?> system, UUID... identityToken)`: Finds arrangements by involved party
- `completeArrangement(IArrangement<?,?> arrangement, ISystems<?,?> system, UUID... identityToken)`: Completes an arrangement

**Relationships**:
- Uses InvolvedParty domain for parties in arrangements
- Uses Product domain for product-related arrangements
- Uses Rules domain for rule-based arrangements

### Specialized Services

#### CerialMaster Domain

The CerialMaster domain manages communication with serial ports.

**Key Service**: `ICerialMasterService<J extends ICerialMasterService<J>>`

**Capabilities**:
- Serial port connection management
- Serial port listing and discovery

**Key Methods**:
- `addOrUpdateConnection(ComPortConnection<?> comPort, ISystems<?,?> system, UUID... identityToken)`: Adds or updates a serial port connection
- `getComPortConnection(Integer comPort)`: Gets a serial port connection by port number
- `listComPorts()`: Lists all COM ports
- `listAvailableComPorts()`: Lists available COM ports

**Relationships**:
- Uses ResourceItem domain to store connection details
- Uses Event domain to record messages sent to and received from COM ports

#### SessionMaster Domain

The SessionMaster domain manages user sessions, authentication, and registration.

**Key Services**:
- `ISessionLoginService<J extends ISessionLoginService<J>>`
- `IUserSessionService<J extends IUserSessionService<J>>`

**Capabilities**:
- User login and logout
- User registration and verification
- Session management

**Key Methods**:
- `loginUser(UserLoginDTO<?> profileServiceDTO, ISystems<?, ?> system, UUID... identityToken)`: Logs in a user
- `logoutUser(ProfileServiceDTO<?> profileServiceDTO, ISystems<?, ?> system, UUID... identityToken)`: Logs out a user
- `registerVisitor(UserRegistrationDTO<?> userRegistrationDTO, ISystems<?, ?> system, UUID... identityToken)`: Registers a visitor
- `getSession(IInvolvedParty<?, ?> involvedParty, ISystems<?, ?> system, UUID... identityToken)`: Gets a session for an involved party

**Relationships**:
- Uses InvolvedParty domain for user representation
- Uses ProfileMaster domain for user profiles

#### ProfileMaster Domain

The ProfileMaster domain manages user profiles.

**Key Service**: `IProfileService<J extends IProfileService<J>>`

**Capabilities**:
- User listing
- Cache management

**Key Methods**:
- `listUsers(String... roles)`: Lists users with specific roles
- `allUsers()`: Lists all users
- `clearCache()`: Clears the cache

**Relationships**:
- Used by SessionMaster domain for user profiles
- Uses InvolvedParty domain for user representation

### Common Usage Patterns

#### Service Access Pattern

Always access services through Guice bound interfaces using IGuiceContext:

```java
// CORRECT: Using IGuiceContext to get a service with proper generic type parameter
IEnterpriseService<?> enterpriseService = com.guicedee.client.IGuiceContext.get(IEnterpriseService.class);
```

#### Reactive Method Invocation Pattern

Invoke service methods using reactive programming patterns:

```java
// CORRECT: Using reactive programming patterns
enterpriseService.getEnterprise(name)
    .chain(enterprise -> {
        // Process the enterprise
        return doSomethingElse(enterprise);
    })
    .subscribe().with(
        result -> {
            // Handle success
        },
        error -> {
            // Handle error
        }
    );
```

#### Transaction Management Pattern

Use ReactiveTransactionUtil for transaction management:

```java
// CORRECT: Using ReactiveTransactionUtil for transaction management
return ReactiveTransactionUtil.withTransaction(session -> {
    // Code that needs to be executed within a transaction
    return entity.persist();
});
```

#### Entity Relationship Management Pattern

Use IManage interfaces for managing relationships between entities:

```java
// Example: Adding a classification to an entity
entity.addClassification("ClassificationType", "ClassificationValue", system, identityToken)
    .subscribe().with(
        result -> {
            // Handle success
        },
        error -> {
            // Handle error
        }
    );
```

## References

1. [TypeSafetyGuidelines.md](TypeSafetyGuidelines.md): Guidelines for ensuring type safety when referencing services.
2. [COMPREHENSIVE_MIGRATION_GUIDE.md](COMPREHENSIVE_MIGRATION_GUIDE.md): Guide for migrating to reactive programming.
3. [IManageInterfacesReactiveMigrationRules.md](IManageInterfacesReactiveMigrationRules.md): Rules for migrating IManage interfaces to reactive programming.
4. [interface_hierarchies.md](documentation/interface_hierarchies.md): Documentation of interface hierarchies.
5. [warehouse_table_hierarchy.md](documentation/warehouse_table_hierarchy.md): Documentation of warehouse table hierarchy.
6. [querybuilder_scd_hierarchy.md](documentation/querybuilder_scd_hierarchy.md): Documentation of query builder hierarchy.
7. [pact-rules.md](pact-rules.md): Rules and guidelines for implementing service interfaces.