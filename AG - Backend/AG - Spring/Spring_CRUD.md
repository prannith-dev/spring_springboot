# CRUD Operations using Spring JDBC (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! Every application in the world, whether it is Facebook, Amazon, or a tiny college project, revolves around just 4 letters: **CRUD**.
Imagine managing a contact list on your mobile phone:
- **C (Create):** Saving a new friend's phone number.
- **R (Read):** Searching your contacts to find your friend.
- **U (Update):** Changing your friend's phone number when they buy a new SIM.
- **D (Delete):** Removing your ex-friend's number permanently!

In Spring JDBC, all database interactions are just these 4 operations. The beauty of Spring JDBC is that for Create, Update, and Delete, we use exactly the **same** method!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

How does Spring JDBC map to CRUD?
1. **CREATE (`INSERT`):** We use `jdbcTemplate.update()`. It returns an `int` (number of rows inserted).
2. **READ (`SELECT`):** We use two different methods depending on what we want:
   - If we expect **Multiple Rows** (e.g., Get All Employees), we use `jdbcTemplate.query()` with a `RowMapper`. It returns a `List<Object>`.
   - If we expect exactly **One Row** (e.g., Get Employee by ID), we use `jdbcTemplate.queryForObject()`.
3. **UPDATE (`UPDATE`):** We use `jdbcTemplate.update()`. Returns an `int` (rows updated).
4. **DELETE (`DELETE`):** We use `jdbcTemplate.update()`. Returns an `int` (rows deleted).

Notice this perfectly! **C, U, and D** all change the database state, so Spring uses a single unified method: `update()`.

---

**3. The Code (Practical Implementation)**

Sir, look at this Tier-1 level code. I am using `NamedParameterJdbcTemplate` because in top companies, we never use `?` placeholders!

```java
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class EmployeeCrudDao {

    private final NamedParameterJdbcTemplate jdbcTemplate;

    public EmployeeCrudDao(NamedParameterJdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // 1. CREATE
    public int createEmployee(String name, double salary) {
        String sql = "INSERT INTO employees (name, salary) VALUES (:name, :salary)";
        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("name", name)
                .addValue("salary", salary);
        return jdbcTemplate.update(sql, params);
    }

    // 2. READ (Single Row)
    public Employee getEmployeeById(int id) {
        String sql = "SELECT * FROM employees WHERE id = :id";
        MapSqlParameterSource params = new MapSqlParameterSource("id", id);
        
        // Use queryForObject for a single result!
        return jdbcTemplate.queryForObject(sql, params, employeeRowMapper());
    }

    // 2. READ (Multiple Rows)
    public List<Employee> getAllEmployees() {
        String sql = "SELECT * FROM employees";
        // Use query for a list of results!
        return jdbcTemplate.query(sql, employeeRowMapper());
    }

    // 3. UPDATE
    public int updateSalary(int id, double newSalary) {
        String sql = "UPDATE employees SET salary = :salary WHERE id = :id";
        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("salary", newSalary)
                .addValue("id", id);
        return jdbcTemplate.update(sql, params);
    }

    // 4. DELETE
    public int deleteEmployee(int id) {
        String sql = "DELETE FROM employees WHERE id = :id";
        MapSqlParameterSource params = new MapSqlParameterSource("id", id);
        return jdbcTemplate.update(sql, params);
    }

    // Centralized RowMapper to avoid duplicate code!
    private RowMapper<Employee> employeeRowMapper() {
        return (rs, rowNum) -> {
            Employee emp = new Employee();
            emp.setId(rs.getInt("id"));
            emp.setName(rs.getString("name"));
            emp.setSalary(rs.getDouble("salary"));
            return emp;
        };
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of students fail and crash their applications?
**The `queryForObject` Exception Trap!** 
If you use `jdbcTemplate.queryForObject(sql, params, rowMapper)` to find an Employee by ID, and that ID does **NOT** exist in the database, what happens? Will it return `null`? 
**NO!** Sir, observe carefully! Spring throws an `EmptyResultDataAccessException` and your app crashes with a 500 Internal Server Error! 
- **The Fix:** You MUST wrap `queryForObject()` in a `try-catch` block for `EmptyResultDataAccessException` and manually return `null` or throw your own Custom Business Exception (e.g., `EmployeeNotFoundException`).

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to a Microsoft interview, they will ask you the **"Auto-Generated Key Trap"**.

- **The Trap:** When you `CREATE` an employee, the database auto-generates the Primary Key (ID). The `jdbcTemplate.update()` method only returns `1` (number of rows inserted). How do you get that newly generated ID back immediately in Java?
  - Freshers will say: "I will write another query: `SELECT MAX(id) FROM employees`." Sir, this is a fatal concurrency disaster! If 10 users insert at the same millisecond, you will fetch someone else's ID!
- **The Tier-1 Solution:** You MUST use Spring's **`KeyHolder`**. 

```java
// Advanced Tier-1 Code for CREATE to get the ID back!
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;

public int createAndReturnId(String name, double salary) {
    String sql = "INSERT INTO employees (name, salary) VALUES (:name, :salary)";
    MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("name", name)
            .addValue("salary", salary);
            
    // The magical KeyHolder!
    KeyHolder keyHolder = new GeneratedKeyHolder();
    
    // Pass the keyHolder to the update method
    jdbcTemplate.update(sql, params, keyHolder);
    
    // Retrieve the auto-generated ID safely!
    return keyHolder.getKey().intValue(); 
}
```

---

**6. Key Takeaways**

1. Spring JDBC massively simplifies CRUD by using `update()` for any query that modifies data (Insert, Update, Delete).
2. For Read operations, use `query()` for a `List`, and `queryForObject()` for a single row.
3. Always handle `EmptyResultDataAccessException` when using `queryForObject()`.
4. To safely retrieve an auto-generated Primary Key after an insert, NEVER use `SELECT MAX(id)`. Always use Spring's `KeyHolder`.

Sir, with this, your Spring JDBC CRUD operations are absolutely invincible!
