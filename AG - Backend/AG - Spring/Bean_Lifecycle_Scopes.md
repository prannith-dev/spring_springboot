# Bean Lifecycle & Scopes (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! A Spring Bean is exactly like a Human Being. It has a strict life journey managed entirely by God (The **Spring IoC Container**).

1. **Birth (Instantiation):** The container uses the constructor to create the baby.
2. **Getting Dressed (Populate Properties/DI):** The container gives the baby clothes (injects the `@Autowired` dependencies).
3. **Coming of Age Ceremony (Initialization):** The baby is now ready for the world. You can write custom code here to open database connections or load files.
4. **Living Life:** The Bean serves thousands of user requests happily.
5. **Death (Destruction):** Tomcat shuts down. The container calls a death method to close database connections, and the Bean goes to the graveyard (Garbage Collection).

What about **Scopes**? Scopes define the *lifestyle* of the Bean.
- **Singleton:** Like the Sun. There is only ONE, and everyone shares it.
- **Prototype:** Like a paper cup. A brand new one is created for every single user, and then thrown away!

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

To crack a Tier-1 interview, you must know the exact chronological order of the Bean Lifecycle:

1. **Instantiation:** Spring calls the constructor (`new MyBean()`).
2. **Dependency Injection:** Spring injects all required dependencies.
3. **`postProcessBeforeInitialization`:** A secret hook where Spring allows `BeanPostProcessor`s to hack/modify your bean.
4. **Custom `init()` method:** Spring calls your custom initialization logic (e.g., `@PostConstruct`).
5. **`postProcessAfterInitialization`:** The secret hook where Spring creates AOP Proxies (for `@Transactional`, etc.).
6. **Bean is Ready!**
7. **Custom `destroy()` method:** When the container shuts down, Spring calls your cleanup code (e.g., `@PreDestroy`).

---

**3. The Code (Practical Implementation)**

Sir, how do we write custom code during the Birth (Init) and Death (Destroy) phases? There are 3 ways, but in modern Spring Boot, we strictly use the **Annotation Approach** (`@PostConstruct` and `@PreDestroy`).

```java
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.stereotype.Component;

@Component
public class DatabaseConnectionBean {

    // 1. Instantiation
    public DatabaseConnectionBean() {
        System.out.println("Step 1: Constructor called. Bean is born!");
    }

    // 2. Initialization (Coming of Age)
    // This runs EXACTLY ONCE after all dependencies are injected!
    @PostConstruct
    public void myInitMethod() {
        System.out.println("Step 2: @PostConstruct called. Opening DB Connection...");
    }

    // 3. Living Life
    public void serveUser() {
        System.out.println("Step 3: Serving the user request.");
    }

    // 4. Death (Destruction)
    // This runs EXACTLY ONCE when the Spring Application is shutting down!
    @PreDestroy
    public void myDestroyMethod() {
        System.out.println("Step 4: @PreDestroy called. Closing DB Connection gracefully...");
    }
}
```

---

**4. Understanding Level Mistakes**

Where do 90% of students fail completely?
**The Constructor vs `@PostConstruct` Trap!**
Freshers write database connection logic directly inside the Constructor. Sir, this is a disaster! 
Why? Because when the Constructor runs, **Dependency Injection has NOT happened yet!** 
If your database logic relies on an `@Autowired` environment variable or another bean, those variables will be `null` inside the constructor! Your app will crash with a massive `NullPointerException`!
- **The Fix:** ALWAYS put startup logic inside `@PostConstruct`. Spring guarantees that `@PostConstruct` will only be called *after* every single dependency is fully injected and ready to use!

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to an Amazon or Microsoft interview, they will torture you with this specific trap:

**The Prototype Destruction Trap (The Deadly Exception!)**
The interviewer will ask: "I have a Bean with `@Scope("prototype")`. I wrote a `@PreDestroy` method in it to close some files. Will it execute when the application shuts down?"

- **Freshers will say:** "Yes Sir! `@PreDestroy` runs for all beans before death!" **WRONG!**
- **The Tier-1 Answer:** **NO! It will NEVER execute!** 
Why? Sir, observe carefully! For a Prototype Bean, Spring creates it, injects dependencies, calls `@PostConstruct`, and hands it to the client. After that, **Spring completely washes its hands of the Prototype Bean!** Spring does not keep a record of Prototype beans in its internal cache. Therefore, when the container shuts down, Spring doesn't even know where the Prototype bean is! 
- **The Golden Rule:** Spring manages the complete lifecycle (birth to death) ONLY for Singletons. For Prototypes, Spring handles the birth, but **YOU** (the developer or the Java Garbage Collector) are responsible for the death and cleanup!

---

**6. Key Takeaways**

1. The correct order is: Constructor -> Dependency Injection -> `@PostConstruct` (Init) -> `@PreDestroy` (Destroy).
2. Never write startup logic in the constructor if it relies on injected dependencies. Use `@PostConstruct`.
3. The **Singleton** scope is the default and is created once per container.
4. The **Prototype** scope creates a new instance every time, and strictly **ignores** `@PreDestroy` methods!
5. `BeanPostProcessor` is the secret engine Spring uses to hack beans before and after initialization (used to build AOP Proxies).

Sir, with this, the complete lifespan of a Spring Bean is exposed! There is nothing outside of this!
