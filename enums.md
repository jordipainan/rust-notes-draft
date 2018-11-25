# Enums and pattern matching

## Defining an Enum

```
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
}

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
}
```

We can represent the same concept in a more concise way using just an enum, rather than an enum inside a struct, by putting data directly into each enum variant.


```
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));
let loopback = IpAddr::V6(String::from("::1"));
```

There’s another advantage to using an enum rather than a struct: each variant can have different types and amounts of associated data.

```
// Example

enum IpAddr {
    V4(u8,u8,u8,u8);
    V6(String)
}

// ...
```

You can put any kind of data inside an enum variant: strings, numeric types, or structs, for example. You can even include another enum!


## The Option Enum and Its Advantages Over Null Values

Option is another enum defined by the standard library.
It encodes the very common scenario in which a value could be something or it could be nothing.
Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is ```Option<T>```.

```
enum Option<T> {
    Some(T),
    None,
}
```
The Option<T> enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. In addition, so are its variants: you can use Some and None directly without the Option:: prefix. The Option<T> enum is still just a regular enum, and Some(T) and None are still variants of type Option<T>.

The <T> syntax means the Some variant of the Option enum can hold one piece of data of any type.

```
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

If we use None rather than Some, we need to tell Rust what type of Option<T> we have, because the compiler can’t infer the type that the Some variant will hold by looking only at a None value.

In general, in order to use an Option<T> value, you want to have code that will handle each variant. You want some code that will run only when you have a Some(T) value, and this code is allowed to use the inner T. You want some other code to run if you have a None value, and that code doesn’t have a T value available. The match expression is a control flow construct that does just this when used with enums.

## The match control flow operator

Rust has an extremely powerful control flow operator called match that allows you to compare a value against a series of patterns and then execute code based on which pattern matches.

```
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Patters that binds to values

Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is how we can extract values out of enum variants.

```
#[derive(Debug)] // So we can inspect the state in a minute
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


fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

### Matching with Option<T>

In the previous section, we wanted to get the inner T value out of the Some case when using Option<T>; we can also handle Option<T> using match as we did with the Coin enum! Instead of comparing coins, we’ll compare the variants of Option<T>, but the way that the match expression works remains the same.

```
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

Matches in Rust are exhaustive: we must exhaust every last possibility in order for the code to be valid.

### The ```_``` Placeholder


Rust also has a pattern we can use when we don’t want to list all possible values. The _ pattern will match any value.

```
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (), // In any other case do nothing
}
```

The match expression can be a bit wordy in a situation in which we only care about one of the cases. For this situation, Rust provides if let.


## Concise control flow with ```if let```

The if let syntax lets you combine if and let into a less verbose way to handle values that match one pattern while ignoring the rest.


```
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}

// Or we can write
if let Some(3) = some_u8_value {
    println!("three");
}
```

The syntax if let takes a pattern and an expression separated by an =. It works the same way as a match, where the expression is given to the match and the pattern is its first arm.
We can also include else's:

```
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```
