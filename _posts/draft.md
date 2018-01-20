---
layout: post
title: Programming Journey 2018 - Derusting some Java and rusting some Rust
github_comments_issueid: "2"
---
<!-- https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet -->

# Programming Journey 2018 - Derusting some Java and rusting some Rust
Almost three weeks in and I'm still holding on to my New Year's resolution (yey).
Welcome to the first update on my programming journey. 
Only want to read about my Java (project name: Proc2X) or my Rust (project name: WaveTab) project? 
Jump right into it:
- [Java Proc2X](#java-project-proc2x)
- [Rust WaveTab](#rust-project-wavetab)

In general, I have been mostly skimming through some tutorials and books on different relevant topics.
Futhermore, I have created the two code repositories and started writing some code.
In this post, I will write mainly about some setup stuff, my first impressions tackling the projects and a rough outline for the weeks to come.
So let's take a look at my Java project.

## Java Project: Proc2X
As a very short reminder: In this project I aim to create a simple rts-style game including exploration and civilization building, where as much as possible is generated procedurally. 
For that, I want to use the Java programming language.

Before even starting, I skimmed through some tutorials and a gitbook named [3d Game Development with LWJGL](https://www.gitbook.com/book/lwjglgamedev/3d-game-development-with-lwjgl/details). 
This book looks like the most promising reference for learning LWJGL, especially for newer LWJGL topics (it seems like there have been some major changes last year, including the way gpu buffers are handled, which are not covered by older tutorials).
Besides that, another important resource for me is [learnopengl.com](https://learnopengl.com/), which offers a pretty extensive series of tutorials of OpenGL.
Whilst these tutorials are for C++, they are still very helpful, as LWJGL is a wrapper around OpenGL for the most part (even on their homepage they suggest to learn the C API first: *If you're just getting started, please familiarize yourself with each [GLFW, OpenGL, OpenAL, ...] API first.*).
In fact, looking at their [wiki](https://github.com/LWJGL/lwjgl3-wiki/wiki) they kind of discourage you from learning OpenGL through LWJGL :D; okay, this statement is a little over the, so please decide for yourself:
>Trying to learn the API via LWJGL is possible (a lot of javadoc is included), but not very productive. 

and
>tl;dr
>- Get familiar with C.
>- Get familiar with the native APIs you use.
>- Read the FAQ and Troubleshooting pages.

With this out of the way, we still want to use LWJGL for re-learning OpenGL (yes, that's right).
So after reading some stuff, I installed the current version of Java 9, my favourite IDE, Intellij IDEA (it's not that I dislike eclipse, I just like this one more), and started up a new maven project.
Including LWJGL 3 in your maven project is easy, there is a maven dependency builder on the [LWJGL website](https://www.lwjgl.org/customize) which lets you pick and choose the components of the LWJGL that you want to use (e.g. OpenGL bindings (the graphics API), GLFW (a widely used windowing api) or JOML (the Java OpenGL math library, I guess it is the equivalent of glm for C++)).

### The Project and Code
With the setup done, let's dive into a little bit of code ([github repository](https://github.com/xy7e/journey-2018-java-procedural-2X))! 
In fact, there is not much interesting to show as of right now.
Regarding the project structure, I am not really sure about what it will look like towards the end of the year, so I just started with some packages I think will be needed.
With this in mind, I expect the project structure to change over time.
Right now it looks something like this:
- graphics
- scene
- utils

Having the initial project structure up and running I started to write some code I thought to be essential for starting the project:
1. A game loop
2. Some rendering stuff
3. A 3D camera that enables to navigate through our 3D world
4. Something that manages (generated) meshes and objects in our 3D world

Implementation-wise most of the stuff I've got so far is based off the first view chapers of [3d Game Development with LWJGL](https://www.gitbook.com/book/lwjglgamedev/3d-game-development-with-lwjgl/details) and [learnopengl](https://learnopengl.com/) (mostly camera-related topics), so I want be talking much about Java code in this post.
Besides the stuff I kept from the book, I've thrown away some of the abstractions suggested in the book, which are made to have a more game engine like setup with code reusable for multiple projects (which I am not aiming for), and added some abstractions in other places (e.g. for scene management).
Also, I don't see the point of running the game engine in a separate thread, when the main thread will go idle from there (maybe I will learn  why this is a good thing in the future).

Now, there are two things worth talking about.
One of these is the game loop.
On this topic, the book suggests using a *fixed time step* game loop (with the additional possibility to skip frames when needed).
When using such a game loop, you are forcing all updates and render calls to run a fixed number of times per seconds.
While this is a very robust way of organizing the game loop and fixed update rates are desireable for stuff like physics or ai, I think you can achieve much smoother movement when processing user input or updating animations as fast as you possibly can.
Following this, I have changed the implementation to process input and render as often as it can, while updating the game logic in a fixed time step (a great explanation of this type of game loop can be found here: http://gameprogrammingpatterns.com/game-loop.html).
Further, I have added a second function to update the game logic, which is also (like render) called as often as possible.
Finally, the game loop looks something like this:

```
while (true) {
    long current = System.nanoTime();
    long deltaNanoTime = current - previous;
    previous = current;
    lag += deltaNanoTime;

    // only call fixed update if a specific amount of time has passed
    // also: do multiple updates (and skip frames) if we got behind due to slow rendering or stuff like that
    while (lag >= NANOS_PER_UPDATE) {
        // update stuff like physics or ai on a fixed cycle
        fixedUpdate();
        lag -= NANOS_PER_UPDATE;
    }

    handleInput();

    // update motion and things like that more frequently
    update(deltaNanoTime / 1_000_000_000f);

    renderSystem.render(window);
}
```

As you can see in this sample, I now have two `update` functions: `fixedUpdate` and `update`, which correspond to the two mentioned before.
Another remark: I am using `System.nanoTime()` instead of `System.currentTimeMillis()`, because of its seemingly higher precision and consistency (which comes at a insignificantly higher computational cost).

The other thing I just have to point out here is my first encounter with *JOML*, the math library, where I learned that reading the docs is better than just assuming that JOML works like glm (my assumption stemmed from the fact that JOML is shipped as an addon to lwjgl, like glm is for OpenGL in a C++ context, for doing vector and matrix algebra).
Look at the following scenario: For my camera, I've set up three vectors (as suggested [here](https://learnopengl.com/#!Getting-started/Camera)), `forward`, `up` and `right`, which point in the direction they are named, while taking the camera's rotation (`pitch` and `yaw`) into consideration.
With such an approach, we have to recalculate these vectors everytime `pitch` or `yaw` are changing, so I ported the C++ code directly to Java using JOML instead of glm.
Then it looked like this:
```
forward.x = (float)(Math.cos(Math.toRadians(yaw)) * Math.cos(Math.toRadians(pitch)));
forward.y = (float)Math.sin(Math.toRadians(pitch));
forward.z = (float)(Math.sin(Math.toRadians(yaw)) * Math.cos(Math.toRadians(pitch)));
forward = forward.normalize();
right = forward.cross(worldUp).normalize();
up = right.cross(forward).normalize();
```

I ran this code and BOOM, all of these vectors were `NaN`.
So, what happened?
Looking at the signature, the methods `cross` and `normalize` return a `Vector3f`, so I thought it would create a new vector and return that (just like glm does).
I then looked into the source code of `Vector3f` and figured that the returned `Vector3f` actually is the vector itself (`this`).
While I understand this design to save unnecessary object creations, it was just unintuitive for me, that a method that modifies the object itself is returning `this` instead of being `void`.
What I've learned from that: Just read the documentation first, dumbass!
Even on the main page of the [JOML github](https://github.com/JOML-CI/JOML) it reads:
>All operations in JOML are designed to modify the object on which the operation is invoked.

From there, it was clear for me, how to do the stuff I want to do.
Finally, one way of implementing it correctly, would be this:
```
forward.x = (float)(Math.cos(Math.toRadians(yaw)) * Math.cos(Math.toRadians(pitch)));
forward.y = (float)Math.sin(Math.toRadians(pitch));
forward.z = (float)(Math.sin(Math.toRadians(yaw)) * Math.cos(Math.toRadians(pitch)));
forward.normalize();
right.set(forward).cross(worldUp).normalize();
up.set(right).cross(forward).normalize();
```

Looking at `right` for example, we first set its values to be those of `forward`.
Then we calculate the cross product with `worldUp`, which modifies `right`.
Lastly, we normalize `right`, just by calling `normalize`.
The way JOML works, allows us to chain calculations like I did here.
That is pretty neat in my opinion.

As a next step, I will start with some procedural generation: terrain generation.
I chose to start with terrain generation, as (I think) it is one of the most common applications for procedural generation, so I thought this might be a good start.
Also I decided to start off with most of the procedural generation stuff before looking at game play features, because creating game play is more fun when there is actually something interesting to look at.

I guess that is all for Java for now, so let's move on to my Rust project.

## Rust Project: WaveTab
The start of my Rust project ([github repository](https://github.com/xy7e/journey-2018-rust-wav2tab)), where I am trying to build a tool that converts raw audio (waves) into guitar tabs, started somewhat similar to my Java project: **Lots** of reading!
Also, as I just started learning the language, there isn't as much code here yet, at least compared to the Java project.

To begin with, I've read through a large portion of *The Book* (as they like to call it), namely [The Rust Programming Language](https://doc.rust-lang.org/book/second-edition/).
It is filled with a lot of information and small examples and I think it is very well written and easy to follow/understand.
When I started coding, I also found [rustbyexample](https://rustbyexample.com) to be a very good resource on how to use the standard library and the language in general.

So, what are my first impressions on the language and what have I achieved so far?
First off, I really like *cargo*, Rust's build system and package manager.
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
Afterwards we will look at some code.

As there isn't really a standard IDE for Rust yet, I looked at the chart on a website called [areweideyet.com](https://areweideyet.com/), to see which code editors and IDEs already plugins that support Rust.
From the pretty extensive list, I chose to go with Visual Studio Code, because I kinda like that editor and there is a comprehensive Rust plugin for that editor.

Having everything installed, the first thing to do is to create a new project.
I decided to code my converter as a library, with an eventual use as a backend for a c# gui at a later stage, so we simply start up our cmd (on windows) in a directory of our choice and run `cargo new wavetab`.
With this command, we tell cargo to create a new library project named *wavetab*.
Cargo then creates a project folder for us which includes the `Cargo.toml` and a source folder containing our `lib.rs`.
For playing around with our library (without the need of our graphical frontend), I added another source file and named it `main.rs`.
In this way, we can either build the library or run `main.rs` by calling `cargo build` or `cargo run` respectively (from the project's root folder in a cmd window).
I also created a folder called `data` in which I will store some *wave* files with different guitar recordings for testing (as of right now there is only one, containing a short note sequence).

Before we dive into some code, I want to some up my plans for approaching the problem of converting a recording into a guitar tab.
I'm planning to look at it from two perspectives: classic dsp and machine learning.
I am no dsp guru, but my intuition for a dsp pipeline that extracts the notes played from the recorded wave (I will call this *signal* in the remainder of this post) goes something like this:
1. Eventually apply some filters to the signal
2. Identify points of time a new note or chord is being played
3. Subdivide the signal to separate subsequently played notes or chords
4. Apply fast fourier transform on signal sections to extract frequencies
5. Convert frequencies to notes (like a') or chords (like E minor)

I imagine such a pipeline to have a bunch of pitfalls and difficult to tune towards different edge cases.
Because of that, I am also looking at the lazy way of doing things: machine learning.
For this, I will (as a first attempt) treat the note extraction problem as a search problem and provide a genetic algorithm with the notes it is able to play and let it figure out which ones to play and when to play those.
In this approach, I expect problems with decreasing amplitudes in the signal when a note is being held for a period of time.
When the algorithm just generates perfect signals from the inputs, there aren't any descreasing amplitudes, which might affect the loss calculations in the optimzation.
But I guess we will see what happens when we get to that point. In the long run, I aim at combining both approaches in some way to use the strengths of each approach.

That being said, let's look at some code!
So first off, we will add two dependencies we will need: *hound* (for loading wave files) and *num* (for some numeric traits).
This is as simple as adding the following to our `Cargo.toml`.
```
[dependencies]
hound = "3.3.0"
num = "0.1.41"
```

Following this, we import these crates in our `lib.rs`:
```
extern crate hound;
extern crate num;
```

Next, I want to split the library's functionality into different modules:
- analysis: parses input signal
- tab_generation: generates a guitar tab from analysed signal
- utils: general utility, extension methods and so on
- plotting: for plotting our signal (and maybe other stuff)

asdf

- start with audio signal loaded from wav: hound
- good when there's something to look at: python plot (piped process)
- extending classes, trait constraints
- vector vs array vs slice



<!--
TODO bis zu diesem Post

Java
- einfache 3D Szene mit Kamera und ein paar Würfeln (wie im learnopengl tutorial, nur dass hier direkt eine Mesh Klasse benutzt werden soll)
-- https://learnopengl.com/#!Getting-started/Camera
DANACH
save scene as number of objects
scene in GameLogic klasse stecken oder so? -> alles mögliche aus Proc2X rausschieben

inheritance hierarchie für sceneobjects -> SceneObject ist abstract, dann können klassen davon abgeleitet werden und noch interfaces mit default methoden eingebaut werden!
-> interfaces bringen gewisse funktionalitäten mit

Rust
- wav laden mit hound
- fft implementierung? -> eher: fange an notenanschläge zu zählen
-- http://sites.music.columbia.edu/cmc/MusicAndComputers/chapter3/03_04.php
-- https://en.wikipedia.org/wiki/Cooley%E2%80%93Tukey_FFT_algorithm
-- http://jakevdp.github.io/blog/2013/08/28/understanding-the-fft/
-- http://www.drdobbs.com/cpp/a-simple-and-efficient-fft-implementatio/199500857
- use python for plotting (for now)
-->
