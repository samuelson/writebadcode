+++
date = "2016-12-16T18:34:06-08:00"
draft = true
title = "Rust First Project"

+++

This is my continued foray into learning Rust. I'm just following along with [this guide](https://doc.rust-lang.org/book/guessing-game.html). We'll se how long it lasts, but so far I like it. It's actually my preferred style of learning something a bit like the [puppet learning vm](https://puppet.com/download-learning-vm) which I used to first learn Puppet and I now actually get to help develop.

This time the tutorial starts with the `cargo new` command, which I'm guessing is the standard method. I'm glad they actually had me do it manually the first time, it makes it seem less magical. Funnliy enough they start with a "Hello, world!" as the default code in the empty project, cute. I also confirmed that you don't need to run `cargo build`, you can just do a `cargo run` and it will both build and run, I said that last time but didn't actually test it.

This time, it looks like we'll be using an outside library, which Rust does with the `use` keyword. So far so good. `use std::io;`... that makes sense.

## The code...

<code>
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
</code>

That's the example code. It's all pretty straightforward, but WTF is the `mut` and `&mut` stuff about. I think I remember some C think about pointers, is this like that? Also the `let`, I assume is a variable declaration. So is `mut` the data type?

Ah, interesting, `let` is for assigning variables, but by default they are immutable! Nicely done, Rust. `mut` let's you create a mutable variable. And it's getting a String object, it looks like. The part in the stdin function call must be to mutate that mutable variable. 

Funny, I just wrote function and asked myself "should I say 'method' instead of 'function'?". For those following along at home, a method is basically a function that belongs to an object, whereas a function can stand alone. I reasoned that since we didn't have an `io` object that this would rightly be called a "function" yet the instructions here say "method"... I'm sticking with "function", sorry Rust we don't have an object unless you're doing some magic behind the scenes.

Yeah... nevermind, I just misread that. `new()` is the static method for `String`, they're calling `stdin` a function. Yes, thank you Rust for agreeing with me. Ugh no wait, so it says `read_line` is a method? Here's the quote:

<pre>
"Methods are like associated functions, but are only available on a particular instance of a type, rather than the type itself."
</pre>

I don't se how this is an instance. String is and instance, but not io::stdin()...but sure, let's roll with this. Moving along, yes it turns out that `&mut` this is like that think a kindof remember from C,  it's a "reference" which is called a pointer in C. You can have multiple references to the same piece of data. The `mut String` part is because the function/method doesn't take a String (i.e. an immutable String) it takes a `&mut String`. But, references are also immutable so it's `&mut guess` instead of `&guess`. This does make sense, but I can tell this immutable thing is going to take a little to get the hang of.

## A brief interlude to talk about functional programming

Another area I've been interested in learning more about is functional programming. For someone who had object oriented programming crammed down their throats it can be a little hard to get the hang of, but functional programming is all about writing functions and combining them. Immutability is also important for functional programming. For someone who is used to incrementing a variable on every time a loop loops, this hurts my head. But on the other hand, recursion is integral to writing good code when doing functional programming, and I've always loved recursion.

## Get back to work!

