## Using generics in Rust

#### Problem

- High-level algorithms for multiple data types

  ```rust
  // Same functions for diferent data types
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
  ```

- General data structure for multiple types

  ```rust
  // Same structures for diferent data types
  struct PointInteger {
      x: i32,
      y: i32
  }
  struct PointFloat {
      x: f32,
      y: f32
  }
  ```

### Use generic functions to reuse algorithms

Using the example of ```largest_x() -> x {}```

```rust
// Using generics, T is a placeholder (can be Q,X ..)
fn largest<T>(list: &[T]) -> T {
    // same code
}
fn main() {
	// T is explicitly specified
	let result: i32 = largest::<i32>();
    println!("{:?}", result);
}
```

Here we have a **problem**, what happen if the passed data does not implement the ```>``` operator ?

*[spoiler]:* Will fail

So in order to resolve that we have **Traits**, which basically defines an interface or a specific behavior the type must allow. So giving a boundary to this type T we can make sure that our function allow any data type that implements this behaviors.

```rust
// T must allow partial ordering and copy traits
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {}
```

### Reuse Structures in Enums and Structs 

- We can do the same as we did in functions

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    let mixed = Point { x: 5, y: 4.0 }; // Won't compile
}
```

```rust
// Mixed types
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    let mixed = Point { x: 5, y: 4.0 };
}
```

- We can do the same with ```enums``` and it's variants

```rust
enum Option<T> {
    Some(T),
    None,
}

fn main() {
    let some_integer = Option::Some(42);
    let some_float = Option::Some(3.14);
    // We need to specify the none value
    let none: Option<i32> = Option::None;
}
```

### Working with generics in struct methods

**No runtime cost using generics !** Because types are known in compilation time.

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
//  ^^^      ^^^
    fn get_x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.get_x());
}
```

We also use generics for implement methods depends on data type:

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl Point<i32> {
//  ^ notice that there is no <T> here
    fn get_x(&self) -> &i32 {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.get_x()); // integer

    let p = Point { x: 5.0, y: 10.0 };
    // will fail ? :)
    println!("p.x = {}", p.get_x()); // float
}
```

### Generics in the rust standard library

#### Options

An ```Option<T>``` can be ```Some``` or ```None```.

```rust
/*
enum Option<T> {
    Some<T>,
    None,
}
*/

// Returns the first element if exists, None otherwise
fn first_element(list: Vec<i32>) -> Option<i32> {
    if list.len() == 0 {
        None
    } else {
        Some(list[0])
    }
}

fn main() {
    let list = vec!(1, 2, 3, 4);
    
    match first_element(list) {
        None => println!("Empty list!"),
        Some(x) => println!("The first element is {}", x),
    }
    
    match first_element(Vec::new()) {
        None => println!("Empty list!"),
        Some(x) => println!("The first element is {}", x),
    }
}
```

#### Result

A ```Result<T,E>``` can be of type ```Ok(T)``` or can be an error ```Err(E)```.

```rust
/*
enum Result<T, E> {
    Ok(T),
    Err(E),
}
*/

// Result can be a number or an Error message
fn get_middle(list: Vec<i32>) -> Result<i32, &'static str> {
    match list.len() {
        0 => Err("empty list"),
        x if x % 2 == 0 => Err("list has even number of elements"),
        // else
        _ => Ok(list[list.len() / 2]),
    }
}

fn main() {
    println!("{:?}", get_middle(vec![1, 2, 3]));
    println!("{:?}", get_middle(vec![1, 2, 3, 4]));
    println!("{:?}", get_middle(Vec::new()));
}
```

#### Collections

Is the most appropriate candidate for using generics, accepts any kind of data.

```rust
// Naive vector implementation
/* 
impl<T> Vec<T> {
   pub const fn new() -> Vec<T>
}
*/
fn main() {
    let mut list: Vec<i32> = Vec::new();
    //let mut list = Vec::new(); // Type inference
    //let mut list = Vec::<i32>::new(); // turbofish ::<>

    list.push(1);
    list.push(2);
    list.push(3);

    println!("{:?}", list);
}

// See also: std::collections
// https://doc.rust-lang.org/std/collections/index.html

// Sequences: Vec, VecDeque, LinkedList
// Maps: HashMap, BTreeMap
// Sets: HashSet, BTreeSet
// Misc: BinaryHeap
```

#### Wrappers

Given to us by the ```std``` lib, some of them are smart pointers, others are containers that helps us to move things around (for example across threads).

How they work are out of the scope of this notes, we will learn when and how to use it.

```rust
use std::rc::Rc;
use std::cell::{Cell, RefCell};
use std::sync::{Arc, Mutex, RwLock};

fn main() {
    // Heap allocated pointers
    let mybox: Box<i32> = Box::new(42); // heap allocated memory
    let myrc: Rc<i32> = Rc::new(42); // reference-counted

    // Cell with internal mutability
    let mycell: Cell<i32> = Cell::new(42); 
    // mycell.set(0);
    // mycell.get();
    let myrefcell: RefCell<i32> = RefCell::new(42); // with read-write lock
    // myrefcell.borrow()
    // myrefcell.borrow_mut()

    // Thread-safe version
    let myarc: Arc<i32> = Arc::new(42); // atomic reference counted Rc<T>
    let mymutex: Mutex<i32> = Mutex::new(42); 
    // mutex.lock()
    let myrwlock: RwLock<i32> = RwLock::new(42);
    // myrwlock.lock()

    // composition examples:
    // Rc<RefCell<T>> 
    // Rc<Vec<RefCell<T>>> 
    
    // Reference: 
    // https://doc.rust-lang.org/book/first-edition/choosing-your-guarantees.html
}

