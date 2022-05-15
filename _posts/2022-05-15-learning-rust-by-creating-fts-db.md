---
layout: post
title: Learning Rust - creating a full-text search engine
categories: [rust]
excerpt: In this post, I try to reproduce the article "Let's build a Full-Text Search engine" to learn more about full-text search databases and Rust.
last_modified_at:
---

1.  [Read and load XML file](#orgfa86592)
2.  [Pass-By-Value or Pass-By-Reference](#org6a4819a)
3.  [Read file](#org7fddb5d)
4.  [Processing the data](#orge96434e)
5.  [Traits](#org2c6373b)
6.  [Searching for a string](#org3ecb930)
7.  [Pass by move](#orga513cf2)

<br>

In this post, I try to reproduce the article "[Let's build a Full-Text Search engine](https://artem.krylysov.com/blog/2020/07/28/lets-build-a-full-text-search-engine/)" to learn more about full-text search databases and Rust. Please note: I graduated in computer engineering, the C language was the first language I ever saw in my life, and I use Python and JS daily. That's the angle I will be using through the post.

<a id="orgfa86592"></a>

## Read and load XML file

Goal number one is to read some data from an XML export, after that load it in memory and perform searches on it. As a data source, I&rsquo;m using the same part of the abstract of English Wikipedia described in the original article, with over 600K documents.

Google: &ldquo;rust read xml file&rdquo;. Check posts. Links led me to crates.io and to the quick-xml crate. More research also resulted in good answers (<https://simplabs.com/blog/2020/12/31/xml-and-rust/>) which taught me different ways to parse files and different crates to use.

``` rust
use std::path::Path;

fn load_documents(path_str: String) {
    // Create a path to the desired file
    let path = Path::new(path_str);
}
```

The code above won&rsquo;t work.


<a id="org6a4819a"></a>

## Pass-By-Value or Pass-By-Reference

Something that caught my attention is that you need to be intentional about passing parameters by reference or by value. The `Path` initializer expects a reference and not a struct `std::string::String`.

``` rust
use std::path::Path;

fn load_documents(path_str: &str) {
    // Create a path to the desired file
    let path = Path::new(&path_str);
}
```

Rust is strictly a pass-by-value language. References are first-class citizens which means that references can be passed - by value - to functions. Therefore, you will often see intentional uses of references to avoid copying values to the stack of inner functions.

Another change in the code is `String` -> `&str` which is the appropriate format for when you pass a regular string as a parameter (check the snippet below).

-   ``String`` is the dynamic heap string type, like Vec: use it when you need to own or modify your string data.
-   ``str`` is an immutable sequence of UTF-8 bytes of dynamic length somewhere in memory. Since the size is unknown, one can only handle it behind a pointer (`&str`).

``` rust
use std::path::Path;

fn load_documents(path_str: &str) {
    // Create a path to the desired file
    let path = Path::new(&path_str);
}

fn main() {
    let documents = load_documents("data");
}
```

- Reference #1: <https://blog.ryanlevick.com/rust-pass-value-or-reference/>
- Reference #2: <https://stackoverflow.com/questions/24158114/what-are-the-differences-between-rusts-string-and-str>
- Reference #3: <https://doc.rust-lang.org/rust-by-example/std_misc/file/open.html>


<a id="org7fddb5d"></a>

## Read file

Next step is to open a file in read-only mode to avoid accidents. To deal with failures, Rust bakes in a type called Result, whose Ok variants carry successful results—the count of bytes transferred, the file opened, and so on—and whose Err variants carry an error code indicating what went wrong. That is, `Result<T, E>` could have one of two outcomes:

-   Ok(T): An element T was found
-   Err(E): An error was found with element E

Rust does not have exceptions, therefore all errors are handled using either the Result type or the panic macro.

``` rust
use std::path::Path;
use std::fs::File;
use std::io::prelude::*;

fn load_documents(path_str: &str) {
    // Create a path to the desired file
    let path = Path::new(&path_str);
    let display = path.display();

    // Open the path in read-only mode, returns `io::Result<File>`
    let mut file = match File::open(&path) {
        Err(why) => panic!("couldn't open {}: {}", display, why),
        Ok(file) => file,
    };

    // Read the file contents into a string, returns `io::Result<usize>`
    let mut xml = String::new();
    match file.read_to_string(&mut xml) {
        Err(why) => panic!("couldn't read {}: {}", display, why),
        Ok(_) => print!("{} ok", display),
    }
}

fn main() {
    let documents = load_documents("data");
}
```

- Reference #1: <https://doc.rust-lang.org/rust-by-example/std_misc/file/open.html>
- Reference #2: <https://doc.rust-lang.org/rust-by-example/error/result.html>
- Reference #3: <https://doc.rust-lang.org/std/result/>


<a id="orge96434e"></a>

## Processing the data

The step below will give you a string with all the content of the XML file. To actually use it and iterate over it, you will need to parse it to an object. quick-xml support an additional feature called serialize provided by another crate called serde which is a framework for serializing and deserializing Rust data structures.

Here a new word appeared: Traits. Let&rsquo;s skip that and come back to it later.

The Serde ecosystem consists of data structures that know how to serialize and deserialize themselves. These data structures implement the Serde&rsquo;s Serialize and Deserialize traits. Besides, Serde provides a derive macro to generate serialization implementations for structs in your own program.

This functionality is based on Rust&rsquo;s #[derive] mechanism, just like what you would use to automatically derive implementations of the built-in Clone, Copy, Debug, or other traits. It is able to generate implementations for most structs and enums including ones with elaborate generic types or trait bounds.

The only step you need to do before running the following code is to add `serde = { version = "1.0", features = ["derive"] }` as a dependency in Cargo.toml.

``` rust
use std::fs::File;
use std::io::prelude::*;
use std::path::Path;
use serde::Deserialize;
use quick_xml::de::{from_str};

#[derive(Debug, Deserialize, PartialEq)]
struct Item {
    title: String,
    url: String,
    r#abstract: String,
    id: Option<u64>
}

#[derive(Debug, Deserialize, PartialEq)]
struct Document {
    doc: Vec<Item>
}

fn load_documents(path_str: &str) -> Document {
    // Create a path to the desired file
    let path = Path::new(&path_str);
    let display = path.display();

    // Open the path in read-only mode, returns `io::Result<File>`
    let mut file = match File::open(&path) {
        Err(why) => panic!("couldn't open {}: {}", display, why),
        Ok(file) => file,
    };

    // Read the file contents into a string, returns `io::Result<usize>`
    let mut xml = String::new();
    match file.read_to_string(&mut xml) {
        Err(why) => panic!("couldn't read {}: {}", display, why),
        Ok(_) => print!("{} ok", display),
    }
    print!("'{}'", xml);
    let document: Document = from_str(&xml).unwrap();
    return document;
}
```

-   Reference #1: <https://serde.rs/data-model.html#types>
-   Reference #2: <https://serde.rs/derive.html>
-   Reference #3: <https://crates.io/crates/quick-xml>


<a id="org2c6373b"></a>

## Traits

Traits are for Rust what interfaces are to other languages. However, not quite. A trait is a collection of methods defined for an unknown type: Self.

Traits appear to me as a way to support duck-typing, in which the implementation of the common methods stays in a separate class (called trait). This separate class has the syntax `impl <trait> for <type>`. The crème de la crème here is that some traits can be automatically derived (compiler implements traits automatically).

They can be used as function parameters to allow the function to accept any type that can do x, where x is some behavior defined by a trait. We can also use trait bounds to refine and restrict generics, such as by saying we accept any type T that implements a specified trait.

-   Reference #1: <https://doc.rust-lang.org/rust-by-example/trait.html>
-   Reference #2: <https://blog.logrocket.com/rust-traits-a-deep-dive/>


<a id="org3ecb930"></a>

## Searching for a string

This part was quite intuitive. I first tried to search by term, using a simple contains, then a regex search.

``` rust
use std::fs::File;
use std::io::prelude::*;
use std::path::Path;
use std::time::Instant;
use serde::Deserialize;
use regex::Regex;
use quick_xml::de::{from_str};


fn search_str(docs: &Vec<Item>, term: &str) -> Vec<usize> {
    let mut indexes = Vec::with_capacity(20);
    for (pos, e) in docs.iter().enumerate() {
        if e.title.contains(term) {
            indexes.push(pos)
        }
    }
    return indexes;
}

fn search_regex_str(docs: &Vec<Item>, term: &str) -> Vec<usize> {
    let formatted = format!(r"(?i)\b{}\b", term);
    let re = Regex::new(formatted.as_str()).unwrap();

    let mut indexes = Vec::with_capacity(20);
    for (pos, e) in docs.iter().enumerate() {
        if re.is_match(&e.title) {
            indexes.push(pos)
        }
    }
    return indexes;
}

fn main() {
    let documents = load_documents("data");
    println!("\n\nTotal {}", documents.doc.len());

    let now = Instant::now();
    let result = search_str(&documents.doc, "cat");
    let elapsed = now.elapsed();
    println!("Elapsed: {:.2?}", elapsed);
    println!("Search: cat; Len: {}; First: {}", result.len(), documents.doc[result[0]].title);

    let now = Instant::now();
    let result = search_str(&documents.doc, "Cat");
    let elapsed = now.elapsed();
    println!("Elapsed: {:.2?}", elapsed);
    println!("Search: Cat; Len: {}; First: {}", result.len(), documents.doc[result[0]].title);

    let now = Instant::now();
    let result = search_str(&documents.doc, "CAT");
    let elapsed = now.elapsed();
    println!("Elapsed: {:.2?}", elapsed);
    println!("Search: CAT; Len: {}; First: {}", result.len(), documents.doc[result[0]].title);
}

// Without regex using contains
// Total 646392
// Elapsed: 293.04ms
// Search: cat; Len: 3176; First: Wikipedia: Telecommunications in Antigua and Barbuda
// Elapsed: 287.67ms
// Search: Cat; Len: 2014; First: Wikipedia: Catatonia
// Elapsed: 227.92ms
// Search: CAT; Len: 13; First: Wikipedia: CATIA

// With regex
// Total 646392
// Elapsed: 4.20s
// Search: cat; Len: 197; First: Wikipedia: Cat
// Elapsed: 4.29s
// Search: Cat; Len: 197; First: Wikipedia: Cat
// Elapsed: 4.12s
// Search: CAT; Len: 197; First: Wikipedia: Cat
```

-   Reference #1: <https://doc.rust-lang.org/std/vec/struct.Vec.html#method.with_capacity>
-   Reference #2: <https://doc.rust-lang.org/std/macro.format.html>


<a id="orga513cf2"></a>

## Pass by move

Back to the main function, at some point, I decided to pass the documents struct by value.

``` rust
fn main() {
    let documents = load_documents("data");
    println!("\n\nTotal {}", documents.doc.len());

    let now = Instant::now();
    let result = search_str(documents.doc, "cat");   // <--------- HERE
    let elapsed = now.elapsed();
    println!("Elapsed: {:.2?}", elapsed);
    println!("Search: cat; Len: {}; First: {}", result.len(), documents.doc[result[0]].title);
}

// ERROR: raises the following on ``documents.doc[result[0]].title``:
// borrow of moved value: `documents.doc`
// value borrowed here after move
// note: move occurs because `documents.doc` has type `std::vec::Vec<Item>`, which does not implement the `Copy` trait
```

The Copy trait is only implementable by data types that can be put on the stack, and because Vec must go on the heap, it cannot implement Copy. What happens here is that the borrow checker is enforcing ownership to make memory safety guarantees without needing a garbage collector. Why ownership do you ask? To make sure different points of the flow won&rsquo;t interfere with each other, imagine that, copying the ``documents`` would copy the Vec struct that contains three values: a length, a capacity, and a pointer to the actual data on the heap. The copy would lead to two pointers to the same data on the heap, if one of them is dropped, pointer in other Vec would now point to junk data.

This is all enforced at compile-time and guarantees that it won&rsquo;t happen at running time.

-   Reference #1: <https://www.reddit.com/r/rust/comments/9fsyy3/rust_passbyvalue_or_passbyreference/>
-   Reference #2: <https://blog.logrocket.com/introducing-the-rust-borrow-checker/>
-   Reference #3: <https://blog.logrocket.com/understanding-ownership-in-rust/>

------

To be continued with a inverted index implementation.
