# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

1. We use `RwLock<Vec<Notification>>` because the notifications list is shared global state that can be accessed by multiple Rocket request handlers at the same time. When a new notification arrives, the application needs write access to push a new item into the `Vec`. When the user asks for the notification list, the application only needs read access to iterate over the existing items and format them. `RwLock` is a good fit here because it allows many readers to access the same data concurrently as long as no writer is active, while still guaranteeing exclusive access for writes. This matches the Receiver app's access pattern: reads can happen often, and writes are relatively small append operations. We do not use `Mutex` here because `Mutex` would also work for correctness, but it is more restrictive: even read-only operations would block each other since only one thread can hold the lock at a time. In other words, `RwLock` gives us the same safety with better concurrency for this read-heavy use case.

2. Rust does not allow us to freely mutate ordinary `static` variables the way Java can mutate static fields through static methods because Rust treats unsynchronized global mutation as unsafe. A mutable global can be accessed from many threads, and that easily creates data races, which Rust prevents at compile time. For that reason, Rust requires shared mutable state to use explicit interior mutability and synchronization primitives such as `RwLock`, `Mutex`, or thread-safe containers like `DashMap`. The `lazy_static` crate helps because it lets us initialize complex global values at runtime while still exposing them as safe static references. So the difference is that Java permits mutation and relies more on runtime discipline, while Rust forces us to make thread safety explicit in the type system before global data can be mutated.

#### Reflection Subscriber-2

1. Yes, I explored code outside the step-by-step tutorial, especially `src/lib.rs` in both the Receiver and Publisher apps. From the Receiver side, I learned that `src/lib.rs` is responsible for shared application configuration and helper types, not just startup boilerplate. The `AppConfig` struct is built with `dotenvy` and Rocket Figment, which means values such as `APP_INSTANCE_ROOT_URL`, `APP_PUBLISHER_ROOT_URL`, and `APP_INSTANCE_NAME` can be injected from environment variables without hardcoding them in the service layer. I also learned why `lazy_static` is used for `REQWEST_CLIENT` and `APP_CONFIG`: both are shared across requests and should only be initialized once. That exploration was useful during testing because it made it clear why we could run three Receiver instances by changing per-process environment variables and why the notification service could read the correct instance URL and name automatically.

2. After finishing the tutorial and testing with multiple Receiver instances on ports `8001`, `8002`, and `8003`, I found that the Observer pattern makes adding more subscribers quite easy. A new Receiver does not require changes to the Publisher's core product logic, it only needs to expose the same notification endpoint and register itself through subscribe. The Publisher then treats every subscriber uniformly and sends notifications to each registered endpoint. This loose coupling is the main advantage: the Publisher only knows that a subscriber can receive updates, not how many instances exist or how each one presents the messages. However, adding more than one Main app instance would not be as easy in the current design. The current Publisher stores subscriber data in local in-memory state, so if we spawn multiple Main instances, each one would have its own separate subscriber list and product state. That means a Receiver subscribed to one Main instance would not automatically be known by the others. To support multiple Main instances cleanly, we would need an external shared database, message broker, or another synchronization mechanism so the publishers share the same subscription and product event state.

3. I did not build a full automated Rust test suite for this tutorial, but I did perform my own manual smoke tests and lightweight Postman adjustments. I reused the imported Postman collection, duplicated the Receiver requests for ports `8002` and `8003`, and then manually tested the full flow: subscribe, create product, publish product, delete product, inspect notification messages, and unsubscribe one Receiver to verify that only the remaining subscribers received later notifications. I also created a small `test/subscribe-unsubscribe` branch in both repositories to keep that verification work isolated from the main feature branches. Even though this was not a full formal test suite, it was still very useful. For future group projects, Postman collections with clearer organization, variables, and a few automated assertions would be very helpful because they make endpoint behavior easier to demonstrate, easier to re-check after changes, and easier for teammates to understand without reading all of the backend code first.
