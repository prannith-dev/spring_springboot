# Transaction Management (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! What is the absolute biggest nightmare in all of software engineering?

Imagine a Bank Transfer. Durga wants to transfer $500 to Ravi.
- **Step 1:** The code deducts $500 from Durga's account. (Success!)
- **Step 2:** Suddenly, the server loses power and crashes.
- **Step 3:** The code to add $500 to Ravi's account NEVER executes!

What just happened? $500 has completely vanished from the universe! The Bank is ruined!
**Transaction Management** is the protective bubble we place around these steps. It enforces a strict rule: **"All or Nothing"** (Atomicity). 
Either all 3 steps succeed, and the bubble turns into concrete (**Commit**), or if even a single step fails, the bubble bursts, and the database magically rewinds itself to exactly how it was before Step 1 (**Rollback**)!

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

To achieve this in Spring Boot, we use the magical **`@Transactional`** annotation. 
Because it is built on Spring AOP, the Proxy Bodyguard intercepts your method:
1. Proxy opens a Database Connection.
2. Proxy says `connection.setAutoCommit(false);` (Starting the bubble).
3. The Proxy executes your actual Business Logic.
4. If it finishes cleanly, Proxy says `connection.commit();`.
5. If it throws an exception, Proxy says `connection.rollback();`.

---

**3. The Code (The Ultimate Implementation)**

Sir, look at this perfect, production-grade Banking Service.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class BankTransferService {

    // The Tier-1 rule: ALWAYS specify rollbackFor = Exception.class!
    @Transactional(rollbackFor = Exception.class)
    public void transferMoney(Long fromAcc, Long toAcc, double amount) throws Exception {
        
        // 1. Deduct from Sender
        accountRepo.deductBalance(fromAcc, amount);
        
        // Let's pretend the server throws a random Checked Exception here!
        if (amount > 1000000) {
            throw new Exception("Suspicious Transfer Blocked!");
        }
        
        // 2. Add to Receiver
        accountRepo.addBalance(toAcc, amount);
        
        // If an exception is thrown, the Proxy automatically catches it and ROLLS BACK!
        // The sender gets their money back instantly!
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of freshers fail and destroy the entire bank?
**The Checked Exception Rollback Trap!**
In Java, there are two types of Exceptions: Unchecked (`RuntimeException`, `NullPointerException`) and Checked (`Exception`, `IOException`, `SQLException`).

- **The Disaster:** By default, Spring's `@Transactional` **ONLY rolls back for Unchecked Exceptions (RuntimeExceptions)**! 
- If your method throws a Checked Exception (e.g., you try to write a file and it throws `IOException`), the Proxy Bodyguard says: "Oh, this is a Checked Exception. I don't care!" and it completely **COMMITS** the half-finished transaction to the database! Durga loses his money, and Ravi gets nothing!
- **The Tier-1 Fix:** ALWAYS use **`@Transactional(rollbackFor = Exception.class)`** to force Spring to roll back for EVERY type of exception, both Checked and Unchecked!

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to a top fintech or Amazon interview, they will torture you with these two architectural traps:

**Trap 1: The `try-catch` Swallow Trap**
The interviewer asks: *"I put `@Transactional` on my method, but when the database crashed, it didn't roll back! Why?"*
- **The Code:**
  ```java
  @Transactional
  public void doWork() {
      try {
          db.save(data);
      } catch (Exception e) {
          System.out.println("Error happened"); 
          // They forgot to rethrow the exception!
      }
  }
  ```
- **The Tier-1 Answer:** AOP Proxies trigger rollbacks by *catching* the exception as it leaves the method! If you put a `try-catch` block inside the method and swallow the error (don't throw it outwards), the Proxy Bodyguard outside the door thinks the method executed perfectly successfully! It will COMMIT the broken transaction! 
- **The Fix:** If you must use `try-catch` inside a transaction, you MUST throw a `RuntimeException` inside the catch block so the Proxy can see it!

**Trap 2: Propagation Behaviors (`REQUIRED` vs `REQUIRES_NEW`)**
- Suppose `Method A` (Create Order) calls `Method B` (Save Audit Log).
- **`REQUIRED` (Default):** `Method B` joins the same bubble as `Method A`. If `Method A` fails later, the entire bubble pops. The Audit Log is erased! 
- **The Nightmare:** But wait! For security and compliance, the Audit Log MUST be saved, even if the Order fails! 
- **`REQUIRES_NEW` (The Fix):** If you put `@Transactional(propagation = Propagation.REQUIRES_NEW)` on `Method B`, it tells Spring: "Pause the main bubble. Create a completely brand new bubble just for the Audit Log. Commit it immediately. Then resume the main bubble."
- Now, even if the main transaction rolls back, the Audit Log is permanently saved to the database!

---

**6. Key Takeaways**

1. **`@Transactional`** ensures the ACID properties (Atomicity, Consistency, Isolation, Durability) by using AOP Proxies to auto-commit or auto-rollback.
2. By default, Spring ONLY rolls back on `RuntimeException`. Always use **`rollbackFor = Exception.class`** to prevent catastrophic partial commits.
3. NEVER swallow exceptions with empty `try-catch` blocks inside a transactional method. The Proxy won't see the error and will commit the data!
4. Master **Propagation**: Use `REQUIRED` (default) to join the same transaction, and `REQUIRES_NEW` to create an independent transaction that commits no matter what (perfect for Audit Logs).
5. Remember the **Self-Invocation Proxy Trap** from the AOP topic! Calling an `@Transactional` method from within the *same class* will completely bypass the transaction!

Sir, with this, the absolute deepest mechanics of Transaction Management are permanently printed in your brain! There is absolutely nothing outside of this!
