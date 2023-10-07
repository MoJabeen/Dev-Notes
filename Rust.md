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
```

# Cargo

Rusts build system and package manager. Libraries used in code referred to as *dependencies*. 

### Create project

$$ cargo\;new\;[name] $$
- Creates toml file
	- Dependencies tracked in .toml file.
- src/ dir with main file

### Running code

Can either build then run the binary or simply use run which will build then execute.

$$ cargo\;build$$
$$cargo\;run$$
For a quick check for compiler errors without building use:
$$cargo\;check$$
#### Release

To op
