The current status of this project is past initial supported release. The first official release was for Windows 10 Creator's Update Edition. We've continued to make improvements beyond the components that were made available with the corresponding SDK. 

The shader language supported in this project HLSL 2016 and beyond, and shader model 6.0 and beyond. The language is intended to match the existing HLSL shader model 5.1 level of support to simplify initial adoption and testing.

Support for advanced language features such as those supported by clang would appear in a future shader model.

The long term plan for this project was originally announced last March at GDC 2016.
The presentation video is [here.](https://www.youtube.com/watch?v=dcDDvoauaz0&t=351s)
A PDF of the [slides](http://1drv.ms/1T8iew9) is also available on that page.

Here is a summary of the key points:

## Backlog of candidate language features
The following language features could be added to the front-end with no impact on core functionality, drivers, or hardware:
* Language Standard Annotations
* References
* Unions
* Visibility: `public, private, protected, friend`
* Generic resource addressing

## Stretch language features
These features are under consideration for longer timeframes depending on user requests.
* `constexpr`
* Assertions
* Function pointers (limited)
* Lambdas (limited)
* Virtuals
* Constructors, destructors, etc.
* Extern functions (linking). This is partially implemented but it's not part of the DXIL standard, and so it's a tool-level-only support.

## Out-of-scope language features
There are currently no plans to implement these features in the language.
* Exceptions
* A stack
* Recursion
* STL Containers
* C99/stdlib compatibility
* Garbage collection

## Candidate hardware features
These are examples of changes to the language that require to changes in the hardware, and so will require updates to the underlying implementation.
* Blender intrinsics
* Programmable blending (aka RT Read)
* Input of arbitrary structures
* More control over IEEE behavior
* Future GPU features
