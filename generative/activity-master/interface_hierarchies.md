# Interface Hierarchies Documentation

## IQueryBuilder Interface Hierarchy

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

Note: The IQueryBuilderWarehouseBaseTable interface is now deprecated and is a marker interface that extends IQueryBuilderSCD. All functionality has been moved to IQueryBuilderSCD.

### Description

The IQueryBuilder hierarchy is designed to provide query building capabilities for different types of entities:

- **IQueryBuilder**: The base interface (from external package com.entityassist.services.querybuilders)
- **IQueryBuilderSCD**: Extends IQueryBuilder to add Slowly Changing Dimension (SCD) functionality, including date range queries, utility methods for date/time conversions, active range and visibility range functionality, as well as a default implementation for getEntityManager, specifically for warehouse tables.
- **IQueryBuilderWarehouseBaseTable**: [Deprecated] Extends IQueryBuilderSCD but adds no additional functionality. This interface is kept for backward compatibility and should not be used in new code.
- **IQueryBuilderWarehouseCoreTable**: Extends IQueryBuilderWarehouseBaseTable to work with IWarehouseCoreTable entities.
- **IQueryBuilderSecurity**: Extends IQueryBuilderSCD to add security-related functionality.
- **IQueryBuilderEnterprise**: Extends IQueryBuilderWarehouseCoreTable to add enterprise-specific filtering.
- **IQueryBuilderFlags**: Extends IQueryBuilderEnterprise to add flag-related functionality.
- **IQueryBuilderValues**: Extends IQueryBuilderFlags to add value-related functionality.
- **IQueryBuilderRelationships**: Extends multiple interfaces (IQueryBuilderFlags, IQueryBuilderValues, IQueryBuilderEnterprise, IQueryBuilderSecurity, IQueryBuilderWarehouseBaseTable, IQueryBuilderClassifications) to provide comprehensive query building capabilities.
- **IQueryBuilderClassifications**: Extends multiple interfaces to provide classification-related functionality.

## WarehouseTable Interface Hierarchy

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

## WarehouseOriginalSourceSystemEnterpriseTable Implementation Hierarchy

```
WarehouseSCDTable (from com.guicedee.activitymaster.fsdm.db.abstraction)
└── WarehouseCoreTable
    └── WarehouseOriginalSourceSystemEnterpriseTable
        └── WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable
            ├── WarehouseRelationshipOriginalSourceSystemEnterpriseTable
            │   └── WarehouseClassificationRelationshipOriginalSourceSystemEnterpriseTable
            │       └── WarehouseClassificationRelationshipTypesOriginalSourceSystemEnterpriseTable
            └── WarehouseSecurityOriginalSourceSystemEnterpriseTable
```

Note: The implementation hierarchy differs from the interface hierarchy. WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable is a class that extends WarehouseOriginalSourceSystemEnterpriseTable and doesn't have a corresponding interface. Also, WarehouseSecurityOriginalSourceSystemEnterpriseTable extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which extends WarehouseOriginalSourceSystemEnterpriseTable, but IWarehouseSecurityTable extends IWarehouseSCDTable directly.

### Description

The WarehouseOriginalSourceSystemEnterpriseTable hierarchy is designed to represent entities in the warehouse:

- **ISCDEntity**: The base interface for entities with Slowly Changing Dimension functionality
- **IWarehouseSCDTable**: Extends ISCDEntity to add warehouse-specific functionality like expiring entities and getting table names
- **IWarehouseSecurityTable**: Extends IWarehouseSCDTable to add security-related functionality, including CRUD permissions and security tokens
- **IWarehouseNameAndDescriptionTable**: Extends IWarehouseSCDTable and IContainsNameAndDescription to add name and description fields to warehouse tables
- **IWarehouseCoreTable**: Extends IWarehouseSCDTable to add security-related functionality, including a method for creating default security
- **IWarehouseTable**: Extends IWarehouseCoreTable, UUID, and IContainsRowRecordInformation. This interface represents a standard warehouse table with security capabilities, SCD functionality, and original source system tracking. It inherits:
  - From IWarehouseCoreTable: Security-related functionality (createDefaultSecurity method)
  - From IWarehouseBaseTable: Entity lifecycle management (expire, isFake, getTableName methods)
  - From ISCDEntity: SCD functionality (effective date and warehouse timestamp methods)
  - From IContainsRowRecordInformation: Original source system tracking (getOriginalSourceSystemUniqueID, getOriginalSourceSystemID methods)
  
  IWarehouseTable serves as the base for more specialized interfaces like IWarehouseRelationshipTable and IResourceData. It uses generic type parameters to maintain type safety and enable method chaining.
