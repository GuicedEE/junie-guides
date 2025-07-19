# Hibernate Reactive 4 Guide

## 1. Introduction to Hibernate Reactive

Creating a new project with Hibernate Reactive isn't hard at all. In this guide, we'll cover all the basic work involved in:

- Setting up and configuring a project
- Writing Java code to define a data model and access the database
- Performance considerations for large-scale projects using Hibernate Reactive

Before you start, we recommend taking a quick look at the very simplistic example program in the session-example directory, which shows off all the "bits" you'll need to get your own program up and running.

### 1.1. Information about Hibernate ORM

This document assumes some passing familiarity with Hibernate ORM or another implementation of JPA. If you've never used JPA before, that's OK, but you might need to refer to the following sources of information at some points in this text:

- The documentation for Hibernate ORM
- The JPA 2.2 specification
- Java Persistence with Hibernate, the latest edition of the book originally titled Hibernate in Action

### 1.2. Setting up a reactive Hibernate project

If you're using Hibernate Reactive outside of the Quarkus environment, you'll need to:

- Include Hibernate Reactive itself, along with the appropriate Vert.x reactive database client, as dependencies of your project
- Configure Hibernate Reactive with information about your database, using Hibernate configuration properties

Or, if you want to use Hibernate Reactive in Quarkus, you can generate a preconfigured skeleton project from the Quarkus website.

#### 1.2.1. Including Hibernate Reactive in your project build

Add the following dependency to your project:

```xml
<dependency>
    <groupId>org.hibernate.reactive</groupId>
    <artifactId>hibernate-reactive-core</artifactId>
    <version>{version}</version>
</dependency>
```

Where `{version}` is the version of Hibernate Reactive you're using.

You'll also need to add a dependency for the Vert.x reactive database driver for your database, one of the following options:

| Database | Driver dependency |
|----------|-------------------|
| PostgreSQL or CockroachDB | `io.vertx:vertx-pg-client:{vertxSqlClientVersion}` |
| MySQL or MariaDB | `io.vertx:vertx-mysql-client:{vertxSqlClientVersion}` |
| DB2 | `io.vertx:vertx-db2-client:{vertxSqlClientVersion}` |
| SQL Server | `io.vertx:vertx-mssql-client:${vertxSqlClientVersion}` |
| Oracle | `io.vertx:vertx-oracle-client:${vertxSqlClientVersion}` |

Where `{vertxSqlClientVersion}` is the version of Vert.x compatible with the version of Hibernate Reactive you're using.

You don't need to depend on the JDBC driver for your database.

#### 1.2.2. Optional dependencies

Optionally, you might also add any of the following additional features:

| Optional feature | Dependencies |
|------------------|--------------|
| An SLF4J logging implementation | `org.apache.logging.log4j:log4j-core` or `org.slf4j:slf4j-jdk14` |
| The Hibernate metamodel generator, if you're using the JPA criteria query API | `org.hibernate.orm:hibernate-jpamodelgen` |
| Hibernate Validator | `org.hibernate.validator:hibernate-validator` and `org.glassfish:jakarta.el` |
| Compile-time checking for your HQL queries | `org.hibernate:query-validator` |
| Second-level cache support via JCache and EHCache | `org.hibernate.orm:hibernate-jcache` along with `org.ehcache:ehcache` |
| SCRAM authentication support for PostgreSQL | `com.ongres.scram:scram-client:3.1` |

You might also add the Hibernate bytecode enhancer to your Gradle build if you want to use field-level lazy fetching.

> Field-level lazy fetching is an advanced feature that most programs don't need. Stick to the basics for now.

There's an example Gradle build included in the example program.

#### 1.2.3. Basic configuration

Hibernate Reactive is configured via the standard JPA `persistence.xml` document which must be placed, as usual, in the `/META-INF` directory.

An example `persistence.xml` file is included in the example program.

The only required configuration that's really specific to Hibernate Reactive is the persistence `<provider>` element, which must be explicit:

```xml
<provider>org.hibernate.reactive.provider.ReactivePersistenceProvider</provider>
```

Otherwise, configuration is almost completely transparent—you can configure Hibernate Reactive pretty much exactly as you would usually configure Hibernate ORM core.

Just like in regular JPA, you should list your entity classes in `persistence.xml`:

```xml
<class>org.hibernate.reactive.example.session.Author</class>
<class>org.hibernate.reactive.example.session.Book</class>
```

A full list of configuration properties recognized by Hibernate may be found in the documentation for Hibernate ORM. You'll never need to touch most of these. The properties you do need at this stage are these three:

| Configuration property name | Purpose |
|----------------------------|---------|
| `jakarta.persistence.jdbc.url` | JDBC URL of your database |
| `jakarta.persistence.jdbc.user` and `jakarta.persistence.jdbc.password` | Your database credentials |

These configuration properties have `jdbc` in their names, but of course there's no JDBC in Hibernate Reactive, and these are simply the legacy property names defined by the JPA specification. In particular, Hibernate Reactive itself parses and interprets the JDBC URL.

> You don't need to specify `hibernate.dialect`. The correct Hibernate Dialect will be determined for you by Hibernate Reactive.

The Vert.x database client has built-in connection pooling and prepared statement caching. You might want to control the size of the connection pool:

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.connection.pool_size` | The maximum size of the reactive connection pool |

We'll learn about more advanced connection pool tuning later, in Tuning the Vert.x pool.

> Hibernate has many configurable things, but many exist only to maintain compatibility with legacy code, and most configuration properties directly related to JDBC or JTA aren't relevant in the context of Hibernate Reactive.

#### 1.2.4. Automatic schema export

You can have Hibernate Reactive infer your database schema from the mapping annotation you've specified in your Java code, and export the schema at initialization time by specifying one or more of the following configuration properties:

| Configuration property name | Purpose |
|----------------------------|---------|
| `jakarta.persistence.schema-generation.database.action` | If `create`, first drop the schema and then export tables, sequences, and constraints.<br>If `create-only`, export tables, sequences, and constraints.<br>If `create-drop`, drop the schema and recreate it on SessionFactory startup. Additionally, drop the schema on SessionFactory shutdown.<br>If `drop`, drop the schema on SessionFactory shutdown.<br>If `validate`, validate the database schema without changing it.<br>If `update`, only export what's missing in the schema. |
| `jakarta.persistence.create-database-schemas` | (Optional) If `true`, automatically create schemas and catalogs |
| `jakarta.persistence.schema-generation.create-source` | (Optional) If `metadata-then-script` or `script-then-metadata`, execute an additional SQL script when exported tables and sequences |
| `jakarta.persistence.schema-generation.create-script-source` | (Optional) The name of the SQL script to be executed |

This feature is extremely useful for testing.

> Hibernate Reactive doesn't support `validate` and `update` with Db2.

Schema export uses blocking operations so starting the factory might require special handling when using it. Failing to do so will cause an exception:

```
io.vertx.core.VertxException: Thread blocked
```

You can solve this issue using `executeBlocking`:

```java
Vertx vertx = ...