Enough distractions, let's finish this tutorial. It makes sense that this needs to be mutable, we need to actuall read the line into the string with the read_line function/method. There is one other little humane touch in this line, the line break at .expect. You can just do that in Rust, I guess. It won't look for the end of line until it hits a ";", I like that. Also the "." syntax is familiar from ruby (and yes, I'm ashamed to say from Visual Basic).

That `.expect()` must be the error handling, that doesn't seem to crazy.

And finally `{}` is the string interpolation for the result. That's actually... kindof great, way more intuitive than that %s bullshit.

## They're Crrrrate

Rust libraries are packaged in something called "crates" which sounds like it's =~ "gems"?

<pre>
A ‘crate’ is a package of Rust code. We’ve been building a ‘binary crate’, which is an executable. rand is a ‘library crate’, which contains code that’s intended to be used with other programs.
</pre>

Yeah, so exactly like a gem. This is kindof handy, they go right in the [dependencies] section of the Cargo.toml file, yeah that's just like gems too. And the versions are tracked in the Cargo.lock. But, they get installed as soon as you build or compile. I wonder if they're stored globally, so you have to worry about version conflicts between project.

## Another brief interlude wherein I install vim syntax highlighting

I use vim. It was actually something I've kindof forced myself to do over the years after my first experience opening it and not only being unable to type anything but have any clue about how to even close the program. It has a very steep learning curve however, and thankfully I didn't give up immediately. I really like [vim Adventures](http://vim-adventures.com/) as a learning tool, although I only went through the free levels. I say steep learning curve, but really it's more jagged with the first part being steep, so every time you internalize a new way of interacting with it you have a big leap in skill. Anyway, I digress from my digression.

I googled for "vim syntax highlighting" and got [this repo](https://github.com/rust-lang/rust.vim), which looks promising because it's under the rust-lang namespace. Like maybe it's actually maintained!

I just noticed that I hadn't set up my dotfiles when I set up this computer, no wonder vim never seemed to be highlighting properly.

## Another brief interlude wherein I set up my dotfiles

When I first started at Puppet (when it was called Puppetlabs) I decided that it would be cool to keep my dotfiles on github. I haven't maintained it incredibly well, but [the base is here](https://github.com/samuelson/eduteam-dotfiles) and I usually tweak it to my own preferences depending on the machine I've installed it on. The repo is a fork of our team base dotfiles, which is a fork of @holman's dotfiles.

Uh oh, now I'm running down the rabbit hole of getting zsh set up, since that's what I use on my mac. I'm going to pause that and get back on task.

Now, I have to remember how to do this. I think I just need to navigate to ~/.vim/bundle/ and add the module there. Yes, that looks right, I see the vim-puppet and other stuff. They're git submodules, I think. So what's the command `git submodule sync`... nope that didn't do anything. Well let me clone the thing first `git submodule add https://github.com/rust-lang/rust.vim` then I'll sync and I think I need `--recursive`. And... it works!

## The actual code for the thing

Back on track, now this is the code from the tutorial. By the way, I always retype code example when I'm trying to learn something, I think it helps you get the language's little idiosyncrasies.

<pre>
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
  println!("Guess the number!");

  let secret_number = rand::thread_rng().gen_range(1, 101);

  println!("The secret number is: {}", secret_number);

  println!("Please input your guess.");

  let mut guess = String::new();

  io::stdin().read_line(&mut guess)
      .expect("Failed to read line");

  println!("You guessed: {}", guess);
}
</pre>

This all actually just makes sense to me. You use `extern` to load an external crate library and then bring it in to the local namespace with `use`. Yeah this is kindof nice. Also I'm again loving the `{}` syntax for string interpolation. I also like the rubylike use of the `.` so that you can use the gen_range() method of the thread_rng() object and just get back a number without having to instantiate that object and keep it around.

## Making it actually work

I like this incremental approach, the program keeps adding little things and then taking a break to explain the new stuff. Here's the next section of code.

<pre>
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
  println!("Guess the number!");

  let secret_number = rand::thread_rng().gen_range(1, 101);

  println!("The secret number is: {}", secret_number);

  println!("Please input your guess.");

  let mut guess = String::new();

  io::stdin().read_line(&mut guess)
      .expect("Failed to read line");

  println!("You guessed: {}", guess);

  match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!")
  
}
</pre>

Yeah that last bit doesn't really fit with any syntax I've experienced. It makes sense on it's face, I guess. So `match` must work with those `Ordering` objects. Also I notice there's `&` in front of secret number. So that would be passing a reference to the number rather than the number itself. They're explaining it more so I'll read on.

Huh, so `cmp` returns an `Ordering` object and `match` is kindof like a case statement for each of those three `Ordering` types. This feels like it's a functional programming thing for some reason I can't explain.

And reading on, the code doesn't even compile because Rust uses a strict typing system. You can't compare a string to an `i32` which is Rust's default.

So two more lines to cast the string to something that can be compared:
<pre>
  let guess: u32 = guess.trim().parse()
      .expect("Please type a number!");
</pre>

My initial thought is that this could probably be done in the same line as the read_line, then you could use an immutable type for `guess` and you woudln't have to recast. Oh, recasting variables is called "Shadowing".

## Loopy

<pre>
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
And the .expect() part is to handle the object that read_line() returns.ddsdd This is not super intuitive to me, but I think I get it. You can tell something to expect
            Ordering::Equal   => println!("You win!")
        }
    }
}
</pre>

Just a loop around the guessing part. Also I did the vim command `gg=G` which does the autoindent again. I'm used to ruby's two spaces per tab, but Rust seems to like four. I may have to change that setting, but I'll run with it for now.

The next section is adding a break to the loop
<pre>
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
</pre>

I really like this, you can just add a section of code surrounded by {} in this context. Totally readable but also really compact.

## The internet is wrong!

The next section is supposed to be about handling the input. You add another {} for the `let guess:` line. Again, this looks awesome, add a code block instead of the `.expect` but.. it doesn't work. I get an error. So... let's run this down!

Step one, am I an idiot who just typed the wrong thing?
Here's the code I typed in:
<pre>
        let guess: u32 = guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
</pre>

I see there is a comma after "continue", but that's also in the tutorial. Capitalization looks good, the spaces are there. Ok, so one word at at time:
'let'... check, 'guess:'... check, 'u32'... check, '='... check, 'match'... damn it. Yeah I missed that word. The code should be this:

<pre>
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
</pre>

It's the same as that other match at the end. So yeah that makes sense if it's an error just skip it, if not just send the number.

And the final exercise was to remove the line where you are told the answer. So that's the game. I was going to make a snarky comment about the game not being fun, but I realized it was fun to play through once to figure out the most effiecient way to guess a number with only knowing if it was too big or two small.

## Final thoughts

Rust is feeling pretty natural to me so far. It actually seems like they designed it with the people using it in mind, so those nice little humane touches like the string interpolation syntax and the way the `match` statements work really show through. I'm not sure if I'll keep doing this play by play, it looks like the next section is more a collection brief explaination of the language.
