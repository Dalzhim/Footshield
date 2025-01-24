# Footshield
Footshield is a prototype meant to explore the creation of a safe subset of the C++ language. It is a clang plugin that generates errors for unsafe constructs, rejecting programs that would otherwise be considered well formed by a conforming C++ compiler.

Because Footshield is a pure subset of the language, C++ programs that do not trigger any of this plugin's additional diagnostics will keep compiling without the plugin.

Additionnally, Footshield is also an experiment meant to explore the design space of tradeoffs by offering an incremental adoption path to legacy applications while also offering convenience and terseness to greenfield projects that aim to be safe from day 1.

# The safe subset of C++
Footshield can generate errors for various unsafe C++ constructs. Here is an exhaustive list:
* Uninitialized variables
* Mutable access to `mutable` member variables within `const` functions
* Base to derived `static_cast`
* C-style casts
* Functional casts
* `const_cast`
* `reinterpret_cast`
* Dereferencing both pointers and references
* Accessing any member of an `union`
* Access to non-`constinit` or non-`const` global variable
* Mutable access to non-`constinit` `thread-local` global variable

# Incremental opt-in
Making incremental adoption as easy as possible is a top priority for this project. This is an explicit acknowledgment that backwards compatibility is paramount. It must be possible to opt into safety one function at a time. Making it as easy as possible to have safety by default is a secondary goal.

## Step #1
The first step towards safety is to enable Footshield as part of your build. All existing conforming C++ code that compiles will keep compiling exactly as before. You are now in a position where you can start getting value out of it.

## Step #2
Footshield introduces a new attribute `[[exp::safe]]` that appertains to functions. This attribute can be introduced one function at a time. Marking a function as being safe means that we're opting into the [safe subset of C++](#The-safe-subset-of-C). All unsafe constructs within this function will now generate errors at compile time. Additionnally, unsafe function calls from within safe functions are ill-formed.

## Step #3
Footshield introduces new compiler flags to opt into specific safety restrictions for all unannotated functions in a translation unit. For example, one could opt into the `const_cast` restriction. This flag can be used to incrementally root out `const_cast` from a legacy codebase and prevent future regressions in modernized code.

Applying such a restriction to a whole translation unit means that it might apply to various different libraries, authored by different entities, making it much harder to modernize incrementally. In order to address this issue, Footshield will introduce new pragmas meant to change the state of safety opt-ins within a translation unit.
```
#pragma GCC safety push
#pragma GCC safety enabled "const_cast"
#pragma GCC safety disabled "reinterpret_cast"
// \[…]
#pragma GCC safety pop
```

# Getting started
TODO

# Current shortcomings of the prototype
While full safety is the goal, this prototype has not yet reached that milestone. Here is
a list of its current shortcomings:
* Overly strict:
  * C-style casts that are equivalent to `static_cast` should be accepted.
  * Functional casts that are equivalent to `static_cast` should be accepted.
  * Uninitialized variables that always get initialized before first use should be accepted.
* Spatial safety is unimplemented.
* Numeric safety is unimplemented.

These shortcomings are not by design. They exist because time was too short to get around to getting it done so far. Some of them also exist because this project doesn't have any novel ideas to explore compared to other projects.

# Future work
The restriction on dereferencing pointers and references severely limits expressivity. In order to restore its use, we plan to explore the following design space:
* Borrow checking
* Lifetime attributes
  * `[[exp::lifetime(0a, 2a)]] int& always_return_rhs(int&, int& rhs);`
