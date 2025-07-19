# WarehouseOriginalSourceSystemEnterpriseTable Class Hierarchy Documentation

## Class Hierarchy

```
SCDEntity (from com.guicedee.activitymaster.fsdm.db.abstraction.builders)
└── WarehouseSCDTable (from com.guicedee.activitymaster.fsdm.db.abstraction)
    └── WarehouseCoreTable
        └── WarehouseOriginalSourceSystemEnterpriseTable
            └── WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable
                ├── WarehouseRelationshipOriginalSourceSystemEnterpriseTable
                │   └── WarehouseClassificationRelationshipOriginalSourceSystemEnterpriseTable
                │       └── WarehouseClassificationRelationshipTypesOriginalSourceSystemEnterpriseTable
                └── WarehouseSecurityOriginalSourceSystemEnterpriseTable
```

Note: The previous documentation mentioned a WarehouseSCDTable class which no longer exists. The hierarchy has been updated to reflect the current structure. WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable now extends WarehouseOriginalSourceSystemEnterpriseTable and sits between WarehouseOriginalSourceSystemEnterpriseTable and both WarehouseRelationshipOriginalSourceSystemEnterpriseTable and WarehouseSecurityOriginalSourceSystemEnterpriseTable.

## Description

The WarehouseOriginalSourceSystemEnterpriseTable class hierarchy implements the interfaces defined in the WarehouseTable interface hierarchy. Here's a description of each class:

- **SCDEntity**: The base class for entities with Slowly Changing Dimension functionality, from the com.guicedee.activitymaster.fsdm.db.abstraction.builders package.
- **WarehouseSCDTable**: Extends SCDEntity and implements IWarehouseSCDTable. Provides basic warehouse functionality like expiring entities and checking if an entity is fake.
- **WarehouseCoreTable**: Extends WarehouseSCDTable and implements IWarehouseCoreTable. Adds security-related functionality, including methods for creating default security access for different types of users.
- **WarehouseOriginalSourceSystemEnterpriseTable**: Extends WarehouseCoreTable and implements IWarehouseTable. Adds functionality for managing original source system IDs and enterprise IDs.
- **WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable**: Extends WarehouseOriginalSourceSystemEnterpriseTable and implements IContainsActiveFlags and IContainsSystem. Adds functionality for managing active flags, system IDs, and archiving entities.
- **WarehouseSecurityOriginalSourceSystemEnterpriseTable**: Extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable and implements IWarehouseSecurityTable. Adds security-related functionality, including CRUD permissions and security tokens.
- **WarehouseRelationshipOriginalSourceSystemEnterpriseTable**: Extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable and implements IWarehouseRelationshipTable. Represents relationships between two warehouse tables, with methods for managing relationship values.
- **WarehouseClassificationRelationshipOriginalSourceSystemEnterpriseTable**: Extends WarehouseRelationshipOriginalSourceSystemEnterpriseTable and implements IWarehouseRelationshipClassificationTable. Adds classification functionality to relationships.
- **WarehouseClassificationRelationshipTypesOriginalSourceSystemEnterpriseTable**: Extends WarehouseClassificationRelationshipOriginalSourceSystemEnterpriseTable and implements IWarehouseRelationshipClassificationTypeTable. Specializes classification relationships for classification types.

## Relationship with QueryBuilderSCD Hierarchy

The WarehouseOriginalSourceSystemEnterpriseTable class hierarchy is related to the QueryBuilderSCD hierarchy through their generic type parameters:

- **WarehouseSCDTable** works with QueryBuilderSCD
- **WarehouseCoreTable** works with QueryBuilderWarehouseCoreTable, which extends QueryBuilderSCD
- **WarehouseOriginalSourceSystemEnterpriseTable** and its subclasses work with their respective query builders

This relationship creates a tight coupling between the two hierarchies, as changes to one hierarchy can affect the other. The query builder hierarchy has been simplified, with all functionality previously in QueryBuilderWarehouseSCDTable now moved to QueryBuilderSCD.

## Comparison with Interface Hierarchy

The class hierarchy generally follows the interface hierarchy, but with some significant differences:

1. **Additional Classes**: The class hierarchy includes WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable that doesn't have a corresponding interface in the interface hierarchy.

2. **Different Inheritance Paths**: In the interface hierarchy, IWarehouseSecurityTable directly extends IWarehouseSCDTable, but in the class hierarchy, WarehouseSecurityOriginalSourceSystemEnterpriseTable extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which extends WarehouseOriginalSourceSystemEnterpriseTable.

