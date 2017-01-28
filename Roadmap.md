The current status of this project is at v0.7 (pre-release). The project will transition to 1.0 (official release) shortly.
The shader language supported in this project is intended to match HLSL fxc shader model 5.1 to simplify adoption and initial testing.

Support for advanced language features such as those supported by clang would appear in a future shader model.

The long term plan for this project was originally announced last March at GDC 2016.
The presentation video is [here.](https://www.youtube.com/watch?v=dcDDvoauaz0&t=351s)
A PDF of the [slides](http://1drv.ms/1T8iew9) is also available on that page.

Here is a summary of the key points:

## Backlog of candidate language features
* Enums
* Bit-fields
* Language Standard Annotations
* References
* Unions
* Function Templates
* Visbility: `public, private, protected, friend`
* Generic resource addressing

## Stretch language features
* `constexpr`
* Assertions
* Function pointers (limited)
* Lambdas (limited)
* Virtuals
* Constructors, destructors, etc.
* Extern functions (linking)

## Out-of-scope language features
* Exceptions
* A stack
* Recursion
* STL Containers
* C99/stdlib compatibility
* Garbage collection

## Candidate hardware features
* Explicit float16/`half` type
* Blender intrinsics
* More atomic operations
* Stereo system values (SV_XXXX)
* Programmable blending (aka RT Read)
* Input of arbitrary structures
* More control over IEEE behavior
* Future GPU features
