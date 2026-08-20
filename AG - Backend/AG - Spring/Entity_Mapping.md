# Entity Mapping (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! What exactly is Entity Mapping?

Imagine you are an Architect. You draw a beautiful blueprint of a house on a piece of paper (The **Java Class**). But the Builder (Hibernate) doesn't know how to translate your paper drawing into a real physical house (The **Database Table**). 
You must use strict architectural symbols (Annotations) to tell the Builder exactly what to do!
- You put a sticker saying "This is a concrete wall" (`@Column`).
- You put a sticker saying "This is the Master Key to the house" (`@Id`).
- You put a sticker on the decorative flower pot saying "Do not cement this to the floor! It is temporary!" (`@Transient`).

These stickers are the Entity Mapping annotations! Without them, Hibernate is completely blind.

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

To build a flawless `@Entity`, you must master these core JPA annotations:

1. **`@Entity`:** Mandatory. It tells JPA, "This class is a database blueprint."
2. **`@Table`:** Optional. Use it if your Java class is `Employee` but the database table is `tbl_emp`.
3. **`@Id`:** Mandatory. Marks the field as the Primary Key.
4. **`@GeneratedValue`:** Tells Hibernate how to generate the Primary Key automatically.
5. **`@Column`:** Customizes the DB column (name, length, unique, nullable).
6. **`@Transient`:** Tells Hibernate to completely IGNORE this field. Do not create a column for it. Do not save it!
7. **`@Enumerated`:** Tells Hibernate how to save Java Enums.
8. **`@Lob`:** Large Object. Used for massive byte arrays (Images/PDFs) or huge text strings (CLOB).

---

**3. The Code (Practical Implementation)**

Sir, look at this Masterpiece Entity! Every line is a potential interview question!

```java
import jakarta.persistence.*;

@Entity
@Table(name = "corporate_users")
public class User {

    // 1. The Primary Key
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // 2. Column Customization
    @Column(name = "email_address", unique = true, nullable = false, length = 100)
    private String email;

    // 3. Enum Mapping (CRITICAL TIER-1 TRAP HERE!)
    @Enumerated(EnumType.STRING)
    private UserRole role; // e.g., ADMIN, USER

    // 4. Large Objects
    @Lob
    private byte[] profilePicture;

    // 5. The Ignore Sticker!
    // Maybe we just use this to calculate age in Java, we don't want it in the DB!
    @Transient 
    private int temporarySessionCode;

    public User() {} // Mandatory Default Constructor!
    
    // Getters and Setters...
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of students fail completely?
**The `GenerationType.AUTO` Disaster!**
When generating Primary Keys, freshers always use `@GeneratedValue(strategy = GenerationType.AUTO)`. 
- What does `AUTO` mean? It means "Let Hibernate decide what is best for the database."
- If you are using MySQL, what does Hibernate do? It doesn't use the standard MySQL Auto-Increment! Instead, it creates an ugly, massive global `hibernate_sequence` table to manually track IDs! It drastically slows down insertions!
- **The Fix:** If you are using MySQL or PostgreSQL, ALWAYS use **`GenerationType.IDENTITY`**. This tells Hibernate to back off and let the Database handle the auto-increment purely natively!

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to a top fintech company, they will torture you with these two deadly traps:

**Trap 1: The Enum `ORDINAL` Corruption Trap (The Database Killer)**
Suppose you have an Enum: `enum Role { ADMIN, USER }`.
If you just write `@Enumerated`, the JPA default is `EnumType.ORDINAL`. 
What happens? Hibernate saves the **Index Number** to the database! `ADMIN` is saved as `0`, `USER` is saved as `1`.

- **The Nightmare:** 6 months later, a new developer adds a new role at the beginning: `enum Role { SUPER_ADMIN, ADMIN, USER }`.
- What happens now? `ADMIN` is now index `1`! But the database is full of `0`s for admins! Instantly, all your existing Admins become Super Admins, and all Users become Admins! **Your entire company database is irreversibly corrupted!**
- **The Tier-1 Fix:** ALWAYS, ALWAYS use **`@Enumerated(EnumType.STRING)`**. This forces Hibernate to save the exact text `"ADMIN"`. It uses slightly more space, but it makes your database 100% immune to index shifts!

**Trap 2: `transient` vs `@Transient`**
What is the difference between the Java `transient` keyword and the JPA `@Transient` annotation?
- `transient` keyword: Tells Java Serialization to ignore this field when sending the object across a network.
- `@Transient` annotation: Tells the JPA/Hibernate engine to ignore this field when saving to the database. (They are completely different concepts!)

---

**6. Key Takeaways**

1. Use `@Column` to customize constraints like `unique`, `nullable`, and `length`.
2. Use `@Transient` to prevent a Java variable from becoming a database column.
3. For MySQL/Postgres, avoid `GenerationType.AUTO`. Strictly use **`GenerationType.IDENTITY`**.
4. **NEVER** save Enums using the default Ordinal type! Always use **`EnumType.STRING`** to prevent catastrophic data corruption.

Sir, with this, the entire blueprint of Entity Mapping is permanently printed in your brain! There is absolutely nothing outside of this!
