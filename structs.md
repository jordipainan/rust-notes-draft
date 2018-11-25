# Using structs to Structure related data

A struct, or structure, is a custom data type that lets you name and package together multiple related values that make up a meaningful group. If you’re familiar with an object-oriented language, a struct is like an object’s data attributes.

## Defining and Instantiating Structs

- Like tuples, the pieces of a struct can be different types.
- Unlike tuples, you'll name each piece of data so it is clear what the values mean
- Structs are more flexible than tuples

```
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}


let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

We don’t have to specify the fields in the same order in which we declared them in the struct.
But we need to specify them all.

To get a specific value from a struct we can use dot notation.

```
user1.email
```

If the instance is mutable we can change the value using dot notation.
But keep in mind one thing, the entire instance must be mutable. Rust does not allow us to mark only certain fields as mutable.

As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance.

```
fn build_user(email: String, username: String) -> User {
    User {
        email: email, // We can write email only, because the field and the parameter have the same name.
		      // email,
		      // Its called the field init shorthand
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

We can create instances from other instances with struct update syntax

```
#![allow(unused_variables)]
fn main() {
    struct User {
        username: String,
        email: String,
        sign_in_count: u64,
        active: bool,
    }

    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotherusername567"),
        active: user1.active,
        sign_in_count: user1.sign_in_count,
    };
    
    let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotherusername567"),
        ..user1
    };
}
```

## Tuple Structs without Named Fields to Create Different Types

You can also define structs that look similar to tuples, called tuple structs. Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields. Tuple structs are useful when you want to give the whole tuple a name and make the tuple be a different type than other tuples.

```
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

## Unit-like structs

You can also define structs that don’t have any fields! These are called unit-like structs because they behave similarly to (), the unit type. Unit-like structs can be useful in situations in which you need to implement a trait on some type but don’t have any data that you want to store in the type itself.
It’s possible for structs to store references to data owned by something else, but to do so requires the use of lifetimes, a Rust feature that we’ll discuss later.


## Printing structs

Putting the specifier :? inside the curly brackets tells println! we want to use an output format called Debug. Debug is a trait that enables us to print our struct in a way that is useful for developers so we can see its value while we’re debugging our code.
Rust does include functionality to print out debugging information, but we have to explicitly opt in to make that functionality available for our struct. To do that, we add the annotation #[derive(Debug)] just before the struct definition.

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1); // use {:#?} in order to change the output (splitted by lines)
}
```

## Method Syntax

Methods are similar to functions: they’re declared with the fn keyword and their name, they can have parameters and a return value, and they contain some code that is run when they’re called from somewhere else. However, methods are different from functions in that they’re defined within the context of a struct (or an enum or a trait object), and their first parameter is always self, which represents the instance of the struct the method is being called on.

### Defining methods

Let's change the area function that has a Rectangle instance as a parameter and instead make an area method defined on the Rectangle struct.

```
#[derive(Debug)]

struct Rectangle {
     width: u32,
     height: u32,
}

impl Rectangle {
    // We dont want to write the struct data only read, thats why &self
    fn area(&self) -> u32 {
	self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("The area of the rectangle is {} square pixels", rect1.area());
}
```

The main benefit of using methods instead of functions, in addition to using method syntax and not having to repeat the type of self in every method’s signature, is for organization. We’ve put all the things we can do with an instance of a type in one impl block rather than making future users of our code search for capabilities of Rectangle in various places in the library we provide.


### Associated functions

Another useful feature of impl blocks is that we’re allowed to define functions within impl blocks that don’t take self as a parameter. These are called associated functions because they’re associated with the struct. They’re still functions, not methods, because they don’t have an instance of the struct to work with. You’ve already used the String::from associated function.

```
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

To call this associated function, we use the :: syntax with the struct name; ```let sq = Rectangle::square(3);```

Each struct is allowed to have multiple impl blocks. 

## Summary

Structs let you create custom types that are meaningful for your domain. By using structs, you can keep associated pieces of data connected to each other and name each piece to make your code clear. Methods let you specify the behavior that instances of your structs have, and associated functions let you namespace functionality that is particular to your struct without having an instance available.
