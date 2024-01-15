# Rust study notes
Notes from [The Rust Programming Language](https://doc.rust-lang.org/stable/book/) book.

## Ownership Rules
- Each value in Rust has an *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

## Referecnces and Borrowing
- Borrowing is the action of creating a reference.
- References are immutable by default
- A *mutable reference* allows the borrower to modify a borrowed value
- The rules of references:
    - At any given time, you can have *either* one mutable reference *or* any number of immutable references
    - References must always be valid.
- Slice is am immutable reference to a contiguous sequence of elements in a collection.
- String literals has the type `&str` and is a slice pointing to the stack memory that stores the string value.

## Data types
- A *tuple* can have different data types for each of its elements and always has a fixed length once declared.
    - Despite having a fixed length, a tuple can be made mutable such that its elements can be changed as long as their data types are maintained.
    - Elements of a tuple can be accessed with a dot notation, such as `t.0`, `t.1`, etc.
- *unit* type: a tuple type with no fields `()` is often called *unit* or *unit type*. Its one value is also called *unit* or *the unit value*. ([ref](https://doc.rust-lang.org/reference/types/tuple.html?highlight=unit%20type#tuple-types))
- An *array* is like a tuple but all elements of an array have to be of the same data tuple.
    - Like a tuple, an array also has a fixed length once defined.
    - Like a tuple, an array can also be made mutable.
    - Unlike a tuple, an array can have repeated elements: `arr = [3; 5]` is equivalent to `arr = [3, 3, 3, 3, 3]`.
    - A multidimensional array: `[[1.0; 10]; 10]`
    - Invalid array element access is checked during runtime by Rust and will result in a runtime error.
- *enums* are defined by enumerating its possible *variants*. Each variant can have data attached and the name of each enum varient becomes a function that constructs an instance of the enum.
    ```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }
    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    ```
- The `Option<T>` enum is included in the prelude and doesn't need be brought into scope explicitly. Its variants are also included in the prelude: you can use `Some` and `None` directly without the `Option::` prefix.
    ```rust
    enum Option<T> {
        None,
        Some(T),
    }
    ```
    - `<T>` is a generic type parameter, which means that the `Some` variant of the `Option` enum can hold one piece of data of any type, and that each concrete type that gets used in place of `T` makes the overall `Option<T>` type a different type.
    - **In order to have a value that can possibly be null, you must explicitly opt in by making the type of that value `Option<T>`. Then, when you use that value, you are required to explicitly handle the case when the value is null.**
    - The `Option<T>` enum has a large number of methods that are useful in a variety of situations. ([ref](https://doc.rust-lang.org/stable/std/option/enum.Option.html))

### Integer overflow
- Rust checks for integer overflow only in debug mode and will cause the program to panic. In release mode, which is compiled with the `--release` flag, Rust performs *two's complement wrapping*, where values greater than the maximum value the type can hold "wrap around" to the minimum of the values the type can hold and vice versa; the program won't panic.
- To explicitly handle the possibility of overflow, you can use these families of methods provided by the standard library for primitive numeric types (see [the associated methods of `u32`](https://doc.rust-lang.org/std/primitive.u32.html) for examples):
    - Wrap in all modes with the `wrapping_*` methods, such as `wrapping_add`.
    - Return the `None` value if there is overflow with the `checked_*` methods.
    - Return the value and a boolean indicating whether there was overflow with the `overflowing_*` methods.
    - Saturate at the value’s minimum or maximum values with the `saturating_*` methods.

## Variable and mutability
- A constant defined by `const` requires annotating the data type. Constants live for the entire lifetime of a program and is inlined to each place they are used.
- An immutable variable defined by `let` cannot be in global scope.
- A global variable is defined by `static`, which, unlike a constant, is not inlined and has only one instance and a fixed location in memory. The data type of a `static` global variable also needs be annotated. A global variable can be made mutable with `static mut`.

## Functions

- A new scope block created with curly brackets is an experssion

    ```rust
    let y = {
        let x = 3;
        x + 1
    }
    ```
    Note that the x + 1 line doesn’t have a semicolon at the end, which is unlike most of the lines you’ve seen so far. Expressions do not include ending semicolons, otherwise it turns into a statement and will not return a value. ([ref](https://doc.rust-lang.org/reference/expressions/block-expr.html))


## Control flow
- The codition in an `if` expression must be a `bool`. Rust will not automatically try to convert non-Boolean types to a Boolean.

- Returning values from loops
    ```rust
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    ```

- Specifying a *loop label* on a loop to be used by `break` or `continue` to apply these keywords to the labeled loop instead of the innormost loop. Loop labels must begin with a single quote.
   ```rust
    fn main() {
        let mut count = 0;
        'counting_up: loop {
            let mut remaining = 10;
            loop {
                if remaining == 9 {
                    break;
                }
                if count == 2 {
                    break 'counting_up;
                }
                remaining -= 1;
            }
            count += 1;
        }
    }
   ```
- The power of `match` control flow construct comes from the expressiveness of the patterns and the fact that the compiler confirms that all possible cases are handled.
    - Patterns that bind to values:
        ```rust
        enum UsState {
            Alabama,
            Alaska,
            // --snip--
        }

        enum Coin {
            Penny,
            Nickel,
            Dime,
            Quarter(UsState),
        }
        fn value_in_cents(coin: Coin) -> u8 {
            match coin {
                Coin::Penny => 1,
                Coin::Nickel => 5,
                Coin::Dime => 10,
                Coin::Quarter(state) => {
                    println!("State quarter from {:?}!", state);
                    25
                }
            }
        }
        ```
    - Matching with `Option<T>` needs be exhaustive (handling all cases):
        ```rust
        fn plus_one(x: Option<i32>) -> Option<i32> {
            match x {
                None => None,
                Some(i) => Some(i + 1),
            }
        }

        let five = Some(5);
        let six = plus_one(five);
        let none = plus_one(None);
        ```
    - Catch-all patterns and the `_` placeholder
        ```rust
        let dice_roll = 9;
        match dice_roll {
            3 => add_fancy_hat(),
            7 => remove_fancy_hat(),
            _ => (),
        }

        fn add_fancy_hat() {}
        fn remove_fancy_hat() {}
        ```
        Note that the unit value `()` is used to express that nothing happens with the `_` arm that catches all other patterns.
- Concise control flow with `if let` can be considered as a syntax sugar for a `match` that runs code when the value matches one pattern and then ignores all other values. Choosing between `match` and `if let` depends on what you’re doing in your particular situation and whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.
    ```rust
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
    ```
    vs
    ```rust
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
    ```

## Structs
- The entirity of a mutable struct instance has to be mutable and Rust does not allow individual fields to be marked as mutable.
- Fields Init shorthand can omit the `key` if variable names match the field names
    ```rust
    User {
        active: true,
        username,
        email,
    }
    ```
- Fields in structs are owned data unless it's a stored reference with specified lifetimes.
- Struct update syntax uses `=` like assignment and moves the data (ownership):
    ```rust
    #[derive(Debug)]
    struct Rectangle {
        width: u32,
        height: u32,
        name: String,
    }

    fn main() {
        let scale = 3;
        dbg!(scale);
        let rect1 = Rectangle {
            width: dbg!(30 * scale),
            height: 50,
            name: String::from("rect1"),
        };

        let rect2 = Rectangle {
            name: String::from("rect2"),
            ..rect1
        };
        dbg!(&rect1);

        let rect3 = Rectangle {
            width: 15,
            ..rect1
        };

        dbg!(&rect1);
        dbg!(&rect2);
        dbg!(&rect3);
    }
    ```
    This would cause compiler error as the `name` field moved to `rect3` and `rect1` can no longer be accessed.
    ```
    error[E0382]: borrow of partially moved value: `rect1`
      --> src/main.rs:28:14
       |
    23 |           let rect3 = Rectangle {
       |  _____________________-
    24 | |             width: 15,
    25 | |             ..rect1
    26 | |         };
       | |_________- value partially moved here
    27 |
    28 |           dbg!(&rect1);
       |                ^^^^^^ value borrowed here after partial move
       |
       = note: partial move occurs because `rect1.name` has type `String`, which does not implement the `Copy` trait

    For more information about this error, try `rustc --explain E0382`.
    ```
    But note that for the `rect2` update, since the `name` field of `rect1` was not used in the update, the filed is not moved and thus `rect1` is still accessible after `rect2` assignment.

- Tuple structs without named fields
   ```rust
   struct Color(i32, i32, i32);
   let black = Color(0, 0, 0);
   ```
- Methods.
    ```rust
    impl Rectangle {
        fn square(size: u32) -> Self {
            Self {
                width: size,
                height: size,
            }
        }
    }
    ```
    - Within an impl block, the type `Self `is an alias for the type that the `impl block is for.
    - Methods must have a parameter named self of type `Self` for their first parameter. Methods can take ownership of `self`, borrow self immutably (`&self`), or borrow `self` mutably (`&mut self`).
    - Having a method that takes ownership of the instance by using just self as the first parameter is rare; this technique is usually used when the method transforms self into something else and you want to prevent the caller from using the original instance after the transformation.
    - All functions defined within an `impl` block are called associated functions because they’re associated with the type named after the `impl`.
    - We can define associated functions that don’t have self as their first parameter (and thus are not methods) because they don’t need an instance of the type to work with.
        ```rust
        impl Rectangle {
            fn square(size: u32) -> Self {
                Self {
                    width: size,
                    height: size,
                }
            }
        }
        ```
    - Each struct is allowed to have multiple `impl` blocks.

## Rust module system
- Paths: A way of naming an item, such as a struct, function, or module.
    - An *absolute* path is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal `crate`.
    - A *relative* path starts from the current module and uses `self`, `super`, or an identifier in the current module.
    - Both absolute and relative paths are followed by one or more identifiers separated by double colons (`::`).
- Modules and use: Let you control the organization, scope, and privacy of paths. (see [Module Cheat Sheet](#modules-cheat-sheet))
- Crates: A tree of modules that produces a library or executable
    - *Binary crates* are programs compile into an executable and must have a function called `main` that defines the entry point.
    - *Library crates* don’t have a `main` function, and they don’t compile to an executable.
    - The *crate root* is a source file that the Rust compiler starts from and makes up the root module of the crate.
- Packages: A Cargo feature that lets you build, test, and share crates
    - A *package* is a bundle of one or more crates that provides a set of functionality. A package contains a `Cargo.toml` file that describes how to build those crates.
    - A package can contain as many binary crates as you like, but **at most only one library crate**. A package must contain at least one crate, whether that’s a library or binary crate.
    - Cargo follows a convention that
        - `src/main.rs` is the crate root of a binary crate with the same name as the package.
        - `src/lib.rs` is the crate root of a library crate with the same name as the package
        - Cargo passes the crate root files to `rustc` to build the library or binary.
        - A package can have multiple binary crates by placing files in the `src/bin` directory: each file will be a separate binary crate.

### Modules Cheat Sheet

- *Start from the crate root*: When compiling a crate, the compiler first looks in the crate root file (usually `src/lib.rs` for a library crate or `src/main.rs` for a binary crate) for code to compile.
- *Declaring modules*: In the crate root file, you can declare new modules
    - Use module name in the `.rs` file instead of `mod.rs`:
    ```
    src/front_of_house.rs (idiomatic style)
    src/front_of_house/mod.rs (older style, still supported path)
    ```
    - Only one style can be used for the same module; otherwise there will be a compiler error. The styles can be mixed for different modules in the same project but it should be avoided.
- *Declaring submodules*: In any file other than the crate root, you can declare submodules.
- *Paths to code in modules*: Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate, as long as the privacy rules allow, using the path to the code.
- *Private vs public*: Code within a module is private from its parent modules by default. To make a module public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use `pub` before their declarations. Items in child modules can use the items in their ancestor modules.
    - If an enum is marked as public, all of its variants are public.
    - Individual fields of a struct needs be marked by `pub` for a field to be public.
    - A struct with a private field needs to have a public associated function as constructor; otherwise it'll be impossible to create an instance of the struct.
- *The `use` keyword*: Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of long paths.
    -  Use `use crate::front_of_house::hosting;` to bring the parent of a function into scope instead of the function itself `use crate::front_of_house::hosting::add_to_waitlist`.
    - Use full path `use std::collections::HashMap;` to bring in structs, enums, and other items with `use`.
    - Create an `alias` with `use .. as`: `use std::io::Result as IoResult;`.
    - Rexporing names with `pub use`: `pub use crate::front_of_house::hosting;`.
    - Use nested paths: `use std::{cmp::Ordering, io};` or with `self`: `use std::io::{self, Write};`
    - The glob operator brings *all* public items into the scope: `use std::collections::*;`.
- Using external packages requires adding the package to `Cargo.toml` and then `use`:
    ```rust
    rand = "0.8.5"
    ```
    The standard `std` library is not required in `Cargo.toml` as it's shipped with Rust.
