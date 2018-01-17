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

Implementation-wise most of the stuff I've got so far is based off the first view chapers of [3d Game Development with LWJGL](https://www.gitbook.com/book/lwjglgamedev/3d-game-development-with-lwjgl/details).
Besides the stuff I kept from the book, I've thrown away some of the abstractions suggested in the book, which are made to have a more game engine like setup with code reusable for multiple projects (which I am not aiming for), and added some abstractions in other places (e.g. for scene management).
Also, I don't see the point of running the game engine in a separate thread, when the main thread will go idle from there (maybe I will learn  why this is a good thing in the future).

Maybe one thing worth talking about is the game loop.
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

As a next step, I will start with some procedural generation: terrain generation.
I chose to start with terrain generation, as (I think) it is one of the most common applications for procedural generation, so I thought this might be a good start.
Also I decided to start off with most of the procedural generation stuff before looking at game play features, because creating game play is more fun when there is actually something interesting to look at.

I guess that is all for Java for now, so let's move on to my Rust project.

- doch noch was: joml ist crazy unintuitive, siehe: `right.set(forward).cross(worldUp).normalize();`

## Rust Project: WaveTab
The start of my Rust project ([github repository](https://github.com/xy7e/journey-2018-rust-wav2tab)), where I am trying to build a tool that converts raw audio (waves) into guitar tabs, started somewhat similar to my Java project: **Lots** of reading!
- https://doc.rust-lang.org/book/second-edition/
- https://rustbyexample.com
- https://crates.io/crates/hound
- visual studio code and rust extension
- fft selbst implementieren?
- skimmed throught large parts of https://doc.rust-lang.org/book/second-edition/
-> looks like a promising language
- I am not a dsp guru, but intuitive pipeline from a dsp point of view might be: filter, find notes, fft, ...
- lazy way: genetic algo (treat it as a search problem), expect problems with decreasing oscillations
- will try both and maybe combine in the end

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
