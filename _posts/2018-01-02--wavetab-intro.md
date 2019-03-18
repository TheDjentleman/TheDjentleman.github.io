---
layout: post
title: Wavetab Dev Diary: Introduction
github_comments_issueid: "1"
---

## The Rust project: A Wave 2 Tab converter
Are you a guitarist and have written a song at some time that you wanted to tab out? Then you probably know the struggle (at least that is something I feel to be cumbersome) of playing a part that you made up over and over to tab it correctly in your favorite tab program. For this case, I imagine a piece of software, that is capable of converting a raw guitar recording (without effects and stuff -> converter might function as a vst sitting right before the audio effects in the audio processing chain) directly into a tab. For this, I am intending to try some standard signal analysis (e.g. with fft's) and/or try different (simpler) machine learning techniques, like genetic algorithms or reinforcement learning.

### Why this for learning Rust?
I chose this project for learning Rust, as I imagine it to be a project that doesn't rely too much on 3rd party libraries (I guess there aren't too many yet for this rather young language). 

## So it begins...
I will approach this project in a learning by doing fashion, where I am learning the language while implementing some simpler algorithms.

In a first step I will make up some higher level concept for the project, to organize the work to be done. At the same time, I will skim through the [Rust book](https://doc.rust-lang.org/book/second-edition/).

Cheers!