# Spring Data JPA — Complete Notes (My Version)

These are my **Spring Data JPA notes** written in a clean, exam/interview-friendly way. I’ve kept the explanations practical and added small clarifications where they help.

---

## 1) Spring Data JPA Overview (Big Picture)

**Spring Data JPA** is a Spring module that makes it easier to work with databases using **JPA** (Java Persistence API).

Here’s the typical flow:

```text
Application
   ↓
Spring Data JPA
   ↓
JPA (Specification)
   ↓
Hibernate (Implementation)
   ↓
JDBC
   ↓
Database
```

### Roles of each layer

| Layer | What it does (in simple words) |
|------|---------------------------------|
| Spring Data JPA | Gives me repositories, query helpers, less boilerplate |
| JPA | The ORM *spec* (rules/interfaces) |
| Hibernate | The common JPA provider that actually generates SQL |
| JDBC | Executes SQL against the database |
| Database | Stores the actual data |

---

## 2) ORM (Object Relational Mapping)

**ORM** means mapping my **Java objects** to **database tables**.

Basic mapping idea:

| Java | Database |
|------|----------|
| Class | Table |
| Object | Row |
| Field | Column |
| Reference | Foreign key |

Example entity:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    private Long id;

    private String name;
}
```

---

## 3) Important JPA Annotations (The Ones I Use Most)

### `@Entity`
Marks a class as a **JPA entity** (i.e., Hibernate will map it to a table).

```text
Java class → DB table
```

### `@Table`
Used when I want to customize table details (how to store in DB).

```java
@Table(name = "users")
```

### `@Id`
Marks the **primary key** field.

```java
@Id
private Long id;
```

### `@GeneratedValue`
Tells Hibernate to auto-generate the ID.

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Common strategies:

| Strategy | Meaning |
|---------|---------|
| `AUTO` | Provider decides the best approach |
| `IDENTITY` | Uses auto-increment column (common in MySQL) |
| `SEQUENCE` | Uses a DB sequence (common in PostgreSQL/Oracle) |
| `TABLE` | Uses a separate table to generate IDs |

### `@Column`
Configure a column (set properties in DB).

```java
@Column(nullable = false, unique = true)
private String email;
```

Properties of column:

- `nullable`
- `unique`
- `length`
- `updatable`
- `columnDefinition`
- `precision`
- `scale`

### `@Enumerated`
Controls how an enum is stored in DB.

```java
@Enumerated
private Status status;
```

Options:

- `ORDINAL` → stores index (0, 1, 2...) (risky if enum order changes)
- `STRING` → stores name ("ACTIVE", "INACTIVE") (**preferred**)

### `@CreationTimestamp`
Auto sets creation time when inserting.

```java
@CreationTimestamp
private LocalDateTime createdAt;
```

### `@UpdateTimestamp` 
Auto updates time when updating.

```java
@UpdateTimestamp
private LocalDateTime updatedAt;
```
---

## 4) Domain Model vs JPA Entity (Architecture Note)

In a clean architecture, try to keep **domain objects** separate from **persistence entities**.

### Domain Model
- Represents **business logic**
- Has business methods (like `addItem()`, validations, rules)
- Lives in the **domain layer**

Examples: `Order`, `Customer`, `OrderItem`

### JPA Entity
- Represents **database structure**
- Has ORM annotations like `@Entity`, `@Table`, `@Column`
- Lives in the **persistence layer**

### Key difference

| Feature | Domain Model | JPA Entity |
|--------|--------------|------------|
| Goal | Business behavior | DB persistence |
| Contains | Business rules/methods | ORM annotations/mapping |
| Layer | Domain | Persistence |

---

## 5) Cascade Types

**Cascade** means: when I perform an operation on the **parent**, Hibernate can automatically apply it to the **children**.

Example:

```java
@OneToMany(cascade = CascadeType.ALL)
private List<OrderItem> items;
```

Cascade options:

| Cascade | Meaning |
|--------|---------|
| `PERSIST` | Saving parent also saves children |
| `MERGE` | Updating/merging parent also merges children |
| `REMOVE` | Deleting parent also deletes children |
| `REFRESH` | Refresh parent → refresh children |
| `DETACH` | Detach parent → detach children |
| `ALL` | All of the above |

---

## 6) Fetch Types

Fetch type decides **when related entities are loaded**.

Two values:

- `LAZY`
- `EAGER`

### LAZY
Child data loads **only when I access it**.

```java
@OneToMany(fetch = FetchType.LAZY)
private List<OrderItem> items;
```

### EAGER
Child data loads **immediately with the parent**.

```java
@ManyToOne(fetch = FetchType.EAGER)
private Customer customer;
```

### Default fetch types (important)
| Relationship | Default |
|------------|---------|
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |
| `@ManyToOne` | EAGER |
| `@OneToOne` | EAGER |

---

## 7) Orphan Removal

`orphanRemoval = true`: if you remove a child from the parent collection (no reference to child), Hibernate deletes it from the database.

Example:

```java
@OneToMany(
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
private List<OrderItem> items;
```

If we do:

```java
order.getItems().remove(item);
```

Hibernate will execute something like:

```sql
DELETE FROM order_items ...
```

### Orphan Removal vs Cascade REMOVE

| Case | Cascade REMOVE | Orphan Removal |
|------|----------------|----------------|
| Parent deleted | child deleted | child deleted |
| Child removed from collection | **not deleted** | **deleted** |

---

## 8) Persistence Context (1st Level Cache)

The **persistence context** is basically Hibernate’s managed cache where entity objects live.

Also called:
- **First-level cache**

What it does:
- Stores managed entities
- Tracks entity changes (dirty checking)
- Syncs changes with DB on flush/commit

Flow:

```text
DB → persistence context → application
```

If an entity is already in the persistence context, Hibernate may **skip querying the DB again**.

---

## 9) Hibernate Entity Lifecycle (Entity States)

An entity can be in one of these states:

```text
Transient → Persistent → Detached → Removed
```

### Transient
Created in Java, not saved, not managed.

```java
User user = new User();
```

### Persistent (Managed)
Entity is tracked by Hibernate.

```java
entityManager.persist(user);
```

### Detached
Entity exists but is no longer managed.

```java
entityManager.detach(user);
```

Changes won’t be saved unless you merge it again.

### Removed
Scheduled for deletion.

```java
entityManager.remove(user);
```

---

## 10) JPQL (Java Persistence Query Language)

**JPQL** is an object-oriented query language.
It queries **entities**, not tables.

Example:

```text
SELECT u FROM User u
```

Basic structure:

```text
SELECT entity
FROM EntityName alias
WHERE condition
```

Example:

```java
@Query("SELECT u FROM User u WHERE u.age > 20")
List<User> findUsers();
```

Common JPQL clauses:

| Clause | Purpose |
|--------|---------|
| `SELECT` | choose fields/entities |
| `FROM` | entity source |
| `WHERE` | filter |
| `JOIN` | association join |
| `ORDER BY` | sorting |

Join example:

```text
SELECT o
FROM Order o
JOIN o.items i
WHERE i.price > 100
```

---

## 11) `@Transactional`

`@Transactional` manages DB transactions automatically.

Example:

```java
@Transactional
public void createOrder() {
    orderRepository.save(order);
}
```

Why use it:
- Atomicity: all-or-nothing
- Consistency
- Automatic rollback on failure (runtime exceptions by default)
- Prevents partial updates

### Without transaction (problem)
```text
save order
save items
error occurs
```

Result:
```text
order saved
items not saved
→ inconsistent DB
```

### With transaction
- All succeed → commit
- Any fails → rollback

---

## 12) Common Transaction / JPA Errors 

### `LazyInitializationException`
Happens when accessing a **LAZY-loaded relationship** after the session/transaction is closed.

Typical situation:

```text
transaction ends (session closed)
then try: order.getItems()
→ LazyInitializationException
```

Fix options (depending on design):
- Access lazy relations inside a transaction
- Use fetch join in queries where needed
- DTO projection (often best for APIs)

---

### `TransactionRequiredException`
Happens when you try update/delete operations without an active transaction.

Example:

```text
entityManager.remove(...)
without transaction
→ TransactionRequiredException
```
