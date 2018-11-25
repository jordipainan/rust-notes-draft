## Using Functional Programming Loops

### Iterators

See docs at: https://doc.rust-lang.org/std/iter/trait.Iterator.html

- Iterator (What we seen before)
- Iterator adaptor: Iterator => Iterator
  - [1,2,3] plus 1 => [2,3,4]
  - [1,2,3] squared => [1,4,9]
- Consuming adaptor: Iterator => Something else
  - Sum of [1,2,3] => 6
  - Max of [1,2,3] => 3

#### Iterator adaptors

```rust
fn main() {
    let numbers = vec![1,2,3,4];
    let plus_one = numbers.iter().map(|x| x+1);
    plus_one.for_each(|x| println!("{}", x));
}
```

- ```map()``` is an iterator adaptor that basically takes a iterator and applies a closure, which is kind of a function on top of every element and then returns another list.
  - Closure example: ```(|params| function_body)``` 
    - ```(|x| x+1)``` => Take x and returns x+1
    - ```(|x| println!("{}", x))``` => Prints x

- ```for_each()``` another iterator that takes a closure and returns each element.

- ```filter()``` is a useful iterator, takes a closure and filter out some elements based on the closure.

  Example: Filters numbers greater than 2

  ```let larget_than_two = numbers.into_iter().filter(|&x| x > 2);```

Closures takes the surrounding scope, so we are able to do something like this:

```rust
fn main() {
    // scope a'
 	let numbers = vec![1,2,3,4];
 	let y = 1;
 	let plus_one = numbers.iter().map(|x| x+y);
 	plus_one.for_each(|x| println!("{}", x));
    // end scope a'
}
```

The surrounding scope is all the main function.

#### Consuming adaptors

```rust
fn main() {
    let numbers: vec![1,2,3,4];
    let sum: i32 = numbers.iter().sum();
    println!("Sum: {}", sum);
    // We use unwrap here because max() returns an Option
    let max: &i32 = numbers.iter().max().unwrap();
    println!("Max: {}", max);
}
```



### When to use functional programming loops ?

- Doing a simple operation on all elements of a list without exceptions

  - Let's take a example of *fold*. This function has a initial value and an accumulator that holds the previous result of the operation performed in the function body, the value is the element used from the iterator. It is useful to make some aggregations.

    ```rust
    fn main() {
        let numbers = vec![1,2,3,4];
        // 0: Initial value
        // sum: Accumulator
        // val: Element
        let sum: i32 = numbers.iter().fold(0, |sum, val| sum+val);
        println!("Sum: {}", sum);
    }
    
    /*	fold() example:
    ---------------------
    |	sum	val	return	|
    |	0*	1	1	  	|	
    |	1	2	3     	|
    |	3	3	6     	|
    |	6	4	10    	|	* Initial value
    ---------------------
    */
    ```

- **Chaining** (chain multiple operations together), for example applying a mathematical operation on elements of a list, then filter out the results and again apply some operations to the returned list.

  ```rust
  fn main() {
      let numbers = vec![1,2,3,4];
      let squared_sum: i32 = numbers.iter()
      							  // Returns an iterator
      							  .map(|x| x*x)
      							  .sum();
      println!("Squared_sum: {}", squared_sum);
  }
  ```

- **Lazy evaluation**: Means that operations are not applied unless we want the result ```collect()```.

  ```rust
  fn main() {
      let numbers = vec![1,2,3,4];
      let plus_one_iter = numbers.iter().map(|x| x+1);
      // This will print: Map { iter: Iter([1,2,3,4]) }
      println!("{:?}", plus_one_iter);
      // This will print: [2,3,4,5]
      let plus_one: Vec<i32> = plus_one.iter().collect();
      println!("{:?}", plus_one);   
  }
  ```

  ```rust
  fn main() {
      let numbers: Vec<32> = (1..) // 1 to infinity
      	.map(|x| x+1) // [2,3,4,5,6,...]
      	.map(|x| x*x) // [4,9,16,25,36,...]
      	.take(5)	  // "take" the first five iterations
      	.collect();   // collect them into a vec
      println!("{:?}", numbers);
  }
  ```
