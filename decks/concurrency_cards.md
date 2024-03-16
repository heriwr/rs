---
Flashcard Deck: Rust Concurrency
---

## What is a `Mutex` and why is it important?

---
* A `Mutex` (short for mutual exclusion) is a synchronization mechanism that enforces exclusive access to shared data in multi-threaded environments.
* Importance:
  * Prevents **data races**: Ensures that only one thread can modify data at a time, maintaining consistency.
* Rust's standard library provides the `Mutex` type.

Example:

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Some shared data we need to protect
    let data = Arc::new(Mutex::new(0)); 

    let data_clone = data.clone(); // Need a clone for the thread

    thread::spawn(move || {
        let mut num = data_clone.lock().unwrap(); // Acquire the lock
        *num += 1; // Modify the data
    });

    // Main thread can also modify the data
    let mut num = data.lock().unwrap();
    *num *= 10;

    println!("Result: {}", num); 
}
```
1. `Arc<Mutex<i32>>`: We wrap our counter (0) in a Mutex for thread safety and then put that in an Arc so it can be shared between threads.
2. `data_clone`: We clone the Arc to pass into the new thread.
3. `lock().unwrap()`: Acquires the lock on the Mutex. .unwrap() handles potential errors here, but proper code would manage them gracefully.
4. **Modify Data**: Once the lock is acquired, we have exclusive access to the num inside.
5. **Lock Released**: The lock is automatically released when num goes out of scope at the end of the thread or main function block.
. . .

## What is a `Cell` in Rust, and why would you use it?

---

* `Cell` provides **interior mutability:** It allows you to modify the contents of a value even when the value itself is declared immutable.
* Use cases:
  * When ownership rules are inconvenient for simple mutations.
  * Within structs to enable mutability of specific fields.
 
 > **Important:** `Cell` doesn't enforce Rust's usual borrowing rules, so it's up to you to ensure safety.

**Example:**

```
use std::cell::Cell;

fn main() {
    // A counter that we want to increment, even though it's 'immutable'
    let counter = Cell::new(0); 

    // Increment the counter several times
    for _ in 0..5 {
        let current_value = counter.get();
        counter.set(current_value + 1);
    }

    println!("Final counter value: {}", counter.get());
}
```

**Explanation:**

1. `Cell::new(0)`: We create a new `Cell` containing the initial counter value of 0.
2. **Immutable** `counter`: Notice that the counter variable itself would normally be immutable if it weren't for `Cell`.
3. `counter.get()`: Retrieves the current value stored within the `Cell`.
4. `counter.set(...)`: Updates the value inside the `Cell`.

**Key Points:**

* Without `Cell`, you wouldn't be able to modify counter after its initial assignment.

> **Important:** `Cell` is useful for scenarios where you need flexibility, but understand the trade-off in terms of compile-time safety guarantees.

. . .

## How do `Mutex` and `Cell` in Rust differ in how they handle mutability?

---

* `Mutex`:
  * Enforces thread-safe exclusivity at compile time.
  * Only one thread can modify the data at a time (requires acquiring a lock).
  * Designed to prevent data races in multi-threaded scenarios.
* `Cell`:
  * Provides interior mutability at runtime.
  * Modifying contents doesn't require exclusive ownership.
  * Bypassess Rust's usual borrowing rules, putting safety responsibility on the programmer.

**Example:**

```
use std::sync::{Mutex, Arc};
use std::cell::Cell;
use std::thread;

// ---  Mutex Example ---
let counter = Arc::new(Mutex::new(0));

thread::spawn({
    let counter = counter.clone();
    let mut num = counter.lock().unwrap();
    *num += 1; 
});

// --- Cell Example ---
let counter = Cell::new(0);