Uni<Void> startHibernate = Uni.createFrom().deferred(() -> {
  emf = Persistence
    .createEntityManagerFactory("demo")
    .unwrap(Mutiny.SessionFactory.class);

  return Uni.createFrom().voidItem();
});

startHibernate = vertx.executeBlocking(startHibernate)
  .onItem().invoke(() -> logger.info("✅ Hibernate Reactive is ready"));
```

#### 1.2.5. Logging the generated SQL

To see the generated SQL as it's sent to the database, either:

- Set the property `hibernate.show_sql` to `true`, or
- Enable debug-level logging for the category `org.hibernate.SQL` using your preferred SLF4J logging implementation.

For example, if you're using Log4J 2 (as above in Optional dependencies), add these lines to your `log4j2.properties` file:

```properties
logger.hibernate.name = org.hibernate.SQL
logger.hibernate.level = debug
```

An example `log4j2.properties` file is included in the example program.

You can make the logged SQL more readable by enabling one or both of the following settings:

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.format_sql` | If `true`, log SQL in a multiline, indented format |
| `hibernate.highlight_sql` | If `true`, log SQL with syntax highlighting via ANSI escape codes |

#### 1.2.6. Minimizing repetitive mapping information

The following properties are very useful for minimizing the amount of information you'll need to explicitly specify in `@Table` and `@Column` annotations which we'll discuss below in Mapping entity classes:

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.default_schema` | A default schema name for entities which do not explicitly declare one |
| `hibernate.default_catalog` | A default catalog name for entities which do not explicitly declare one |
| `hibernate.physical_naming_strategy` | A `PhysicalNamingStrategy` implementing your database naming standards |

> Writing your own `PhysicalNamingStrategy` is an especially good way to reduce the clutter of annotations on your entity classes, and we think you should do it for any nontrivial data model.

#### 1.2.7. Nationalized character data in SQL Server

By default, SQL Server's `char` and `varchar` types don't accommodate Unicode data. So, if you're working with SQL Server, you might need to force Hibernate to use the `nchar` and `nvarchar` types.

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.use_nationalized_character_data` | Use `nchar` and `nvarchar` instead of `char` and `varchar` |

Alternatively, you can configure SQL Server to use the UTF-8 enabled collation `_UTF8`.

### 1.3. Writing the Java code

With that out of the way, we're all set to write some Java code!

As is the case in any project that uses Hibernate, your persistence-related code comes in two main pieces:

- A representation of your data model in Java, which takes the form of a set of annotated entity classes, and
- A larger number of functions which interact with Hibernate's APIs to perform the persistence operations associated with your various transactions.

The first part, the data or "domain" model, is usually easier to write, but doing a great and very clean job of it will strongly affect your success in the second part.

> Take your time with this code, and try to produce a Java model that's as close as reasonable to the relational data model. Avoid using exotic or advanced mapping features when they're not really needed. When in the slightest doubt, map a foreign key relationship using `@ManyToOne` with `@OneToMany(mappedBy=…​)` in preference to more complicated association mappings.

The second part of the code is much trickier to get right. This code must:

- Manage transactions and reactive sessions
- Construct reactive streams by chaining persistence operations invoked on the reactive session
- Fetch and prepare data needed by the UI
- Handle failures

> Some responsibility for transaction and session management, and for recovery from certain kinds of failure, can be best handled in some sort of framework code.

#### 1.3.1. Mapping entity classes

We won't have much to say about the entity classes here, simply because the principles behind mapping entity classes in Hibernate Reactive, along with the actual mapping annotations you'll use, are all identical to regular Hibernate ORM and other implementations of JPA.

For example:

```java
@Entity
@Table(name="authors")
class Author {
    @Id @GeneratedValue
    private Integer id;

    @NotNull @Size(max=100)
    private String name;

    @OneToMany(mappedBy = "author", cascade = PERSIST)
    private List<Book> books = new ArrayList<>();

    Author(String name) {
        this.name = name;
    }

    Author() {}

    // getters and setters...
}
```

You're quite free to mix and match:

- The regular JPA mapping annotations defined in the package `jakarta.persistence` with
- The advanced mapping annotations in `org.hibernate.annotations`, and even
- Annotations like `@NotNull` and `@Size` defined by Bean Validation.

A full list of object/relational mapping annotations may be found in the documentation for Hibernate ORM. Most mapping annotations are already supported in Hibernate Reactive, though there are still a handful of limitations at this time.

##### Common JPA annotations

The most common and useful mapping annotations include these standard JPA annotations:

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Declares an entity class (a class with its own database table an persistent identity) |
| `@MappedSuperclass` | A superclass that declares common persistent fields of its `@Entity` subclasses |
| `@Embeddable` or `@Embedded` | Declare an embeddable class (a class without its own persistent identity or database table) |
| `@Inheritance` | Defines how inheritance hierarchies should be mapped to database tables |
| `@Id` | Specifies that a field of an entity holds the persistent identity of the entity, and maps to the primary key of its table |
| `@IdClass` | Specifies a class representing the composite primary key of the entity (for entities with multiple `@Id` fields) |
| `@EmbeddedId` | Specifies that a field of an entity holds its composite primary key represented as an `@Embeddable` class |
| `@GeneratedValue` | Specifies that an identifier is a system-generated surrogate key |
| `@Version` | Specifies that a field of an entity holds a version number used for optimistic locking |
| `@Enumerated` | Maps a field holding an enum |
| `@ManyToOne` | Declares a many-to-one association to a second entity |
| `@OneToOne` | Declares a one-to-one association to a second entity |
| `@OneToMany` | Declares a one-to-many association to a second entity |
| `@Table` | Specifies a mapping to a database table |
| `@SecondaryTable` | Specifies a mapping to a second database table |
| `@Column` | Specifies a mapping to a database column |
| `@JoinColumn` | Specifies a mapping to a database foreign key |

