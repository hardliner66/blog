+++
title = "Introducing SOL"
author = ["Steve Biedermann"]
description = "Introducing the helper library I made for the game I'm working on, what it contains and why it exists."
date = 2025-04-11T00:00:00+01:00
  tags = ["odin", "gamedev"]
  categories = ["writeup", "coding"]
draft = false
[taxonomies]
  tags = ["odin", "gamedev"]
  categories = ["writeup", "coding"]
[extra]
  ben_said = "hi"
+++

# Introducing SOL

This project is the result of me literally [nerd sniping](https://xkcd.com/356/) myself.

**TLDR**: Its basically collection of Odin modules that I implemented to make my life easier,
while working on my [game](https://iamhardliner.itch.io/black-vs-white).

- [Modules](#modules)
  - [iter](#iter)
  - [fixed\_dynamic\_array](#fixed-dynamic-array)
    - [fixed\_dynamic\_array/iter](#fixed-dynamic-array-iter)
  - [expression\_evaluator](#expression-evaluator)
  - [stack\_tracking\_allocator](#stack-tracking-allocator)
  - [opaque](#opaque)
- [History](#history)

## Modules
### iter
An interface and helper functions for implementing iterators.

### fixed_dynamic_array
A container using a fixed size, heap allocated buffer as its backing data structure, while
providing an interface similar to a dynamic array.

This is basically the heap allocated version of Odin's `Small_Array`.

#### fixed_dynamic_array/iter
An iterator over the values in a `Fixed_Dynamic_Array` which provides access functions,
that allow safe mutation of the underlying data. Even during iteration.

### expression_evaluator
A simple expression evaluator, using precedence climbing to allow reconfiguration of how precedence
is handled, which provides the possibility to add custom operators and pass variables.

The expression evaluator provides three procedures.

- `parse`: Parses a string containing an expression into an `Expression_Block`
- `eval_expr`: Evaluates an `Expression_Block`
- `eval`: Combination of `parse` and `eval_expr`

While `eval` is simpler to use, its recommended to `parse` and `eval_expr` individually,
so you can cache the result of parse and re-use it.

### stack_tracking_allocator
Basically the same as Odin's `Tracking_Allocator`, but it also stores the stack trace for
each allocation.

**Windows only**

### opaque
Helper types, which can be used to handle data in a type-erased way

## History
It started when I decided to pre-allocate as much memory as possible for the game I'm working on.
The biggest use cases were enemies and projectiles, which there can be a lot in the game.

The first version just heap allocated a huge slice with enough space to hold the maximum amount
of entities I could have while the game was running. This worked fine, but you need to know which
entities are alive, so you can process them. To do that, I made every entity have a HP value and
when iterating over them, I checked if that values was > 0.

That was problematic for a few reasons. It introduces an if-check in the loop, which can
harm performance and lead to bugs if you forget to do the check.
It also made spawning of entities more complicated, because you need to search for a dead entity
first, so you can initialize a new entity there. And finally, as you have no way of really knowing
where each alive entity is located in the array, you need to iterate over way more entities than
necessary. You can kind of set a lower bound by remembering what the highest index was,
but that was more of a band-aid than an actual fix.

In order to remedy this, I spent some time looking at the standard library of Odin,
which is where I found an interesting container type called a `Small_Array`.

A `Small_Array` is a stack allocated array-like type, but with the interface of
a dynamic array. So you can add and remove items dynamically, while the container
makes sure that all the invariants are kept:
- Elements are in a contiguous memory region
- You can't access outside the virtual bounds of the array
  - virtual, because accessing memory is technically valid, as long as you stay inside the bounds of the backing array
  - semantically that memory should be considered out of bounds though, which is what the container will uphold
- After the container is created, no additional memory is allocated unless you call something like resize

It only had one drawback, which is that size of the backing array has to be known at compile time,
which was at odds with my goal of being able to configure as much as possible in the game without
having to recompile.

After some research, I decided to create a new container based on how `Small_Array` works,
but using a heap allocated slice instead of a stack allocated array.
Sadly I couldn't come up with a proper name, so I named it `Fixed_Dynamic_Array`,
because it used a fixed size of backing storage, while providing an interface similar to how a dynamic array works.
Now I can read the length from a config file at runtime and still have similar guarantees to `Small_Array`.

Once that was done, the next problem was the cleanup. Namely when an entity dies,
it should be removed from the array. But if you do that naively while you loop over the array,
you might accidentally skip an element or process an element twice. So I kept the HP and
removed dead elements in a second pass.

For this problem I also found a solution in the standard library. There is a nice function
called `unordered_remove`, which takes the valid element and puts it where the removed element is.
Then you just need to decrement the length and you removed an element without having to move
all the elements after one position up. That means that if an entity dies, you can do an
unordered remove and skip incrementing the loop counter, so you process the new element,
which is now at the position of the removed element, instead of skipping it.

Having to manually manage a loop counter is quite bothersome, so I started experimenting
with how for loops work in Odin, in order to create an iterator that would properly
handle removed elements. At first, I just stored the expected length and skipped
incrementing the index if it had changed, but I realized that there are more operations
that can safely be done, if they are properly accounted for.

So I added functions to the iterator, that could safely be used to manipulate the
the array, while still making sure no elements are skipped or iterated twice.

At this point I was quite happy with how the container worked. Iteration was easy again.
Removal was easy. And all elements were neatly organized in a contiguous fashion.

So I went back to working on the game. My next goal was, to clean up the config.
It had grown from just a few settings to well over 50 values and many of them were
in weird places, were it not only made no sense for them to be there, but I also
had to break the structure multiple times to accommodate for new things. So it was
definitely time to clean up. Not only for my sake, but also so my collaborators
wouldn't need to search where a setting was every time, because I had to change something.

While doing that, I had the idea to also add scaling to the game, so that enemies
could tank more, the farther the game went on. At first I hard coded the formulas
into the game, but having a new cleaned up config file, I thought it might be
nice to add the possibility to change the scaling formula through the config,
so I spent some time implementing an expression parser/evaluator with support
for custom operators and variables.

After that, I started to doing some cleanup and refactoring of the rest of the code,
when Karl Zylinski dropped a blog post about [handle-based arrays](https://zylinski.se/posts/handle-based-arrays/).

While reading it, I noticed that he also created an iterator, but he had figured out
a cleaner way to use them in a for loop.

My version:
```odin
iter := make_iter(arr)
for it := next(&iter); it != nil; it = next(&iter) {
    // use it
}
```

His version:
```odin
iter := make_iter(arr)
for it in next(&iter) {
    // use it
}
```

So I started experimenting with the iterator code again. When I figured out how
this worked, I thought it might be a nice to have a generic iterator interface,
which you could re-use and base other iterators on.

This is where the nerd-snipe happened, because this was not something that I actually
needed. Just something that seemed interesting to do.

I created a first prototype, but soon realized that different iterators can have
different types for their internal state, so if I wanted a truly generic iterator,
I had to get rid of the state type somehow and to do that, I chose to transmute my
way to a working implementation.

But that was really ugly and hacky, so I started implementing an opaque type,
which could be used for type erasure, while keeping the ugly transmutes away
from the actual code. I ended up creating three variants of the opaque type.
A stack allocated version, a heap allocated version and a pointer version.
Each had their own benefits and drawbacks. The version I chose for the iterator
was the stack allocated version, because I didn't want to heap allocate every time an
iterator is created and I also didn't want to force users to remember to free the iterator
when they're done.

This worked quite nicely, but because it was stack allocated,
you needed to specify the size at compile time. So to actually make this work, I had to define a
maximum size an iterator (including state) could have. At first I saw it as a necessary evil,
in order to have an iterator, that only know what type it returned and not what kind of state
it had, but every other day I would start thinking about it again, because I thought there
has to be a better way to do this.

That thought is what led me experiment with slices, which ended in me creating
[this monstrosity](https://github.com/hardliner66/sol/blob/main/experiments/ongoing/slice_abuse/slice_abuse.odin).
Knowing a bit more about how the internals worked, i was able to abuse how slices work
in order to create an iterator that would work with value (`for v in slice`) and reference (`for &v in slice`) semantics,
without it having any sort of backing container. It is basically a massive hack,
which abuses implementation details about slices, for loops and `deferred_out` in order to work.
(_Note: Calling a function that that uses a `deferred_*` attribute in the head of a loop will cause the deferred function to run on every iteration_)

Sadly the `deferred_*` functions can't be used with generic functions, which is one of the reasons
why this approach was not a good fit for creating proper iterators. Which, thinking about how
hacky all of it is, is probably for the better.

Seeing that my efforts didn't lead anywhere, I paused working on the iterator stuff
and went back to working on the game. This time I wanted to tackle memory management.
As most things already used fixed size containers, I thought it might be a good idea
to start using an arena allocator and pre-allocating all the memory the game needs at
initialization.

While I was at it, I thought I'd also start cleaning up some of the memory leaks.

With Odin's `Tracking_Allocator`, this should be a piece of cake, right? Well,
in theory, yes, in practice, not quite. The problem is that, while the tracking
allocator stores the location of where the allocation was made, its usefulness depends on
if and how the location is passed to the allocation functions.

If a function takes an optional location argument and passes that to the function it
calls, the allocation will be attributed to the location where you called the initial function.
If a function doesn't take a location or somewhere in the chain the location is not passed down
properly, the location is gonna be somewhere inside the standard library of Odin.
So while I had all the locations the allocations came from, for some of them it was
almost impossibly to know which part of my code it actually came from.

In order to remedy this, I borrowed another thing from the standard library.
I built a version of the tracking allocator, which would not only store the location,
but also a stack trace for each allocation. While this might not be feasible for all programs,
it helped me track down all the parts that didn't properly clean up after themselves.

Finally, we arrive at today. After having played around with and experimented with the iterator
code for quite some time now, I finally realized how I could get rid of the opaque type altogether.
So now I can finally rest and stop thinking about how to get rid of those extra bytes we were carrying around.

I realized that for the most part iterators will be pretty short lived. You create one,
you use it to iterate over a collection and then you're done with it.
That means, I can ignore type erasure for this use-case, because we can just use the
typed iterator (with the state type) directly anyway.

With that out of the way, creating an iterator that is unaware of its state can be done by
taking a pointer to the state aware iterator and casting it to be state unaware.
The interfaces line up anyway and because everything is passed by pointer, the cast
just works and internally it can still work with the appropriate types.