thread::spawn(move || {
    let mut num = counter.get();
    num += 1;
    counter.set(num);
});
```

**Explanation:**

* **Mutex Example:**

1. `Arc<Mutex<i32>>`:  We use `Arc` to create a shared reference to a `Mutex` containing our counter (initially 0).

2. **Thread Creation:** We spawn a new thread with a closure that increments the counter.

3. `counter.clone()`:  The closure needs its own copy (clone) of the Arc to access the counter in the other thread.

4. `lock().unwrap()`: Inside the thread, the closure acquires the lock on the Mutex. This ensures exclusive access to the counter, preventing data races.

5. `*num += 1`:  With the lock held, the value inside the Mutex is incremented.

* **Cell Example**

1. `Cell::new(0)`: Here, we create a `Cell` directly containing the counter value (0).

2. **Thread Creation (Similar)**: We spawn a similar thread with a closure to modify the counter.

3. `let mut num = counter.get();`: The closure retrieves the current value from the `Cell` using `get()`.

4. `num += 1`:  The retrieved value is incremented outside of the `Cell`. `Cell` itself doesn't enforce exclusive access.

5. `counter.set(num);`: Finally, the incremented value is explicitly set back into the `Cell` using `set()`.

**Key Differences:**

* **Mutex:** Offers compile-time safety guarantees for thread-safe access. Requires acquiring a lock, potentially leading to overhead if there's high contention.
* **Cell:** Bypasses Rust's borrowing rules, allowing for mutation without exclusive ownership. The programmer is responsible for ensuring thread safety (e.g., using manual synchronization techniques if needed).
Choosing Between Mutex and Cell:

* **Use Mutex:** When data races are a concern in multi-threaded scenarios and you need strong guarantees about data consistency.
* **Use Cell:** For simple interior mutability within structs or when ownership rules are inconvenient for specific modifications, but be mindful of potential thread safety issues if not addressed carefully.
. . .

## What is a `RefCell` in Rust, and how does it differ from `Cell`?

---

* RefCell provides interior mutability with dynamic borrow checking: It allows you to modify data even within immutable references.
* Key difference from Cell: RefCell enforces Rust's borrowing rules (no simultaneous mutable and immutable references) at runtime, causing a panic if violated.
* Use cases: When you need mutability in situations where the compiler cannot guarantee safety at compile time.

. . .

## What is a `RwLock` and why would you choose it over a `Mutex`?

---

* `RwLock` (Reader-Writer Lock) provides finer-grained concurrency control:
  * **Multiple readers:** Can simultaneously access the shared data.
  * **Exclusive writer:** Only one writer can modify the data at a time.
* **Choose over `Mutex` when:**
  * You have mostly reads and infrequent writes.
  * Increased read performance is critical.

. . .

## What is the difference between a 'Send' and a 'Sync' Type?

---

* **Send:** A type is 'Send' if it's safe to transfer ownership between threads.
 > Primitive types like integers, as well as many standard library types, implement Send.
 
* **Sync:** A type is 'Sync' if it's safe to share a reference (&T) between threads.
 > Types composed entirely of 'Send' types are usually 'Sync'.

. . .

## How does a `Mutex` work?

---

**Acquiring the Lock:** To use a Mutex, a thread must first acquire its lock. Think of it as obtaining the only key to the padlock.
**Exclusive Access:** Once a thread has the lock, it gains exclusive access to the protected data. Other threads have to wait their turn.
**Releasing the Lock:** When the thread is done modifying the data, it must release the lock, akin to placing the key back for the next thread to use.

> A thread tries to get the lock; if it succeeds, it has exclusive access. Crucially, if another thread tries to acquire the same lock while it's held by another, it will be blocked (put to sleep) until the lock is released.

```
use std::sync::Mutex;
use std::thread;

let counter = Mutex::new(0);

let threads = (0..10).map(|_| {
    thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    })
}).collect::<Vec<_>>();

for thread in threads {
    thread.join().unwrap();
}

