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

</details>

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


