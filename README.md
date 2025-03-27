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
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

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

#### Reflection Publisher-3