##### Useful Hibernate annotations

These Hibernate annotations are also quite useful to know about:

| Annotation | Purpose |
|------------|---------|
| `@Cache` | Enables second-level caching for an entity |
| `@Formula` | Maps field to SQL expression instead of a column |
| `@CreationTimestamp`, `@UpdateTimestamp` | Automatically assign a timestamp to a field |
| `@OptimisticLocking` | Enables optimistic locking for entities with no `@Version` field |
| `@FilterDef` and `@Filter` | Define a Hibernate filter |
| `@FetchProfile` | Defines a Hibernate fetch profile |
| `@Generated` | Defines a property generated by the database |
| `@ColumnDefault` | Specifies a SQL expression used to assign a default value to a column (use in combination with `@Generated(INSERT)`) |
| `@GenericGenerator` | Selects a custom id generator |
| `@DynamicInsert` and `@DynamicUpdate` | Generate SQL dynamically with only needed columns (instead of using static SQL generated at startup) |
| `@Fetch` | Specifies the fetching mode for an association |
| `@BatchSize` | Specifies the batch size for batch fetching an association |
| `@Loader` | Specifies a named query used to fetch an entity by id (for example, when `find(type, id)` is called) in place of the default SQL generated by Hibernate |
| `@SqlInsert`, `@SqlUpdate`, `@SqlDelete` | Specify custom DML for entity operations |
| `@NaturalId` | Marks a field or fields as an alternative "natural" identifier (unique key) of the entity |
| `@Nationalized` | Use `nchar`, `nvarchar`, or `nclob` selectively for one particular column |
| `@Immutable` | Specifies that an entity or collection is immutable |
| `@SortNatural` or `@SortComparator` | Maps a `SortedSet` or `SortedMap` |
| `@Check` | Declares a SQL check constraint to be added to DDL |

##### Bean Validation annotations

Information about Bean Validation annotations may be found in the documentation for Hibernate Validator.

> For defining a required field, we prefer to use the `@NotNull` annotation from Bean Validation instead of JPA's more verbose `@Basic(optional=false)`. Similarly, we prefer to define the length of a text field using `@Size(100)` rather than `@Column(length=100)`.

#### 1.3.2. Getters and setters

When using Hibernate Reactive outside the Quarkus environment, you'll need to write your entity classes according to the usual JPA conventions, which require:

- Private fields for persistent attributes, and
- A nullary constructor.

It's illegal to access persistent fields from outside the entity class. Therefore, external access to persistent fields must be intermediated via getter and setter methods defined by the entity class.

> If you access fields of an unfetched entity instance from code outside the entity class, you'll obtain bogus null or default (zero) values!

When you use Hibernate Reactive in Quarkus, these requirements are relaxed, and you can use public fields instead of getters and setters if you prefer.

#### 1.3.3. equals() and hashCode()

Entity classes should override `equals()` and `hashCode()`. People new to Hibernate or JPA are often confused by exactly which fields should be included in the `hashCode()`, so please keep the following principles in mind:

- You should not include mutable fields in the hashcode, since that would require rehashing any collection containing the entity whenever the field is mutated.
- It's not completely wrong to include a generated identifier (surrogate key) in the hashcode, but since the identifier is not generated until the entity instance is made persistent, you must take great care to not add it to any hashed collection before the identifier is generated. We therefore advise against including any database-generated field in the hashcode.
- It's OK to include any immutable, non-generated field in the hashcode.

We therefore recommend identifying a natural key for each entity, that is, a combination of fields that uniquely identifies an instance of the entity, from the perspective of the data model of the program. The business key should correspond to a unique constraint on the database, and to the fields which are included in `equals()` and `hashCode()`.

That said, an implementation of `equals()` and `hashCode()` based on the generated identifier of the entity can work if you're careful.

> If you can't identify a natural key, it might be a sign that you need to think more carefully about some aspect of your data model. If an entity doesn't have a meaningful unique key, then it's impossible to say what event or object it represents in the "real world" outside your program.

Note that even when you've identified a natural key, we still recommend the use of a generated surrogate key in foreign keys, since this makes your data model much easier to change.

#### 1.3.4. Identifier generation

One area where the functionality of Hibernate Reactive diverges from plain Hibernate is in the area of id generation. Custom identifier generators written to work with Hibernate ORM and JDBC will not work in the reactive environment.

Sequence, table, and UUID id generation is built in, and these id generation strategies may be selected using the usual JPA mapping annotations: `@GeneratedValue`, `@TableGenerator`, `@SequenceGenerator`.

On MySQL, an autoincrement column may be used by specifying `@GeneratedValue(strategy=GenerationType.IDENTITY)`

Custom id generators may be defined by implementing `ReactiveIdentifierGenerator` and declaring the custom implementation using `@GenericGenerator`.

Natural ids—including composite ids—may be assigned by the program in the usual way.

The standard id generation strategies defined by the JPA specification may be customized using the following annotations:

| Annotation | Purpose |
|------------|---------|
| `@SequenceGenerator` | Configure a generator based on a database sequence |
| `@TableGenerator` | Configure a generator based on a row of a database table |

For example, sequence id generation may be specified like this:

```java
@Entity
@Table(name="authors")
class Author {
    @Id @GeneratedValue(generator = "authorIds")
    @SequenceGenerator(name = "authorIds",
               sequenceName = "author_ids",
             allocationSize = 20)
    Integer id;
    ...
}
```

You can find more information in the JPA specification.

