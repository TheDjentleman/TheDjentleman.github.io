---
layout: post
title: Of fractals and voronois
github_comments_issueid: "3"
---
<!-- https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet -->

# Of fractals and voronois
blubb

Only want to read about my Java (project name: Proc2X) or my Rust (project name: WaveTab) project? 
Jump right into it:
- [Java Proc2X](#java-project-proc2x)
- [Rust WaveTab](#rust-project-wavetab)

more blubblors

## Java Project: Proc2X
blabla

### The Input System
design goals, intuition behind it

### Procedural terrain
setup, plane deren verts angehoben oder abgesenkt werden

#### 1. Fractal Terrain
a

#### 2. Voronoi Terrain
b

#### 3. Marching Cubes Terrain
c

## Rust Project: WaveTab
Looking at more noise: the noise I've recorded using my guitar.

### first step of dsp chain: onset detection
asdf

### fft
asdf

<!--
TODO bis zu diesem Post

Java
save scene as number of objects
scene in GameLogic klasse stecken oder so? -> alles mögliche aus Proc2X rausschieben

irgendwann input system erklären

inheritance hierarchie für sceneobjects -> SceneObject ist abstract, dann können klassen davon abgeleitet werden und noch interfaces mit default methoden eingebaut werden!
-> interfaces bringen gewisse funktionalitäten mit

https://www.youtube.com/watch?v=wbpMiKiSKm8
http://catlikecoding.com/unity/tutorials/noise-derivatives/
https://www.youtube.com/watch?v=F2TfRaZ6COQ
fractal noise terrain
voronoi terrain (https://www.redblobgames.com/)
https://www.redblobgames.com/maps/terrain-from-noise/
marching cubes terrain

fractal noise = summing octaves of perlin noise  (https://gamedev.stackexchange.com/a/68170/83580)
https://stackoverflow.com/a/36833930/1861380
https://www.classes.cs.uchicago.edu/archive/2015/fall/23700-1/final-project/MusgraveTerrain00.pdf
http://webstaff.itn.liu.se/~stegu/simplexnoise/SimplexNoise.java
https://github.com/Auburns/FastNoise_Java

irgendwas low poly mit hex grid anstatt triangles?


Rust
- wav laden mit hound
- fft implementierung? -> eher: fange an notenanschläge zu zählen
-- http://sites.music.columbia.edu/cmc/MusicAndComputers/chapter3/03_04.php
-- https://en.wikipedia.org/wiki/Cooley%E2%80%93Tukey_FFT_algorithm
-- http://jakevdp.github.io/blog/2013/08/28/understanding-the-fft/
-- http://www.drdobbs.com/cpp/a-simple-and-efficient-fft-implementatio/199500857
- use python for plotting (for now)

onset detection
https://github.com/andreasjansson/onset-detection
https://arxiv.org/pdf/1712.02567.pdf
https://stackoverflow.com/questions/294468/note-onset-detection
http://bingweb.binghamton.edu/~ahess2/Onset_Detection_Nov302011.pdf
http://www.nyu.edu/classes/bello/MIR_files/2005_BelloEtAl_IEEE_TSALP.pdf
http://librosa.github.io/librosa/generated/librosa.onset.onset_backtrack.html#librosa.onset.onset_backtrack
-->
