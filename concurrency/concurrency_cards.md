---
Flashcard Deck: Rust Concurrency
---

## What is?

What is a `Mutex`?

---
A `Mutex` (short for mutual exclusion) acts like a padlock for your data. It guarantees that only one thread can access and modify a shared resource at a time, preventing chaotic **data races** and preserving data integrity. 

> Rust's Mutex type from the standard library provides this essential synchronization tool.

. . .

What is a `Cell`?

---

A `Cell` allows you to get and set values within an immutable type, even if you don't own the data. It tracks mutability at runtime.

. . .

What is a `RefCell`?

---

A `RefCell` is similar to `Cell`, but enforces borrowing rules at runtime.

> If you try to get both a mutable and immutable reference simultaneously, it will panic.

. . .

What is a `RwLock`?

---

"Reader-writer" lock.

> It's a more flexible version of `Mutex`, allowing multiple readers at once, or a single writer.

. . .

How does a `Mutex` work?

---

**Acquiring the Lock:** To use a Mutex, a thread must first acquire its lock. Think of it as obtaining the only key to the padlock.
**Exclusive Access:** Once a thread has the lock, it gains exclusive access to the protected data. Other threads have to wait their turn.
**Releasing the Lock:** When the thread is done modifying the data, it must release the lock, akin to placing the key back for the next thread to use.

> A thread tries to get the lock; if it succeeds, it has exclusive access. Crucially, if another thread tries to acquire the same lock while it's held by another, it will be blocked (put to sleep) until the lock is released.

. . .

Why use a `Mutex`?

---

**Preventing Data Races:** In multi-threaded programs, data races occur when multiple threads try to access and modify the same data simultaneously without coordination. Mutexes prevent this by ensuring only one thread operates on the data at a time.
**Enforcing Synchronization:** Mutexes establish order and control in concurrent programs, guaranteeing that shared resources are used in a predictable and safe manner.

> Data races are a nightmare in concurrent programming—they lead to bugs that are incredibly difficult to track down. Mutexes shield you from these by imposing structure on how shared data can be used.  Think of them like traffic lights regulating access to a busy intersection.

. . .

What is **Mutex Poisoning**?

---

**Panic While Holding a Lock:** If a thread panics (crashes) while holding a Mutex lock, the `Mutex` becomes poisoned.
**Poisoned Protection:** A poisoned `Mutex` prevents further attempts to acquire the lock, signaling that the protected data might be in an inconsistent or corrupt state.

> Mutex poisoning is a safety mechanism in Rust. Imagine a thread has the key, modifies data, but then suddenly crashes – the data could be left in a bad state.  Poisoning the mutex stops other threads from unknowingly messing with potentially corrupt data, helping to isolate the problem caused by the panic.

. . .

What is ```mpsc```?

---

**Multiple producer, single consumer.** 

> For scenarios where several threads send data to one receiver.


. . .

How do you create new threads in Rust?

---

You use the `std::thread::spawn` function to create a new thread. You give it a closure (a function-like block of code) that defines the work the thread will do.

```use std::thread;
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

Describe a static item in Rust.

---

* Has a constant initializer (value must be known at compile time).
* Exists for the entire duration of the program's execution (never dropped).
* Is ready to use even before the `main` function starts.

. . .

What is `std::sync::Arc` in Rust?

---

"Atomically reference counted" smart pointer.

> Enables thread-safe owneership of shared data by tracking how many references to the data exist.

. . .

Why are "shared" and "exclusive" more accurate terms than "immutable" and "mutable" when describing Rust references?

---

* **"Shared"** (`&T`) emphasises that multiple references to the same data can exist simultaneously.
* **"Exclusive"** (`&mut T`) highlights the guarantee that no other reference can access the data while the exclusive reference is in use, allowing modification.
