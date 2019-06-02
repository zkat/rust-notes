# Rust Bumps

This is a collection of notes I have on sharp edges I've encountered in my
journey to learn Rust, in hopes that I can use them in the future to target
contributions to Rust proper, or to share with others who might be interested.

## Contribution Process

It wasn't immediately clear what the process of actually contributing to Rust
proper is.

### Solutions

I went to `#beginners` in the Rust discourse, and they pointed me at the
forum, where I'd need to bring it up, try and get some buy-in around my change
concept, then put together an RFC when enough(?) people have shown interest,
and then finally that RFC would point to an implementation.

## Builders

The builder pattern is super common in Rust. Almost obnoxiously so. To the
point where I wonder if Rust should have some sort of first-class support or
new language feature that largely obsoletes the need for the pattern.

### Solutions

In general the pattern is often replaced by things like variadic functions or
ad-hoc option arguments in other languages.

This isn't a terrible pattern, but even going as far as introducing
https://crates.io/crates/typed-builder into the language or into the Book
would be nice.

## Multi-type errors

The book doesn't really cover a pretty basic case in earnest: the situation
where you have multiple different error returns and you want to use `?`.

### Solutions

Establishing an error-definition pattern a-la https://crates.io/crates/failure
in the Book would be super helpful, if not integrating `failure` itself.

Update: I'm supposed to use `failure`, not `error-chain`, per @ag_dubs. SIGH

Update: The plot thickens! https://twitter.com/hoodie_de/status/1135192684916879360 claims I should not be returning failure::Error from public APIs but this is not clear at all and I'm not sure what the expected alternative is.

## Returning iterators

Iterators are super duper useful! Infortunately, it's super hard to figure out
how to -return- them from functions. The compiler errors when you first try to
do this are incredibly opaque and confusing.

### Solution

It turns out these days, you have `-> impl Iterator` which makes this work,
but this was not very obvious and the compiler errors were SUPER CONFUSING
about how to fix this. It might go a long way to have the compiler be aware of
at _least_ the Iterator case. Those compiler messages, wwhoo.

## WTF is `AsRef`?

I kept seeing this `AsRef` stuff while working with `Path`-related functionality and didn't really understand why folks were doing that instead of `path: &Path` in function signatures.

### Solutions

This isn't really covered in the Book, and maybe it should. A friend proceeded to explain the use and intent: It's used as a perf-cheap version of `From` to convert from one immutable ref to another immutable ref. In this **particular** case, it was being used so you could pass Paths, PathBufs, _or_ strings into the function and have it work with any of those types.

It would be nice to make a note in the Book about this being a convention, or becoming one?

## Weird File API

The File-related APIs are really strange and factored in such a way that I had a really hard time finding what I needed, compared to the relatively flat `fs` module in Node.js.

I think the biggest pain is that some things seem to live in `fs::`, some in `io::` and others in `fs::File`, and it wasn't clear when one vs the other was used. The toplevel `fs::` utilities are actually pretty handy, though!

### Solutions

idk. Changing this fundamentally would have a huge impact, unfortunately, and
Rust is very stability-oriented these days. I kinda wish there were a simpler,
higher-level, zero-cost crate for doing these things, optionally with
`Future`s, and for the `fs` and `io` stuff to largely be considered "internal
use only" building blocks, if even that. I just plain don't like it and find
it hard to work with. Maybe [`Tokio`](https://crates.io/crates/tokio) is the
answer, once it's updated with futures@0.3?

## Literally no chown?

So the `fs` API has literally no `chown` functionality. I know it's pretty
niche, but I needed it!

### Solutions

This is exposed through direct bindings by
[`nix`](https://crates.io/crates/nix). There's also a higher-level
[`chownr`](https://crates.io/crates/chownr) crate by yours truly that does the
recursive bit.

## Everyone uses `extern crate`

...but you're supposed to just use `use`. This is mostly an annoyance because everyone's docs are outdated and still refer people to importing using this. As an additional note, a lot of places combine this statement with `#![macro_use]` when importing macros.

### Solutions

Highlighting that `extern crate` isn't used anymore, in the Book, might be
super helpful here. The rest is just waiting out the ecosystem. I'm not really
sure what the actual expected alternative to `#![macro_use]` is supposed to
be, but I had some luck directly `use`-ing the macro into my scope like a
function? So a macro named `foo` in crate `bar` would be imported with `use
bar::foo;` and then the macro seemed to work...

## No `cargo version` built-in

Coming from Node, this feels like a sharp corner: I have to manually edit
Cargo.toml, change the version, run `cargo build` to update the `Cargo.lock`
so I don't end up with a random diff, then I have to manually `git ci -a -m
'vX.Y.Z'`, `git tag -a vX.Y.Z`, fill in a message, `git push --follow-tags`,
and **then** `cargo publish`. Just for a basic release. I'm used to at least
having a baseline of `npm version patch && git push --follow-tags && npm pub`.

### Solution

Having a _baseline_ `cargo version` built in would go a long way towards
making publishing as a newbie a smoother process.

## No `nyc` equivalent for coverage

There's no single coverage tool you can just install that integrates with
`cargo test` and gives useful coverage feedback in the command line, works
with coveralls/codecov, etc. The best you get is something [as described in
this 3-year-old forum
thread](https://users.rust-lang.org/t/howto-generating-a-branch-coverage-report/8524)
which... honestly I'm not going to bother because this is an immense amount of
fucking around with stuff just to get my percentages?

I'm so used to having [`nyc`](https://npm.im/nyc) available :()

### Solution

Rust -really- needs an `nyc` port.
