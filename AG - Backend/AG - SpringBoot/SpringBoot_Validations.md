# Spring Boot Entity Validations: Essential Notes

## 1. What is Entity Validation?
Entity Validation (using Jakarta/Hibernate Validator) is a framework that allows you to define strict rules for what data your application will accept. Instead of writing manual `if/else` statements, you define these rules by simply attaching tags (annotations) directly to your Java variables.

**Example Annotations:**
- `@NotBlank`: Ensures a String is not null and not just empty spaces.
- `@NotNull`: Ensures a value actually exists.
- `@Min(0)`: Ensures a number cannot be negative.
- `@Email`: Ensures a String is formatted like a real email address.

## 2. How it Differs from Custom Exceptions

It is critical to understand when to use Validations vs. Custom Exceptions. They serve two completely different purposes:

### Entity Validations (`@Valid`)
- **Purpose:** Input Validation (Checking the "Shape" of the data).
- **When it runs:** Immediately at the Controller (The Front Door).
- **Needs Database?** No.
- **Example:** "Did the user leave the train name blank?" or "Is the ticket price negative?"

### Custom Exceptions (`throw new TrainNotFoundException()`)
- **Purpose:** Business Logic Validation (Checking the "Meaning" of the data).
- **When it runs:** Inside the Service layer.
- **Needs Database?** Yes, almost always.
- **Example:** "Does this specific Train ID exist in my database?" or "Are there enough available seats left on this coach?"

## 3. How to Implement It
Applying validations requires three simple steps:
1. **Dependency:** Add `spring-boot-starter-validation` to your `pom.xml`.
2. **Entity Rules:** Place annotations (like `@Min` or `@NotBlank`) on your Entity variables (e.g., inside `Train.java`).
3. **Controller Trigger:** Add the `@Valid` annotation right before `@RequestBody` in your Controller methods. This acts as the trigger to enforce the rules.

## 4. Why Combining Both is the Standard Architecture

Using both Entity Validations (`@Valid`) and Custom Exceptions together is the standard architecture for modern enterprise applications. Here is a detailed breakdown of exactly why this combination is so incredibly useful:

### 1. Performance and Database Protection
Every time your application talks to the database, it consumes memory and time.

If a user sends a request to create a train but forgets to include the train's name (`""`), your application shouldn't waste time opening a database connection just to figure out the data is useless.
**How it helps:** Entity Validation (`@NotBlank`) acts as a firewall. It stops the bad request instantly at the Controller and rejects it. The Service layer and Database are completely protected from processing junk data, which keeps your server fast and reduces load.

### 2. Catching Multiple Errors at Once
When you write manual `if` statements (like `if (delay < 0) throw error;`), the code stops at the very first `if` statement that fails. If a user made 5 mistakes in their JSON request, they only see the first error. They fix it, hit send, and then get hit by the second error. This is very frustrating for users!

**How it helps:** Entity Validations evaluate the entire object at once. If the train name is blank, the price is negative, and the delay is negative, `@Valid` catches all three mistakes simultaneously. Your `GlobalExceptionHandler` can then return a single JSON array listing all 3 errors, allowing the frontend to highlight all the incorrect boxes in red at the same time.

### 3. Separation of Concerns (Clean Code)
A core rule in software engineering is that a class should only have one job (Single Responsibility Principle).

- The **Controller's job** is to receive HTTP requests and make sure the raw data is formatted correctly (Input Validation).
- The **Service's job** is to execute business rules and talk to the database (Business Logic Validation).

**How it helps:** If you put manual `if (price < 0)` checks inside your Service or Entity, you are mixing these jobs. By using `@Valid` annotations, you remove all the messy `if/else` validation blocks from your code. Your Java files become much shorter, cleaner, and easier for other developers to read.

### 4. Security
Hackers often try to break applications by sending massive amounts of text or weirdly formatted data into input fields to see if the server crashes.

**How it helps:** Annotations like `@Size(max = 50)` on a String or `@Min(0)` on a number automatically block malicious data payloads at the very outer edge of your application. The bad data is rejected before it can cause memory overflow issues or reach your database queries.

### Summary
By combining them, you create a two-layered defense system:

- **Layer 1 (The `@Valid` Firewall):** Instantly rejects badly shaped data, keeping your server fast and secure while giving the frontend a complete list of formatting mistakes.
- **Layer 2 (The Custom Exceptions):** Cleanly handles complex business rules (like checking if a Train ID exists in the database) without cluttering your code with complex error-handling logic.
