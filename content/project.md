---
layout: single
permalink: /projects/
title: Projects
description: "Personal projects and other work"
disableComments: true
---

## [diffsitter](https://github.com/afnanenayet/diffsitter)

A semantic difftool that parses a file's AST to compute more meaningful diffs.

## [oars](https://github.com/afnanenayet/oars)

A rust library for orthogonal array generation and verification. It contains
constructors for orthogonal arrays as well as functions that can verify whether
a given point set is a valid (strong) orthogonal array. Development is
currently in progress, as we are currently working on efficiently generating
strong orthogonal arrays. The library is available at
[crates.io](https://crates.io/crates/oars).

## [pcg-rs](https://github.com/afnanenayet/pcg-rs)

A port of the PCG random number generation library, written in pure Rust.

## [nib](https://github.com/afnanenayet/nib)

{{< figure src="https://raw.githubusercontent.com/afnanenayet/nib/master/.github_assets/sample_image.png">}}

A WIP renderer written in Rust.

## [ensh](https://github.com/afnanenayet/Enayet-Shell)

Ensh stands for "Enayet Shell." It is a basic shell written in Rust, designed
to be modern and efficient. It is currently a stable viable shell for MacOS and
Linux. It uses Travis CI for testing and the Cargo package manager to manage
unit tests. At the time of writing (can't always promise I'll update this page
as soon as I update the project), the `ensh` binary size is only 160K, which I'm
working on paring down.

I originally started this project to learn Rust, and learn Rust as a
systems programming language. This project also taught me about process
management in Unix and a lot about maintaining and deploying a product.

## [cttp](https://github.com/afnanenayet/cttp)

A multithreaded HTTP 1.1 server written in C. I wrote this to get a better handle
on BSD sockets and because I haven't really written any code that deals with
networking. The server can handle a decent number of mime types, and functions
as a fairly basic HTTP server. It can receive `GET` requests to a particular path,
and will serve the file from that path to a browser.  The server is also
multithreaded (using POSIX threads) and can accept multiple connections at
once.

`cttp` uses the `pthreads` library and [cmocka](http://cmocka.org)
to manage unit testing. Travis CI is used for deployment and testing.

## tiny search engine (TSE)

For a software development class, we created a small search engine in C.
It downloads the contents of a webpage, indexes it, and ranks the results
of a boolean query and displays it to the user. It follows the Unix development
philosophy. At the instruction of our professor and Dartmouth College, the source code
cannot be made public, but I can provide the source code upon inquiry.
Email me if you want to see the source.

## Android apps

### FreeLoop

{{<figure src="/images/freeloop_screen.webp" title="Screenshot of FreeLoop" height="400em" align="center" >}}

An app that replicates a guitar looper pedal for free. Available on
the [Play Store](https://play.google.com/store/apps/details?id=com.enayet.loopr).
Has over 6,000 downloads.

### [21.Days](https://github.com/afnanenayet/21.Days)

Created with three other students at Dartmouth College, 21.Days is designed
to help users consistently build a habit. It utilizes Firebase and good app
design practices.

### Minigma

{{<figure src="/images/minigma_screen.webp" title="Screenshot of Minigma" height="400em"  >}}

An encryption app that utilizes Android intents to allow a user to send encrypted
messages through any app on their phone. Download on the
[Play Store](https://play.google.com/store/apps/details?id=com.enayet.minigma).

### Battery Informatics

{{<figure src="/images/bat_info_screen.webp" title="Screenshot of Battery Informatics" height="400em" >}}

Provides diagnostic information about a phone's battery, including charge
capacity, temperature, and rate of discharge. Download on the
[Play Store](https://play.google.com/store/apps/details?id=com.enayet.powinfo).

### [MyRuns](https://github.com/afnanenayet/MyRuns6)

A fitness/run tracker app created for a computer science class at Dartmouth College.
Encourages good development practices, modularity, and MVC. Uses a variety of
Google/Android services like location, accelerometer, and is multithreaded.
Also uses machine learning to infer the type of activity a user is carrying out.
