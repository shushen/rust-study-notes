# Rust study notes

## Data types
- *unit* type: a tuple type with no fields `()` is often called *unit* or *unit type*. Its one value is also called *unit* or *the unit value*. ([ref](https://doc.rust-lang.org/reference/types/tuple.html?highlight=unit%20type#tuple-types))

## Functions

- A new scope block created with curly brackets is an experssion

    ```
    let y = {
        let x = 3;
        x + 1
    }
    ```
    Note that the x + 1 line doesn’t have a semicolon at the end, which is unlike most of the lines you’ve seen so far. Expressions do not include ending semicolons, otherwise it turns into a statement and will not return a value. ([ref](https://doc.rust-lang.org/reference/expressions/block-expr.html))

