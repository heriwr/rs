<details>
    <summary>Modules</summary>

# Explain how Rust modules control code organization and visibility.

- **Rust modules:** Group code (functions, structs, etc.) into logical units for better organization.
- **Visibility control:** Items inside a module are private by default. Use `pub` to make them accessible in other parts of the program (`pub fn`, `pub struct`).

Code Example:

```
mod authentication { 
    pub fn login(username: &str, password: &str) -> bool { 
        // ...
    }
}

// In another file
use authentication::login; 
```

# What is a module?
A container for zero or more items, organizing code and potentially creating namespaces. Modules can nest arbitrarily.

# What is a module item?
A named module, surrounded by braces and prefixed with the mod keyword. Introduces a new module into the crate structure.

# How do modules interact with types?
Modules and types exist within the same namespace. You cannot declare a type (struct, enum, etc.) with the same name as an existing module in the same scope (and vice-versa).

# Can you use the unsafe keyword before the mod keyword?
Syntactically allowed, but semantically rejected. This is primarily to enable macros to manipulate syntax involving the unsafe keyword in relation to modules.

# How does Rust determine module filenames?
Module name mirrors the file name (plus ".rs" extension).
Module path mirrors directory structure.
## Example
```
crate::util::config would likely be found in util/config.rs
```

# Describe the alternate way to define a module's content.
Place module contents in a file named "mod.rs" within a directory named after the module.
Example: 
```
crate::util content could be in util/mod.rs.
```

> Can't mix this with a regular .rs file for the same module.

# Why is the path attribute used on modules?
To customize the file path used to load the module's contents, overriding default filename conventions.

# How does the path attribute's behavior change when used within inline modules?
mod-rs files: Paths are relative to the directory containing the mod-rs file.
non-mod-rs files: Paths are relative, starting with a directory named after the containing non-mod-rs file.

# Which built-in attributes are meaningful for modules?
Back:
* `cfg` (conditional compilation)
* `deprecated`
* `doc` (documentation comments)
* `lint` check attributes (like allow, warn, etc.)
* `path` (covered earlier)
* `no_implicit_prelude` (disables automatic use std::prelude::v1::*;)


</details>

<details>
    <summary>Use Statements</summary>

# How can I shorten long paths when referring to items in Rust code?
    
 Use the `use` declaration to create aliases or bring items directly into scope.
    
    - **Example:** `use std::collections::HashMap;`

# Why might I see a `use` declaration at the top of a Rust file?
    
    
- **Readability:** `use` statements clarify which external modules or items are being used, making the code easier to understand.
    - **Name Conflicts:** Prevent naming clashes when different modules contain items with the same name.


# Describe different ways to use the `use` keyword in Rust.
    
- **Back:**
    
    - **Aliasing:** `use std::io::Read as FileRead;`
    - **Namespace:** `use std::io::*` (brings all items from `std::io` into scope)
    - **Nested:** `use std::collections::{HashMap, BTreeSet};`

</details>

