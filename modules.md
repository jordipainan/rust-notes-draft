# Modules

Rust has a module system that enables the reuse of code in an organized fashion.

A module is a namespace that contains definitions of functions or types, and you can choose whether those definitions are visible outside their module (public) or not (private).

- The mod keyword declares a new module. Code within the module appears either immediately following this declaration within curly brackets or in another file.
- By default, functions, types, constants, and modules are private. The pub keyword makes an item public and therefore visible outside its namespace.
- The use keyword brings modules, or the definitions inside modules, into scope so it’s easier to refer to them.

## ```mod``` and the filesystem

A crate is a project that other people can pull into their projects as a dependency. To create a library, pass the --lib option into cargo.
```
cargo new comun_module --lib
```

Every module definition in Rust starts with the mod keyword.

```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2+2, 4);
    }
}

mod network {
    fn connect() {
    }
}
```

If we wanted to call this function from code outside the network module, we would need to specify the module and use the namespace syntax :: like so: network::connect().
We can also have multiple modules in the file and modules inside modules.

```
// 'src/lib.rs'

mod client;
mod network;

// 'src/network.rs'

fn connect() { }
mod server {
    fn connect() { }
}
```

## Controlling visibility with ```pub```

We use the extern crate command to bring the library crate into scope.

```
extern crate x;

fn main() {
    x::y::function();
}
```
The default state of all code in Rust is private: no one else is allowed to use the code. 

To tell Rust to make a function public, we add the pub keyword to the start of the declaration.

After you specify that a function such as x::y is public, not only will your call to that function from your binary crate be allowed, but also the warning that the function is unused will go away. Marking a function as public lets Rust know that the function will be used by code outside of your program. Rust considers the theoretical external usage that’s now possible as the function “being used.” Thus, when a function is marked public, Rust will not require that it be used in your program and will stop warning that the function is unused.

Overall, these are the rules for item visibility:

- If an item is public, it can be accessed through any of its parent modules.
- If an item is private, it can be accessed only by its immediate parent module and any of the parent’s child modules.


Example that will not compile:

```
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

An example that works:

```
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

## Bringing Names into Scope with the ```use``` Keyword

Rust’s use keyword shortens lengthy function calls by bringing the modules of the function you want to call into scope. Here’s an example:

```
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

Because enums also form a sort of namespace like modules, we can bring an enum’s variants into scope with use as well.

```
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```


We can use ```super``` to move up one module in the hierarchy from our current module, like this:
```
super::client::connect();
```

Also we can use ```*``` to import all module:
```
use TrafficLight::*;
```