If you have very particular requirements, you can check out the Javadoc of `ReactiveIdentifierGenerator` for information on how to implement your own custom reactive identifier generator.

#### 1.3.5. JSON Mapping

Like in Hibernate ORM, it's possible to map a JSON field using the `SqlTypes.JSON` or an embeddable.

> Supports for JSON is limited to PostgreSQL and MySQL. Only PostgreSQL supports mapping a JSON using an embeddable

Hibernate Reactive will recognize fields of type `io.vertx.core.json.JsonObject` as JSON columns:

```java
@Entity
class Example {
    JsonObject jsonField;
}
```

Finally, the `org.hibernate.reactive.StringToJsonConverter` converter is available for mapping a string to a JSON column:

```java
@Entity
class Example {
    @Convert(converter = StringToJsonConverter.class)
    String json;
}
```

#### 1.3.6. Custom types

Hibernate custom types based on the `UserType` interface are targeted toward use with JDBC, and depend on interfaces defined by JDBC. So Hibernate Reactive features an adaptor that exposes a partial implementation of JDBC to the `UserType` implementation.

Therefore, some existing `UserType` implementations will work in Hibernate Reactive, depending upon precisely which features of JDBC they depend on.

> Where possible, use a JPA attribute converter instead of a custom type, since attribute converters are not in any way tied to JDBC.

You may specify a custom type by annotating a field of an entity class with the Hibernate `@Type` annotation.

#### 1.3.7. Attribute converters

Any JPA `AttributeConverter` works in Hibernate Reactive. For example:

```java
@Converter
public class BigIntegerAsString implements AttributeConverter<BigInteger, String> {
    @Override
    public String convertToDatabaseColumn(BigInteger attribute) {
        return attribute == null ? null : attribute.toString(2);
    }

    @Override
    public BigInteger convertToEntityAttribute(String string) {
        return string == null ? null : new BigInteger(string, 2);
    }
}
```

You'll need to use one or both of these annotations:

| Annotation | Purpose |
|------------|---------|
| `@Converter` | Declares a class implementing `AttributeConverter` |
| `@Convert` | Specifies an `AttributeConverter` converter to use for a field of an entity class |

You'll find more information in the Javadoc for these annotations and in the JPA specification.

#### 1.3.8. APIs for chaining reactive operations

When you write persistence logic using Hibernate Reactive, you'll be working with a reactive `Session` most of the time. Just to make things a little more confusing for new users, the reactive `Session` and its related interfaces all come in two flavors:

- `Stage.Session` and friends provide a reactive API based around Java's `CompletionStage`, and
- `Mutiny.Session` and friends provide an API based on Mutiny.

You'll need to decide which API you want to use!

If you take the time to look over the types `Stage.Session` and `Mutiny.Session`, you'll notice they're almost identical. Choosing between them is a matter of deciding which reactive API you want to use for working with reactive streams. Your decision won't affect what you can do with Hibernate Reactive. On the other hand, we've sent a lot of feedback and requests for improvement to the Mutiny team, and we think it's now the case that Hibernate Reactive code is simpler and cleaner with Mutiny.

These are the most important operations on reactive streams that you'll need all the time when working with Hibernate Reactive:

| Purpose | Java CompletionStage | Mutiny Uni |
|---------|---------------------|------------|
| Chain non-blocking operations | `thenCompose()` | `chain()` |
| Transform streamed items | `thenApply()` | `map()` and `replaceWith()` |
| Perform an action using streamed items | `thenAccept()` | `invoke()` and `call()` |
| Perform cleanup (similar to finally) | `whenComplete()` | `eventually()` |

In this introduction, our code examples usually use Mutiny. If you're more familiar with `CompletionStage`, you can refer to the above table to help you understand the code.

When we use the term reactive stream in this document, we mean:

- A chain of `CompletionStages`, or
- A chain of Mutiny `Unis` and `Multis`

that is built by the program in order to service a particular request, transaction, or unit of work.

#### 1.3.9. Obtaining a reactive session factory

Whatever you decide, the first step to getting a reactive session is to obtain a JPA `EntityManagerFactory` just as you usually would in plain ol' regular JPA, for example, by calling:

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("example");
```

Now, `unwrap()` the reactive `SessionFactory`. If you want to use `CompletionStages` for chaining reactive operations, ask for a `Stage.SessionFactory`:

```java
Stage.SessionFactory sessionFactory = emf.unwrap(Stage.SessionFactory.class);
```

Or, if you prefer to use the Mutiny-based API, `unwrap()` the type `Mutiny.SessionFactory`:

```java
Mutiny.SessionFactory sessionFactory = emf.unwrap(Mutiny.SessionFactory.class);
```

Reactive sessions may be obtained from the resulting reactive `SessionFactory`.

> It's also possible to construct a reactive `SessionFactory` via programmatic configuration based on Hibernate's `ServiceRegistry` architecture, by using a `ReactiveServiceRegistryBuilder`. But that's outside the scope of this document.

#### 1.3.10. Obtaining a reactive session

Persistence operations are exposed via a reactive `Session` object. It's very important to understand that most operations of this interface are non-blocking, and execution of SQL against the database is never performed synchronously. Persistence operations that belong to a single unit of work must be chained by composition within a single reactive stream.

Also remember that a Hibernate session is a lightweight object that should be created, used, and then discarded within a single logical unit of work.

> That is to say, you should reuse the same session across multiple persistence operations within a single reactive stream representing a certain transaction or unit of work, but don't share a session between different concurrent reactive streams!

To obtain a reactive `Session` from the `SessionFactory`, use `withSession()`:

```java
sessionFactory.withSession(
        session -> session.find(Book.class, id)
                .invoke(
                    book -> ... //do something with the book
                )
);
```

The resulting `Session` object is automatically associated with the current reactive stream, and so nested calls to `withSession()` in a given stream automatically obtain the same shared session.

Alternatively, you may use `openSession()`, but you must remember to `close()` the session when you're done. And you must take great care to only access each session from within exactly one Vert.x context. (See Sessions and Vert.x contexts more on this).

```java
Uni<Session> sessionUni = sessionFactory.openSession();
sessionUni.chain(
        session -> session.find(Book.class, id)
                .invoke(
                    book -> ... //do something with the book
                )
                .eventually(session::close)
);
```

#### 1.3.11. Using the reactive session

The `Session` interface has methods with the same names as methods of the JPA `EntityManager`. You might already be familiar with the following session operations defined by JPA:

| Method name and parameters | Effect |
|----------------------------|--------|
| `find(Class,Object)` | Obtain a persistent object given its type and its id (primary key) |
| `persist(Object)` | Make a transient object persistent and schedule a SQL insert statement for later execution |
| `remove(Object)` | Make a persistent object transient and schedule a SQL delete statement for later execution |
| `merge(Object)` | Copy the state of a given detached object to a corresponding managed persistent instance and return the persistent object |
| `refresh(Object)` | Refresh the persistent state of an object using a new SQL select to retrieve the current state from the database |
| `lock(Object)` | Obtain a pessimistic lock on a persistent object |
| `flush()` | Detect changes made to persistent objects association with the session and synchronize the database state with the state of the session by executing SQL insert, update, and delete statements |
| `detach(Object)` | Disassociate a persistent object from a session without affecting the database |
| `getReference(Class,id)` or `getReference(Object)` | Obtain a reference to a persistent object without actually loading its state from the database |

If you're not familiar with these operations, don't despair! Their semantics are defined in the JPA specification, and in the API documentation, and are explained in innumerable articles and blog posts. But if you already have some experience with Hibernate or JPA, you're right at home!

> Just like in Hibernate ORM, the session is considered to be unusable after any of its methods throws an exception. If you receive an exception from Hibernate Reactive, you should immediately close and discard the current session.

Now, here's where Hibernate Reactive is different: in the reactive API, each of these methods returns its result in a non-blocking fashion via a Java `CompletionStage` (or Mutiny `Uni`). For example:

```java
session.find(Book.class, book.id)
       .invoke( book -> System.out.println(book.title + " is a great book!") )
