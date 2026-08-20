# CRUD APIs with JPA (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! Building a CRUD API is exactly like building a Bank Teller system.

1. **Create (`POST`):** A customer walks in with cash. The Teller opens a new account.
2. **Read (`GET`):** A customer asks for their balance. The Teller looks up the ledger.
3. **Update (`PUT`/`PATCH`):** A customer moves to a new city. The Teller updates their address.
4. **Delete (`DELETE`):** A customer closes their account permanently.

The Customer is the **Frontend App (React/Postman)**. The Teller is your **`@RestController`**. The Vault Manager is your **`JpaRepository`**. The entire flow is perfectly standardized!

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

To build a flawless Tier-1 CRUD API, you must strictly follow the HTTP rules:
- **`@PostMapping`**: Used ONLY for creating new records. Returns `201 Created`.
- **`@GetMapping`**: Used ONLY for reading data. Must never modify the database! Returns `200 OK`.
- **`@PutMapping`**: Replaces the *entire* object. Returns `200 OK`.
- **`@PatchMapping`**: Updates only *specific fields* of an object.
- **`@DeleteMapping`**: Deletes a record. Returns `204 No Content` or `200 OK`.

The Unbroken Chain: `Request` -> `@RestController` -> `@Service` (Business Rules & Validation) -> `JpaRepository` -> Database.

---

**3. The Code (Practical Implementation)**

Sir, look at this perfect, production-grade Service and Controller architecture!

**The Service Layer (The Brains):**
```java
@Service
public class EmployeeService {

    @Autowired
    private EmployeeRepository repository;

    // 1. CREATE
    public Employee createEmployee(Employee emp) {
        return repository.save(emp); 
    }

    // 2. READ
    public Employee getEmployeeById(Long id) {
        return repository.findById(id)
            .orElseThrow(() -> new RuntimeException("Employee with ID " + id + " not found!"));
    }

    // 3. UPDATE
    public Employee updateEmployee(Long id, Employee updatedData) {
        Employee existingEmp = getEmployeeById(id); // Fetch the existing one first!
        
        existingEmp.setName(updatedData.getName());
        existingEmp.setSalary(updatedData.getSalary());
        
        return repository.save(existingEmp); // Save the merged object!
    }

    // 4. DELETE
    public void deleteEmployee(Long id) {
        if (!repository.existsById(id)) {
            throw new RuntimeException("Cannot delete! Employee ID " + id + " does not exist!");
        }
        repository.deleteById(id);
    }
}
```

**The Controller Layer (The Teller):**
```java
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    @Autowired
    private EmployeeService service;

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED) // Return 201 Created!
    public Employee create(@RequestBody Employee emp) {
        return service.createEmployee(emp);
    }

    @GetMapping("/{id}")
    public Employee get(@PathVariable Long id) {
        return service.getEmployeeById(id);
    }

    @PutMapping("/{id}")
    public Employee update(@PathVariable Long id, @RequestBody Employee emp) {
        return service.updateEmployee(id, emp);
    }

    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {
        service.deleteEmployee(id);
        return "Employee Deleted Successfully!";
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of developers fail completely?
**The `PUT` vs `PATCH` Disaster!**
- **`PUT`** means "Replace Completely". If an Employee has Name, Age, and Email, and the client sends a PUT request with just `{ "name": "New Name" }`, a lazy developer will blindly overwrite the database record. The Age and Email will instantly become `null` in the database! Data is destroyed!
- **`PATCH`** means "Partial Update". It is much harder to implement in Java because you have to use Reflection or write manual `if(field != null)` checks to only update the fields the client actually sent.
- **The Tier-1 Rule:** If a client uses `PUT`, they MUST send the entire object payload!

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you sit in a top fintech or Amazon interview, they will torture you with these two deadly traps:

**Trap 1: The Missing `update()` Method Trap**
The interviewer asks: *"I see `repository.save()`, `findById()`, and `deleteById()`. Where is `repository.update()` in Spring Data JPA?"*

- **Freshers will say:** "I don't know, maybe I have to write a custom `@Query`." **WRONG!**
- **The Tier-1 Answer:** There is NO `update()` method in JPA! You use **`save()`** for BOTH Insert and Update!
- **The Magic:** How does `.save()` know whether to Insert or Update? 
  - If the `Employee` object you pass to `save()` has a `null` Primary Key, JPA says: "Ah, this is a brand new baby!" and fires an `INSERT` query.
  - If the `Employee` object has a Primary Key (e.g., `id = 5`), JPA says: "Wait! Let me check the database." It executes a `SELECT` query. If ID 5 exists, JPA fires an `UPDATE` query! This is called **Save-or-Update** magic!

**Trap 2: The Blind Delete Crash (`EmptyResultDataAccessException`)**
Look at the `deleteEmployee()` method in the Service code above. Why did I write `repository.existsById(id)`?
- If you blindly write `repository.deleteById(99)` and ID 99 does not exist in the database, Spring will crash violently with an `EmptyResultDataAccessException`! 
- **Tier-1 Rule:** NEVER blindly delete! ALWAYS check if the ID exists first (`existsById`), OR catch the specific exception and translate it into a clean `404 Not Found` response for the API client!

---

**6. Key Takeaways**

1. A RESTful CRUD API must strictly map to HTTP Methods: POST (Create), GET (Read), PUT/PATCH (Update), DELETE (Delete).
2. Never put business logic in the `@RestController`. The Controller only manages HTTP traffic; the `@Service` handles the real logic.
3. JPA does not have an `update()` method. The **`save()`** method handles both Inserts (if ID is null) and Updates (if ID exists).
4. Beware of blindly deleting records. Always use **`existsById()`** to prevent ugly 500 Internal Server Error crashes when the record isn't found.

Sir, with this, the entire architecture of building bulletproof CRUD APIs with JPA is permanently printed in your brain! There is absolutely nothing outside of this!
