---
title: "The main() Course"
categories: [C++]
tags: [C++, Imp]
---
A fanciful little post about li'l old `main()`.

![cherry-tomatoes](../../assets/cherry-tomatoes.jpg)    

> “The main() function <s>of color</s> should be to serve expression.”  
> — Henri Matisse

The function `main()` is a normal program's [entry point](http://en.cppreference.com/w/cpp/language/main_function) [[**§basic.start.main**](http://eel.is/c++draft/basic.start.main)].

### Minimalism
The shortest conforming C++ executable program is [[^1]]:

[^1]: The requirement that every source file end with a non-escaped newline was [removed](https://stackoverflow.com/a/8172348/135862) in C++11, but is required in C and pre-C++11.


```cpp
int main(){}
```

A few interesting things to about this snippet:

1. In a *conforming* implementation we must specify the return type `int`.  
   However, some compilers will only [warn](https://wandbox.org/permlink/z1qDbe9lnEg88TYe) on the even minimaler `main(){}`.
3. We do not *need* to `return` a value from `main()`.   
   *"If control flows off the end of... main, the effect is equivalent to a return with `0`"*.  
   [Shafik Yaghmour](https://twitter.com/shafikyaghmour) points out in the comments that this curious behavior is due to [*the C99 standard finally having to bow to sloppy existing practices*](https://stackoverflow.com/a/31396971/135862).


### Modernism
As kindly suggested by [@krzaq](https://twitter.com/KrzaQ2), let's not forget the modern incarnation:

```cpp
auto main() -> int {}
```
Which uses `auto` with the trailing return type `int`.

### Arguments
Sometimes we want to pass command line arguments to our program. 

Although `main` cannot be overloaded *within* a program, it actually has multiple possible signatures.  
In addition to the empty `int main()` we have already seen, it may also be (the obliquely phrased) *"a function of `(int,`pointer to pointer to`char)` returning `int`"*  and on some platforms may support *any further (optional) parameters* beyond the first familiar two.  

This gives us the familiar `int main(int argc, char* argv[])` where:

1. `argc` is the number of command line arguments;
2. `argv` is an array of pointers to `char*` null-terminated multibyte strings that represent the arguments passed to the program from the execution environment;
3. When `argc` is nonzero, `argv[0]` will be the name of the executable or `""` where unavailable;
4. Regardless, even when `0 == argc`, `nullptr == argv[argc]` is guaranteed.

We can write the contrived minimal "`main` with args": 

```cpp
int main(int,char**){}
```

In fact, all of the following are valid `main` signatures:

- `int main()` No command line arguments;
- `int main(int,char**){}` Minimal program with args; 
- `int main(int argc, char* argv[])` Most common variant; 
- `int main(int ac, char* av[])` Alternate names, `argc` and `argv` are just a convention;
- `int main(int argc, char** argv)` Alternate array type: *"pointer to pointer to`char`"* 
- `int main(int const, char const * const * const )` We can specify the args as `const`;

When `argv` is of type `char* argv[]`, the array length is (obviously) determined by the number of program arguments and thus dynamically sized for each program execution. It is *not* a literal array (whose size is known at compile time) as often indicated by this syntax. 

### Beware The Imp 😈

Beyond the first two arguments, [all 3 major compilers gladly accept:](https://godbolt.org/g/k9Ta6Q)  

```cpp
int main(int, char**, char**){}
```
Support for and what the third argument means or contains is *implementation defined* so [[^2]]:  
<p style="text-align: center;font-weight: bold;">😈 Beware The Imp 😈</p>  

[^2]: *The Imp* must not be confused the the *Nasal Demon* 👿 of [*undefined behavior*](https://en.wikipedia.org/wiki/Undefined_behavior). 

A very common extension (e.g. Windows and Linux) is passing a third argument of type `char*[]` pointing at an array of pointers to the execution environment variables.  

### Arguments Over Arguments
From here on, we are in The Imp's quagmire.  

#### Alternate Types
The underlying type of the `argv` strings must be `char`.  
[Given](https://godbolt.org/g/U3zjbB):

```cpp
int main(int, signed char** av) // use signed char instead of char
{ 
    return av[0][0]; 
}
```
- Clang chokes with  
  `error: second parameter of 'main' (argument array) must be of type 'char **'`
- GCC works ***fine*** but:  
  `warning: second argument of 'int main(int, signed char**)' should be 'char **'`
- MSVC does not object at all.

#### More Arguments
What if we [add more arguments](https://godbolt.org/g/2zes5c):

```cpp
int main(int, char**, char**, char**){}
```

- Clang reasonably rejects with `error: too many parameters (4) for 'main': must be 0, 2, or 3`;
- MSVC gladly accepts this;
- GCC accepts the code with the curious:  
  `warning: 'int main(int, char**, char**, char**)' takes only zero or two arguments`.  
  But we've already seen that it happily accepts the *three* argument version as well. Curious. 

What the 4th argument means, I don't know.

In fact, both gcc and MSVC [seem quite nonchalant](https://godbolt.org/g/AZVwfg) about any extra arguments.

```cpp
int main(int,  
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, 
char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char**, char** 
){}
```

#### Extensions

MSVC also supports a non-standard, Microsoft-Specific alternative function:  
`wmain(int argc, wchar_t *argv[], wchar_t *envp[])`   
for working with wide Unicode characters on the Windows command line. Read more about it [here](https://docs.microsoft.com/en-us/cpp/c-language/using-wmain).

### Termination

There are surprisingly many of ways of leaving `main()`. See my [Terminators](http://videocortex.io/2016/terminators/) post for more details.   
<p style="text-align: center;font-weight: bold;"><img src="../../assets/terminator_robot.png"></p>


### Summary
Trusty old `main()` is the first function we meet when writing our first "Hello World". We take it for granted and don't often give it a second thought. But it is a special function with some unique behaviors, tricks and secrets - some of them dark. So...   
  
<p style="text-align: center;font-weight: bold;">😈 Beware The Imp 😈</p>
<p style="text-align: center;"><img src="https://media.giphy.com/media/ykBfNG10gwIRW/giphy.gif"></p>


### Guru Pop-Quiz
***What is the shortest conforming C++ program with exception handling (contrived as it may be)?  
Seriously - try it.***

<details>
<summary>Hint...</summary>
<br>Can you match or beat 27 character?
<br><br>
<details>
  <summary>I give up! Enlighten me!!</summary>
  <br><code class="highlighter-rouge">
  int main()try{}catch(...){}
  </code><br>
  <br>Ha! Use the obscure <a href="http://en.cppreference.com/w/cpp/language/function-try-block">function-try-block</a>.
</details>
</details>
<br>

### UBnacci
<p style="text-align: center;font-weight: bold;">👿<br>👿<br>👿👿<br>👿👿👿<br>👿👿👿👿👿<br>👿👿👿👿👿👿👿👿<br>👿👿👿👿👿👿👿👿👿👿👿👿👿</p>

The [standard says](http://eel.is/c++draft/basic.start.main#3.sentence-1):  

> The function main shall not be used within a program.

[cppreference.com says](http://en.cppreference.com/w/cpp/language/main_function):  

>The main function has several special properties:  
>  1. It cannot be used anywhere in the program: in particular, **it cannot be called recursively**

Disregarding and ignoring this rule puts us into **UB** (*undefined behavior*) territory.  

However, speaking of tiny programs, I was surprised to see that this program, which I call **UBnacci**,  [works](https://wandbox.org/permlink/hOlPca0rANAgabwC) on all major compilers:  

```cpp
// UBnacii: Do not do try this at home! Here be Nasal Demons!
int main(int argc, char** = nullptr) // defaults to nullptr
{ return argc < 2 ? argc : main(argc-1) + main(argc-2); }
```
This tiny app will calculate the Fibonacci number of `argc` by recursivly calling `main()` itself!  
To make this even more fun, note that the unnamed `char**` argument defaults to `nullptr` so that `main()` can be called with a single argument!   

Again, this is just a fun little app, but **DO NOT do this in real code!**



<p style="text-align: center;font-weight: bold;">👿👿👿👿👿👿👿👿👿👿👿👿👿<br>👿👿👿👿👿👿👿👿<br>👿👿👿👿👿<br>👿👿👿<br>👿👿<br>👿<br>👿</p>





*If you found this post interesting, have a better quiz answer, you think I missed something or simply want to discuss this further, please leave a message in the comments below, on Twitter, Reddit or find me on [the C++ Slack channel](https://cpplang.now.sh/).*

*Credits: 
[banner](https://pixabay.com/en/cherry-tomatoes-tomato-food-2152341/) ::
[giphy](http://gph.is/29iaGnN) :: 
[Collapsable Markdown section](https://github.com/dear-github/dear-github/issues/166#issuecomment-236342209)*

#### Footnotes