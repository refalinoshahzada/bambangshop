# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Do we need an interface (or trait in Rust) in BambangShop, or is a single Model struct enough?

In the Observer pattern, an interface (or a trait in Rust) establishes a contract that all subscribers must adhere to, enabling different implementations to coexist. Rust traits facilitate polymorphic behavior and ensure that all observers implement essential methods, such as `update`, as seen in the Publisher-1 example.  

In the case of BambangShop, we currently define a `Subscriber` struct without a trait. This approach works as long as all subscribers share the same structure and behavior. However, if we later introduce multiple subscriber types—such as email-based or webhook-based subscribers—a trait would be useful to enforce a common behavior, like:  

```rust
trait Subscriber {
    fn notify(&self, notification: &Notification);
}
```

Since we only have one `Subscriber` struct for now, a trait isn't strictly necessary, but adding one would enhance flexibility.

2. Is using Vec (list) sufficient, or is DashMap necessary?

Since the `id` in `Program` and the `url` in `Subscriber` must be unique, a key-value storage structure like a **hashmap/dictionary** is ideal.  

### **a. Limitations of Using `Vec<Subscriber>`:**  
- Removing a subscriber requires **iterating** through the list.  
- If the number of subscribers grows too large, this approach becomes inefficient.  

### **b. Advantages of Using `DashMap<String, Subscriber>` (Map/Dictionary):**  
- Since dictionary keys must be **unique**, `url` uniqueness is naturally enforced.  

Given our need for efficient lookups, **DashMap is the better choice** for managing subscribers.

3. Do we need DashMap, or can we implement Singleton instead?

The Singleton pattern guarantees a single global instance, which we already accomplish using `lazy_static!`:

```rust
lazy_static! {
    pub static ref SUBSCRIBERS: DashMap<String, Subscriber> = DashMap::new();
}
```

However, Singleton by itself does not provide thread safety. `DashMap` in Rust is specifically built for concurrent access, preventing data races without the need for manual locking.

- If we replace `DashMap` with `HashMap` and use Singleton, we would need to wrap it in a `Mutex` or `RwLock` to ensure thread safety, which introduces locking overhead.

Since `DashMap` offers both Singleton behavior and thread safety, it is the optimal choice.

In conclusion, we should keep `DashMap` rather than replacing it with just a Singleton.

#### Reflection Publisher-2

1. **Why should we separate the "Service" and "Repository" layers from the Model?**  

In the MVC pattern, the Model typically handles both business logic and data access, but best practices suggest separating these concerns to improve code structure:  

- **Separation of Concerns (SoC)**:  
  - The Repository is responsible for database operations, such as querying and persisting data.  
  - The Service layer contains the core business logic and coordinates various operations.  
  - The Model simply defines the data structure without handling logic or database interactions.  

- **Scalability & Maintainability**:  
  - Keeping business logic and database access separate makes the code easier to modify, extend, and test.  
  - The Service layer enables business rules to evolve without affecting database queries.  
  - The Repository abstracts the database, making it easier to switch technologies (e.g., PostgreSQL to MongoDB).  

- **Testability**:  
  - The Service layer can be tested independently without requiring a database.  
  - The Repository can be mocked in tests to eliminate database dependencies.  

In **BambangShop**, the NotificationService and SubscriberRepository follow this layered approach, ensuring modularity and maintainability.  

2. **What are the drawbacks of relying only on the Model without separate Service and Repository layers?**  

If the Service and Repository layers are not used, the Model would be responsible for both business logic and database interactions. This can lead to several issues:  

- **Tightly Coupled Code**:  
  - Models like Notification, Subscriber, and Program would directly manage database interactions.  
  - Any change in business logic would require modifying the Model itself, making the system harder to maintain.  

- **Increased Dependencies Between Models**:  
  - Certain features, such as sending notifications upon subscribing/unsubscribing, would require direct interaction between models.  
  - This tight coupling makes refactoring and debugging more difficult.  

- **Limited Extensibility**:  
  - Adding new features like logging or validation would require modifying the Model, violating the **Single Responsibility Principle (SRP)**.  

Without Service and Repository layers, the system becomes more complex, harder to scale, and less maintainable.  

3. **How does Postman assist in testing our API?**  

Postman is a powerful tool for API testing that helps validate and debug requests in **BambangShop**.  

- **API Testing Capabilities**:  
  - It allows sending API requests to endpoints like:  
    - **Subscribe:** `/subscribe/<product_type>`  
    - **Unsubscribe:** `/unsubscribe/<product_type>?<url>`  
  - It verifies API responses, ensuring correct HTTP status codes and JSON payloads.  

- **Key Features**:  
  - **Collections & Environments**: Group API requests and switch between local and production setups.  
  - **Automated Testing**: Use scripts to validate responses automatically.  
  - **Mock Servers**: Simulate API responses to test frontend functionality without a backend.  
  - **API Documentation**: Generate interactive documentation to help team members understand endpoints.  

By leveraging Postman, developers can efficiently test and debug APIs, improving the development workflow.

#### Reflection Publisher-3

1. **Which type of Observer Pattern is implemented in our system?**  

In **BambangShop**, we use the **Push Model** of the Observer Pattern.  

- The **Publisher (NotificationService)** actively sends updates to **Subscribers**.  
- Whenever a product is created, deleted, or promoted, the `notify` method immediately dispatches notifications.  
- Subscribers receive updates via the `update` method, which asynchronously sends an HTTP POST request to their registered URL.  

This approach ensures that subscribers receive real-time updates as soon as the publisher decides to send them.  

2. **What would happen if we switched to the Pull Model instead?**  

If **BambangShop** adopted a **Pull Model**, subscribers would need to manually request updates instead of receiving them automatically. Here are the pros and cons:  

**Advantages of the Pull Model**  
- **Reduced Load on the Publisher**: Notifications are retrieved only when needed, avoiding unnecessary broadcasts.  
- **Better Handling of Offline Subscribers**: Since updates are fetched on demand, subscribers won’t miss notifications due to temporary unavailability.  
- **More Control for Subscribers**: Each subscriber can decide when to retrieve updates, reducing redundant notifications if real-time data isn’t necessary.  

**Disadvantages of the Pull Model (for our case)**  
- **Increased Complexity for Subscribers**: They must periodically check for updates, which adds additional processing overhead.  
- **Delayed Notifications**: Since subscribers fetch data at intervals (e.g., every 10 minutes), they won’t receive real-time updates.  
- **Higher Resource Consumption**: Frequent polling by multiple subscribers can lead to unnecessary system load.  

For **BambangShop**, where real-time notifications are essential for product events, the **Push Model** is more efficient. The Pull Model would introduce delays and unnecessary polling, making it unsuitable.  

3. **What are the consequences of removing multi-threading from the notification process?**  

Currently, **NotificationService::notify** utilizes **multi-threading (thread::spawn)** to send notifications asynchronously. If multi-threading were removed, the following issues would arise:  

- **Slower Notifications**: Updates would be sent sequentially, causing significant delays, especially with a large number of subscribers.  
- **Increased API Response Time**: The `create`, `delete`, and `publish` operations in **ProductService** would take longer, as the system would have to wait for all notifications to be sent before responding.  
- **Greater Risk of Failures**: If a single notification request fails or times out, all subsequent notifications would be blocked, reducing reliability.  

By using **multi-threading**, **BambangShop** ensures that notifications are sent efficiently and in parallel, preventing delays and improving scalability.