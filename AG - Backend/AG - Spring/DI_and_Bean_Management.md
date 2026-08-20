# Dependency Injection & Bean Management (Tier-1 Interview Notes)

**1. The Real-Life Analogy (The Main Essence)**

Sir, observe carefully! Imagine a massive Tier-1 Hospital (The **Spring IoC Container**). 
The Hospital manages different resources (These are **Spring Beans**): MRI Machines, Surgeons, and Syringes.

Now, how does the hospital manage these Beans?
- **Singleton Scope:** Does the hospital buy a new MRI machine for every single patient? No! They buy exactly **ONE** MRI machine and share it with everyone. 
- **Prototype Scope:** Do they share one syringe for 100 patients? Sir, that is a disaster! They use a **NEW** syringe for every single patient.

When a Surgeon (Dependent Bean) needs an MRI Machine (Dependency), the Hospital Manager (IoC Container) automatically wheels the MRI machine into the surgery room. The Surgeon does not go to the factory to build the machine! This is **Dependency Injection (DI)**!

Are you getting the point? Let's tear this apart!

---

**2. Detailed but Simple Explanation (The Unbroken Chain)**

How do we tell the Spring Container to manage these Beans? There are two golden ways:

1. **`@Component` (For your own code):** When you write a class (e.g., `Surgeon`), you put `@Component` on top of it. At startup, Spring scans your folders (Component Scanning), finds the annotation, and says, "Ah! I must create an object of this class and keep it in my Container!" 
   - *(Note: `@Service`, `@Repository`, and `@Controller` are just special child versions of `@Component`).*
2. **`@Bean` (For 3rd party code):** What if you need a `RestTemplate` or a `HikariDataSource` object? You cannot open the source code of the Spring library and type `@Component` inside it! 
   - Instead, you create a `@Configuration` class, write a method that returns the object, and put `@Bean` on the method. Spring calls this method, takes the returned object, and puts it in the Container.

**Bean Scopes (The Lifecycle):**
- **Singleton (Default):** Exactly 1 object is created per Spring Container.
- **Prototype:** A brand new object is created every time you ask for it (`@Autowired` or `getBean()`).
- **Request:** 1 object per HTTP Request (Web apps only).
- **Session:** 1 object per User Session (Web apps only).

---

**3. The Code (Practical Implementation)**

Sir, look at this pure architecture!

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Scope;

// --- APPROACH 1: @Component for OUR code ---
@Component
@Scope("prototype") // We want a new Syringe every time!
public class Syringe {
    public void inject() { System.out.println("Using a fresh syringe!"); }
}

@Component
// Default is Singleton! We don't need to specify @Scope("singleton")
public class Surgeon {
    private final Syringe syringe;

    // Constructor Injection!
    public Surgeon(Syringe syringe) { this.syringe = syringe; }
}

// --- APPROACH 2: @Bean for 3rd Party / Custom configuration ---
@Configuration
public class HospitalConfig {

    // We can't put @Component inside the third-party RestTemplate class, 
    // so we create it manually and give it to Spring using @Bean!
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

---

**4. Understanding Level Mistakes**

Where do junior developers fail completely?
**The `@ComponentScan` Disaster:** Freshers put `@Component` on a class, run the app, and get a massive `NoSuchBeanDefinitionException`. Why?
- Because Spring Boot only scans the package where your `@SpringBootApplication` main class is located, and its sub-packages!
- If your Main class is in `com.tier1.app` and your DAO class is in `com.tier1.dao` (a sibling package, not a sub-package), Spring is totally blind to it! 
- **The Fix:** You must manually tell Spring where to look using `@ComponentScan(basePackages = {"com.tier1.app", "com.tier1.dao"})`.

---

**5. Loopholes & Exceptions (Tier-1 Traps)**

If you go to a top company, they will ask you the **Singleton-Prototype Trap**. This is a 100% guaranteed, deadly interview question!

**The Trap:** 
Look at the code in Section 3. `Surgeon` is a **Singleton**. `Syringe` is a **Prototype**.
If 5 patients visit the Surgeon, how many `Syringe` objects are created?
- **Freshers will say:** "Syringe is Prototype! So 5 Syringes are created!" **WRONG!**
- **The Tier-1 Answer:** Only **ONE** Syringe is created! 
Why? Because `Surgeon` is a Singleton. It is created EXACTLY ONCE when the application starts. During that one creation, Spring resolves its dependencies and injects ONE `Syringe` into the constructor. After that, Spring never calls the constructor again! Even though `Syringe` is a prototype, it is permanently trapped inside the Singleton Surgeon! All 5 patients will get injected with the exact same dirty syringe!

**The Tier-1 Fix:** How do we force the Singleton to get a fresh Prototype every time?
1. **ObjectFactory / Provider:** Inject `ObjectFactory<Syringe>` instead of `Syringe`. Then call `syringeFactory.getObject()` every time you need a fresh one.
2. **`@Lookup` Method Injection:** Tell Spring to override a method at runtime to fetch a fresh bean. (Amazon loves this answer).

---

**6. Key Takeaways**

1. `@Component` is for auto-detecting your own classes. `@Bean` is for manually adding 3rd party classes to the Container.
2. The default Bean Scope is **Singleton** (One instance per application).
3. Use **Prototype** scope when you need a completely fresh, unshared object every time.
4. **NEVER** inject a Prototype Bean directly into a Singleton Bean, or the Prototype will act like a Singleton! Always use `ObjectFactory` or `@Lookup` to solve this trap.

Sir, with this, your Spring Bean management concepts are absolutely invincible! There is nothing outside of this!
