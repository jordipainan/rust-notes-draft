# Generic Types, Traits and Lifetimes

Generics is a tool for effectively handling the duplication of concepts.
Generics are abstract stand-ins for concrete types or other properties. When we’re writing code, we can express the behavior of generics or how they relate to other generics without knowing what will be in their place when compiling and running the code.

## Generic data types, a way for removing duplicates

For example, say we had two functions: one that finds the largest item in a slice of i32 values and one that finds the largest item in a slice of char values. How would we eliminate that duplication? 

We can use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types.

### In function definitions

When defining a function that uses generics, we place the generics in the signature of the function where we would usually specify the data types of the parameters and return value.

Example:

- Duplicated

```
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

- Unduplicated

```
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```


### In struct definitions

To define a Point struct where x and y are both generics but could have different types, we can use multiple generic type parameters.

```
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}

// Or using the same type

struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let both_integer = Point { x: 5, y: 7 };
}
```

### In Enum Definitions

```
enum Option<T> {
    Some(T),
    None,
}
```

```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### In method definitions

```
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

## Traits: Defining Shared Behavior

A trait tells the Rust compiler about functionality a particular type has and can share with other types.
Traits are similar to a feature often called interfaces in other languages, although with some differences.

### Defining a Trait

A type’s behavior consists of the methods we can call on that type.
Trait definitions are a way to group method signatures together to define a set of behaviors necessary to accomplish some purpose.

```
pub trait Summary {
    fn summarize(&self) -> String;
}
```

We declare a trait using the trait keyword and then the trait’s name, which is Summary in this case. Inside the curly brackets, we declare the method signatures that describe the behaviors of the types that implement this trait.

After the method signature, instead of providing an implementation within curly brackets, we use a semicolon. Each type implementing this trait must provide its own custom behavior for the body of the method. The compiler will enforce that any type that has the Summary trait will have the method summarize defined with this signature exactly.

### Implementing a trait on a Type

Implementing a trait on a type is similar to implementing regular methods. The difference is that after impl, we put the trait name that we want to implement, then use the for keyword, and then specify the name of the type we want to implement the trait for.

```
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

One restriction to note with trait implementations is that we can implement a trait on a type only if either the trait or the type is local to our crate.
We can’t implement external traits on external types.

This restriction is part of a property of programs called coherence, and more specifically the orphan rule, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa. Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.


### Default implementations

Sometimes it’s useful to have default behavior for some or all of the methods in a trait instead of requiring implementations for all methods on every type. Then, as we implement the trait on a particular type, we can keep or override each method’s default behavior.

```
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

// Now we can call summarize

let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summarize());
```

The syntax for overriding a default implementation is the same as the syntax for implementing a trait method that doesn’t have a default implementation.

Default implementations can call other methods in the same trait, even if those other methods don’t have a default implementation. In this way, a trait can provide a lot of useful functionality and only require implementors to specify a small part of it.

```
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```


### Traits as arguments

```
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

In the body of notify, we can call any methods on item that come from the Summary trait, like summarize.

#### Trait Bounds

The impl Trait syntax works for short examples, but is syntax sugar for a longer form. This is called a 'trait bound', and it looks like this:

```
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}

// See the difference
pub fn notify(item1: impl Summary, item2: impl Summary) {
pub fn notify<T: Summary>(item1: T, item2: T) {
```
##### Multiple trait bounds with ```+```

```
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

#### ```where``` clauses for clearer code

```
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

### Returning Traits

We can use the impl Trait syntax in return position as well, to return something that implements a trait:

```
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

This signature says, "I'm going to return something that implements the Summary trait, but I'm not going to tell you the exact type. In our case, we're returning a Tweet, but the caller doesn't know that.

### Using trait bounds to Conditionally Implement methods

By using a trait bound with an impl block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits.

Example:
```
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

We can also conditionally implement a trait for any type that implements another trait. Implementations of a trait on any type that satisfies the trait bounds are called blanket implementations and are extensively used in the Rust standard library. For example, the standard library implements the ToString trait on any type that implements the Display trait. The impl block in the standard library looks similar to this code:

```
impl<T: Display> ToString for T {
    // --snip--
}
```
Because the standard library has this blanket implementation, we can call the to_string method defined by the ToString trait on any type that implements the Display trait.

## Validating References with Lifetimes

Every reference in Rust has a lifetime, which is the scope for which that reference is valid.

### Preventing Dangling References with Lifetimes

The main aim of lifetimes is to prevent dangling references, which cause a program to reference data other than the data it’s intended to reference.

Example of dangling:

```
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}

error[E0597]: `x` does not live long enough
  --> src/main.rs:7:5
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

### The borrow Checker

The Rust compiler has a borrow checker that compares scopes to determine whether all borrows are valid.

```
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+

```

### Generic Lifetimes in Functions

When we’re defining a function, we don’t know the concrete values that will be passed into this function, so we don’t know whether the if case or the else case will execute. We also don’t know the concrete lifetimes of the references that will be passed in, so we can’t look at the scopes.

The borrow checker can’t determine this either, because it doesn’t know how the lifetimes of x and y relate to the lifetime of the return value. To fix this error, we’ll add generic lifetime parameters that define the relationship between the references so the borrow checker can perform its analysis.

### Lifetime Annotation Syntax

[Jordi needs to stop for today ... Brain exploding in 3, 2, 1 ..]
