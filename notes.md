# Rust study notes

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
- *unit* type: a tuple type with no fields `()` is often called *unit* or *unit type*. Its one value is also called *unit* or *the unit value*. ([ref](https://doc.rust-lang.org/reference/types/tuple.html?highlight=unit%20type#tuple-types))

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
            width: 15,
            ..rect1
        };

        dbg!(&rect1);
        dbg!(&rect2);
    }
    ```
    This would cause compiler error as the `name` field moved to `rect2` and `rect1` can no longer be accessed.
    ```
    error[E0382]: borrow of partially moved value: `rect1`
    --> src/main.rs:22:10
       |
    17 |       let rect2 = Rectangle {
       |  _________________-
    18 | |         width: 15,
    19 | |         ..rect1
    20 | |     };
       | |_____- value partially moved here
    21 |
    22 |       dbg!(&rect1);
       |            ^^^^^^ value borrowed here after partial move
       |
       = note: partial move occurs because `rect1.name` has type `String`, which does not implement the `Copy` trait

    For more information about this error, try `rustc --explain E0382`.
    ```
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
