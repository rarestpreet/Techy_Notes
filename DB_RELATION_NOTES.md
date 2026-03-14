# Relational Mapping Cheat Sheet (Spring JPA Examples) 

## Overview

Relational mapping define how database tables connect.

**Main relationship types**
- One-to-One
- One-to-Many
- Many-to-One
- Many-to-Many

**Core terms**
- **Owning side**: the side that **writes the relationship** (controls the FK / join table / join column). Usually the side with `@JoinColumn` or `@JoinTable`.
- **Inverse side**: mapped using `mappedBy`. It **does not update** the foreign key. Can still access relation (with extra queries).
- **Join column**: a foreign key column in a table (`..._id`).
- **Join table**: an intermediate table used to connect two tables (typical for many-to-many; sometimes created unintentionally).
- **Mapped by**: define column of owning side entity which handles mapping.

---

## 1) One-to-One

### Concept
Each row in **Table A** corresponds to exactly one row in **Table B**.

Example: `User <---> Profile`

---

### 1.1 Unidirectional One-to-One

Only `User` knows about `Profile`.

**Typical table structure**
```
user
----
id
name
profile_id (FK -> profile.id)

profile
-------
id
bio
```

**Code**

```java
public class Profile {
    private Long id;
    private String bio;
}
```

```java
public class User {
    private Long id;
    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id") // owning side -> has FK column
    private Profile profile;
}
```

---

### 1.2 Bidirectional One-to-One

Both entities reference each other.

**Owning side: `User`** (it has the FK / join column)

```java
public class User {
    private Long id;
    private String name;

    @OneToOne
    @JoinColumn(name = "profile_id")
    private Profile profile;
}
```

**Inverse side: `Profile`** (`mappedBy` points to the field name in `User`)

```java
public class Profile {
    private Long id;
    private String bio;

    @OneToOne(mappedBy = "profile")
    private User user;
}
```

**Rule of thumb:** keep it bidirectional only if you truly need navigation from both sides.

---

## 2) Many-to-One

### Concept
Many rows in **Table A** relate to **one row in Table B**.

Example: `Many Orders ---> One Customer`

**Table structure**
```
customer
--------
id
name

orders
------
id
customer_id (FK -> customer.id)
```

**Code**

```java
public class Customer {
    private Long id;
    private String name;
}
```

```java
public class Order {
    private Long id;

    @ManyToOne
    @JoinColumn(name = "customer_id") // owning side (FK lives here)
    private Customer customer;
}
```

---

## 3) One-to-Many

### Concept
One row in **Table A** relates to **many rows in Table B**.

Example: `Customer ---> Many Orders`

---

### 3.1 Unidirectional One-to-Many (Use with caution)

Only `Customer` knows about `Order`.

```java
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany
    private List<Order> orders;
}
```

⚠️ **Important:** With this mapping, JPA often creates an **extra join table** (because there is no FK column managed on the `Order` side).

---

### 3.2 Bidirectional One-to-Many (Recommended)

This is usually the best model: **`@ManyToOne` on the many-side** and `mappedBy` on the one-side.

**Owning side: `Order`**

```java
public class Order {
    private Long id;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

**Inverse side: `Customer`**

```java
public class Customer {
    private Long id;

    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<>();
}
```

**Why this is preferred**
- No unnecessary join table
- FK is stored naturally in `orders.customer_id`
- Clear ownership and update behavior

---

## 4) Many-to-Many

### Concept
Many rows in **Table A** relate to many rows in **Table B**.

Example: `Students <---> Courses`

**Table structure**
```
student
-------
id

course
------
id

student_course
--------------
student_id (FK -> student.id)
course_id   (FK -> course.id)
```

**Code**

```java
public class Student {
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"), //owning side
        inverseJoinColumns = @JoinColumn(name = "course_id") //inverse side
    )
    private List<Course> courses;
}
```

```java
public class Course {
    private Long id;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students;
}
```

**Practical note:** In real applications, many-to-many often needs extra columns (like `enrolledAt`, `status`, `grade`). In that case, model it as an **association entity** (e.g., `Enrollment`) using two `@ManyToOne` relations.

---

## 5) Key Rules (Very Important)

### Owning side
- Contains `@JoinColumn` or `@JoinTable`
- Responsible for writing relationship changes to DB
- **The `mappedBy` side never owns the FK**

### Inverse side (`mappedBy`)
- Defined like:
  ```java
  mappedBy = "fieldName"
  ```
- `fieldName` must match the **owning entity’s field** exactly
- Used to make relationships **bidirectional without duplicate columns/tables**

---

## 6) Best Practices (Recommended Defaults)

- Prefer **`@ManyToOne` + `@OneToMany(mappedBy=...)`** instead of unidirectional `@OneToMany`
- Avoid unnecessary join tables
- Keep relationships **unidirectional** unless you need navigation from both sides
- Choose the owning side intentionally (it impacts updates and schema)

---

## 7) Quick Summary Table

| Relationship  | FK / Join Location              | Extra Table Needed |
|---------------|----------------------------------|--------------------|
| One-to-One    | In owning entity (FK column)     | No                 |
| Many-to-One   | In “many” side (FK column)       | No                 |
| One-to-Many   | Usually in “many” side (FK)      | Sometimes (if uni) |
| Many-to-Many  | Join table                       | Yes                |

---

## Key Insight

JPA can fetch entity **A through B** *without extra tables* when:
- the **owning side** is correct, and
- the relationship uses a proper **join column** (foreign key) rather than an unintended join table.

---