```

On the other hand, methods with no meaningful return value just return `CompletionStage<Void>` (or `Uni<Void>`).

```java
session.find(Book.class, id)
       .call( book -> session.remove(book) )
       .call( () -> session.flush() )
```

> The session will be flushed automatically at the end of a unit of work if—and only if—you use a transaction, as described below in Transactions. If you don't use a transaction, and forget to flush the session explicitly, your persistence operations might never be sent to the database!

An extremely common mistake when using reactive streams is to forget to chain the return value of a "void-like" method. For example, in the following code, the `flush()` operation is never executed, because `invoke()` doesn't chain its return value to the tip of the stream.

```java
session.find(Book.class, id)
       .call( book -> session.remove(book) )
       .invoke( () -> session.flush() )   //OOPS, WRONG!!
```

So remember:

- You must use `thenCompose()`, not `thenAccept()`, when calling "void-like" methods that return `CompletionStage`.
- In Mutiny, you must use `call()`, not `invoke()`, when calling "void-like" methods that return `Uni`.

The same problem occurs in the following code, but this time it's `remove()` that never gets called:

```java
session.find(Book.class, id)
       .call( book -> {
           session.remove(book);   //OOPS, WRONG!!
           return session.flush();
       } )
```

If you already have some experience with reactive programming, there's nothing new to learn here. But if you are new to reactive programming, just be aware that you're going to make this mistake, in some form, at least once!

#### 1.3.12. Queries

Naturally, the `Session` interface is a factory for `Query` instances which allow you to set query parameters and execute queries and DML statements:

| Method name | Effect |
|-------------|--------|
| `createQuery()` | Obtain a `Query` for executing a query or DML statement written in HQL or JPQL |
| `createNativeQuery()` | Obtain a `Query` for executing a query or DML statement written in the native SQL dialect of your database |
| `createNamedQuery()` | Obtain a `Query` for executing a named HQL or SQL query defined by a `@NamedQuery` annotation |

That `createQuery()` method produces a reactive `Query`, allowing HQL / JPQL queries to be executed asynchronously, always returning their results via a `CompletionStage` (or `Uni`):

```java
session.createQuery("select title from Book order by title desc")
       .getResultList()
       .invoke( list -> list.forEach(System.out::println) )
```

The `Query` interface defines the following important operations:

| Method name | Effect |
|-------------|--------|
| `setParameter()` | Set an argument of a query parameter |
| `setMaxResults()` | Limit the number of results returned by the query |
| `setFirstResult()` | Specify a certain number of initial results to be skipped (for result pagination) |
| `getSingleResult()` | Execute a query and obtain the single result |
| `getResultList()` | Execute a query and obtain the results as a list |
| `executeUpdate()` | Execute a DML statement and obtain the number of affected rows |

> The Hibernate Reactive Query API doesn't support `java.util.Date` or its subclasses in `java.sql`, nor `java.util.Calendar`. Always use `java.time` types like `LocalDate` or `LocalDateTime` for specifying arguments to temporally-typed query parameters.

For JPA criteria queries, you must first obtain the `CriteriaBuilder` using `SessionFactory.getCriteriaBuilder()`, and execute your query using `Session.createQuery()`.

```java
CriteriaQuery<Book> query = factory.getCriteriaBuilder().createQuery(Book.class);
Root<Author> a = query.from(Author.class);
Join<Author,Book> b = a.join(Author_.books);
query.where( a.get(Author_.name).in("Neal Stephenson", "William Gibson") );
query.select(b);
return session.createQuery(query).getResultList().invoke(
        books -> books.forEach( book -> out.println(book.title) )
);
```

#### 1.3.13. Fetching lazy associations

In Hibernate ORM, a lazy association is fetched transparently when the association is first accessed within a session. In Hibernate Reactive, on the other hand, lazy association fetching is an asynchronous process that produces a result via a `CompletionStage` (or `Uni`).

Therefore, lazy fetching is an explicit operation named `fetch()`, a static method of `Stage` and `Mutiny`:

```java
session.find(Author.class, author.id)
       .chain( author -> Mutiny.fetch(author.books) )
       .invoke( books -> ... )
