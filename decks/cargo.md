# How do you start a new Rust project named "hello_world"?

**Command:**

```bash 
cargo new hello_world
```

**What this does:**

* **Creates a project directory:**  Generates a new directory named "hello_world".
* **Sets up essential files:**  Includes:
    * `Cargo.toml`:  Project metadata and dependencies.
    * `src/main.rs`:  Your project's starting source code.
* **Initializes a Git repository:** (Optional, configurable with Cargo flags)

# How do you compile and execute a Rust project?

**Command:**

```bash
cargo run
```

**Actions:**

1. **Compiles:** If changes were made, Cargo first rebuilds your project.
2. **Runs:** Executes the resulting binary from the compiled code. 

**Additional Notes:**

* **Project Root:**  Run this command from the root directory of your Rust project (where `Cargo.toml` is located).
* **Build-Only:**  Use `cargo build` if you want to compile without immediate execution.

# How do you check a Rust project for errors without running it? 

**Command:**

```bash
cargo check
```

**Purpose:**

* **Fast Compilation:**  `cargo check` compiles the project to verify code correctness without generating an executable binary.
* **Error Detection:** Catches type errors, lifetime issues, unused code, and more.
* **Development Workflow:**  Ideal for rapid feedback loops during development.


