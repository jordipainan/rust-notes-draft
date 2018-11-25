# Writing and Running a Rust Program

- File: main.rs
- Content:
```
fn main() {
	println!("Hello, world!");
}
```

- Compile: ``` rustc main.rs ```
- Exec: ``` ./main ```

# Creating, Building and Running a Cargo Project
Is the Rust's build system and package manager.

- Version: ``` cargo --verison ```<br>
- Create project: ``` cargo new hello_project ```<br>
We have a file called Cargo.toml which is Cargo's configuration format (TOML = Tom's Obvious, Minimal Language). 
Also we have a folder called src with a main.rs file inside and it initialized a new Git repository with a .gitignore.
- Build: ``` cargo build ``` Creates an executable file in target/debug/ <br>
- Running cargo build for the first time also causes Cargo to create a new file at the top level: Cargo.lock. <br> 
This file keeps track of the exact versions of dependencies in your project.
- Run: ``` cargo run ``` Compiles the code and executes it. <br>
- Check: ``` cargo check ``` Is much faster than ``` cargo build ``` because it skips the step of producing and executable. <br>
If you’re continually checking your work while writing the code, using ``` cargo check ``` will speed up the process!

- Building for release: ``` cargo build --release ``` Compiles with optimizations. Use this for benchmarks

# To Cargo or not to Cargo

With simple projects, Cargo doesn’t provide a lot of value over just using rustc, but it will prove its worth as your programs become more intricate. 
With complex projects composed of multiple crates, it’s much easier to let Cargo coordinate the build. 