```

Of course, this isn't necessary if you fetch the association eagerly.

> It's very important to make sure you've fetched all the data that will be needed before passing control to the process that renders the UI! There is no transparent lazy fetching in Hibernate Reactive, so patterns like "open session in view" will not help at all.

Sometimes you might need to chain multiple calls to `fetch()`, for example:

```java
Mutiny.fetch( session.getReference(detachedAuthor) )
       .chain( author -> Mutiny.fetch(author.books) )
       .invoke( books -> ... )
```

> `fetch()` isn't recursive! You can't fetch an association belonging to an unfetched entity without fetching the entity instance first.

#### 1.3.14. Field-level lazy fetching

Similarly, field-level lazy fetching—an advanced feature, which is only supported in conjunction with Hibernate's optional compile-time bytecode enhancer—is also an explicit operation.

To declare a lazy field we usually use the JPA `@Basic` annotation:

```java
@Basic(fetch=LAZY) String isbn;
```

An optional one-to-one association declared `@OneToOne(fetch=LAZY)` is also considered field-level lazy.

> This annotation has no effect at all unless the entity is processed by the bytecode enhancer during the build. Most Hibernate users don't bother with this, since it's often an inconvenience.
> On the other hand, if you're running Hibernate Reactive in Quarkus, the bytecode enhancer is always enabled, and you won't even notice it's there.

A lazy field is only fetched if we explicitly request it by calling an overloaded version of the `fetch()` operation:

```java
session.find(Book.class, book.id)
       .chain( book -> session.fetch(book, Book_.isbn) )
       .invoke( isbn -> ... )
```

Note that the field to fetch is identified by a JPA metamodel `Attribute`.

> We don't encourage you to use field-level lazy fetching unless you have very specific requirements. It's almost always more efficient to fetch all the fields of an entity at once. Field-level lazy fetching is every bit as vulnerable to N+1 selects as lazy association fetching.

#### 1.3.15. Transactions

The `withTransaction()` method performs work within the scope of a database transaction.

```java
session.withTransaction( tx -> session.persist(book) )
```

The session is automatically flushed at the end of the transaction.

For a given `Session` object, nested calls to `withTransaction()` occur within the same shared transaction context. However, notice that the transaction is a resource local transaction only, delegated to the underlying Vert.x database client, and does not span multiple datasources, nor integrate with JPA container-managed transactions.

> Hibernate Reactive does not currently support distributed (XA) transactions.

For extra convenience, there's a method that opens a session and starts a transaction in one call:

```java
sessionFactory.withTransaction( (session, tx) -> session.persist(book) )
```

This is probably the most convenient thing to use most of the time.

## 1.4. Integrating with Vert.x

At runtime, interaction with the database happens on a Vert.x thread, typically the event loop thread. When you write code that creates and destroys Hibernate Reactive sessions, it's important to understand how sessions relate to threads and Vert.x contexts.

### 1.4.1. Sessions and Vert.x contexts

Remember how in regular old Hibernate JPA, you're not supposed to share a session between multiple threads? Well, the idea here is essentially similar, it's just that the notion of a "thread" is a little more slippery, or at least more technical. You need to be able to replace the idea of a "thread" with the idea of a chain of callbacks occurring on a reactive stream, all within the scope of a certain Vert.x local context.

When you create a session using `withSession()` or `withTransaction()`, it's automatically associated with the current Vert.x local context, and propagates with the local context, as mentioned above in Obtaining a reactive session. And you're only allowed to use the session from the thread that owns this local context. If you screw up, and use it from a different thread, you might see this error:

```
HR000068: This method should exclusively be invoked from a Vert.x EventLoop thread; ...
```

On the other hand, if you use `openSession()`, you'll have to manage the association between sessions and contexts yourself. Now, that's in principle straightforward, but you'd be surprised how often people mess up.

> The Hibernate session is not thread-safe (nor "stream-safe"), so using it across different threads (or reactive streams) may cause bugs that are extremely hard to detect. Don't say we didn't warn you!

More than a few users are surprised by this restriction. But we insist that it's completely natural. An atomic operation of the session, from your point of view as a user of the session, is a method like `flush()`, `find()`, or `getResultList()`. Any one of those methods can result in multiple interactions with the database. And between such interactions, the session is simply not in a well-defined state. Reactive streams are a kind of thread, and it's quite unreasonable to expect that reactive programming should vanish away your concurrency problems in a shimmering puff of pixie dust. That's not how these things work.

For example, I bet you would like to be able to write code like this:

```java
List<CompletionStage> list = ...
for (Entity entity : entities) {
    list.add(session.persist(entity));
}
CompletableFuture.allOf(list).thenCompose(session::flush);
```

Well, we're sorry, but that's just not allowed. Parallel reactive streams may not share a session. Each stream must have its own session.

### 1.4.2. Executing code in a Vert.x context

What if you need to run a block of code within the scope of a Vert.x context, but the current thread isn't associated with a `Context`? One solution is to obtain a Vert.x `Context` object using `getOrCreateContext()` and then call `runOnContext()` to execute the code in that context.

```java
Context currentContext = Vertx.currentContext();
currentContext.runOnContext( event -> {
    // Here you will be able to use the session
});
```

Within the block of code passed to `runOnContext()`, you'll be able to use the Hibernate Reactive session associated with the context.

### 1.4.3. Vert.x instance service

The `VertxInstance` service defines how Hibernate Reactive obtains an instance of Vert.x. The default implementation just creates one the first time it's needed. But if your program requires control over how the Vert.x instance is created, or how it's obtained, you can override the default implementation and provide your own `VertxInstance`. Let's consider this example:

```java
public class MyVertx implements VertxInstance {

  private final Vertx vertx;

  public MyVertx() {
    this.vertx = Vertx.vertx();
  }

  @Override
  public Vertx getVertx() {
    return vertx;
  }

}
```

One way to register this implementation is to configure Hibernate programmatically, for example:

```java
Configuration configuration = new Configuration();
StandardServiceRegistryBuilder builder = new ReactiveServiceRegistryBuilder()
        .addService( VertxInstance.class, new MyVertx() )
        .applySettings( configuration.getProperties() );
