# Common collections

Unlike the built-in array and tuple types, the data these collections point to is stored on the heap, which means the amount of data does not need to be known at compile time and can grow or shrink as the program runs. Each kind of collection has different capabilities and costs, and choosing an appropriate one for your current situation is a skill you’ll develop over time.

- A vector allows you to store a variable number of values next to each other.
- A string is a collection of characters. We’ve mentioned the String type previously, but in this chapter we’ll talk about it in depth.
- A hash map allows you to associate a value with a particular key. It’s a particular implementation of the more general data structure called a map.

## Storing Lists of Values with Vectors

Vectors allow you to store more than one value in a single data structure that puts all the values next to each other in memory. Vectors can only store values of the same type.

#### In order to create a new vector:

```
let v: Vec<i32> = Vec::new();
// Or
let v = vec![1, 2, 3]; // Which infers the type
```

The Vec<T> type provided by the standard library can hold any type, and when a specific vector holds a specific type, the type is specified within angle brackets.

To add elements in a Vector --> ```v.push(3);```

#### In order to access an element:

```
let v = vec![1, 2, 3, 4, 5];
let third: &i32 = &v[2];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);

// We can use the indexing syntax or the get method
```

#### Iterating over the values:

```
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}


let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}

// To change the value that the mutable reference refers to,
// we have to use the dereference operator (*) to get to the 
// value in i before we can use the += operator.
```
#### Use ```pop``` to remove

#### Using an enum to store multiple types

At the beginning of this chapter, we said that vectors can only store values that are the same type. This can be inconvenient; there are definitely use cases for needing to store a list of items of different types. Fortunately, the variants of an enum are defined under the same enum type, so when we need to store elements of a different type in a vector, we can define and use an enum!

```
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

## Storing UTF-8 Encoded text with Strings
A String is a wrapper over a Vec<u8>.

Rust has only one string type in the core language, which is the string slice str that is usually seen in its borrowed form &str.

The String type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type.

Rust’s standard library also includes a number of other string types, such as OsString, OsStr, CString, and CStr.

#### Creating a new String

```
let mut s = String::new();

let data = "initial contents";

// Using the to_string method to create a String from a string literal
let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();

// We can also use the function String::from to create a String from a string literal
let s = String::from("initial contents");
```

#### Updating a String

```
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2);


let mut s = String::from("lo");
s.push('l');
```


#### Combining string
Often, you’ll want to combine two existing strings. One way is to use the + operator.
For more complicated string combining, we can use the format! macro:

```
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = s1 + "-" + &s2 + "-" + &s3;


let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");
let s = format!("{}-{}-{}", s1, s2, s3);
```

#### Indexing 

In many other programming languages, accessing individual characters in a string by referencing them by index is a valid and common operation. However, if you try to access parts of a String using indexing syntax in Rust, you’ll get an error.


#### Slicing strings

``` 
let s = &hello[0..4];
```

#### Methods for Iterating Over Strings

```
for c in "नमस्ते".chars() {
    println!("{}", c);
}

// Another


for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```


## Storing Keys with Associated Values in Hash Maps

The type HashMap<K, V> stores a mapping of keys of type K to values of type V. It does this via a hashing function, which determines how it places these keys and values into memory.


#### Creating a new hash map

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

Just like vectors, hash maps store their data on the heap.

Another way of constructing a hash map is by using the collect method on a vector of tuples, where each tuple consists of a key and its value. The collect method gathers data into a number of collection types, including HashMap. For example, if we had the team names and initial scores in two separate vectors, we could use the zip method to create a vector of tuples where “Blue” is paired with 10, and so forth. Then we could use the collect method to turn that vector of tuples into a hash map.

```
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

#### Hash Maps and Ownership

For types that implement the Copy trait, like i32, the values are copied into the hash map. For owned values like String, the values will be moved and the hash map will be the owner of those values.

```
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point, try using them and
// see what compiler error you get!
```

#### Accessing values in a HashMap

We can get a value out of the hash map by providing its key to the get method.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

Here, score will have the value that’s associated with the Blue team, and the result will be Some(&10). The result is wrapped in Some because get returns an Option<&V>; if there’s no value for that key in the hash map, get will return None.

We can iterate over each key/value pair in a hash map in a similar manner as we do with vectors, using a for loop.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

#### Overwriting a value

If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced.

```
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

#### Inserting a value only if the key has no value

Hash maps have a special API for this called entry that takes the key you want to check as a parameter. The return value of the entry function is an enum called Entry that represents a value that might or might not exist.

```
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

The or_insert method on Entry is defined to return a mutable reference to the value for the corresponding Entry key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value.

#### Updating a value based on the old value

Code that counts how many times each word appears in some text. We use a hash map with the words as keys and increment the value to keep track of how many times we’ve seen that word. If it’s the first time we’ve seen a word, we’ll first insert the value 0.

```
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

