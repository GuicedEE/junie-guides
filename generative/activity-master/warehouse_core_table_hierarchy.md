# WarehouseCoreTable Hierarchy Documentation

## Interface and Class Hierarchy

```
Interface Hierarchy:
IBaseEntity (from com.entityassist.services.entities)
└── ISCDEntity (from com.guicedee.activitymaster.fsdm.client.services.builders)
    └── IWarehouseSCDTable (from com.guicedee.activitymaster.fsdm.client.services.builders.warehouse.base)
        ├── IWarehouseSecurityTable
        └── IWarehouseCoreTable
            └── IWarehouseTable
                └── ... (WarehouseTable interface hierarchy)

Implementation Hierarchy:
BaseEntity (from com.entityassist)
└── SCDEntity (from com.guicedee.activitymaster.fsdm.db.abstraction.builders)
    └── WarehouseSCDTable (from com.guicedee.activitymaster.fsdm.db.abstraction)
        └── WarehouseCoreTable
            └── WarehouseOriginalSourceSystemEnterpriseTable
                └── WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable
                    ├── WarehouseRelationshipOriginalSourceSystemEnterpriseTable
                    │   └── ... (WarehouseRelationshipOriginalSourceSystemEnterpriseTable hierarchy)
                    └── WarehouseSecurityOriginalSourceSystemEnterpriseTable
```

> **Note**: The implementation hierarchy has been updated to reflect the current structure. WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable now extends WarehouseOriginalSourceSystemEnterpriseTable and sits between WarehouseOriginalSourceSystemEnterpriseTable and both WarehouseRelationshipOriginalSourceSystemEnterpriseTable and WarehouseSecurityOriginalSourceSystemEnterpriseTable. The previously documented CoreEntity class has been merged into WarehouseCoreTable as they contained duplicate functionality.

## Description

The WarehouseCoreTable hierarchy is designed to provide entity functionality with active flag management and security features:

- **IBaseEntity**: The base interface from the external package com.entityassist.services.entities, providing basic entity operations
- **ISCDEntity**: Extends IBaseEntity to add Slowly Changing Dimension (SCD) functionality, including effective dates and warehouse timestamps
- **IWarehouseSCDTable**: Extends ISCDEntity to provide a base interface for warehouse entities
- **IWarehouseCoreTable**: Extends IWarehouseSCDTable to add security-related functionality

## Implementation Classes

- **BaseEntity**: The base implementation class from the external package com.entityassist
- **SCDEntity**: Extends BaseEntity and implements ISCDEntity, adding fields and methods for SCD functionality
- **WarehouseSCDTable**: Extends SCDEntity and implements IWarehouseSCDTable, providing basic warehouse functionality
- **WarehouseCoreTable**: Extends WarehouseSCDTable and implements IWarehouseCoreTable, adding active flag management and security-related functionality

## IWarehouseCoreTable Interface Details

### Purpose and Responsibilities

The IWarehouseCoreTable interface is responsible for adding security-related functionality to warehouse entities. It extends the IWarehouseSCDTable interface, which provides basic warehouse functionality.

### Methods

- **createDefaultSecurity(ISystems, UUID...)**: Creates default security access for different types of users
- **configureSecurityEntity(S)**: Configures a security entity for this entity

### Generic Type Parameters

- **J**: The implementing class type (self-referential for method chaining)
- **Q**: The query builder type, extending IQueryBuilderWarehouseCoreTable
- **I**: The ID type, extending UUID
- **S**: The security table type, extending WarehouseSecurityOriginalSourceSystemEnterpriseTable

## IWarehouseTable Interface Details

### Purpose and Responsibilities

The IWarehouseTable interface extends IWarehouseCoreTable, UUID, and IContainsRowRecordInformation. It represents a standard warehouse table with security capabilities, SCD functionality, and original source system tracking. It serves as the base for more specialized interfaces like IWarehouseRelationshipTable and IResourceData.

### Methods

IWarehouseTable inherits methods from its parent interfaces:
- From IWarehouseCoreTable: createDefaultSecurity method for security-related functionality
- From IWarehouseBaseTable: expire, isFake, getTableName methods for entity lifecycle management
- From ISCDEntity: Methods for managing effective dates and warehouse timestamps
- From IContainsRowRecordInformation: getOriginalSourceSystemUniqueID, setOriginalSourceSystemUniqueID, getOriginalSourceSystemID, setOriginalSourceSystemID methods for tracking original source system information

