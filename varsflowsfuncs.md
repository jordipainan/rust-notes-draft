# Common programming concepts

None of the concepts presented in this chapter are unique to Rust.<br>
We will learn about variables, basic types, functions, comments and control flow.

## Variables and mutability

- By default variables are immutable.

```
// This will throw a compilation error
fn main() {
    let x = 5;
    println!("My number is {}", x);
    x = 6;
}

// This will compile
fn main() { 
    let mut x = 5; ... 
}
```

There is a difference between variables and constants: 
- Like immutable variables, constants are values that are bound to a name and are not allowed to change.
- We declare constants using the ```const``` keyword instead of the ```let``` keyword.
- The type of the value must be annotated in constants.
- Constants can be declared at any scope, including global scope, which makes them usefull for values that many parts of code need to know about.
- Constants may be set only to a constant expression, not the result of a function call or any other value that could only be computed at runtime.

```
// 100_000 is the same as 100000
// Rust ignores '_' and we can use it to see the values in a friendly way
const MAX_POINTS: u32 = 100_000
```

Finally we have the Shadowing: We can declare a new variable with the same name as a previous variable, and the new variable shadows the previous.
```
// First binds the variable x to 5
// Then it shadows x by repeating let x = 
// taking the original value and adding 1, so the value will be 6 and so on
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;
    println!("The value of x is: {}", x);
}
```
Shadowing is different than marking a variable as mut, because we will get a compile-time error if we try to reassign to this variable without using the let keyword again.<br>
By using let we can perform few transformations on a value but have the variable be immutable after those transformations have been completed.<br>
Another difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value but reuse the same name.


## Data types

Rust is a statically typed language, which means that it must know the types of all variables at a compile time.
In cases when many types are possible we must add a type annotation.

```
let guess: u32 = "42".parse().expect("Not a number!");
```

### Scalar Types
A scalar type represents a single value. Rust has four primary scalar types: integers, floating-point numbers, booleans and characters.
#### Integers
|Length |Signed|Unsigned|
|-------|------|--------|
|8-bit  |i8    |u8      |
|16-bit |i16   |u16     |
|32-bit |i32   |u32     |
|64-bit |i64   |u64     |
|128-bit|i128  |u128    |
|arch   |isize |usize   |


Each signed variant can store numbers from -(2^(n - 1)) to 2^(n - 1) - 1 inclusive, where n is the number of bits that variant uses.
We can write integer literas in various formats:
- Decimal: 98_222
- Hex    : 0xff
- Octal  : 0o77
- Binary : 0b1111_0000
- Byte (u8 only): b'A'

If we're unsure of which type of integer use ```i32``` is generally the fastest.

#### Floating-Point
```
fn main() {
    let x = 2.0;       // f64
    let y: f32 = 3.0;  // f32
}
```

#### Boolean
True and false as always.

#### Character
Specified in a single quotes, double quotes are for string literals.<br>
```char``` type represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII.<br>
Be carefull with chars in Rust because human intuition for what a "character" is may not match up with what a char is in Rust.

### Compound types
Can group multiple values into one type. Rust has two primitive compound types: tuples and arrays.

#### Tuples
- Have fixed length
- We declare a tuple like this: ```let tup: (i32, f64, u8) = (500, 6.4, 1);```
- To get individual values out of a tuple, we can use pattern matching to dstructure a tuple value:
  ```
  fn main() {
      let tup = (500, 6.4, 1);
      let (x, y, z) = tup;
      println!("The value of y is: {}", y);
  }
  ```
- In addition to destructuring through pattern matching, we can access a tuple element directly by using a period (.) followed by the index of the value we want to access:
  ```let first_element = x.0;```
- This program creates a tuple, x, and then makes new variables for each element by using their index. As with most programming languages, the first index in a tuple is 0.

#### Arrays

- Unlike the tuples, every element of an array must have the same type.
- Have fixed length, like tuples.
- Example: ```let a: [i32; 5] = [1,2,3,4,5];```
- Accessing elements: a[0] ... a[n-1]

## Functions

### Basic
Function definitions in Rust start with fn and have a set of parentheses after the function name. The curly brackets tell the compiler where the function body begins and ends.

```
fn example_function(arg_a, arg_b) { ... }
```

### Statements and Expressions
- Statements are instructions that perform some action and do not return a value. 
- Expressions evaluate to a resulting value

- This a statement: ```let a = 6;``` The assignment does not returns the value of the assignment. So ```x = y = 6``` is not working in Rust.
- This is a expression: 5 + 6 that evaluates the value 11

```
fn main() {
    let x = 5;
    let y = {
        let x = 3;
	x + 1
    };
    println!("The value of y is: {}", y);
}
```

This expression:
```
{
    let x = 3;
    x + 1
}
```
Is a block that, in this case, evaluates to 4. That value gets bound to y as a part of the let statement.<br>
Note the x + 1 line without a semicolon at the end, which is unlike most of the lines you’ve seen so far. Expressions do not include ending semicolons.<br>
If you add a semicolon to the end of an expression, you turn it into a statement, which will then not return a value. Keep this in mind.


### Functions with return values 

Functions can return values to the code that calls them. We don’t name return values, but we do declare their type after an arrow (->). <br>
In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. <br>
You can return early from a function by using the return keyword and specifying a value, but most functions return the last expression implicitly. <br>
Here’s an example of a function that returns a value:
```
fn five() -> i32 {
    5
}
fn main() {
    let x = five();
    println!("The value of x is: {}", x);
}
```

## Comments

Always ```//```.

## Control Flow

### ```if``` Expressions

```
fn main() {
   let x = 5;
   if x < 5 {
      println!("{}", x);
   } else {
       println!("{}", x+1);
   }
}
```
Blocks of code associated with the conditions in if expressions are sometimes called arms, just like the arms in match expressions.<br>
Condition must be a bool. <br>
Also we have ```else if``` statements. Using too many else if expressions can clutter your code, so if you have more than one, you might want to refactor your code.<br>

Because if is an expression, we can use it on the right side of a let statement.

```
fn main(i32: y) {
 let x = if y < 5 {
     5
 } else {
     6
 }; 
}
```

Remember that blocks of code evaluate to the last expression in them, and numbers by themselves are also expressions.


### Loops

#### ```loop``` 

Keyword used in previous examples. Stops with ```break```
One of the uses of a loop is to retry an operation you know can fail, such as checking if a thread completed its job. <br>
However, you might need to pass the result of that operation to the rest of your code. If you add it to the break expression you use to stop the loop, it will be returned by the broken loop:
```
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    assert_eq!(result, 20);
}
```

#### ```while```
```
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

#### ```for```
```
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

And if we want to do a reverse loop ? <br>
The way to do that would be to use a Range, which is a type provided by the standard library that generates all numbers in sequence starting from one number and ending before another number.<br>
Here’s what the countdown would look like using a for loop and another method we’ve not yet talked about, rev, to reverse the range:
```
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

