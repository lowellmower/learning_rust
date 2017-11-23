Lexicon:

Enumerations: often referred to as enums. An enumeration is a type that can have a fixed set of values, and those values are called the enum’s variants.
Variants: The fixed set of values of an enumeration. Example being cmp::Orderings variants Greater, Equal, Less
Growable: A mutable list which can be dynamically sized
The Prelude: The list of things that Rust automatically imports into every Rust program
Result: An enum, just as String is, to which there is a generic type and a type specific to packages, e.g. `io::Result` which has its own variants. For example, the variants for a `Result` returned from `match` will be `Ok` and 
`Err`

DAY ONE:
Notes:
*“!” identifies macros, eg println!() is a macro where as println would be a defined function, example err:
```
hello_world :> rustc main.rs
error[E0423]: expected function, found macro `println`
--> main.rs:2:5
|
2 |     println("Hello, World!");
|     ^^^^^^^ did you mean `println!(...)`?
error: aborting due to previous error
```
* Compiling and running are in separate steps similar to C with clang
* Cargo (package manager)
* Start a project with cargo and some things noticed are:
	- initiates git
	- makes a gitignore file with:
	```
	/target/
	**/*.rs.bk
	```
	- Creates an authors file
	- puts code in a /src dir (similar to go)

Speaking of go, it already feels like RUST has some of the things right from the beginning, specifically, Go is notoriously troublesome in package management and dependency resolution with little in the way of a "built in" option. I've always defaulted to glide and even then, it is not ideal... Rust appears to ship with a solution in mind and out of the box but insinuates there are alternatives if the programmer chooses so:

"The last line, [dependencies], is the start of a section for you to list any crates (which is what we call packages of Rust code) that your project will depend on so that Cargo knows to download and compile those too. "

"Cargo expects your source files to live inside the src directory so that the top-level project directory is just for READMEs, license information, configuration files, and anything else not related to your code"

Also seems easy to change a program which may have been started without Cargo simply by moving the system's files into a /src dir and creating the Cargo.TOML file.

Slightly different build process than `rustc` - instead use `cargo build` which puts the binary in a path of `$PROJECT_PATH/target/debug/binary`:
```
hello_cargo [] :> cargo build
   Compiling hello_cargo v0.1.0 (file:///Users/lowellmower/rust/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 4.38 secs
hello_cargo [] :> ./target/debug/hello_cargo 
Hello, world!
```

I'm loving the built in sem ver (link) and that this major part of writing software was included from the very beginning.

Basic steps for using Cargo:
```
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build 
# or cargo run
```

[Cargo docs](http://doc.crates.io/guide.html) - just so sad though - Y U NO HTTPS?!

I LOVE THE COMPILER:
```
guessing_game [master] :> cargo run
   Compiling guessing_game v0.1.0 (file:///Users/lowellmower/rust/guessing_game)
error: expected expression, found `,`
  --> src/main.rs:19:32
   |
19 |     io::stdnin().read_line(&mut, guess).expect("Failed to read line");
   |                                ^

error: expected one of `.`, `;`, `<`, `?`, `break`, `continue`, `false`, `for`, `if`, `loop`, `match`, `move`, `return`, `true`, `unsafe`, `while`, `}`, or an operator, found `,`
  --> src/main.rs:19:32
   |
19 |     io::stdnin().read_line(&mut, guess).expect("Failed to read line");
   |                                ^ expected one of 18 possible tokens here
```

One must explicitly allow for mutation of variables:
```
let foo = 5; // immutable
let mut bar = 5; // mutable
```

The compiler will catch this also, so if you've allocated a new variable which is immutable and there is a place within the program where that is mutated, then it will bark at you:
```
16 |     let guess = String::new();
   |         ----- consider changing this to `mut guess`
...
19 |     io::stdin().read_line(&mut guess).expect("Failed to read line");
   |                                ^^^^^ cannot borrow mutably
```

I know this doesn't sound like world shattering news but it sure feels like it. In my mind this is like having on the fly constants which, from the sounds of it, will be garbage collected. <3 

The types:

"String is a string type provided by the standard library that is a growable, UTF-8 encoded bit of text" - growable? I like the sounds of this...

types are declared before (like Java) `&mut guess` and when passed as params to a function `read_line(&mut guess)` - odd that we don't see to care that it is of type String but more that it is `&mut` - also `&` indicates it is a reference giving us a way to let multiple parts of our code access one piece of data without needing to copy that data into memory multiple times.

"For now, all you need to know is that like variables, references are immutable by default. Hence, we need to write &mut guess rather than &guess to make it mutable"

The Result types are enumerations, often referred to as enums. An enumeration is a type that can have a fixed set of values, and those values are called the enum’s variants. Chapter 6 will cover enums in more detail

`.expect()`, if provided as a static method of type Result (or in this case `io::Result`) will check for errors and return the msg provided, crashing the program at that point.

DAY TWO:
Not sure how I feel about having to explicitly declare an external crate atop having to have the import statement `use` as well, e.g.
```
extern crate rand;

use rand::Rng;
```
Though the documentation reads that the "traits" brought into scope with use are something which is documented in the crate's docs and I suppose this is similar to the python import strategy of `from` `use`

Just like Go, I enjoy the static typing and the type inference

I like how shadowing gives the illusion and feel of type casting, in that converting a string to an integer feels pyhtony:
```
let mut str_or_int_var = String::new();
// assignment of some value to the var of type string
let str_or_int_var: u32 = str_or_int_var.trim().parse()
    .expect("Failed to parse str to int");
```
The colon (`:`) after tells Rust we’ll annotate the variable’s type.

I like how rust will infer the type used in future and past comparisons of the variable, e.g. if one is looking to compare an unsigned 32 bit integer with a randomly generated number which may result in a number larger than 32 bits, the compiler will warn:
```
warning: literal out of range for u32
  --> src/main.rs:14:54
   |
14 |     let secret_num = rand::thread_rng().gen_range(1, 10100000000);
   |                                                      ^^^^^^^^^^^
   |
   = note: #[warn(overflowing_literals)] on by default
```

DAY THREE:
Variables and mutability - by default variables are immutable and must be declared as mutable otherwise

Immutability is not as limiting as it sounds given rust's allowance for shadowing. The difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value, but reuse the same name

```
// this is allowed and x is freely able to change
let mut x = 5;
x = 6;

// this is also allowed but x will be immutable after
// the change, essentially recreating the old variable
let y = 10;
let y = y + 2;

// another pattern where shadowing is useful
// GOOD (will allow "dynamic" reassignment)
let spaces = "   ";
let spaces = spaces.len();

// BAD (compile error type mismatch)
let mut spaces = "   ";
spaces = spaces.len();
```
