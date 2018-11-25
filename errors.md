# Error handling

Rust groups errors into two major categories: recoverable and unrecoverable errors. For a recoverable error, such as a file not found error, it’s reasonable to report the problem to the user and retry the operation. Unrecoverable errors are always symptoms of bugs, like trying to access a location beyond the end of an array.

Rust doesn’t have exceptions. Instead, it has the type Result<T, E> for recoverable errors and the panic! macro that stops execution when the program encounters an unrecoverable error.

## Unrecoverable Errors with ```panic!```

When the panic! macro executes, your program will print a failure message, unwind and clean up the stack, and then quit.

By default, when a panic occurs, the program starts unwinding, which means Rust walks back up the stack and cleans up the data from each function it encounters.

The alternative is to immediately abort, which ends the program without cleaning up. Memory that the program was using will then need to be cleaned up by the operating system. We enable this in Cargo.toml using:
```
[profile.release]
panic = 'abort'
```

Example:

```
fn main() {
    panic!("crash and burn");
}
```

## Recoverable errors with ```Result```

```Result``` enum is defined as having two variants, ```Ok``` and ```Err```, as follows:

```
// T and E are generic type parameters
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

What you need to know right now is that T represents the type of the value that will be returned in a success case within the Ok variant, and E represents the type of the error that will be returned in a failure case within the Err variant. Because Result has these generic type parameters, we can use the Result type and the functions that the standard library has defined on it in many different situations where the successful value and error value we want to return may differ.

Example opening a file:

```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

## Matching on Different Errors

Example:

```
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

The type of the value that ```File::open``` returns inside the ```Err``` variant is ```io::Error```, which is a struct provided by the standard library. This struct has a method ```kind``` that we can call to get an ```io::ErrorKind``` value. The enum ```io::ErrorKind``` is provided by the standard library and has variants representing the different kinds of errors that might result from an ```io``` operation. The variant we want to use is ```ErrorKind::NotFound```, which indicates the file we’re trying to open doesn’t exist yet.

The condition if ```error.kind() == ErrorKind::NotFound``` is called a match guard: it’s an extra condition on a match arm that further refines the arm’s pattern. This condition must be true for that arm’s code to be run; otherwise, the pattern matching will move on to consider the next arm in the match.

The condition we want to check in the match guard is whether the value returned by error.kind() is the NotFound variant of the ErrorKind enum. If it is, we try to create the file with File::create. However, because File::create could also fail, we need to add an inner match statement as well. When the file can’t be opened, a different error message will be printed. The last arm of the outer match stays the same so the program panics on any error besides the missing file error.

### Shortcuts for Panic on Error: ```unwrap``` and ```expect```

Using match works well enough, but it can be a bit verbose and doesn’t always communicate intent well. 

If the Result value is the Ok variant, unwrap will return the value inside the Ok. If the Result is the Err variant, unwrap will call the panic! macro for us.

Example:

```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

Another method, expect, which is similar to unwrap, lets us also choose the panic! error message. Using expect instead of unwrap and providing good error messages can convey your intent and make tracking down the source of a panic easier.

```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### Propagating Errors

When you’re writing a function whose implementation calls something that might fail, instead of handling the error within this function, you can return the error to the calling code so that it can decide what to do. This is known as propagating the error and gives more control to the calling code.

Example:

```
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

### A Shortcut for Propagating Errors: the ```?``` Operator

```
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

The ? placed after a Result value is defined to work in almost the same way as the match expressions.

If the value of the Result is an Ok, the value inside the Ok will get returned from this expression, and the program will continue. If the value is an Err, the Err will be returned from the whole function as if we had used the return keyword so the error value gets propagated to the calling code.

The ? operator can only be used in functions that have a return type of Result.

## To ```panic!``` or Not to ```panic!```

When code panics, there’s no way to recover. You could call panic! for any error situation, whether there’s a possible way to recover or not, but then you’re making the decision on behalf of the code calling your code that a situation is unrecoverable.

Returning Result is a good default choice when you’re defining a function that might fail. In rare situations, it’s more appropriate to write code that panics instead of returning a Result. 

The panic! macro signals that your program is in a state it can’t handle and lets you tell the process to stop instead of trying to proceed with invalid or incorrect values. The Result enum uses Rust’s type system to indicate that operations might fail in a way that your code could recover from. You can use Result to tell code that calls your code that it needs to handle potential success or failure as well.


