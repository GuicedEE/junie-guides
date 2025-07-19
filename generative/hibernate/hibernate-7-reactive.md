# 📘 Topic Rules: Hibernate Reactive (Version 7)

These rules define correct usage patterns for Hibernate Reactive (version 7.x), focused on Vert.x 5 applications using JPMS modularity and avoiding traditional frameworks (e.g., Spring, Quarkus). It assumes usage of `Mutiny.SessionFactory`, `Mutiny.Session`, and `Uni`-based chaining.

---

## 🔖 General Principles

* Hibernate Reactive is **non-blocking** and built on top of **Vert.x Mutiny API**.
* `SessionFactory` must be managed as a singleton.
* All operations must use `Uni<T>` (or `Multi<T>` for streaming).
* Transactions must be explicitly controlled using `.withTransaction()`.
* Entities must be annotated with `@Entity`, and all mappings must be JPMS module-aware.
* Configuration is done via `hibernate.cfg.xml` or `persistence.xml` (with limitations).
* When using `persistence.xml`, only core JDBC properties like `jakarta.persistence.jdbc.url`, `jakarta.persistence.jdbc.user`, `jakarta.persistence.jdbc.password`, and Hibernate-specific properties (e.g., `hibernate.dialect`, `hibernate.show_sql`) are considered.

---

## 🧱 SessionFactory Initialization

```java
SessionFactory sessionFactory = Persistence
    .createEntityManagerFactory("my-unit")
    .unwrap(Mutiny.SessionFactory.class);
```

* Use `persistence.xml` with `jakarta.persistence.jdbc.url`, `user`, and `password` pointing to the Testcontainers-managed database.
* For modular builds, ensure class/module visibility using `--add-opens` or JPMS exports.

---

## 🥪 Transactions

### 🔹 Pattern: withTransaction

```java
sessionFactory.withTransaction((session, tx) -> {
    return session.persist(entity);
});
```

* This guarantees rollback on failure.
* Returns a `Uni<Void>` or `Uni<T>` depending on chained ops.

### 🔹 Pattern: Manual Control (Discouraged unless needed)

```java
sessionFactory.openSession()
  .flatMap(session -> session.beginTransaction()
    .flatMap(tx -> session.persist(entity)
      .flatMap(v -> tx.commit())
      .eventually(() -> session.close())
    )
  );
```

* Prefer `.withTransaction()` to ensure cleanup.

---

## 📥 CRUD Patterns

### 🔸 Create

```java
sessionFactory.withTransaction((session, tx) -> session.persist(new MyEntity(...)))
```

### 🔸 Read

```java
sessionFactory.withSession(session -> session.find(MyEntity.class, id))
```

### 🔸 Update

```java
sessionFactory.withTransaction((session, tx) -> session.merge(entity))
```

### 🔸 Delete

```java
sessionFactory.withTransaction((session, tx) -> session.remove(entity))
```

---

## 🧠 Utility Interface

```java
@FunctionalInterface
public interface HibernateTransactionalOperation<T> {
    Uni<T> apply(Mutiny.Session session, Mutiny.Transaction tx);
}

public class HibernateReactiveUtil {
    public static <T> Uni<T> runTransaction(SessionFactory sf, HibernateTransactionalOperation<T> op) {
        return sf.withTransaction(op);
    }
}
```

---

## 🥪 Testing with Testcontainers

* Use [Testcontainers](https://www.testcontainers.org/) to spin up PostgreSQL, MySQL, or MariaDB in isolation.
* Dynamically inject JDBC URL, username, and password into `persistence.xml` using system properties:

```java
PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
    .withDatabaseName("test")
    .withUsername("test")
    .withPassword("test");

postgres.start();

System.setProperty("jakarta.persistence.jdbc.url", postgres.getJdbcUrl());
System.setProperty("jakarta.persistence.jdbc.user", postgres.getUsername());
System.setProperty("jakarta.persistence.jdbc.password", postgres.getPassword());
```

* Then initialize the factory:

```java
SessionFactory sessionFactory = Persistence
    .createEntityManagerFactory("test-unit")
    .unwrap(Mutiny.SessionFactory.class);
```

* Avoid `H2` unless your tests are not DB-specific. Use production-dialect DBs for real behavior coverage.
* Use `.withTransaction(...)` in test setup and teardown.
* Avoid blocking calls (`.get()`, `sleep()`); use `Uni.await().indefinitely()` only for isolated tests.

---

## 🚫 Anti-Patterns

* ❌ Do not call `.toCompletableFuture().get()` — defeats non-blocking model
* ❌ Do not manage sessions or transactions manually without `.withTransaction()` unless required
* ❌ Do not share `Mutiny.Session` between threads or across requests
* ❌ Avoid JPA-native lazy loading proxies (prefer DTO pattern or use eager fetch joins)

---

## 🧵 Threading and Event Loop

* Hibernate Reactive schedules work on Vert.x context thread using Mutiny
* If integrating with Vert.x Event Bus, always return `Uni<T>` to ensure context preservation
* Use `.subscribe().with(...)` or `.subscribeAsCompletionStage()` only at the outermost layer

---

## 📌 Related Topics

* [Hibernate Reactive Docs](https://hibernate.org/reactive/)
* [Mutiny API Reference](https://smallrye.io/smallrye-mutiny/)
* [Testcontainers Java](https://www.testcontainers.org/modules/databases/)
* Vert.x Context-aware Logging & Error Handling
* JPMS Module Config for Hibernate Reactive

---

📌 Use these rules when building modular, reactive applications with Hibernate 7.x and Vert.x 5.
