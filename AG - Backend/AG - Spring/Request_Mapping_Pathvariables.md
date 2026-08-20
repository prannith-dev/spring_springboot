# Request Mapping & Path Variables (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! In the last topic, we saw the `DispatcherServlet` (The Receptionist) guiding customers to the correct Employee (Controller). But how does the Receptionist know which cabin belongs to which employee? 

- **`@RequestMapping`:** This is the **Sign Board** on the employee's door! It tells the Receptionist, "If a customer asks for `/api/employees`, send them to this cabin!"
- **`@PathVariable`:** Imagine the customer walks in wearing a T-shirt with the number "101" printed on it. The employee looks at the shirt and instantly knows, "Ah, you are asking for Employee ID 101!" The data is baked directly into the URL path itself (`/api/employees/101`).
- **`@RequestParam`:** Imagine the customer walks in carrying a filled-out form or holding a slip of paper. The data is attached separately at the end of the URL (`/api/employees?id=101`). 

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

1. **`@RequestMapping`:** This annotation maps HTTP requests to specific classes or methods. 
   - If you put it on the **Class**, it becomes the "Base URL" for the entire class.
   - If you put it on a **Method**, it becomes the specific endpoint.
   - *Modern Shortcut:* Instead of writing `@RequestMapping(method = RequestMethod.GET)`, Spring gives us `@GetMapping`, `@PostMapping`, `@PutMapping`, etc.
2. **`@PathVariable`:** Used to extract values from the URI path. Ideal for RESTful APIs when identifying a specific resource (e.g., getting a specific user by ID).
3. **`@RequestParam`:** Used to extract values from the query string (e.g., `?name=durga&age=45`) or from HTML form data. Ideal for filtering, searching, or sorting.

---

**3. The Code (Practical Implementation)**

Sir, look at this pure REST API Controller. The routing is crystal clear!

```java
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/employees") // Class-level: Base URL for everything in this class!
public class EmployeeController {

    // 1. PathVariable Example
    // URL: GET http://localhost:8080/api/v1/employees/99
    @GetMapping("/{empId}")
    public String getEmployeeById(@PathVariable("empId") int id) {
        // Spring automatically extracts '99' from the URL and puts it into the 'id' variable!
        return "Fetching details for Employee ID: " + id;
    }

    // 2. RequestParam Example (Search/Filter)
    // URL: GET http://localhost:8080/api/v1/employees/search?department=IT&salary=50000
    @GetMapping("/search")
    public String searchEmployees(
            @RequestParam("department") String dept, 
            @RequestParam("salary") double minSalary) {
        
        return "Searching employees in " + dept + " with salary > " + minSalary;
    }

    // 3. Multiple Path Variables
    // URL: GET http://localhost:8080/api/v1/employees/99/projects/5
    @GetMapping("/{empId}/projects/{projectId}")
    public String getProjectForEmployee(
            @PathVariable int empId,       // If variable name exactly matches the {name}, 
            @PathVariable int projectId) { // you don't need the ("name") attribute inside the annotation!
        
        return "Employee " + empId + " is working on Project " + projectId;
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of students fail completely?
1. **Mixing up PathVariable and RequestParam:** 
   - If the URL is `/users/101`, and you write `@RequestParam int id`, your app will crash with `400 Bad Request` because there is no `?id=101`.
   - If the URL is `/users?id=101`, and you write `@PathVariable int id`, your app will crash with `404 Not Found` because the path `/users` does not match `/users/{id}`.
2. **Variable Name Mismatch:** If your URL path says `/{userId}` but your Java method says `@PathVariable int id`, Spring will panic! The name inside `{}` MUST perfectly match the Java variable name. If they are different, you MUST explicitly tell Spring: `@PathVariable("userId") int id`.

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you sit in an Amazon interview, they will torture you with these two deadly traps:

**Trap 1: The Primitive Wrapper Trap (`@RequestParam`)**
By default, if a client forgets to send a `@RequestParam` in the URL, Spring throws a massive `400 Bad Request` error! It is mandatory by default!
- **Freshers fix it by writing:** `@RequestParam(required = false) int age`.
- **The Disaster:** If the client doesn't send the age, Spring tries to assign `null` to the variable. But `int` is a primitive! It cannot hold `null`! Your app crashes with an `IllegalStateException`!
- **Tier-1 Solution:** ALWAYS use Wrapper Classes (`Integer`, `Double`) if `required = false`. `Integer age` can safely hold `null` without crashing. Or, provide a default value: `@RequestParam(defaultValue = "18") int age`.

**Trap 2: The Regex Path Restriction**
What if you have `@GetMapping("/{id}")` for users, but a hacker types `/api/users/DROP_TABLE`? Your Java code tries to parse "DROP_TABLE" as an `int` and crashes with `NumberFormatException`.
- **Tier-1 Solution:** Restrict the `@PathVariable` using Regex directly in the mapping!
- `@GetMapping("/{id:[0-9]+}")` -> Now, if the user types anything other than numbers, Spring immediately throws a `404 Not Found` before the code even executes!

---

**6. Key Takeaways**

1. `@RequestMapping` is the map that routes the URL to the exact Java method.
2. `@PathVariable` extracts data embedded *inside* the URI path (identifying resources).
3. `@RequestParam` extracts data from the query string after the `?` (filtering, sorting).
4. Beware of the `required = false` trap! Never use primitive data types if a parameter is optional. Use Wrapper classes or `defaultValue`.

Sir, with this, your routing foundation is absolutely unbreakable! There is nothing outside of this!
