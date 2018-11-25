# Understanding Ownership

## Ownership

Ownership is Rust’s most unique feature, it enables Rust to make memory safety guarantees without needing a garbage collector.

Memory is managed through a system of ownership with a set of rules that the compiler checks at compile time.

### The Stack and the Heap

Both the stack and the heap are parts of memory that is available to your code to use at runtime, but they are structured in different ways.
The stack uses a LIFO strategy, adding data is called pushing onto the stack, and removing data is called popping off the stack.
The stack is fast because of the way it accesses the data: it never has to search for a place to put new data or a place to get data from because that place is always the top. Another property that makes the stack fast is that all data on the stack must take up a known, fixed size.

Data with a size unknown at compile time or a size that might change can be stored on the heap instead. 
The heap is less organized: when you put data on the heap, you ask for some amount of space. The operating system finds an empty spot somewhere in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. 
This process is called allocating on the heap, sometimes abbreviated as just “allocating.” 
Pushing values onto the stack is not considered allocating. Because the pointer is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you have to follow the pointer.

Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there.
A processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap). Allocating a large amount of space on the heap can also take time.

When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

### Ownership Rules

- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Variable Scope

```
#![allow(unused_variables)]
fn main() {
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid. {Calling drop method autocatically}
}
```

### Memory and allocation. An example with strings

```
#![allow(unused_variables)]
fn main() {
    let s = String::from("hello");
    let mut s = String::from("hello"); // Here we are requesting memory
    s.push_str(", world!"); // push_str() appends a literal to a String
    println!("{}", s); // This will print `hello, world!`
}
```

In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s immutability.

With the String type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents.
This means:
- The memory must be requested from the operating system at runtime.
- We need a way of returning this memory to the operating system when we’re done with our String.

In C for example we need to allocate memory and let it free when no longer use it. But in Rust the memory is automatically returned once the variable that owns it goes out of scope.

### Ways Variables and Data Interact

Multiple variables can interact with the same data in different ways in Rust

This bind the value 5 to x, then make a copy of the value in x and bind it to y. We now have two variables, x and y, and both equal 5.
This is happening because integers are simple values with a known, fixed size, and these two 5 values are pushed onto the stack.
```
let x = 5
let y = x
```

Now let's look at the String version
```
let s1 = String::from("hello");
let s2 = s1;
```
A String is made up of three parts, shown on the left: a pointer to the memory that holds the contents of the string, a length, and a capacity. This group of data is stored on the stack. On the right is the memory on the heap that holds the contents.

<img src="https://doc.rust-lang.org/book/2018-edition/img/trpl04-01.svg" alt="" width="300px" height="300px">

The length is how much memory, in bytes, the contents of the String is currently using. The capacity is the total amount of memory, in bytes, that the String has received from the operating system. The difference between length and capacity matters, but not in this context.
When we assign s1 to s2, the String data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to.

<img src="https://doc.rust-lang.org/book/2018-edition/img/trpl04-02.svg" alt="" width="300px" height="300px">

Also we can copy the heap data, but it can be very expensive in terms of runtime performance if the data on the heap were large.

The last portion of code has a triky bug.
When s2 and s1 go out of scope, they will both try to free the same memory. This is known as a double free error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.
Rustaceans we are not afraid of that because Rust simply considers s1 is no longer valid and therefore we dont need to free anything when s1 goes out of scope.

More in depth, when compile this code trying to use s1 ...
```
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);

``` 
The compiler thows an error:
```
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait

```

In other languajes its called shallow copy, but because Rust also invalidates the first variable we will call it a MOVE (s1 was moved into s2).

In addition, there’s a design choice that’s implied by this: Rust will never automatically create “deep” copies of your data. Therefore, any automatic copying can be assumed to be inexpensive in terms of runtime performance.

If we do want to deeply copy the heap data of the String, not just the stack data, we can use a common method called clone.

```
#![allow(unused_variables)]
fn main() {
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
}
```

Lets talk about the following chunk of code:
```
let x = 5;
let y = x;
println!("x = {}, y = {}", x, y);
```
Seems to contradict what we just learned: we don’t have a call to clone, but x is still valid and wasn’t moved into y.
The reason is that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. 
That means there’s no reason we would want to prevent x from being valid after we create the variable y. 
In other words, there’s no difference between deep and shallow copying here, so calling clone wouldn’t do anything different from the usual shallow copying and we can leave it out.
Rust has a special annotation called the Copy trait that we can place on types like integers that are stored on the stack. If a type has the Copy trait, an older variable is still usable after assignment.
So what types are Copy? Check the official Rust documentation :)
But as a general rule, any group of simple scalar values can be Copy. You need to know that Rust won’t let us annotate a type with the Copy trait if the type, or any of its parts, has implemented the Drop trait.


### Returning values and scope

Returning values can also transfer ownership.

```
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 goes out of scope but was
  // moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("hello"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function.
}

// takes_and_gives_back will take a String and return one.
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```
The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless the data has been moved to be owned by another variable.
Taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? It’s quite annoying that anything we pass in also needs to be passed back if we want to use it again.
Luckily for us, Rust has a feature for this concept, called references.


## References and Borrowing

Here is how you would define and use a calculate_length function that has a reference to an object as a parameter instead of taking ownership of the value:
```
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1); // &s1 is a reference to s1 and 
				     // they allow you to refer to some 
                                     // value without taking ownership of it

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

The opposite of referencing by using & is dereferencing, which is accomplished with the dereference operator, *. We will see later.

Because we do not own the variable, the value it points to will not be dropped when the reference goes out of scope.

The scope in which the variable s is valid is the same as any function parameter’s scope, but we don’t drop what the reference points to when it goes out of scope because we don’t have ownership. When functions have references as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.

We call having references as function parameters borrowing. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back.

Example: Will fail

```
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

```
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

We’re not allowed to modify something we have a reference to. We need the ownership!

### Mutable references

The following code will fix the error that we got before:

```
fn main() {
    let mut s = String::from("hello");

    change(&mut s); // mutable reference
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}

```

Mutable references have one big restriction: you can only have one mutable reference to a particular piece of data in a particular scope. This code will fail:

```
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

The benefit of having this restriction is that Rust can prevent data races at compile time.
As always, we can use curly brackets to create a new scope.
```
#![allow(unused_variables)]
fn main() {
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 goes out of scope here, so we can make a new reference with no problems.

let r2 = &mut s;
}
```

A similar rule exists for combining mutable and immutable references.
We also cannot have a mutable reference while we have an immutable one. Users of an immutable reference don’t expect the values to suddenly change out from under them!

You should be carefull with dangling pointers. A dangling pointer is  that references a location in memory that may have been given to someone else, by freeing some memory while preserving a pointer to that memory.


## Slices

Another data type that does not have ownership is the slice. Slices let us reference a contiguous sequence of elements in a collection rather than the whole collection.


### String slices

```
let s = String::from("hello world");
let hello = &s[0..5]; // start..end+1
		      // Can be written as [..2]
let world = &s[6..=10] // start..=end (Includes last element)
                       // Can be written as [3..]
// Or we can write
let w_len = world.len()
let slice = &s[3..len]

// We can also drop both values to take a slice of the entire string
&s[..]
```

The type that signifies "string slice" is written as &str
```
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

### Other slices

Slices are not specific to strings. Consider the following example:
```
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
```

# Summary

The concepts of ownership, borrowing, and slices ensure memory safety in Rust programs at compile time. The Rust language gives you control over your memory usage in the same way as other systems programming languages, but having the owner of data automatically clean up that data when the owner goes out of scope means you don’t have to write and debug extra code to get this control.
