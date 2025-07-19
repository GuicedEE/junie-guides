# QueryBuilderSCD Class Hierarchy Documentation

## Class Hierarchy

```
QueryBuilder (from com.entityassist.querybuilder)
└── QueryBuilderSCD (from com.guicedee.activitymaster.fsdm.db.abstraction.builders)
    ├── QueryBuilderWarehouseCoreTable
    │   └── QueryBuilderWarehouseTableTable
    ├── QueryBuilderSecurities
    └── QueryBuilderRelationship
        └── QueryBuilderClassifications
            └── QueryBuilderRelationshipClassification
                └── QueryBuilderRelationshipClassificationTypes
```

Note: The class hierarchy has been updated to reflect the current structure. All functionality previously in QueryBuilderWarehouseSCDTable has been moved to QueryBuilderSCD. QueryBuilderWarehouseTableTable and QueryBuilderSecurities are classes that extend QueryBuilderSCD. The hierarchy for QueryBuilderEnterprise, QueryBuilderFlags, and QueryBuilderValues has been simplified.

## Interface Hierarchy

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

## Description

The QueryBuilderSCD hierarchy is designed to provide query building capabilities for Slowly Changing Dimension (SCD) entities. The hierarchy has been simplified, with a single implementation of QueryBuilderSCD in the codebase:

**QueryBuilderSCD (from com.guicedee.activitymaster.fsdm.db.abstraction.builders)**: Extends QueryBuilder from the com.entityassist.querybuilder package and implements IQueryBuilderSCD. Works with SCDEntity entities and serves as the base class for more specialized query builders.

This implementation provides methods for:

- Querying entities within date ranges (inDateRange, inDateRangeSpecified)
- Updating entities with warehouse timestamps (update)
- Managing entity lifecycle (onCreate, onUpdate, onDeleteUpdate)
- Ordering results (latestFirst)
- Managing warehouse dates and partitions
- Overriding getEntityManager to use Mutiny.Session
- Setting default values for new entities
- Querying entities in active and visible ranges
- Deleting entities with a specific active flag type
- Archiving entities
- Updating entities with a new status

### IQueryBuilderWarehouseBaseTable

This interface is now deprecated. It extends IQueryBuilderSCD but adds no additional functionality. It is kept for backward compatibility and should not be used in new code. All functionality previously in this interface has been moved to IQueryBuilderSCD.

### QueryBuilderWarehouseCoreTable

This class extends QueryBuilderSCD and implements IQueryBuilderWarehouseCoreTable. It is designed to work with WarehouseCoreTable entities and provides additional methods for managing security-related functionality.

## Relationship with Other Hierarchies

The QueryBuilderSCD hierarchy is related to other hierarchies in the system:

- **Relationship with Entity Hierarchies**: 
  - QueryBuilderSCD works with SCDEntity entities and WarehouseSCDTable entities
  - QueryBuilderWarehouseCoreTable works with WarehouseCoreTable entities
  - Each specialized query builder works with its corresponding entity type

- **Relationship with Interface Hierarchy**: 
  - QueryBuilderSCD implements IQueryBuilderSCD
  - QueryBuilderWarehouseCoreTable implements IQueryBuilderWarehouseCoreTable
  - Each specialized query builder implements its corresponding interface

## Related Documentation

- [Interface Hierarchies](interface_hierarchies.md)
- [WarehouseTable Class Hierarchy](warehouse_table_hierarchy.md)
- [WarehouseCoreTable Hierarchy](warehouse_core_table_hierarchy.md)

## Potential Issues

1. **Complex Generic Type Parameters**: The implementation uses complex generic type parameters with self-referential bounds, which can make the code harder to understand. For example, `J extends QueryBuilderSCD<J, E, I>` creates a recursive type definition that can be difficult to follow.

2. **Inheritance vs. Composition**: The system relies heavily on inheritance, which can lead to the fragile base class problem. Changes to base classes can unexpectedly affect subclasses, making the system harder to maintain. Consider using composition instead of inheritance for some functionality.

3. **Tight Coupling**: The QueryBuilderSCD hierarchy is tightly coupled with the entity hierarchy through generic type parameters, which can make it difficult to change one without affecting the other. For example, changing the structure of SCDEntity would likely require changes to QueryBuilderSCD.

4. **Incomplete Implementation**: The onDeleteUpdate method returns true without performing any actual logic, which suggests that it might be incomplete or not fully implemented. Consider adding proper implementation or documenting why this behavior is intentional.

5. **Documentation Gaps**: Some methods lack comprehensive documentation, which can make it difficult for developers to understand their purpose and usage. Consider adding more detailed documentation, especially for complex methods.

6. **Complex Inheritance Structure**: The class hierarchy has a complex inheritance structure with multiple levels, which can make it difficult to understand and maintain. Consider simplifying the hierarchy or providing better documentation to explain the relationships.

7. **Reactive Implementation**: The system uses Mutiny for reactive programming, but not all methods return Uni types. Consider making the API more consistent by ensuring all methods that could benefit from reactive programming return Uni types.

8. **Security Concerns**: Security-related functionality is spread across multiple classes, which can make it difficult to ensure that security is consistently applied. Consider centralizing security logic or providing better documentation on how security should be implemented.

9. **Error Handling**: Some methods don't have explicit error handling, which could lead to unexpected behavior in edge cases. Consider adding more robust error handling and documenting expected behavior in error scenarios.