StandardServiceRegistry registry = builder.build();
SessionFactory sessionFactory = configuration.buildSessionFactory( registry );
```

Alternatively, you could implement the `ServiceContributor` interface.

```java
public class MyServiceContributor implements ServiceContributor {
  @Override
  public void contribute(StandardServiceRegistryBuilder serviceRegistryBuilder) {
    serviceRegistryBuilder.addService( VertxInstance.class, new MyVertxProvider() );
  }
}
```

To register this `ServiceContributor`, add a text file named `org.hibernate.service.spi.ServiceContributor` to `/META-INF/services/`.

```
org.myproject.MyServiceContributor
```

## 1.5. Tuning and performance

Once you have a program up and running using Hibernate Reactive to access the database, it's inevitable that you'll find places where performance is disappointing or unacceptable.

Fortunately, most performance problems are relatively easy to solve with the tools that Hibernate makes available to you, as long as you keep a couple of simple principles in mind.

First and most important: the reason you're using Hibernate Reactive is because it makes things easier. If, for a certain problem, it's making things harder, stop using it. Solve this problem with a different tool instead.

> Just because you're using Hibernate in your program doesn't mean you have to use it everywhere.

Second: there are two main potential sources of performance bottlenecks in a program that uses Hibernate:

- Too many round trips to the database, and
- Memory consumption associated with the first-level (session) cache.

So performance tuning primarily involves reducing the number of accesses to the database, and/or controlling the size of the session cache.

But before we get to those more advanced topics, we should start by tuning the connection pool.

### 1.5.1. Tuning the Vert.x pool

In Basic configuration we already saw how to set the size of the Vert.x database connection pool. When it comes time for performance tuning, you can further customize the pool and prepared statement cache via the following configuration properties:

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.vertx.pool.max_wait_queue_size` | The maximum connection requests allowed in the wait queue |
| `hibernate.vertx.pool.connect_timeout` | The maximum time to wait when requesting a pooled connection, in milliseconds |
| `hibernate.vertx.pool.idle_timeout` | The maximum time a connection may sit idle, in milliseconds |
| `hibernate.vertx.pool.cleaner_period` | The Vert.x connection pool cleaner period, in milliseconds |
| `hibernate.vertx.prepared_statement_cache.max_size` | The maximum size of the prepared statement cache |
| `hibernate.vertx.prepared_statement_cache.sql_limit` | The maximum length of prepared statement SQL string that will be cached |