### Generic Type Parameters

- **J**: The implementing class type (self-referential for method chaining)
- **Q**: The query builder type, extending IQueryBuilderWarehouseCoreTable
- **I**: The ID type, extending UUID
- **S**: The security table type, extending IWarehouseSecurityTable

## WarehouseCoreTable Implementation Details

### Purpose and Responsibilities

The WarehouseCoreTable class extends WarehouseSCDTable and implements IWarehouseCoreTable. It adds active flag management and security-related functionality, as well as methods for finding and deleting entities.

### Fields

- **referenceId**: A transient identifier for the entity
- **activeFlag**: The active flag for the entity
- **dateTimeOffsetFormatter**: A formatter for date time offset

### Methods

- **getActiveFlag()**: Returns the active flag for the entity
- **setActiveFlag(ActiveFlag)**: Sets the active flag for the entity and returns the entity for method chaining
- **find(I)**: Finds the entity with the given ID
- **findAll()**: Finds all entities of this type
- **delete()**: Deletes the entity
- **getReferenceId()**: Gets the reference ID
- **setReferenceId(String)**: Sets the reference ID
- **getDateTimeOffsetFormatter()**: Gets the formatter for date time offset
- **configureSecurityEntity(S)**: Abstract method to configure a security entity
- **createDefaultSecurity(ISystems, UUID...)**: Creates default security access for different types of users
- Various methods for creating default security access for different types of users

## Relationship with Other Parts of the System

### Relationship with IQueryBuilder Hierarchy

The IWarehouseCoreTable interface is related to the IQueryBuilder hierarchy through its generic type parameter Q, which extends IQueryBuilderWarehouseCoreTable. This creates a bidirectional relationship between entities and query builders:

- IWarehouseCoreTable entities work with IQueryBuilderWarehouseCoreTable query builders
- IQueryBuilderWarehouseCoreTable query builders work with IWarehouseCoreTable entities

The QueryBuilderWarehouseCoreTable class implements IQueryBuilderWarehouseCoreTable and provides methods for working with WarehouseCoreTable objects, including methods for managing active flags, deleting entities, archiving entities, and updating entities with new statuses.

### Relationship with WarehouseOriginalSourceSystemEnterpriseTable Hierarchy

The WarehouseCoreTable class is part of the WarehouseOriginalSourceSystemEnterpriseTable hierarchy:

- WarehouseSCDTable extends SCDEntity and implements IWarehouseSCDTable
- WarehouseCoreTable extends WarehouseSCDTable and implements IWarehouseCoreTable
- WarehouseOriginalSourceSystemEnterpriseTable extends WarehouseCoreTable

This hierarchy provides a consistent approach to entity management, with each level adding specific functionality.

## Related Documentation

- [Interface Hierarchies](interface_hierarchies.md)
- [WarehouseOriginalSourceSystemEnterpriseTable Class Hierarchy](warehouse_table_hierarchy.md)
- [QueryBuilderSCD Class Hierarchy](querybuilder_scd_hierarchy.md)

## Potential Issues

1. **Complex Generic Type Parameters**: The extensive use of generic type parameters with self-referential bounds (e.g., `J extends IWarehouseCoreTable<J, Q, I, S>`) adds complexity and can make the code harder to understand.

2. **Tight Coupling**: The IWarehouseCoreTable interface is tightly coupled with the IQueryBuilderWarehouseCoreTable interface through its generic type parameter, which can make it difficult to change one without affecting the other.

3. **Inheritance vs. Composition**: The system relies heavily on inheritance, which can lead to the fragile base class problem. Changes to base classes can unexpectedly affect subclasses, making the system harder to maintain. Consider using composition instead of inheritance for some functionality.

4. **Security Concerns**: Security-related functionality is spread across multiple classes, which can make it difficult to ensure that security is consistently applied. Consider centralizing security logic or providing better documentation on how security should be implemented.

5. **Reactive Implementation**: The system uses Mutiny for reactive programming, but not all methods return Uni types. Consider making the API more consistent by ensuring all methods that could benefit from reactive programming return Uni types.

6. **Error Handling**: Some methods don't have explicit error handling, which could lead to unexpected behavior in edge cases. Consider adding more robust error handling and documenting expected behavior in error scenarios.

7. **Documentation Gaps**: Some methods lack comprehensive documentation, which can make it difficult for developers to understand their purpose and usage. Consider adding more detailed documentation, especially for complex methods.