println!("Final counter value: {}", counter.lock().unwrap());
```

. . .

## What is **Mutex Poisoning** in Rust??

---

* **Cause:** Occurs when a thread holding a Mutex lock panics (crashes unexpectedly).
* **Effect:** The Mutex becomes poisoned, preventing other threads from acquiring it.
* **Purpose:** Safety mechanism to protect potentially corrupted data:
    * The panicking thread might have left the shared data in an inconsistent state.
    * Poisoning helps isolate the issue and prevents further corruption.

. . .

## What does `mpsc` stand for in Rust, and what problem does it solve?

---

* `mpsc`: Multiple Producer, Single Consumer
* **Solves:** Structured communication between threads where:
    * Several threads generate data.
    * A single thread processes the data in order.
* **Think of it like:** A conveyor belt feeding items into a single processing machine.

**Example:**

```
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    // Producer thread
    thread::spawn(move || {
        for i in 0..10 {
            tx.send(i).unwrap();
        }
    });

    // Consumer thread
    for received in rx {
        println!("Got: {}", received);
    }
}
```

**Code Breakdown:**

1. `mpsc::channel()`:

   * This line creates an mpsc channel using the mpsc::channel function from the standard library.
   * The channel consists of two parts:
        * tx (transmitter): This is used to send data to the channel.
        * rx (receiver): This is used to receive data from the channel.
2. **Producer Thread:**

    * We spawn a new thread using `thread::spawn`.
    * Inside the thread closure:
        * A loop iterates from 0 to 9.
        * In each iteration, the current value (`i`) is sent through the transmitter (`tx.send(i).unwrap()`) to the channel.
        * `.unwrap()` is used here for simplicity, but proper error handling (e.g., checking for potential send errors) would be necessary in a real-world application.
3. Consumer Thread:

    * The `main` thread acts as the consumer in this example.
    * A `for` loop iterates over the values received (`rx`) from the channel.
    * Each received value is printed using `println!`.

**Key Points:**

* **Multiple Producers:** The code demonstrates how multiple threads (the producer thread and potentially more) could send data through the same `tx` channel.
* **Single Consumer:** The `main` thread acts as the single consumer, receiving data in the order it was sent using the `rx` channel.
* **Order Preservation:** `mpsc` channels guarantee that messages are received in the same order they were sent.

. . .

## How do you create and manage new threads in Rust?

---

* **Creation:** Use the `std::thread::spawn` function, providing a closure containing the thread's code.
* **Management:**
    * The returned `JoinHandle` allows you to wait for the thread to finish.
    * `.join().unwrap()` blocks the current thread until the spawned thread completes.

```
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi from the spawned thread: {}", i);
            thread::sleep(Duration::from_millis(500));
        }
    });

    handle.join().unwrap(); // Wait for the thread to finish
}
```

**Explanation:**

1. `use std::thread; & use std::time::Duration;`:

    * These lines import necessary modules from the standard library:
        * `std::thread` provides functions for working with threads.
        * `std::time::Duration` allows us to define a time delay.
2. `fn main() { ... }`:

    * This defines the `main` function, the entry point of the program.
3. `let handle = thread::spawn(|| { ... });`:

    * Here, we create a new thread using `thread::spawn`.
    * The argument to `spawn` is a closure (anonymous function) that defines the code the new thread will execute.
    * The closure body contains a loop that prints a message ten times, with a half-second delay between each iteration using ``thread::sleep(Duration::from_millis(500))``.
    * The `spawn` function returns a `JoinHandle`, which represents the spawned thread. We store this handle in the `handle` variable.
4. `handle.join().unwrap();`:

    * This line calls the `join` method on the `JoinHandle`.
    * `join` waits for the spawned thread to finish its execution before continuing with the following code in the `main` thread.
    * `.unwrap()` is used here for brevity, but proper error handling (e.g., checking for potential errors during the join operation) would be necessary in real-world applications.

**Key Points:**

* This code demonstrates how to create a new thread using thread::spawn and how to wait for it to finish using handle.join().
* The JoinHandle allows us to manage the spawned thread.

> * **Error Handling:**  The example code uses `.unwrap()` for simplicity. However, in production Rust code, it's vital to handle potential errors that could occur when joining a thread. The `JoinHandle` might panic if the thread it represents has already panicked. Use `Result` and proper error handling techniques instead.

> * **Detached Threads:** Threads are not automatically joined when their `JoinHandle` goes out of scope.  If you don't explicitly `.join()`  a thread, it will continue running in the background, even after `main` or the parent thread exits.  For situations where you don't need to wait on the thread's completion, consider intentionally detaching it.

. . .

## What is a static item in Rust?

---

* **Key Features:**
    * **Global Constant:** a single value known at compile time
    * **Lifetime:** Exists for the entire program duration.
* **Use Cases:**
    * Fixed configuration values
    * Data shared between threads (use with synchronization mechanisms)

. . .

## What is `std::sync::Arc` and why use it in Rust?

---

* **Arc (Atomically Reference Counted):** A smart pointer for managing shared ownership in multi-threaded scenarios.
* **Key Feature:** Ensures thread-safe reference counting:
    * Tracks how many references exist to the shared data.
    * Automatically dealloctes the data when the last reference is dropped.
* **Use case:** When multiple threads need to access and potentially modify the same data safely.

. . .

## In Rust, why are "shared" and "exclusive" better descriptors for references than "immutable" and "mutable"?

---

**Focus on behavior:** These terms describe how references can be used, not just whether the underlying data itself might change.
**Shared (`&T`):**
    * Emphasizes the ability to have multiple simultaneous references.
    * Doesn't imply that the data itself cannot be changed (e.g., through interior mutability).
**Exclusive (`&mut T`):**
    * Underlines the guarantee of no other concurrent access, enabling safe modification.

. . .

## What is a condition variable (Condvar) in Rust, and why are they useful?

---

* **Purpose:** Allows threads to wait until a specific condition becomes true and receive notification when it does.
* **Key Points:**
    * Often Paired with Mutex: Protects the shared data related to the condition.
    * Avoids Busy Waiting: Threads can sleep efficiently until notified, rather than continuously checking a condition.
* **Typical Use Cases:**
    * Producer-consumer scenarios
    * Coordination between multiple threads

**Code Example:**

```
use std::sync::{Arc, Mutex, Condvar};
use std::thread;

