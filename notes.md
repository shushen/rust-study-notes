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
