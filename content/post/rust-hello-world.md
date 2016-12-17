+++
title = "Rust: Hello World"
date = "2016-12-16T16:59:11-08:00"
draft = true

+++

I've been thinking that learning a new language and documenting the process on this blog might be a fun way to live up to the "Write Bad Code" idea. There are two languages I'm currently interested in at the moment: Go and Rust. Although go looks pretty cool, it seems like it's more geared toward the same niches as ruby. Rust, on the other hand, looks to be intended as more of a systems level language like C/C++.

C was one of the first languages I tried to learn way back when. I'd done my first programming in Apple Basic, writing dumb programs on the school computers in Elementary school. When I got older I wanted something a little more robust so I tried to learn C. I actually didn't know that C and C++ weren't the same. We had Prodigy, but no Internet, and my 13 year old self didn't think to actually try to do research on the topic. Anyway, I started struggling through some C++, finding it completely different from Basic and being thoroughly confused. I didn't have any kind of "C++" for kids book, so basically this dream didn't go anywhere. In retrospect, there were several reasons for this, not the least of which was that C++ kindof sucks. I also think I just didn't know enough about computers to really figure out what I was doing. There was no Stack Exchange back then, but if I had it to do over again, I would've started being annoying on the Prodigy bulletin boards related to programming.

Oh well, I finally got to learn more core programming concepts in high school and college, through the strange lenses of Visual Basic and Java (and a tine bit of Assembly), but I always felt like I was missing that close to the hardware programming.

I first heard about Rust almost exactly one year ago, at least I think so based on the date of [this hackaday article](http://hackaday.com/2015/12/18/programming-with-Rust/). I've accidentally learned some C from playing with arduinos, so I'd really like to move up to a full fleged systems programming language.

From what I understand, Rust is basically aimed at the same target as C, but without a lot of the irritating things about C. So for a few blog posts, I'll be follwing [this introduction](https://doc.rust-lang.org/book/). I'd really like to get to the point where I can write device drivers and do that kindof low level work.

I'm running archlinux (which I highly recommend), so installing Rust is really easy `sudo pacman -S rust`. The nice thing about archlinux is that you don't you generally get close to the latest release but still get all the niceties of package management.

## First Steps with Rust

So, the guide says I should run `rustc --version` which tells me that Archlinux installed version 1.13.0 which turns out to be the current version even if I'd compiled from source.

It looks like the first code is the traditional Hello world.
<code>
fn main() {
    println!("Hello, world!");
}
</code>

Compile and run and... it works! I've written something in Rust.

My initial thought is that it looks quite C like, but I notice there is none of that stuff that threw me off when I've tried to learn C. Like includes and void main(void) and all that.

Interesting thing is that `println!` is apparently a macro not a function call. I remember macros from Assembly, where they're basically like copy/pasting code in, I wonder if they're the same in Rust?

## Cargo: Build System and Package Manager?

Now, I'm supposed to be installing something called `cargo` which is the build system/package manager for Rust. Those two concepts don't really seem to go together but what the hell, let's do this Rust! Hmm, apparently "Cargo comes installed with Rust itself", I say bullshit, at least on Archlinux. But, Archlinux is generally pretty reasonable about these things, so with `sudo pacman -S cargo` I'm back in business.

Ah now this guide is telling me to undo what I just did because it's going to teach me the right way. Ok, I get it. Not my favorite pedagogical method, but sometimes you've gotta do it.

This isn't actually too terrible, I'd rather learn the right way early. It's having me create a captial C Cargo.toml as the config file for me little package and move the files to `./src` yeah that actually makes sense.

Also this is kindof cool; you can build with `cargo build` and it'll drop an executeable in a subdirectory, and the run with `cargo run`. Or... you can just use `cargo run` and it will rebuild if there are any changes and then run.  Nice, that's a little closer to the interpreted language feel.

It creates a file called `Cargo.lock`, which I think is a bit like `gemfile.lock`? That's it, I've now hello worlded.

Oh wait, apparently there is another even easier way to create a new project...

`cargo new hello_world --bin`

Yeah, so that just does all the steps we manually did. Fair enough, I'd rather know what and why than have it be magic, I guess.

## Final thought

That's it, I just did the first section of the guide. I think I like Rust, although the live-blogging aspect of it might make it seem cooler than it really is. I hope you got something out of this, maybe you should try the actually guide I linked to rather than my ramblings...
