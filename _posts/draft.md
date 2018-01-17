---
layout: post
title: Programming Journey 2018 - Derusting some Java and rusting some Rust
github_comments_issueid: "2"
---
<!-- https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet -->

# Programming Journey 2018 - Derusting some Java and rusting some Rust
Already two weeks in and I'm still holding on to my New Year's resolution (yey).
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
As for the project structure, I am not really sure about what it will look like towards the end of the year, so I just started with some package I think will be needed.
With this in mind, I expect the project structure to change over time.
Right now it looks something like this:
- a
- b

Having the initial project structure up and running I started to write some code I thought to be essential for starting the project:
1. A game loop
2. Some rendering stuff
3. A 3D camera that enables to navigate through our 3D world
4. Something that manages (generated) meshes and objects in our 3D world

For the game loop ... fixed time step?

for implementing the stuff i have followed the aforementioned resources and this: https://www.youtube.com/watch?v=BHD7IFguVes&list=PL11uLCLEeKW5HsMBjwfuElr_EwoBqxoQL

why is there no lighting?

not much interesting code to see fo far: for the large part just an adaption of stuff shown in the book (the book is more taylored towards reusable code for other games, 
we skip that and reduce some abstractions). maybe one remarkable change some more insight about: the game loop

also i dont see the point of running the gameengine in a different thread, when the main thread will go idle from there (maybe i will learn sometime why this is a good thing)

- intellij idea
- lwjgl 3
- https://www.lwjgl.org/guide
- https://www.gitbook.com/book/lwjglgamedev/3d-game-development-with-lwjgl/details
- https://www.youtube.com/watch?v=BHD7IFguVes&list=PL11uLCLEeKW5HsMBjwfuElr_EwoBqxoQL
- https://learnopengl.com/
- starte mit simpelstem game loop (einfaches update -> fixedupdate kommt später dann)
- schaffe grundlage um ein mesh zu generieren und zu zeichnen (ohne licht!)
- bastele erst an proceduralem zeug
- projektstruktur wird sich vermutlich (leider) öfter mal ändern da ich noch keine ahnung hab was am besten ist
- also skimmed through https://www.gitbook.com/book/lwjglgamedev/3d-game-development-with-lwjgl/details
-> don't really like the structure of the book, might be helpful for examples though

## Rust Project: WaveTab
- https://doc.rust-lang.org/book/second-edition/
- https://rustbyexample.com
- https://crates.io/crates/hound
- visual studio code and rust extension
- fft selbst implementieren?
- skimmed throught large parts of https://doc.rust-lang.org/book/second-edition/
-> looks like a promising language

<!--
TODO bis zu diesem Post

Java
- einfache 3D Szene mit Kamera und ein paar Würfeln (wie im learnopengl tutorial, nur dass hier direkt eine Mesh Klasse benutzt werden soll)
-- https://learnopengl.com/#!Getting-started/Camera

Rust
- wav laden mit hound
- fft implementierung? -> eher: fange an notenanschläge zu zählen
-- http://sites.music.columbia.edu/cmc/MusicAndComputers/chapter3/03_04.php
-- https://en.wikipedia.org/wiki/Cooley%E2%80%93Tukey_FFT_algorithm
-- http://jakevdp.github.io/blog/2013/08/28/understanding-the-fft/
-- http://www.drdobbs.com/cpp/a-simple-and-efficient-fft-implementatio/199500857
- use python for plotting (for now)
-->
