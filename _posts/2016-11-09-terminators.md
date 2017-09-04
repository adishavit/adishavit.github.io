---
title:  "Terminators"
categories: [C++]
tags: [C++]
---
#### *Hasta la vista, baby!*   

![Terminators](../../assets/terminators.jpg)  

C++ provides surprisingly numerous ways for a program to terminate. These include both normal and unexpected termination. Some older ways are depricated (e.g. `std::unexpected()`) and some are new (e.g. [§25.2.4.2 algorithms.parallel.exceptions](http://eel.is/c++draft/algorithms.parallel.exceptions#2)).  

When writing robust software and libraries, it is important to be be aware of the various ways your code might abruptly end. Similarly, many of these conditions might occur at module-boundaries such as DLLs. 

Amongst the standard C++ program terminators we find:

* [`std::exit()`](http://en.cppreference.com/w/cpp/utility/program/exit)
* [`std::quick_exit()`](http://en.cppreference.com/w/cpp/utility/program/quick_exit)
* [`std::_Exit()`](http://en.cppreference.com/w/cpp/utility/program/_Exit)
* [`std::abort()`](http://en.cppreference.com/w/cpp/utility/program/abort)
* [`std::terminate()`](http://en.cppreference.com/w/cpp/error/terminate)
* [`std::unexpected()`](http://en.cppreference.com/w/cpp/error/unexpected)(deprecated)
* [`std::signal()`](http://en.cppreference.com/w/cpp/utility/program/signal)
* [`std::raise()`](http://en.cppreference.com/w/cpp/utility/program/raise)

Additionally, OS-specific terminators include:

* [`__fastfail()`](https://msdn.microsoft.com/en-us/library/dn774154.aspx)[ Microsoft specific ]

Most of these functions have subtle contexts, conditions and effects that should be considered by checking their documentation. Some, but not all, of these functions have facilities for setting and even chaining custom handlers. Some of these handler even have both C and C++ `std` versions. 

* What happens when you call one of these functions yourself?  
* What is the difference between them?
* How are they ordered?
* What are their interdependencies?
* What happens to global and static variables and Singletons? 

<p align="center">
  <img src="../../assets/terminator_robot.png" alt="Sublime's custom imageTerminator"/>
</p>

#### *Come with me if you want to live! (not really)*

To better understand all these possibilities, I created a [GraphViz diagram][GitHub] showing the various program termination flows as defined by the standard.  
The orange path shows normal program termination.
[![The call graph](https://raw.githubusercontent.com/adishavit/Terminators/master/termination_graph.png)](https://cdn.rawgit.com/adishavit/Terminators/master/termination_graph.svg)

The full res SVG is [here](https://cdn.rawgit.com/adishavit/Terminators/master/termination_graph.svg).  
The diagram repo is [here][GitHub].  
The image is taken directly from the repo, so any updates there will be reflected here.

Some notable feature differences between these functions:

|                   | Set Return Value | Set Custom Handler via      | Has C Version | More             |
|-------------------|:----------------:|-----------------------|:-------------:|------------------|
| `exit()`     |         ✓        | `atexit()`         |       ✓       | Destroys objects |
| `quick_exit()` |         ✓        | `at_quick_exit()`  |       ✓       |                  |
| `_Exit()`      |         ✓        |                       |       ✓       |                  |
| `abort()`      |                  |                       |       ✓       |                  |
| `terminate()`  |                  | `set_terminate()`  |               |                  |
| `unexpected()` |                  | `set_unexpected()` |               | Deprecated       |
| `raise()`      |                  | `signal()`         |       ✓       |                  |
| `__fastfail()`      |         ✓        |                       |               |                  |

<p align="center">
  <img src="../../assets/terminator_robot.png" alt="Sublime's custom imageTerminator"/>
</p>

#### *I'll be back!*

This project is already a couple of years old. I'm posting about it since it has recently received renewed attantion due to a flurry of tweets on the subject. Independently, [JF Bastien][jfbastian-terminator] created [a project][jfbastian-terminator] that shows how many of these paths can be invoked and traversed in a program. Check it out [here][jfbastian-terminator], output [here](http://ideone.com/6uF5s9).

This diagram is automatically generated by Graphviz. Other layout apps may give prettier results. I would be happy to learn of them.
Corrections, additions, pull-requests, updates and layout improvements will be gladly accepted.  

*If you found this post amusing, dawnting, surprising, awful etc. do feel free to leave a comment. If you think something is missing or you think more details are required for completeness, leave a comment, open an issue or make a PR.*

*Acknowledgments:
[banner](https://youtu.be/Ol-X-ZzrvkM?t=22s) :: 
[rawgit.com](https://rawgit.com/) ::
[www.tablesgenerator.com](http://www.tablesgenerator.com/) ::
[icon](http://pixeljoint.com/pixelart/1187.htm)*

[GitHub]:[https://github.com/adishavit/Terminators]
[jfbastian-terminator]: (https://github.com/jfbastien/terminator)