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

* `Cell` provides **interior mutability:** It allows you to modify the conents of a value even when the value itself is declared immutable.
* Use cases:
  * When ownership rules are inconvenient for simple mutations.
  * Within structs to enable mutability of specific fields.
 
 > **Important:** `Cell doesn't enforce Rust's usual borrowing rules, so it's up to you to ensure safety.

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

> `Cell` is useful for scenarios where you need flexibility, but understand the trade-off in terms of compile-time safety guarantees.

. . .

## What is a `RefCell`?

---

A `RefCell` is similar to `Cell`, but enforces borrowing rules at runtime.

> If you try to get both a mutable and immutable reference simultaneously, it will panic.

. . .

## What is a `RwLock`?

---

"Reader-writer" lock.

> It's a more flexible version of `Mutex`, allowing multiple readers at once, or a single writer.

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

## Why use a `Mutex`?

---

**Preventing Data Races:** In multi-threaded programs, data races occur when multiple threads try to access and modify the same data simultaneously without coordination. Mutexes prevent this by ensuring only one thread operates on the data at a time.
**Enforcing Synchronization:** Mutexes establish order and control in concurrent programs, guaranteeing that shared resources are used in a predictable and safe manner.

> Data races are a nightmare in concurrent programming—they lead to bugs that are incredibly difficult to track down. Mutexes shield you from these by imposing structure on how shared data can be used.  Think of them like traffic lights regulating access to a busy intersection.

. . .

## What is **Mutex Poisoning**?

---

**Panic While Holding a Lock:** If a thread panics (crashes) while holding a Mutex lock, the `Mutex` becomes poisoned.
**Poisoned Protection:** A poisoned `Mutex` prevents further attempts to acquire the lock, signaling that the protected data might be in an inconsistent or corrupt state.

> Mutex poisoning is a safety mechanism in Rust. Imagine a thread has the key, modifies data, but then suddenly crashes – the data could be left in a bad state.  Poisoning the mutex stops other threads from unknowingly messing with potentially corrupt data, helping to isolate the problem caused by the panic.

. . .

## What is ```mpsc```?

---

**Multiple producer, single consumer.** 

> For scenarios where several threads send data to one receiver.


. . .

## How do you create new threads in Rust?

---

You use the `std::thread::spawn` function to create a new thread. You give it a closure (a function-like block of code) that defines the work the thread will do.

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

. . .

## Describe a static item in Rust.

---

* Has a constant initializer (value must be known at compile time).
* Exists for the entire duration of the program's execution (never dropped).
* Is ready to use even before the `main` function starts.

. . .

## What is `std::sync::Arc` in Rust?

---

"Atomically reference counted" smart pointer.

> Enables thread-safe owneership of shared data by tracking how many references to the data exist.

. . .

## Why are "shared" and "exclusive" more accurate terms than "immutable" and "mutable" when describing Rust references?

---

* **"Shared"** (`&T`) emphasises that multiple references to the same data can exist simultaneously.
* **"Exclusive"** (`&mut T`) highlights the guarantee that no other reference can access the data while the exclusive reference is in use, allowing modification.

. . .

## What is a condition variable in Rust, and why would you use one?

---

A **condition variable** (Condvar) provides a mechanism for threads to signal each other when a particular condition becomes true.

> It's typically used in conjunction with a `mutex` to protect shared data.

**Use Cases:**
* Producer-consumer scenarios (one thread produces data, another consumes it).
* Implementing barriers (waiting for multiple threads to reach a point.)

```
use std::sync::{Arc, Mutex, Condvar};
use std::thread;

// Shared data between threads
let pair = Arc::new((Mutex::new(false), Condvar::new())); 
let pair2 = pair.clone();

thread::spawn(move || {
    let (lock, cvar) = &*pair2;
    let mut started = lock.lock().unwrap();
    *started = true;
    println!("Notifying waiting thread...");
    cvar.notify_one(); 
});

let (lock, cvar) = &*pair;
let mut started = lock.lock().unwrap();
while !*started {
    println!("Waiting for the start signal...");
    started = cvar.wait(started).unwrap(); 
}
println!("Received start signal!");
```

. . .

## Describe the mechanism of waiting and notfication with a condition variable (condvar).

---

* **Waiting:**
  1. A thread aquires a mutex lock.
  2. While the desired condition is not met, the thread calls `condvar.wait(mutex)`.
     * This atomically releases the mutex and blocks the thread.
* **Notification**
  1. Another thread acquires the mutex lock.
  2. It changes the shared data to signal the condition.
  3. It calls `condvar.notify_one()` (wakes up one waiting thread) or `condvar.notify_all()` (wakes up all waiting threads).

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
