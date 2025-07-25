---
author:
  - Dainish Jabeen
bibliography:
  - refs.bib
date: 2023-10-01
title: Rust
---
# Introduction

Rust is balancing high level ergonomics and low-level control, allowing safe control over low level details.

Rust is a compiler static based language, meaning to run the code you first compile using **rustc**.

# Basics

- fn for function
- curly bracket based
- uses semicolons for line separation   

```rust
fn main() {
	println!("meh");
}
//comments

//curly brackets for formatted print

println!("{0} meh {1}", "ex1","ex2")
//Output : ex1 meh ex2


//params and return tuple
fn meh(pair:(i32,bool)) -> (bool,i32) {

}

```

#### Debug Printing

Format std::fmt has traits fmt::Debug and fmt::Display. All types that wush to use a trait requires an implementation. The types in the std library have an automatic implementation. A new implementation can be derived when using the Debug trait for a new type using **attributes**:

```rust

#[derive(Debug)]
struct Meh(i32);

fn main() {

	println!("{:?}",Meh(2)); // is now printable {:?} - debug trait
	println!("{:#?}",Meh(2)); //pretty printing
}

```


### Display Printing

Manually implementing a new type for display printing:

Your choice of format is setup in the impl write!.

```rust

use std::fmt;

struct Meh(i32);

impl fmt::Display for Meh {
	//Use this signature for fmt 
	fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
		write!(f,"{}",self.0); //self.0 is for the value in Meh (has no name)
	}
}

```

## Variables

Can be inferred, will be automatically be immutable add mut. Scope is within {} block. Can be shadowed ie temp overwritten within inner block. Better to always initialise. 

```rust

const THRESHOLD: i32 = 10;

let meh: f64 = 1.0 //float64 var
let hem: i32 = 99 //int var
let blah: u32 = 19 //unsigned int
let mut blah:u32 = 19 //mutable unsigned int

let meh = 1.0 //dont need to be type annotated


let arr : [i32; len] = []; //use arr as pointer for params
let len = arr.len()

let x = arr[1 .. 4] //slice or array start index 1 to 4

//non fixed arrays are vectors
let mut vec = Vec::new(i32);

let long_tupe = (1u8,2u16,true) //tupes hold multiple types

struct Meh {
	name : String,
	age : u8,
}

enum Blah {
	x,
	Y(char), //struct
	Z( inital:char)
}

let press = Blah::(Y('d')); //enum value 

```

## Literals

Can use underscores in numbers for easier reading ie 1_000 is 1000. Can also use e notation (assumes f64). Add sufix of type to number.

```rust

println!("{}",1u32)

```

### Conversion

```rust

let decimal = 65.4321_f32;

let integer = decimal as u8; //needs explicit conversion

//For custom types need new impl of std::convert::From, from allows you to create yourself from another type

use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let num = Number::from(30);
    
    
    let num: Number = int.int
    println!("My number is {:?}", num);
}

//Into is used when going from a var to another var

fn main() {
    let int = 5;
    // Try removing the type annotation
    let num: Number = int.into();
    println!("My number is {:?}", num);
}

//! Use TryFrom and TryInto for fallible conv that may cause Err()

//use .to_string() for conv to string

struct Circle {
    radius: i32
}

println!("{}", circle.to_string());

//string to int
let parsed: i32 = "5".parse().unwrap();

```

### Loops

```rust

 for i in 0..xs.len() + 1 { // Oops, one element too far!
        match xs.get(i) { //safely access element without compile errors
            Some(xval) => println!("{}: {}", i, xval),
            None => println!("Slow down! {} is too far!", i),
            //use to stop compile error
        }
    }


// loops inclusive a to exclusive b
for i in a..b {

}

// use xs.expect(i) to exit code if out of bounds.

//inf loop
loop {

}

//Use 'labels to specify which loop to break

'outer: loop {
	println!("Entered the outer loop");

	'inner: loop {
		println!("Entered the inner loop");

		// This would break only the inner loop
		//break;

		// This breaks the outer loop
		break 'outer;
	}

	println!("This point will never be reached");
}

println!("Exited the outer loop");

//set value using loop

let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

```

### Conditions


```rust

//has match instead switch

match val {

	ex => func,
	ex3 => fun3,
	a..b => range,
	_ => default,

}

//can also use match to set a value

let x = match s {
	1 => 2,
	3 => 4,
}

match array {
        // Binds the second and the third elements to the respective variables
        [0, second, third] =>
        // Single values can be ignored with _
        [1, _, third] => println!(
         // You can also bind some and ignore the rest
        [-1, second, ..] => println!(
        // Or store them in another array/slice (the type depends on
        // that of the value that is being matched against)
        [3, second, tail @ ..] => println!(
        // Combining these patterns, we can, for example, bind the first and
        // last values, and store the rest of them in a single array
        [first, middle @ .., last] => println!(
    }

struct Foo {
        x: (u32, u32),
        y: u32,
    }

match foo {
	Foo { x: (1, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),

	// you can destructure structs and rename the variables,
	// the order is not important
	Foo { y: 2, x: i } => println!("y is 2, i = {:?}", i),


//! Can also include if statments in the match (guard)

let number: u8 = 4;

    match number {
        i if i == 0 => println!("Zero"),
        i if i > 0 => println!("Greater than zero"),
        _ => unreachable!("Should never happen."),
    }

//Binding to condition value and use it usin @

 match age() {
        0             => println!("I haven't celebrated my first birthday yet"),
        // Could `match` 1 ..= 12 directly but then what age
        // would the child be? Instead, bind to `n` for the
        // sequence of 1 ..= 12. Now the age can be reported.
        n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
        n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),

//use allows for auto scoping

enum Status {
    Rich,
    Poor,
}

use crate::Status::*

let m = Rich //same as Status::Rich

```

# Cargo

Rusts build system and package manager. Libraries used in code referred to as *dependencies*. 

### Create project

$$ cargo\;new\;[name] $$
- Creates toml file
	- Dependencies tracked in .toml file.
- src/ dir with main file

### Running code

Can either build then run the binary (found in target/debug) or simply use run which will build then execute.

$$ cargo\;build$$
$$cargo\;run$$
For a quick check for compiler errors without building use:
$$cargo\;check$$
#### Release

To optimise code for release use:

$$cargo\;build\;--release$$ This creates a binary in target/release instead of target/debug