3. **Multiple Inheritance**: The interface hierarchy uses multiple inheritance (e.g., IWarehouseRelationshipClassificationTable extends both IWarehouseTable and IWarehouseRelationshipTable), but the class hierarchy uses single inheritance with composition.

4. **Implementation of Multiple Interfaces**: Some classes implement multiple interfaces. For example, WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable implements IContainsActiveFlags and IContainsSystem.

5. **Parallel Hierarchies**: The WarehouseOriginalSourceSystemEnterpriseTable and WarehouseRelationshipOriginalSourceSystemEnterpriseTable hierarchies are parallel in the implementation, both extending from WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, but in the interface hierarchy, IWarehouseRelationshipTable extends IWarehouseTable.

## Related Documentation

- [Interface Hierarchies](interface_hierarchies.md)
- [WarehouseCoreTable Hierarchy](warehouse_core_table_hierarchy.md)
- [QueryBuilderSCD Class Hierarchy](querybuilder_scd_hierarchy.md)

## Potential Issues

1. **Complex Inheritance Structure**: The class hierarchy has a complex inheritance structure with multiple levels, which can make it difficult to understand and maintain. For example, WarehouseClassificationRelationshipTypesOriginalSourceSystemEnterpriseTable extends WarehouseClassificationRelationshipOriginalSourceSystemEnterpriseTable, which extends WarehouseRelationshipOriginalSourceSystemEnterpriseTable, which extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which extends WarehouseOriginalSourceSystemEnterpriseTable, which extends WarehouseCoreTable, which extends WarehouseSCDTable, creating a deep inheritance chain.

2. **Inconsistent Inheritance Paths**: The class hierarchy has been improved to better align with the interface hierarchy, but some inconsistencies remain. For example, WarehouseSecurityOriginalSourceSystemEnterpriseTable extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which extends WarehouseOriginalSourceSystemEnterpriseTable, but IWarehouseSecurityTable extends IWarehouseSCDTable directly.

3. **Tight Coupling**: The classes are tightly coupled through their generic type parameters, which can make it difficult to change one class without affecting others. For example, WarehouseOriginalSourceSystemEnterpriseTable has a generic parameter for a security table (S extends WarehouseSecurityOriginalSourceSystemEnterpriseTable<S, ?, I>), creating a dependency between these classes.

4. **Complex Generic Type Parameters**: The extensive use of generic type parameters with self-referential bounds (e.g., J extends WarehouseSCDTable<J, Q, I>) adds complexity and can make the code harder to understand. This is particularly evident in classes like WarehouseClassificationRelationshipTypesOriginalSourceSystemEnterpriseTable with six generic type parameters.

5. **Circular Dependencies**: There appear to be circular dependencies in the class hierarchy. For example, WarehouseOriginalSourceSystemEnterpriseTable has a generic parameter for a security table (S extends WarehouseSecurityOriginalSourceSystemEnterpriseTable<S, ?, I>), but WarehouseSecurityOriginalSourceSystemEnterpriseTable extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which extends WarehouseOriginalSourceSystemEnterpriseTable. This circular dependency can lead to initialization issues and makes the code harder to understand.

6. **Lack of Documentation**: Many classes and methods lack comprehensive documentation, which can make it difficult for developers to understand their purpose and usage.

7. **Security Concerns**: Security-related functionality is spread across multiple classes (WarehouseSecurityOriginalSourceSystemEnterpriseTable, WarehouseCoreTable), which can make it difficult to ensure that security is consistently applied. The relationship between these security-related components is not always clear.

8. **Parallel Hierarchies**: The class hierarchy and interface hierarchy are parallel but not perfectly aligned, which can lead to confusion about which class implements which interface. For example, WarehouseSecurityOriginalSourceSystemEnterpriseTable implements IWarehouseSecurityTable, but it extends WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable, which doesn't have a corresponding interface.

9. **Implementation Inheritance**: The class hierarchy relies heavily on implementation inheritance, which can lead to the fragile base class problem. Changes to base classes can unexpectedly affect subclasses, making the system harder to maintain.

10. **Redundant Code**: There appears to be redundant code in the class hierarchy. For example, both WarehouseSecurityOriginalSourceSystemEnterpriseTable and WarehouseActiveFlagSytemIdOriginalSourceSystemEnterpriseTable have methods for managing active flags, which can lead to inconsistencies if one implementation changes but the other doesn't.
