---
name: Rust Concurrency Flashcard Deck
---

What is a ```Mutex```?

---
A ```Mutex``` (short for mutual exclusion) acts like a padlock for your data. It guarantees that only one thread can access and modify a shared resource at a time, preventing chaotic **data races** and preserving data integrity. Rust's Mutex type from the standard library provides this essential synchronization tool.

. . .

How does a ```Mutex``` work?

---

...
...

Why use a ```Mutex```?

---



. . .

What is **Mutex Poisoning**?

---



. . .

What is ```mpsc```?

---

**Multiple producer, single consumer.** For scenarios where several threads send data to one receiver.
