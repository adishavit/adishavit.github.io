---
title: "Affine Space Types"
categories: [C++, Mathematics, API, Design]
tags: [C++, Mathematics, API, Design]
---
Well defined semantics for positions and displacements.

![star-trails](../../assets/star-trails-banner.jpg)    

## The Affine Space

I recently came across a geometric structure that deserves to be better known: ***The Affine Space***.  
In fact, like many abstract mathematical concepts, it is so fundamental that we are all subconsciously familiar with it though may have never considered its mathematical underpinnings.   

Strangely enough, once I assimilated the concept, my [Baader-Meinhof phenomenon](https://science.howstuffworks.com/life/inside-the-mind/human-brain/baader-meinhof-phenomenon.htm) kicked in and it seemed that everyone is suddenly talking about the Affine Space. I'll be linking to multiple resources that do a much better job than I can at explaining them.  

This post will introduce the Affine Space structure and focus mainly on its role in the C++ type system, the standard library and for creating new strong types.      

#### CAVEAT EMPTOR

This post is *not* about other common *"affine"* topics such as:

- ***Affine Transformations***: In computer graphics and image processing, [geometric *affine transformations*](https://en.wikipedia.org/wiki/Affine_transformation) are parametric shape deformations where parallel lines (in e.g. 2D or 3D) remain parallel after the transformation;
- ***Affine Type Systems***: I really wanted to title this post *Affine Types*, however in Type-Theory [*affine type systems*](https://en.wikipedia.org/wiki/Substructural_type_system#Affine_type_systems) are well defined.    

Although these may be indirectly and mathematically related, any such link is beyond the scope of this post.   Similarly, I will not delve into many of the deeper mathematical aspects, details and connections.    

My focus (and perhaps the main reason you may find this post interesting and different from other resources) will be on the formalization and its role in better API design.  

**WARNING:** I am not a mathematician. The explanations, intuitions and interpretations are my own and thus may be utterly wrong. If you find such errors, please drop me a line in the comments (or elsewhere) so I can be enlightened and correct the mistakes.    
I am writing this as a programmer to a programmer, so although familiarity with elementary-school Linear Algebra is assumed, that you are a mathematician is not.

Also, I am treading a fine line between mathematical purity and pragmatic C++, so even where not explicitly stated assume an asterisk reasonably ignoring edge-cases with undefined-behaviors. 

## Motivating Examples 

### Pointer Arithmetic

Consider the [following C code](https://godbolt.org/g/r9ikBQ):
 
```cpp
typedef float T;         // declare T
T   arr[42];             // an array of T
T*  beg = &arr[0];       // pointer to first element of the array
T*  end = beg + 42;      // pointer to one past the end of the array (OK, not UB)
int cnt = end - beg;     // element count or offset
T* last = beg + cnt - 1; // pointer to the last element
int neg = beg - end;     // a negative or backwards offset from 'end'
```
There are four *types* in use in this snippet:

1. `T` (`float`)
2. `T[42]`
3. `T*`
4. `int`. 

`T` is arbitrarily chosen here as `float` and `T[42]` is not really relevant to this example (it is here to avoid UB).  

We are interested in the interactions of the two remaining *separate* types: `int` a signed integer and `T*` a pointer.  

Observe that:

- We can add integers *to* pointers and subtract integers *from* pointers [[^1]].   
The result is a ***pointer type***.  
- We can also subtract *pointers* to get a (signed) ***integer type*** [[^2]].
- Integers can (obviously) also be added, subtracted and multiplied by each other to get other integers (closure under addition and multiplication [[^3]]).  
  Note however, that *mathematically* for integers *subtraction* is not really a standalone operation (operator) but really just a notational shorthand (or syntactic sugar) for addition with the inverse (the right-hand-side multiplied by `-1`).     

[^1]: In C++ the correct type is in fact [`std::ptrdiff_t`](http://en.cppreference.com/w/cpp/types/ptrdiff_t) which on some platforms may not be an `int`. 

[^2]: For the pedantic, yes, only pointers to elements of the *same* array (including the pointer one past the end of the array) may be subtracted from each other, otherwise the behavior is undefined.

[^3]: I am quietly ignoring any undefined behavior due to signed overflow.

*However*, this code does not compile:

```cpp
beg + end; // Error C2110: '+': cannot add two pointers
42 - beg;  // Error C2113: '-': pointer can only be subtracted from another pointer
beg * 3;   // Error C2296: '*': illegal, left operand has type 'T *'
```

MSVC's errors are very informative here and teach us that the addition, subtraction and multiplication operations are only allowed for some type combinations and, as in this case, not for others.

What does adding two addresses *mean*? What does it *mean* to multiply an address by a scalar (negative or otherwise)?   These questions do not have a clear answer (or any answer for that matter) and thus these operations are not permitted. 

> Given **"meaning"** == **"valid semantics"**;   
> We desire: **"no meaning"** == **"invalid semantics"** and;  
> **"invalid semantics"**  ⇒ **invalid or rejected syntax**.    

The ***value*** of a pointer is just a number, an integer, an index, specifying the "cell-address" *offset* in some memory address space. So does this mean that `T* x` and using `*x` are just **syntactic** sugar in C for  some `base_addr[x]`?  
**No**. Even in C, and most likely even in earlier languages, there is a **semantic** distinction of types that does not allow pointers to behave as regular (indexing) integers.  
In other words, a pointer is an integral *value* (typically unsigned) with non-arithmetic semantics (and hence a special syntax).

In C++, which is generally more strongly typed than C, the correct integer type to use for pointer arithmetic in *not* `int` but is an implementation-defined standard type called [`std::ptr_diff`](http://en.cppreference.com/w/cpp/types/ptrdiff_t) (which may or may not be an `int`). 

The final observations are about ***commutativity***:  

- Addition is commutative, so both `beg + 1` and `1 + beg` are identical valid pointers to the second element.  
- Subtraction is obviously *not* commutative, so while `end - 1` is a valid pointer to the last element, `1 - end` is meaningless and will not compile.    
Again, the compiler is enforcing the special semantics.

**Pop Quiz:** 

1. Given two integers `a` and `b`, find the mid-point `m` (up to truncation).
2. Given two pointers into an array `a` and `b`, find the element in the middle `m` (up to truncation).
3. Is the simplest code identical? If not why not? 
4. **BONUS:** Could (2) be hypothetically solved with just one additive operation and one multiplication, while maintaining pointer semantics?


<p align="center">👇</p>

### Iterators
Iterators are a generalization and abstraction of pointers. [With a few minor tweaks](https://godbolt.org/g/6HssFm):  

```cpp
typedef float T;                // declare T
std::vector<T> vec = {0,1,2,3}; // a vector of Ts
auto beg = vec.begin();         // iterator to first element in the vector
auto end = vec.end();           // end iterator 
int cnt = end - beg;            // element count or offset
auto last = beg + cnt - 1;      // iterator to the last element
int neg = beg - end;            // a negative or backwards offset from 'end'
```
The code is essentially the same. Again we see the interaction between the iterator type `std::vector<T>::iterator` and an offsetting integer. In this case the correct integral type to use would actually be `std::vector<T>::iterator::difference_type`.  
This code works because `std::vector<>` iterators are [`RandomAccessIterator`](http://en.cppreference.com/w/cpp/concept/RandomAccessIterator)s and thus support the `+` and `-` operators as used here.   
Of course, `beg + end`, `42 - beg` and `beg * 3` are still invalid expressions and do not compile.

Non-`RandomAccessIterator` iterators do not support these operators, but we may still achieve something similar:

```cpp
typedef float T;                    // declare T
std::set<T> set={0,1,2,3};          // an set of Ts
auto beg = set.begin();             // iterator to first element in the set
auto end = set.end();               // end iterator 
auto cnt = std::distance(beg, end); // element count or offset

auto last = beg;                    // copy 'beg' into `last`
std::advance(last,cnt - 1);         // increment 'last'  to the last element

auto neg = std::distance(end, beg); // a negative or backwards offset from 'end'
```
The function `std::distance()` functions as the offset finding subtraction operator and using it is only syntactically different. Conversely, although `std::advance()` functions like the integer offseting operator `+`, its API and semantics are different, since it mutates its input argument and returns `void`. I guess it is more akin the sematics of the `+=` operator (more on mutation later). So we must initilize `last` with two (non-composable) steps.  
We may be seeing here an API design decision that may not have fully considered iterators and their operators in the Affine Space context... [[^4]]

[^4]: At least for the non-`InputIterator` cases.

<p align="center">⌚</p>

### `<chrono>` Types

But enough with pointer-like examples. Consider the following code: 

```cpp
using namespace std::chrono;
auto beg = system_clock::now(); // start time
// ... do something worthwhile
auto end = system_clock::now(); // end time
auto dur = end - beg;           // total duration
auto almost = beg + dur - 1ms;  // just before end
auto next = beg + 2*dur;        // time when taking twice as long
auto ago = beg - end;           // a negative or backwards time offset from 'end'
```
The C++11 `<chrono>` library is the standard way to measure time (in C++20 it will also portably handle dates). At its heart, `<chrono>` distinguishes between two distinct types:  

1. `std::chrono::time_point<>` - a point in time
2. `std::chrono::duration<>`   - a time interval

In the snippet above, `beg`, `end`, `almost`, `next` and `ago` are all `time_point`s.  
The variables `dur`, `2*dur` and the literal `1ms` are `duration`s.

As expected, `beg + end`, `42ms - beg` and `beg * 3` are again invalid expressions and do not compile.  
This is no coincidence. [Howard Hinnant](https://github.com/HowardHinnant) the original `chrono` designer was well aware of Affine Space semantics.  

Are you seeing the pattern yet?  

<p align="center">🚀</p>

Up to now all the types we've seen have been one dimensional types like indexes, addresses and points along the time line. But as its name implies, Affine Space semantics also extend to higher dimensions. 

## Motivating Counter-Examples 

### Point Arithmetic

As you may have surmised by now, what all these examples have in common is the concept of a type representing a location, a point or a position and a second *associated type* representing a displacement (shift, offset, duration). These concepts naturally generalize to higher dimensions. 

Consider this innocuous [OpenCV](http://opencv.org) code snippet:

```cpp
// Vector math is fine
cv::Vec2i jump = { 0,42 };        // a 2D vector: up
cv::Vec2i step = { 42,0 };        // a 2D vector: right
cv::Vec2i jump_fwd = jump + step; // the jump direction (vector addition) 
cv::Vec2i run = 2 * step;         // faster motion      (vector scaling) 
cv::Vec2i bump = -step;           // inverted direction (vector scaling)
// ...
```

In higher dimensions, vectors take the role of the scalars for the displacements (shift, offsets) that we saw previously. Vector math is well defined in linear algebra and all is well. (`cv::Vec2i` is a 2D vector with integer elements.) 

We continue:

```cpp
cv::Point pos = { 0,0 };                      
cv::Point next_pos = pos + cv::Point{step}; // WAT?
cv::Point dir = next_pos - pos;             // Umm..
cv::Point huh = -pos;                       // Huh?  
cv::Point hmm = pos * 2;                    // Hmmm...
```

At first glance it would appear that the `cv::Point` type represents a 2D point. However, not only does it not easily interact with the `cv::Vec2i` type (requiring an explicit cast), in fact, it supports all the same arithmetic operations as the vector type. It does not expose the semantics of a "true" point type where **"invalid semantics"**  ⇒ **invalid or rejected syntax**.

<p style="text-align: center;"><img src="../../assets/this-is-affine.jpg" width="400px"/></p>

This type of API may be convenient, but as the Mars Climate Orbiter developers learned so is [storing thruster force in *"pounds of force"* and/or *"newtons"*](https://en.wikipedia.org/wiki/Mars_Climate_Orbiter) as bare untyped numbers.

Not to pick only on OpenCV, the [Eigen C++ template library for linear algebra](http://eigen.tuxfamily.org/index.php?title=Main_Page), provides *only* vectors and ignores the concept of points althogether.  

**Easy Quiz:** Given *N* `cv::Point`s, find their centroid (center-of-gravity).

<p align="center">☁️</p>

### Aloof Points 

My last counter-example is from [PCL, The Point Cloud Library](http://www.pointclouds.org/). Surely a C++ library dedicated to manipulating points would provide points with meaningful semantics.    
Sadly this is not the case: 

```cpp
pcl::PointXYZ origin = {0,0,0};
origin.x += 1; 
origin.y += 2; 
origin.z += 42; 
```
The only meaningful operations one can perform on points, is member access. This causes code using PCL point types to be very verbose and error prone. There isn't even a vector type in PCL. When needed, the Eigen vector types are used (though there are no semantic relations and operators between the types).     

When [someone asked about this API omission](https://stackoverflow.com/questions/25697967/operator-operator-in-pointxyz), the [reply](https://stackoverflow.com/a/25699136/135862) was:  
*"Why would they ? what is the meaning for ,e.g., the + operator? this is not defined mathematically."*  
This is technically correct for the `+` operator but as we've seen, definietly a cop-out with regards to point subtraction and displacement types.   

<p align="center">🔥</p>

## The Affine Space

### Definition

Intuitively:  

- A *point* is a position specified with coordinate values (e.g. location, address etc.).
- A *vector* is specified as the difference between two points (e.g. shift, offset, displacement, duration etc.).

If an *origin* is specified, then a *point* can be represented by a *vector* from the *origin*, however, a point is still *not* a vector in *coordinate-free* concepts.


There are [many](https://en.wikipedia.org/wiki/Affine_space) [mathematical](http://www.cis.upenn.edu/~cis610/geombchap2.pdf) [definitions](http://mathworld.wolfram.com/AffineSpace.html) of an *affine space*, such as [*"a vector space that has forgotten its origin"*](https://ncatlab.org/nlab/show/affine+space). Here's one with a minimal math jargon:  

1. An *affine space* has two types of entities (i.e. *types*): ***points*** and ***vectors***. 
2. All the *vectors* form a *vector space*:
    1. ***Closure*** under the usual two operations: 
        - **Addition of vectors** 
        - **Multiplication by a scalar**.
    2. Vector *subtraction* and negation is "syntactic sugar" for addition with the right-hand-side multiplied by -1.
    3. A *Linear Combination* of vectors is also a vector and is the weighted sum of one of more vectors. 
3. An affine space has the following properties:
    1. There is a unique vector *v* related to a pair of points *p* and *q*, defining two operations:
        - **Subtract two points to get a vector**: *p-q=v*
        - **Add a vector to a point to get another point**: *p+v=q*
    2. An ***Affine Combination* of points into another *point*** is a weighted sum of one of more *points* when the total sum of the weights is exactly 1.
    3. An ***Affine Combination* of points into a *vector*** is a weighted sum of one of more *points* when the total sum of the weights is exactly 0.
    4. A weighted sum of one of more *points* when the total sum of the weights is neither 1 nor 0 is **undefined**.

The *Affine Combination* properties 3.2 and 3.3 are not strictly definitions since they can be directly derived from 3.1 by showing that such a weighted sum can be factored to a sum of vectors (3.3) plus a point (3.2) ([single line proof](http://mrl.snu.ac.kr/~jehee/DDA/DDA_CourseNote.pdf#page=20)). 

**IMPORTANT**: Note that an affine combination of points is *not* a general linear combination and is only defined when the weights sum to 0 or 1, and in each case the result is a different type - a vector or a point respectively. 

### Types

We can translate these definitions to types and operator type signatures  
(`<arg types> -> <return type>`):

1. An *affine space* API consists of three inter-related types: `Point`, `Vector`, `scalar`
2. `Vector` closure consists of:
    1. Addition (infix): 
        - `Vector + Vector -> Vector`
    2. Commutative scalar multiplication (infix): 
        - `scalar * Vector -> Vector` 
        - `Vector * scalar -> Vector`
    3. As syntactic sugar, we can also define (infix) subtraction:
        - `Vector - Vector -> Vector` 
3. We also defined the following operator between `Point` and `Vector`:
    1. Find displacement as (infix) point subtraction:
        - `Point - Point -> Vector`
    2. Displace point as (infix) addition:
        - `Point + Vector -> Point`
        - `Point - Vector -> Point` (syntactic sugar)
    
The types `Point` and `Vector` have the same dimension. They often also share the exact same representation e.g. a tuple of one of more numbers. The numbers are typically (but not always) of the same type as `scalar`. Of course, `Point` and `Vector` may be template classes parameterized by aspects like, element type, precision etc. In `<chrono>` for example, `chrono::duration<>` is a *family* of types and its internal representation generally differs from that of `chrono::time_point<>`.    

If our types are mutable, we can also add the following operators:

- `Vector *= scalar -> Vector`
- `Vector += Vector -> Vector`
- `Vector -= Vector -> Vector`
- `-Vector -> Vector` (unary) negation
- `Point += Vector -> Point`
- `Point -= Vector -> Point`

We cannot add `Point -= Point -> Vector` since the return type is a `Vector` and not the left hand side `Point`.



------

### Draft notes:

- https://eli.thegreenplace.net/2018/affine-transformations/ 
- https://youtu.be/A9-fT-QTRH8 - 

<https://www.one-tab.com/page/ka_HUOj3S2qWfvIfl8ES_A>


<https://youtu.be/jJyKp2Hzee0>

<https://eli.thegreenplace.net/2018/affine-transformations/>

affine combinations are not supported by pointers
mention relation to strong types and unit types.
mutatability
atypical for Abstract Algebras to have 2 types of interacting objects
Category Theory

### Footnotes