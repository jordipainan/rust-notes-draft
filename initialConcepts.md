# Let's practice the fundamentals

## Basic input from CLI
```
// Gets the input of the user and prints it
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

``` use std::io; ```

To obtain user input and then print the result as output, we need to bring the io (input/output) library into scope.<br>
The io library comes from the standard library (which is known as ```std```).<br>
Rust comes with a variety of things in its standard library. However, if you had to manually import every single thing that you used, it would be very verbose. But importing a lot of things that a program never uses isn't good either. A balance needs to be struck.
By default, Rust brings only a few types into the scope of every program in the prelude.<br>
If a type you want to use isn’t in the prelude, you have to bring that type into scope explicitly with a use statement.<br>
Using the ```std::io``` library provides you with a number of useful features, including the ability to accept user input.<br>
The prelude is the list of things that Rust automatically imports into every Rust program.<br>
On a technical level Rust inserts ``` extern crate std; ``` into the crate root of every crate (see crate as a package), and ``` user std::prelude::v1::*; ``` into every module.

``` let mut guess = String::new();  ``` <br>

This line do a lot. First we create a mutable variable called guess (let mut guess). By default every Rust variable is not mutable.<br>
And then we create and String type (provided by the std lib) using the new function (returns a new instance of a string). The  ```::new``` indicates that new is an associated function of the String type (a function is implemented on a type rather than on a particular instance of a String). In C we call this a static method. <br>

```
io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
```

If we hadn't listed the use ```std::io``` line at the beginning we could have written this function call as ```std::io::stdin```.<br>
Then we call the ```read_line``` method passing one argument.<br>
The & indicates that this argument is a reference, which give you a way to let multiple parts of your code access one piece of data without needing to copy that data into memory multiple times. We will see references later.
<br>
Finally we have the ```.expect("Something")```. The ```read_line``` also returns a value, in this case an ```io::Result```. Rust has a number of types named ```Result``` in its std library.<br>
The result types are enumerations (a type that can have a fixed set of values), for result the variants are Ok or Err and the purpose is to encode error-handling information.<br>
```io::Result``` has an ```expect``` method.

In the ```println!("abc: {}", guess)``` we are calling the print method with ```!``` indicating that is a macro (we will explain later) and with ```{}``` in the argument as a placeholder for the second argument.
Example:
```
let x = 5;
let y = 6;
println!("x: {} and y: {}", x, y);
```

## Crates, loops, match ...

Import rand in order to generate a random number in ```Cargo.toml``` file:
```
[dependencies]
rand = "0.5.5"
```
Cargo has a mechanism that ensures you can rebuild the same artifact every time you or anyone else builds your code.<br> 
Cargo will use only the versions of the dependencies you specified until you indicate otherwise.<br>
The Cargo.lock file, which was created the first time you ran cargo build. When you build a project for the first time, Cargo figures out all the versions of the dependencies that fit the criteria and then writes them to the Cargo.lock file.
When you build your project in the future, Cargo will see that the Cargo.lock file exists and use the versions specified there rather than doing all the work of figuring out versions again.

- ```cargo update``` is a command for updating this crates ingnoring the Cargo.lock.

```
// lets Rust know we’ll be using the rand crate as an external dependency.
// This also does the equivalent of calling use rand, so now we can call
// anything in the rand crate by placing rand:: before it
extern crate rand;

use std::io;
use std::cmp::Ordering;
// Rng trait, will cover traits later
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    //println!("The secret number is: {}", secret_number);
    loop {
    	println!("Please input your guess.");

    	let mut guess = String::new();

    	io::stdin().read_line(&mut guess)
        	.expect("Failed to read line");

    	println!("You guessed: {}", guess);

    	// We've already defined a variable called guess but in Rust we can
    	// shadow the previous value of guess with a new one. This feature is often
    	// used in situation in which you want to convert a value from one type to another type
    	// Trim deletes the \n when pressing INTRO and parse parses the string into int
    	// u32 is the type of the new guess (32-bit unsigned integer)
        // Switching from an expect call to a match expression is how we generally move
        // from crashing on an error to handling the error
    	let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            // _ is a catchall value for processing all possible errors
	    Err(_) => continue,
        }

    	// Comparing guess and secret_number
    	// We use a match expression to decide what to do next based on which variant
    	// of Ordering was returned from the call to cmp
    	match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
            // Quitting after correct guess
	    break;
    	}
    }
}
```
- A match expression is made up of arms. An arm consists of a pattern and the code that should be run  if the value given to the beginnning of the match expression fits that arm's pattern.
  Rust takes the value given to match and looks through each arm’s pattern in turn. The match construct and patterns are powerful features in Rust.
- We can use ```cargo doc --open``` command which will build documentation provided by all of our dependencies locally and open it in our browser