- **IWarehouseRelationshipTable**: Extends IWarehouseTable to represent relationships between two warehouse tables
- **IWarehouseRelationshipClassificationTable**: Extends IWarehouseRelationshipTable to add classification functionality
- **IWarehouseRelationshipClassificationTypeTable**: Extends IWarehouseRelationshipClassificationTable for classification types
- **IResourceData**: Extends IWarehouseTable to represent resource items with data

## Relationship Between Hierarchies

The two hierarchies are related through their generic type parameters:

- IQueryBuilder interfaces work with entities that implement ISCDEntity
- IQueryBuilderSCD works with entities that implement ISCDEntity and IWarehouseSCDTable
- IQueryBuilderSecurity works with entities that implement ISCDEntity, but has methods that work with IWarehouseSecurityTable

## Related Documentation

- [WarehouseOriginalSourceSystemEnterpriseTable Class Hierarchy](warehouse_table_hierarchy.md)
- [WarehouseCoreTable Hierarchy](warehouse_core_table_hierarchy.md)
- [QueryBuilderSCD Class Hierarchy](querybuilder_scd_hierarchy.md)

## Potential Issues

1. **Complex Inheritance Structure**: Both hierarchies have complex inheritance structures with multiple levels and multiple inheritance paths, which can make them difficult to understand and maintain. The WarehouseOriginalSourceSystemEnterpriseTable hierarchy in particular has many specialized interfaces with overlapping functionality.

2. **Tight Coupling**: The two hierarchies are tightly coupled through their generic type parameters, which can make it difficult to change one hierarchy without affecting the other. For example, IWarehouseTable has a generic parameter for a security table (S extends IWarehouseSecurityTable<S, ?, I>), creating a dependency between these interfaces.

3. **Multiple Inheritance**: Some interfaces like IQueryBuilderRelationships and IWarehouseRelationshipClassificationTable extend multiple interfaces, which can lead to the diamond problem if not carefully managed. This is particularly evident in the WarehouseOriginalSourceSystemEnterpriseTable hierarchy where interfaces often extend multiple other interfaces.

4. **Incomplete Implementation**: Some methods in interfaces like IQueryBuilderSecurity appear to be incomplete (e.g., the canRead method returns without performing any security checks), which could lead to security vulnerabilities.

5. **External Dependencies**: The base IQueryBuilder interface is from an external package, which means changes to that package could affect the entire hierarchy. This creates a dependency on an external codebase that may evolve independently.

6. **Generic Type Parameter Complexity**: The extensive use of generic type parameters with self-referential bounds (e.g., `J extends IQueryBuilderSCD<J, E, I>`) adds complexity and can make the code harder to understand. This is particularly evident in interfaces like IWarehouseRelationshipClassificationTypeTable with six generic type parameters.

7. **Lack of Documentation**: Many interfaces and methods lack comprehensive documentation, which can make it difficult for developers to understand their purpose and usage. While some interfaces like IWarehouseSecurityTable have good documentation, others have minimal or no documentation.

8. **Security Concerns**: Security-related functionality is spread across multiple interfaces and classes (IWarehouseSecurityTable, IWarehouseCoreTable, IQueryBuilderSecurity), which can make it difficult to ensure that security is consistently applied. The relationship between these security-related components is not always clear.

9. **Parallel Hierarchies**: The IQueryBuilder and WarehouseOriginalSourceSystemEnterpriseTable hierarchies are parallel but not perfectly aligned, which can lead to confusion about which query builder should be used with which table type. For example, IWarehouseSecurityTable works with IQueryBuilderSecurity, but IWarehouseTable works with IQueryBuilderSCD.

10. **Interface Bloat**: Some interfaces like IWarehouseRelationshipClassificationTable extend multiple interfaces that provide similar functionality, leading to interface bloat and potential confusion about which methods come from which parent interface.
