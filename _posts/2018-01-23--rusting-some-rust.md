---
layout: post
title: Wavetab Dev Diary - Getting started with Rust
github_comments_issueid: "2"
---

Welcome to the first update on my Rust project. 

In general, I have been mostly skimming through some tutorials and books on different relevant topics.
Futhermore, I have created the code repository and started writing some code.
In this post, I will write mainly about some setup stuff and my first impressions tackling the project.

So, in my Rust project ([github repository](https://github.com/nyro-dev/wavetab-rust)), where I am trying to build a tool that converts raw audio (waves/signals) into guitar tabs, started with ... **lots** of reading!
Also, as I just started learning the language, there isn't much code here yet.

To begin with, I've read through a large portion of *The Book* (as they like to call it), namely [The Rust Programming Language](https://doc.rust-lang.org/book/second-edition/).
It is filled with a lot of information and small examples and I think it is very well written and easy to follow/understand.
When I started coding, I also found [rustbyexample](https://rustbyexample.com) to be a very good resource on how to use the standard library and the language in general.

So, what are my first impressions on the language and what have I achieved so far?
First off, I really like *cargo*, Rust's build system and package manager, as it is very easy to use.
Cargo is also used for creating projects, which is helpful because it also creates the necessary project structure for you.
Rust enforces a specific project structure, but it is pretty straight forward:
- In the project root directory, you have a file (`Cargo.toml`) that contains information about the project and external dependencies, as well as a folder named *src* where you put (you guessed it) your source files
- You wan't an executable? Then you need a source file named `main.rs` 
- Your project is a library? Then you need a source file named `lib.rs`
- `main.rs` (or `lib.rs`) is the entry point of your program (or library)
- In Rust you group functionality in *modules*, where the module name equals the source file name
- I think that describes the biggest part of it

When you are looking for 3rd party libs (so called *crates*, I think), you can just look for existing stuff at [crates.io](https://crates.io/).

### Project Setup and Code
Next, I want to give a very brief description on how I've set up my programming environment and the project itself.
Afterwards we will look at some code ("Warning": In this post I will try to explain some of the basic concepts in Rust, thus this section will end up pretty lengthy; For the posts to come, I will focus more on the exciting stuff I coded).

As there isn't really a standard IDE for Rust yet, I looked at the chart on a website called [areweideyet.com](https://areweideyet.com/), to see which code editors and IDEs already have plugins that support Rust.
From the pretty extensive list, I chose to go with Visual Studio Code, because I kinda like that editor and there is a comprehensive Rust plugin for that editor (although I belief only parts of the plugin are working for me right now, but it wasn't too bad until now).

Having everything installed, the first thing to do is to create a new project.
I decided to code my converter as a library, with an eventual use as a backend for a c# gui at a later stage, so we simply start up our cmd (on windows) in a directory of our choice and run `cargo new wavetab`.
With this command, we tell cargo to create a new library project named *wavetab*.
Cargo then creates a project folder for us which includes the `Cargo.toml` and a source folder containing our `lib.rs`.
For playing around with our library (without the need of our graphical frontend), I added another source file and named it `main.rs`.
In this way, we can either build the library or run `main.rs` by calling `cargo build` or `cargo run` respectively (from the project's root folder in a cmd window).
I also created a folder called `data` in which I will store some *wave* files with different guitar recordings for testing (as of right now there is only one, containing a short note sequence).

Before we dive into some code, I want to sum up my plans for approaching the problem of converting a recording into a guitar tab.
I'm planning to look at it from two perspectives: classic dsp and machine learning.
I am no dsp guru, but my intuition for a dsp pipeline that extracts the notes played from the recorded wave (I will call this (discrete) *signal* in the remainder of this post) goes something like this:
1. Eventually apply some filters to the signal
2. Identify the points of time a new note or chord is being played
3. Subdivide the signal to separate subsequently played notes or chords
4. Apply fast fourier transform on signal sections to extract frequencies
5. Convert frequencies to notes (like a') or chords (like E minor)

I imagine such a pipeline to have a bunch of pitfalls and to be difficult to tune towards different edge cases.
Because of that, I am also looking at the lazy way of doing things: machine learning.
For this, I will (as a first attempt) treat the note extraction problem as a search problem and provide a genetic algorithm with the notes it is able to play and let it figure out which ones to play and when to play those.
In this approach, I expect problems with decreasing amplitudes in the signal when a note is being held for a period of time.
When the algorithm just generates perfect signals from the inputs, there aren't any decreasing amplitudes, which might affect the loss calculations in the optimzation.
But I guess we will see what happens when we get to that point. 
In the long run, I aim at combining both approaches in some way to use the strengths of each approach.

That being said, let's look at some code!
So first off, we will add two dependencies we will need: *hound* (for loading wave files) and *num* (for some numeric traits, a trait is a little bit like an interface).
This is as simple as adding the following to our `Cargo.toml`.
```
[dependencies]
hound = "3.3.0"
num = "0.1.41"
```

Following this, we import these crates in our `lib.rs`:
```rust
extern crate hound;
extern crate num;
```

Next, I wanted to split the library's functionality into different modules:
- analysis: parses input signal
- tab_generation: generates a guitar tab from analysed signal
- utils: general utility, extension methods and so on
- plotting: for plotting our signal (and maybe other stuff)

We define all of the modules of our library in `lib.rs` by adding
```rust
pub mod plotting;
mod analysis;
mod tab_generation;
mod utils;
```
Here, `mod` is the keyword for, you guessed it, a module.
By adding `pub` in front of the plotting module, we make it accessible outside of the library (i.e. for callers of the library; In the current state, I am calling the plotting function from outside the library).
With `mod <module-name>`, we are just declaring, that there is a module with the given name.
Because of that, we also have to add the following source files: `plotting.rs`, `analysis.rs`, `tab_generation.rs` and `utils.rs` (the file names have to match the module names).
As an alternative, we could have implemented the modules in the `lib.rs` file, by also adding the module functionality:
```rust
mod analysis {
    // functionality
}
```
This can be useful when we don't want to add source files for stuff like small sub modules.

The last thing we have in our `lib.rs` is the definition of a struct named `Wavetab`, which will act like a facade for the library's functionality. 
In Rust, we define class-like structures by first defining a `struct` (this is kind of comparable to C++ structs, except all fields are private), which bundle a bunch of values.
```rust
pub struct Wavetab {
    wave: Vec<i16>,
    sample_rate: u32
}
```
So, our Wavetab struct consists of a field named *wave*, which is a vector (a dynamically sized array; `Vec<T>` declares a generic type, just like in many other languages), holding 16 bit integers (`i16`), and a field named *sample_rate*, which is an unsigned 32 bit integer.
In *wave* we will store the input signal and in *sample_rate*, the sampling rate at which the original continuous audio signal was sampled.
Also, the struct must be public, to be accessible from outside the library.

Now, to add callable functions to a struct, we *associate* functions with it  
```rust
impl Wavetab {
    // functions
}
```
In Rust, function definitions start with the keyword `fn`, followed by the functions name, its parameters and then the return type after a `->`.
Knowing how to define a function, we want to associate four functions with *Wavetab*:
1. A constructor `new` (the name *new* is by convention, it could be called anything)
2. A convenience function `from_file` to create the struct with data from a file
3. A getter function for the wave (which should return an immutable reference to enable *borrowing* of the variable)
4. The facade function `convert_wave` to do the conversion of the loaded signal.

Our constructor is pretty unspectacular:
```rust
pub fn new(wave: Vec<i16>, sample_rate: u32) -> Wavetab {
    Wavetab { wave, sample_rate }
}
```
There are just a view things to note here:
- We pass in the signal vector and the sampling rate
- The way `wave` is passed, the created instance will take *ownership* of the vector
- Our parameters have exactly the same names as the values of our struct. In this situation, we can initialize the struct by typing `Wavetab { wave, sample_rate }` instead of `Wavetab { wave: wave, sample_rate: sample_rate }` (we will see that in our `from_file` function)
- The last expression in a function that does not end with a `;` will be returned from the function. We also could have added the `return` keyword in front the expression.

So, in the last section I've thrown two Rust buzzwords at you: *borrowing* and *ownership*.
Before going through some more code, I want to elaborate a little bit on the concept of ownership in Rust (note that this won't be an extensive review, but much rather a brief introduction on the topic. For further readings I want to refer to [The Book](https://doc.rust-lang.org/book/second-edition/ch04-00-understanding-ownership.html)).

What is ownership?
Well, in short, it is the way memory (and thus (e.g. struct) instances, values ("instances" of primitive types like integers) and references ("pointers")) is handled in Rust.
It defines a very intuitive, yet strict set of rules on how you have to use and manage variables that you are creating at runtime.
Through this rule set, the Rust compiler is able to detect and prevent common problems you may face in other languages, like dangling pointers or unintended mutation of the same instance from different locations.
Further, it allows for detection (at least to some extend, which is far more than in any other language I know of) of possible race conditions in multi-threaded applications.
On the other hand, many things that might lead to unintended behavior have to be wrapped in special structs (like the reference counting smart pointer `Rc`) or explicitly annotated with keywords (like `mut`), enabling you to find potential sources of error much faster.

So, from my understanding, there are 3 major modes of ownership and borrowing:
- taking ownership
- immutable borrowing
- mutable borrowing

Let's look at some code, to see what these things are about:
```rust
let book = Book::new("Some book");

borrow(&book);  // pass immutable reference (&)

// time passes...

get_back_borrowed_books();  // get back all borrowed books -> all immutable reference are deleted

borrow_it_to_your_best_friends_infant(&mut book);  // pass mutable reference (&mut), there can always be only one at a time

// more time passes...

get_back_borrowed_books();

donate(book);  // pass the instance itself -> transfer ownership
```
Imagine you own *Some book* and you've already read through it multiple times.
At this point, you might not need it anymore.
Also, you heard that one of your friends really wants to read this book, but cannot find it anywhere.
Hence, because you are a nice person, you decide to borrow your copy of the book to your friend (`borrow(&book)`).
Now, your friend is able to read, but not modify (you don't do that to borrowed stuff), your book.
Actually, in Rust you are a magician and *could* lend your book to multiple of your friends at the same time (just so you know that you are allowed to share multiple immutable references to an instance).
During this period of time, you also are unable to modify your book (whilst it is still *your* book, you cannot just modify it, because it is at your friend's place).
After some time, you want to get your book back, because another friend of yours wants to borrow your book for his little infant.
You, as a wise person, decide to allow this friend to modify your book (because it is likely that the kid will do it).
As you couldn't modify your book while borrowed by someone, no one could do so as well, thus you have to get your book back first, so that no one else has access to it.
Now, you can share your book with your friend's infant (`borrow_it_to_your_best_friends_infant`), without the risk that other borrowers of your book might stumble upon missing (ripped apart) pages while reading.
The kid now got a mutable reference to your book (and in general you can only have one mutable reference to an instance at the same time).
More time passes, and you get your book back once again.
You look at this mess of book, that got a sweet new coloring and a few innovative half-pages, and decide you don't want it anymore and `donate` it.
By donating your book for charity, you transfer ownership of your book to the donatary.
Now, you don't have access to the book anymore.

Ok, so after this mess of an analogy for trying to explain ownership, maybe you should consult [The Book](https://doc.rust-lang.org/book/second-edition/ch04-00-understanding-ownership.html), and then we move on and look at our `from_file` function: 
```rust
pub fn from_file(file_path: &str) -> Wavetab {
    let mut reader = match hound::WavReader::open(file_path) {
        Ok(r) => r,
        Err(_) => panic!("There was an error reading the file")
    };
    let wav_spec = reader.spec();
    let samples: Vec<i16> = reader.samples::<i16>().map(|sample| {
        match sample {
            Ok(s) => s,
            Err(_) => panic!("Broken sample")
        }
    }).collect();

    Wavetab {
        wave: samples,
        sample_rate: wav_spec.sample_rate
    }
}
```
There is a lot more going on there, so let's break this down.
The only parameter for the function is a string (reference) holding the path to a wave file.
We are borrowing the string passed to the functions, as we don't want to take ownership of it, nor do we want to modify it.
The first thing to do in the function is to open the wave file using hound's `WavReader`.
Here, we are using pattern matching (keyword `match`) to do some error handling.
The function `hound::WavReader::open(...)` returns Rust's `Result<T, E>` enum ([link](https://doc.rust-lang.org/book/second-edition/ch09-02-recoverable-errors-with-result.html)), which returns the `WavReader` instance wrapped inside the `Ok` struct in case of success and the error wrapped in `Err` in a failure case (e.g. when provided an incorrect path).
Hence, in the pattern matching block, we are extracting the `WavReader` instance in case of success or "throw an exception" (`panic!(...)` is the equivalent of raising an exception and terminating the program) in case of failure.
Note that `reader` is a mutable variable here, because for some reason the function used further below (`reader.samples`) mutates the instance.
The next step is pretty straightforward: `reader.spec()` returns a struct that holds the information from the wave file header (like the sampling rate, encoding and stuff).
Following this, we are reading the (discrete) signal from the file using the `reader`, some functional programming stuff and pattern matching:
1. We want to store all signal samples in a vector of 16 bit integers: `let samples: Vec<i16> = ...`
2. Because of that, we want our reader to read the samples as 16 bit integers: `reader.samples::<i16>()`
3. If we stopped at this point, we would get an compiler error, as `reader.samples::<i16>()` wrappes every sample in a `Result`, thus we have to unpack it
    - We can achieve that by using `map` (applies a given function to all elements of a collection), which takes a [closure](https://doc.rust-lang.org/book/second-edition/ch13-01-closures.html) as an argument.
    - The closure syntax is Rust looks like this: `|arguments| { body }`
    - So, our closure takes one sample as an argument and uses pattern matching (just like before) to unwrap the sample, while handling broken samples (there is also a function for `Result`, called `unwrap`, which could be used as well, but in case of an error, our programm would just panic (I know it does exactly this now, but we could also decide to just skip broken samples))
4. The last thing to do: `reader.samples::<i16>()` actually returns an iterator (I think), so we have to call the `collect` function, to turn it into a vector.

As a final step, we create the instance of our struct with the values read from the file.

Now that we have defined our library entry point, there are two topics left I want to talk about in this post.
The first thing is *extension traits*.
Having programmed in C# for a while, I came to really like extension methods, which can be used to extend already existing classes with additional functions, which are then treated as if they were part of the class.
In Rust you can achieve the same, so I decided it might be handy to extend `Vec<T>` by a few convenience functions for our project.
You can see all of the functions I have added so far in this piece of code:
```rust
pub trait ArrExt<T> {
    fn min_val(&self) -> &T; // calculates the minimum value of the slice
    fn max_val(&self) -> &T; // calculates the maximum value of the slice
    fn abs(&self) -> Vec<T>; // replaces every element with its abolsute value

    // inplace functions
    fn abs_inplace(&mut self);
}
```
In Rust, traits define a bunch of functions (kind of like an interface) for an unknown type.
You can then implement the functions of a trait for a specific type (also types from the standard library), hence extending the types functionality.
So, in the last code fragment, we are defining a generic trait named `ArrExt` with four functions.

Next, I will show the implementation of one of those functions and point out a few things.
```rust
impl<T: PartialOrd + Signed> ArrExt<T> for [T] {  
    fn min_val(&self) -> &T {
        let min_val = self.iter().fold(None, |min, x| match min {
            None => Some(x),
            Some(y) => Some(if x < y { x } else { y })
        }).unwrap();

        min_val
    }
    
    // ...
}
```
The first interesting thing to see occurs directly in the first line `impl<T: PartialOrd + Signed> ArrExt<T> for [T]`.
Let's break this down:
- `impl<T: PartialOrd + Signed>`: This defines a so called [trait bound](https://rustbyexample.com/generics/bounds.html) for our generic type `T`. In our project we are most likely only concerned with numeric data, this is also true for the functions defined in our `ArrExt` trait, thus we have to ensure that `T` is of some numeric type and implements some needed functionality, like ordering functions (`<, >, <=, >=`, ensured by the `PartialOrd` trait) and mathematical functions (in our case `abs`, ensured by the `Signed` trait, which is actually a trait from the extern crate *num*)
- `impl<...> ArrayExt<T> for [T]`: This means we are implementing our trait functions for the array of type `T`. So, why for arrays and not for vectors, like we wanted to? The answer is pretty easy. From my understanding, an array is a more abstract concept than a vector. For example, if we have a function that expects a vector as an argument, we can only pass vectors to it, on the other hand, a function that expects an array as an argument can either take vectors, slices (kinda like sub-vectors, think of python slices) or arrays!

Further, the code shows the implementation of the `min_val` function.
There isn't really much new or fancy to talk about here.
It uses the fold operator (which is a common operation in functional programming) to collapse the array into a single value.
Maybe a word or two about `Some` and `None`.
There is no `null` in Rust, instead you can use the [`Option<T>`](https://rustbyexample.com/std/option.html) enum, which returns a value of type `T` wrapped in `Some(value)` if there is a value or returns `None` if there is none.
When folding to find the minimum value of the array, we have to provide an initial value to compare against, so we could either provide a very high value that we expect to be higher than the highest value in the array (which is not really an elegant solution), or we can start with `None` and use pattern matching to just use the value, when comparing to `None` in the first step of the fold.
As the result of our fold is then of type `Option<T>`, we are calling `unwrap` to obtain the actual value.
Note: this might cause a panic, if the array is empty, as the value will then be `None`. 

The second thing I want to talk about is something visual: plotting.
As available plotting crates for Rust are not too satisfying, I figured I'd just run python in a subprocess and use matplotlib for some plotting.
Although not really performance oriented, it gets the job done, and performance isn't really a concern as of right now anyway.

So, the thing I wanted to plot is the discrete signal I have just loaded from a wave file.
To plot this using pythons matplotlib, we have to perform several steps, given the signal as a vector (reference) `wave`:
1. Convert each value of the vector to a String: `wave.iter().map(|i| i.to_string()).collect()`
2. Build two python lists as Strings (e.g. using a loop)
    - `x_arr`: One containing the values from `0` to `length(wave) - 1`, this will look like this: `"[0,1,2]"` for a vector of length `3`.
    - `y_arr`: The other one contains the String representation of the actual values
3. Build the process' execution string. Here we write our python script to execute: `let exec_str = format!("import matplotlib.pyplot as plt\nplt.plot({}, {})\nplt.show()", &x_arr[..], &y_arr[..]);` 
4. Spawn a new process that runs python and set up a pipe: `let mut process = Command::new("python").stdin(Stdio::piped()).spawn()`
5. Write our python code to the python process' stdin to issue our plot drawing: `stdin.write_all(exec_str.as_bytes())`
6. Wait for the process to finish, otherwise we won't see anything: `process.wait().unwrap();`

Putting it all together and adding respective error handling, we end up with something like this: 
```rust
pub fn plot_wave<T: ToString>(wave: &Vec<T>) {
    // map every vector element to its string representation
    let wave_val_str: Vec<String> = wave.iter().map(|i| i.to_string()).collect();

    // build python arrays [a, b, c, ...]
    let mut x_arr = String::from("[");
    let mut y_arr = String::from("[");

    // iterate over all elements of our array
    // enumerate returns a tuple of (index, current value), just like in python
    for (i, st) in wave_val_str.iter().enumerate() {
        x_arr.push_str(&i.to_string()[..]);  // push the index (as a string slice &[]) to the x array
        y_arr.push_str(&st[..]);  // push the value to the y array (the slicing is not necessary here, but I do it for consistency)

        // add a comma separator or a closing square brackets
        if i < wave_val_str.len() - 1 {
            x_arr.push_str(",");
            y_arr.push_str(",");
        } else {
            x_arr.push_str("]");
            y_arr.push_str("]");
        }
    }

    // build a string that contains our python code, using the format macro (https://rustbyexample.com/hello/print.html)
    let exec_str = format!("import matplotlib.pyplot as plt\nplt.plot({}, {})\nplt.show()", &x_arr[..], &y_arr[..]);

    // set a stdin pipe to pipe our commands to the process running python (pipe to python process stdin)
    let mut process = match Command::new("python").stdin(Stdio::piped()).spawn() {
        Err(why) => panic!("Couldn't spawn python process: {}", why.description()),
        Ok(process) => process
    };
    {
        let ref mut stdin = process.stdin.as_mut().unwrap();  // get the process' stdin
        stdin.write_all(exec_str.as_bytes()).expect("Failed to write python command");  // write our python script
        stdin.write_all(b"\n").expect("Failed to write python command");  // close it out with a newline
    }
    process.wait().unwrap();  // wait for the process to exit
}
```
If we now open up our `main.rs` and import our library, we can load a signal from a wave file and plot it using python:
```rust
extern crate wavetab;

use std::env;

use wavetab::Wavetab; // bring the struct into scope
use wavetab::plotting; // plotting utils

fn main() {
    // collect provided arguments
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
    
    // we pass the one argument: the file path
    // the first entry in args is the path to the executable, thus the argument we've provided is at index 1
    let file_path = &args[1];

    // create a Wavetab instance from the given wave file
    let wt = Wavetab::from_file(file_path);
    
    // plot the loaded wave
    plotting::plot_wave(wt.wave());
}
```
In this piece of code, we are passing the path to our wave file as an argument to the main function (accessible via `env::args`).
When running this by calling `cargo run data/short_seq.wav` from our console, we get to see a plot like this:

![wave plot](/images/blog/02_derusting/wave.png)

With this plot, I will close this post.
All in all, even with the compiler complaining about my code on a frequent basis, I really enjoy programming in Rust so far, and I am looking forward to developing my project in it.

So, we finally made it (phew), that's all for now.

Stay tuned for my next update and cheers!