```

All values in Rust are stack allocated by default.

##### Box

Values can be *boxed* (allocated in the heap) by creating a `Box<T>`. A box is a smart pointer to a heap allocated value of type `T`. When a box goes out of scope, its destructor is called, the inner object is destroyed, and the memory in the heap is freed.

Boxed values can be dereferenced using the `*` operator; this removes one layer of indirection.

```rust
use std::mem;

#[allow(dead_code)]
#[derive(Debug, Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

#[allow(dead_code)]
struct Rectangle {
    p1: Point,
    p2: Point,
}

fn origin() -> Point {
    Point { x: 0.0, y: 0.0 }
}

fn boxed_origin() -> Box<Point> {
    // Allocate this point in the heap, and return a pointer to it
    Box::new(Point { x: 0.0, y: 0.0 })
}

fn main() {
    // (all the type annotations are superfluous)
    // Stack allocated variables
    let point: Point = origin();
    let rectangle: Rectangle = Rectangle {
        p1: origin(),
        p2: Point { x: 3.0, y: 4.0 }
    };

    // Heap allocated rectangle
    let boxed_rectangle: Box<Rectangle> = Box::new(Rectangle {
        p1: origin(),
        p2: origin()
    });

    // The output of functions can be boxed
    let boxed_point: Box<Point> = Box::new(origin());

    // Double indirection
    let box_in_a_box: Box<Box<Point>> = Box::new(boxed_origin());

    println!("Point occupies {} bytes in the stack",
             mem::size_of_val(&point));
    println!("Rectangle occupies {} bytes in the stack",
             mem::size_of_val(&rectangle));

    // box size = pointer size
    println!("Boxed point occupies {} bytes in the stack",
             mem::size_of_val(&boxed_point));
    println!("Boxed rectangle occupies {} bytes in the stack",
             mem::size_of_val(&boxed_rectangle));
    println!("Boxed box occupies {} bytes in the stack",
             mem::size_of_val(&box_in_a_box));

    // Copy the data contained in `boxed_point` into `unboxed_point`
    let unboxed_point: Point = *boxed_point;
    println!("Unboxed point occupies {} bytes in the stack",
             mem::size_of_val(&unboxed_point));
}
```

##### Rc

In the majority of cases, ownership is clear: you know exactly which variable owns a given value. However, there are cases when a single value might have multiple owners. For example, in graph data structures, multiple edges might point to the same node, and that node is conceptually owned by all of the edges that point to it. A node shouldn't be cleaned up unless it doesn't have any edges pointing to it.

To enable multiple ownership, Rust has a type called `Rc<T>`, which is an abbreviation for *reference counting*. The `Rc<T>` type keeps track of the number of references to a value which determines whether or not a value is still in use. If there are zero references to a value, the value can be cleaned up without any references becoming invalid.

We use the `Rc<T>` type when we want to allocate some data on the heap for multiple parts of our program to read and we can’t determine at compile time which part will finish using the data last. If we knew which part would finish last, we could just make that part the data’s owner, and the normal ownership rules enforced at compile time would take effect. Note that `Rc<T>` is only for use in single-threaded scenarios.

For example, we want to reproduce this scenario:

![Two lists that share ownership of a third list](https://doc.rust-lang.org/book/second-edition/img/trpl15-03.svg)

With ```Box<T>``` it is impossible, guess why? *[spoiler]*: Value moved

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

/* 
We could have called a.clone() rather than Rc::clone(&a)
but Rust’s convention is to use Rc::clone in this case.
*/
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

The call to `Rc::clone` only increments the reference count (not *deep copy*), which doesn't take much time. Using `Rc<T>` allows a single value to have multiple owners, and the count ensures that the value remains valid as long as any of the owners still exist.

Via immutable references, `Rc<T>` allows you to share data between multiple parts of your program for reading only. If `Rc<T>` allowed you to have multiple mutable references too, you might violate one of the borrowing rules.

##### Cell and RefCell

Cell types come in two flavors: `Cell<T>` and `RefCell<T>`. `Cell<T>` implements interior mutability by moving values in and out of the `Cell<T>`. To use references instead of values, one must use the `RefCell<T>` type, acquiring a write lock before mutating.

*Interior mutability* is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data; normally, this action is disallowed by the borrowing rules. To mutate data, the pattern uses `unsafe` code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing.

With references and `Box<T>`, the borrowing rules’ invariants are enforced at compile time. With `RefCell<T>`, these invariants are enforced *at runtime*. With references, if you break these rules, you’ll get a compiler error. With`RefCell<T>`, if you break these rules, your program will panic and exit.

The `RefCell<T>` type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

A consequence of the borrowing rules is that when you have an immutable value, you can’t borrow it mutably. For example, this code won’t compile:

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}
```

However, there are situations in which it would be useful for a value to mutate itself in its methods but appear immutable to other code. Code outside the value’s methods would not be able to mutate the value. Using `RefCell<T>` is one way to get the ability to have interior mutability. But `RefCell<T>` doesn't get around the borrowing rules completely: the borrow checker in the compiler allows this interior mutability, and the borrowing rules are checked at runtime instead. If you violate the rules, you’ll get a `panic!` instead of a compiler error.



We will see multi-thread useful types later. 

- ```Arc<T>```
- ```Mutex<T>```
- ```RwLock<T>```

  

Also you can get more info and examples at 

https://manishearth.github.io/blog/2015/05/27/wrapper-types-in-rust-choosing-your-guarantees/