let pair = Arc::new((Mutex::new(false), Condvar::new())); 
let pair2 = pair.clone();

thread::spawn(move || {
    let (lock, cvar) = &*pair2;
    let mut started = lock.lock().unwrap();
    *started = true; // Set the condition
    cvar.notify_one(); 
});

let (lock, cvar) = &*pair;
let mut started = lock.lock().unwrap();
while !*started { 
    started = cvar.wait(started).unwrap(); 
}
println!("Condition met!"); 
```

**Code Breakdown**

1. **Setup:**

    * `Arc<Mutex<bool>, Condvar>>`:
        * We create a shared data structure wrapped in an `Arc` for use across threads.
        * It contains a `Mutex` protecting a boolean flag (`started`) and a `Condvar`.
`pair2 = pair.clone()`: We clone the `Arc` to access the shared data in both threads.
2. **Producer Thread (`thread::spawn(...)`)**

    * `(lock, cvar) = &*pair2`: Obtains references to the Mutex and Condvar from the shared data structure.
    * `lock.lock().unwrap()`: Acquires the lock on the Mutex.
    * `*started = true`: Sets the boolean flag to true, indicating the condition is met.
    * `cvar.notify_one()`: Sends a signal through the Condvar, waking up a waiting thread if there is one.
3. **Consumer Thread (Main Execution)**

* `(lock, cvar) = &*pair`: Obtains references to the `Mutex` and `Condvar`.
* `lock.lock().unwrap()`: Acquires the lock on the `Mutex`.
* `while !*started { ... }`: Enters a loop that continues while the `started` flag is false.
* `cvar.wait(started).unwrap()`:
    * This is the core Condvar interaction; the thread releases the lock and waits for a signal.
    * When signaled, it re-acquires the lock and the loop re-checks the condition.
4. **Condition Met:**

    * Once the producer thread sets the condition, the consumer thread will be notified, exit the loop, and print "Condition met!".
  
**Key Points:**

> * **Synchronization:** The `Mutex` ensures that only one thread accesses and modifies the `started` flag at a time.
> * **Efficient Waiting:** The `Condvar` allows the consumer thread to sleep until the condition is signaled, avoiding wasteful busy-waiting.

. . .

## How do threads use a condition variable (condvar) to wait and signal in Rust?

---

**Waiting Thread:**

1. **Acquire Lock:** Obtain the `Mutex` associated with the condition.
2. **Check Condition:** If the condition isn't met yet:
    * **Wait and Release:** Call `condvar.wait(mutex)` to atomically release the lock and go to sleep.
**Signaling Thread:**

1. **Acquire Lock:** Obtain the same `Mutex`.
2. **Modify State:** Change the shared data so the condition becomes true.
3. **Notify:** Call `condvar.notify_one()` (to wake one waiter) or `condvar.notify_all()` (to wake all waiters).

. . .
 
## What is thread parking in Rust?

* Thread parking avoids "busy waiting", whee a thread continuously checks a condition in a loop, wasting CPU cycles.
* Instead, the thread parks itself until the condition is met and signaled by another thread, thus conserving resources.

. . .

## What is a deadlock in concurrent programming?

---

A deadlock occurs when two or more threads are stuck forever, each waiting for a resource held by the other. This leads to program freeze.

> Think of a traffic jam at a 4-way intersection where everyone's trying to go first, and no one can move.

. . .

## How does a livelock differ from a deadlock?

* **Livelock:** Threads are still active, constantly changing state in response to each other, but without any real progress.
* **Deadlock:** Threads are completely frozen, each blocked on the other's resource.

> Imagine two people constantly apologising and trying to let the other pass through a doorway - they move around, but no one gets through.

## Define atomicity in the context of concurrency.

---

Atomicity guarantees that a set of operations appears to happen either completely or not at all. Other threads cannot see partial or intermediate states of operations.

> Think of a database transaction; it should either fully succeed (e.g., transfer money) or fully fail, leaving the accounts unaltered.

## Name two synchronisation primitives beyond mutexes.

* **Semaphores:** Control access to a limited number of resources (like a fixed pool or parking spaces).
* **Conditional Variables:** Allow threads to wait until a specific condition becomes true (like waiting for a queue to have items before consuming).

## What is the advantage of lock-free or wait-free programming?

---

* **Performance:** In high contention scenarios, they can be faster than traditional locking techniques.
* **Avoidance of deadlocks/livelocks:** Carefully designed algorithms ensure progress.
* **Downside:** Increased complexity, making these techniques harder to implement correctly.