Finally, for more advanced cases, you can write your own code to configure the Vert.x client by implementing `SqlClientPoolConfiguration`.

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.vertx.pool.configuration_class` | A class implementing `SqlClientPoolConfiguration` |

### 1.5.2. Enabling statement batching

An easy way to improve performance of some transactions with almost no work at all is to turn on automatic DML statement batching. Batching only helps in cases where a program executes many inserts, updates, or deletes against the same table in a single transaction.

All you need to do is set a single property:

| Configuration property name | Purpose |
|----------------------------|---------|
| `hibernate.jdbc.batch_size` | Maximum batch size for SQL statement batching |

(Again, this property has `jdbc` in its name, but Hibernate Reactive repurposes it for use with the reactive connection.)

> Even better than DML statement batching is the use of HQL update or delete queries, or even native SQL that calls a stored procedure!

### 1.5.3. Association fetching

Achieving high performance in ORM means minimizing the number of round trips to the database. This goal should be uppermost in your mind whenever you're writing data access code with Hibernate. The most fundamental rule of thumb in ORM is:

- Explicitly specify all the data you're going to need right at the start of a session/transaction, and fetch it immediately in one or two queries,
- And only then start navigating associations between persistent entities.

Without question, the most common cause of poorly-performing data access code in Java programs is the problem of N+1 selects. Here, a list of N rows is retrieved from the database in an initial query, and then associated instances of a related entity are fetched using N subsequent queries.

> Hibernate code which does this is bad code and makes Hibernate look bad to people who don't realize that it's their own fault for not following the advice in this section!

Hibernate provides several strategies for efficiently fetching associations and avoiding N+1 selects:

- Outer join fetching,
- Batch fetching, and
- Subselect fetching.

Of these, you should almost always use outer join fetching. Batch fetching and subselect fetching are only useful in rare cases where outer join fetching would result in a cartesian product and a huge result set. Unfortunately, outer join fetching simply isn't possible with lazy fetching.

> Avoid the use of lazy fetching, which is often the source of N+1 selects.

It follows from this tip that you shouldn't need to use `Stage.fetch()` or `Mutiny.fetch()` very often!

Now, we're not saying that associations should be mapped for eager fetching by default! That would be a terrible idea, resulting in simple session operations that fetch the entire database! Therefore:

> Most associations should be mapped for lazy fetching by default.

It sounds as if this tip is in contradiction to the previous one, but it's not. It's saying that you must explicitly specify eager fetching for associations precisely when and where they are needed.

If you need eager fetching in some particular transaction, use:

- `left join fetch` in HQL,
- A fetch profile,
- A JPA `EntityGraph`, or
- `fetch()` in a criteria query.

You can find much more information about association fetching in the documentation for Hibernate ORM.

### 1.5.4. Enabling the second-level cache

A classic way to reduce the number of accesses to the database is to use a second-level cache, allowing cached data to be shared between sessions.

Hibernate Reactive supports second-level cache implementations that perform no blocking I/O.

> Make sure you disable any disk-based storage or distributed replication used by your preferred cache implementation. A second-level cache which uses blocking I/O to interact with the network or disk-based storage will at least partially negate the advantages of the reactive programming model.

Configuring Hibernate's second-level cache is a rather involved topic, and quite outside the scope of this document. But in case it helps, we're testing Hibernate Reactive with the following configuration, which uses EHCache as the cache implementation, as above in Optional dependencies:

| Configuration property name | Property value |
|----------------------------|----------------|
| `hibernate.cache.use_second_level_cache` | `true` |
| `hibernate.cache.region.factory_class` | `org.hibernate.cache.jcache.JCacheRegionFactory` |
| `hibernate.javax.cache.provider` | `org.ehcache.jsr107.EhcacheCachingProvider` |
| `hibernate.javax.cache.uri` | `/ehcache.xml` |

If you're using EHCache, you'll also need to include an `ehcache.xml` file that explicitly configures the behavior of each cache region belonging to your entities and collections.

> Don't forget that you need to explicitly mark each entity that will be stored in the second-level cache with the `@Cache` annotation from `org.hibernate.annotations`.

You can find much more information about the second-level cache in the documentation for Hibernate ORM.

### 1.5.5. Session cache management

Entity instances aren't automatically evicted from the session cache when they're no longer needed. (The session cache is quite different to the second-level cache in this respect!) Instead, they stay pinned in memory until the session they belong to is discarded by your program.

The methods `detach()` and `clear()` allow you to remove entities from the session cache, making them available for garbage collection. Since most sessions are rather short-lived, you won't need these operations very often. And if you find yourself thinking you do need them in a certain situation, you should strongly consider an alternative solution: a stateless session.

### 1.5.6. Stateless sessions

An arguably-underappreciated feature of Hibernate is the `StatelessSession` interface, which provides a command-oriented, more bare-metal approach to interacting with the database.

You may obtain a reactive stateless session from the `SessionFactory`:

```java
Stage.StatelessSession ss = getSessionFactory().openStatelessSession();
```

A stateless session:

- Doesn't have a first-level cache (persistence context), nor does it interact with any second-level caches, and
- Doesn't implement transactional write-behind or automatic dirty checking, so all operations are executed immediately when they're explicitly called.

For a stateless session, you're always working with detached objects. Thus, the programming model is a bit different:

| Method name and parameters | Effect |
|----------------------------|--------|
| `get(Class, Object)` | Obtain a detached object, given its type and its id, by executing a select |
| `fetch(Object)` | Fetch an association of a detached object |
| `refresh(Object)` | Refresh the state of a detached object by executing a select |
| `insert(Object)` | Immediately insert the state of the given transient object into the database |
| `update(Object)` | Immediately update the state of the given detached object in the database |
| `delete(Object)` | Immediately delete the state of the given detached object from the database |

> There's no `flush()` operation, and so `update()` is always explicit.

In certain circumstances, this makes stateless sessions easier to work with, but with the caveat that a stateless session is much more vulnerable to data aliasing effects, since it's easy to get two non-identical Java objects which both represent the same row of a database table.

> If you use `fetch()` in a stateless session, you can very easily obtain two objects representing the same database row!

In particular, the absence of a persistence context means that you can safely perform bulk-processing tasks without allocating huge quantities of memory. Use of a `StatelessSession` alleviates the need to call:

- `clear()` or `detach()` to perform first-level cache management, and
- `setCacheMode()` to bypass interaction with the second-level cache.

> Stateless sessions can be useful, but for bulk operations on huge datasets, Hibernate can't possibly compete with stored procedures!

When using a stateless session, you should be aware of the following additional limitations:

- Persistence operations never cascade to associated instances,
- Changes to `@ManyToMany` associations and `@ElementCollections` cannot be made persistent, and
- Operations performed via a stateless session bypass callbacks.

### 1.5.7. Optimistic and pessimistic locking

Finally, an aspect of behavior under load that we didn't mention above is row-level data contention. When many transactions try to read and update the same data, the program might become unresponsive with lock escalation, deadlocks, and lock acquisition timeout errors.

There's two basic approaches to data concurrency in Hibernate:

- Optimistic locking using `@Version` columns, and
- Database-level pessimistic locking using the SQL `for update` syntax (or equivalent).

In the Hibernate community it's much more common to use optimistic locking, and Hibernate makes that incredibly easy.

> Where possible, in a multiuser system, avoid holding a pessimistic lock across a user interaction. Indeed, the usual practice is to avoid having transactions that span user interactions. For multiuser systems, optimistic locking is king.

That said, there is also a place for pessimistic locks, which can sometimes reduce the probability of transaction rollbacks.

Therefore, the `find()`, `lock()`, and `refresh()` methods of the reactive session accept an optional `LockMode`. You can also specify a `LockMode` for a query. The lock mode can be used to request a pessimistic lock, or to customize the behavior of optimistic locking:

| LockMode type | Meaning |
|---------------|---------|
| `READ` | An optimistic lock obtained implicitly whenever an entity is read from the database using select |
| `OPTIMISTIC` | An optimistic lock obtained when an entity is read from the database, and verified using a select to check the version when the transaction completes |
| `OPTIMISTIC_FORCE_INCREMENT` | An optimistic lock obtained when an entity is read from the database, and enforced using an update to increment the version when the transaction completes |
| `WRITE` | A pessimistic lock obtained implicitly whenever an entity is written to the database using update or insert |
| `PESSIMISTIC_READ` | A pessimistic for share lock |
| `PESSIMISTIC_WRITE` | A pessimistic for update lock |
| `PESSIMISTIC_FORCE_INCREMENT` | A pessimistic lock enforced using an immediate update to increment the version |

## 1.6. Custom connection management and multitenancy

Hibernate Reactive supports custom management of reactive connections by letting you define your own implementation of `ReactiveConnectionPool`, or extend the built-in implementation `DefaultSqlClientPool`.

| Configuration property name | Value |
|----------------------------|-------|
| `hibernate.vertx.pool.class` | A class which implements `ReactiveConnectionPool` |

A common motivation for defining a custom pool is the need to support multitenancy. In a multitenant application, the database or database schema depends on the current tenant identifier. The easiest way to set this up in Hibernate Reactive is to extend `DefaultSqlClientPool` and override `getTenantPool(String tenantId)`.

For multitenancy, might also need to set the following configuration property defined by Hibernate ORM:

| Configuration property name | Value |
|----------------------------|-------|
| `hibernate.tenant_identifier_resolver` | (Optional) A class which implements `CurrentTenantIdentifierResolver` |

If you don't provide a `CurrentTenantIdentifierResolver`, you can specify the tenant id explicitly when you call `openSession()`, `withSession()`, or `withTransaction()`.

You don't need a custom pool if you're using discriminator-based multitenancy. In that case, all you need is to declare the `@TenantId` properties of your entities, just like in Hibernate ORM 6.