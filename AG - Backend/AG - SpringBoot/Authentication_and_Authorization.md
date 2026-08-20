# Authentication & Authorization (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! What is the exact difference between Authentication and Authorization? Freshers mix them up every single day!

Imagine you are traveling at the Airport.
- **Authentication (AuthN):** You arrive at the security gate. The officer looks at your face, and looks at your Passport. He is verifying: *Are you exactly who you claim to be?* (Checking Username & Password). If your passport is fake, you are kicked out of the airport! (`401 Unauthorized`).
- **Authorization (AuthZ):** You are successfully inside the airport (Authenticated). You walk up to the VIP First-Class Lounge. The guard looks at your boarding pass. It says "Economy Class". The guard stops you! You are a valid passenger, but you do NOT have the *Permission/Role* to enter this specific room! (`403 Forbidden`).

**Authentication = Identity. Authorization = Permissions.**
Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

**The Authentication Flow (How Spring proves your Identity):**
1. User sends Username/Password.
2. The Bouncer creates an **Authentication Token**.
3. It passes it to the **`AuthenticationManager`** (The Detective).
4. The Detective calls the **`UserDetailsService`**, which goes to your Database and fetches the real User record.
5. The Detective uses **`PasswordEncoder`** to compare the passwords.
6. If it matches, Spring places the User's identity into the **`SecurityContextHolder`** (A highly secure vault stored in the server's RAM for that specific request).

**The Authorization Flow (How Spring checks your Permissions):**
You can secure your application at two levels:
1. **URL Level:** Using `.requestMatchers("/admin").hasRole("ADMIN")` in the Security Config.
2. **Method Level:** Using the magical **`@PreAuthorize`** annotation directly on your Service methods!

---

**3. The Code (The Ultimate Implementation)**

Sir, look at this! If you want to connect Spring Security to your actual MySQL database, you MUST implement the `UserDetailsService` interface!

**1. The Custom User Details Service (Authentication)**
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        // 1. Fetch from Database
        UserEntity user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("User not found!"));

        // 2. Convert to Spring Security's User object!
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getEmail())
            .password(user.getPassword()) // Must be BCrypt hashed!
            .roles(user.getRole()) // e.g., "ADMIN" or "USER"
            .build();
    }
}
```

**2. Method-Level Security (Authorization)**
```java
@Service
public class SalaryService {

    // You MUST add @EnableMethodSecurity in your config for this to work!
    @PreAuthorize("hasRole('ADMIN')")
    public void updateSalary(Long employeeId, double newSalary) {
        // Only an ADMIN can ever execute this method!
        System.out.println("Salary updated!");
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of freshers fail completely?
**The 401 vs 403 Confusion!**
- If a client calls your API and forgets to send the JWT Token (or sends the wrong password), you MUST return **`401 Unauthorized`**. (AuthN failed).
- If a client logs in perfectly as a "USER", and tries to call the `updateSalary` endpoint (which is for ADMINs only), you MUST return **`403 Forbidden`**. (AuthZ failed).
- If you return a 401 when it should be a 403, frontend React developers will write code that violently kicks the user back to the login screen for no reason!

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to a top fintech or Amazon interview, they will torture you with these two absolute nightmare traps:

**Trap 1: The `ROLE_` Prefix Trap (The Silent 403 Forbidden)**
The interviewer asks: *"I saved the role 'ADMIN' in my database. I wrote `.hasRole("ADMIN")` in my config. I logged in successfully, but I get a 403 Forbidden! Why?"*

- **The Tier-1 Answer:** Spring Security has a heavily guarded secret! When you use `.hasRole("ADMIN")`, Spring secretly appends **`ROLE_`** to it behind your back! It looks for `"ROLE_ADMIN"`!
- Since your database only has `"ADMIN"`, they don't match! You are locked out!
- **The Fix:** You have two options:
  1. Save `"ROLE_ADMIN"` in your database instead of `"ADMIN"`.
  2. Stop using `hasRole()`. Use **`.hasAuthority("ADMIN")`** instead! `.hasAuthority` checks the exact string without appending any secret prefixes! 

**Trap 2: The `SecurityContextHolder` ThreadLocal Trap**
The interviewer asks: *"An Admin calls my API. Inside my Service layer, I spawn a new background Thread using `@Async` to send an email. The background Thread instantly crashes with an `AccessDeniedException` saying the user is not authenticated. Why?"*

- **The Nightmare:** `SecurityContextHolder` uses a Java **`ThreadLocal`** variable. This means the Authentication identity is glued like superglue to the **Main HTTP Thread**. 
- When you spawn a new background Thread, the superglue breaks! The background thread wakes up, checks the `SecurityContextHolder`, finds it completely EMPTY, and panics! It thinks an unauthenticated hacker is trying to run code!
- **The Tier-1 Fix:** You must manually copy the context to the new thread, or configure Spring Security to use `MODE_INHERITABLETHREADLOCAL` so that child threads automatically inherit the parent thread's security identity!

---

**6. Key Takeaways**

1. **Authentication (401)** verifies Identity. **Authorization (403)** verifies Permissions.
2. Implement **`UserDetailsService`** to connect Spring Security to your custom database tables.
3. Use **`@PreAuthorize`** to lock down specific Service methods with extreme precision.
4. Beware the secret **`ROLE_`** prefix trap! Use **`.hasAuthority()`** if you don't save prefixes in your database.
5. The **`SecurityContextHolder`** is strictly glued to a single thread. Background `@Async` threads will lose the security context unless explicitly configured!

Sir, with this, the absolute deepest mechanics of Authentication, Authorization, and the ThreadLocal background trap are permanently printed in your brain! There is absolutely nothing outside of this!
