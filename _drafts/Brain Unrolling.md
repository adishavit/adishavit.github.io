---
title: "Brain Unrolling"
categories: [C++]
tags: [C++, C++20, generators, coroutines]
---
> Generators and the Sweet Syntactic Sugar of Coroutines

![windmills](../../assets/windmills.jpg)

## The Cambrian

### Good Old Functions

Say we want to iterate over all the elements of a vector.  
We can write a function - a.k.a. ***sub-routine*** - to do that:

```cpp
void vectorate(std::vector<int> const& v)
{
    for (auto e: v)                 // 1. iterate
        std::cout << e << '\n';     // 2. do something: print e
}
```

If we want to draw a line on some image or device, then algorithms in books and resources will typically look something like this function (a simplified [Bresenham](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm) variant):

```cpp
void drawline(int x0, int y0, int x1, int y1) // TODO: Test Code
{
    int dy=y1-y0;
    int x=x0;
    int y=y0;
    int p=2*dy-dx;
    while(x<x1)                     // iterate
    {
       putpixel(x,y,7);             // do something: call putpixel()
       if(p>=0)
       {
           y=y+1;
           p=p+2*dy-2*dx;
       }
       else
       {
           p=p+2*dy;
       }
       x=x+1;
    }
}
```

In both examples, the functions do ***two*** things:

1. They iterate over a sequence, i.e. the vector or the pixel positions along a line;
2. They do some operation with each of the sequence elements

If we wanted to sum the vector elements, we would need to re-write `vectorate()` to sum the elements instead of (or in addition to) printing them.

Similarly, `drawline()` assumes that a function `putpixel()`:

1. is available in scope (for compilation and linking);
2. has the correct signature;
3. will do the right thing when called;
4. will actually return control to `drawline()` for continued operation.

A larger issue here is that both functions are "closed" in the sense that they only return after they have iterated over the *whole* sequence. They *eagerly* process a whole sequence.  

### Callbacks

One common way to overcome some of these limitations is by passing external *callback functions* to our own functions. Some C++ mechanisms that allow this are function-pointers or using "Callable" template parameter types (or Concepts in C++20) for passing e.g. lambdas. This is exactly how predicates work in many STL algorithms for example.

A few well known problems with callbacks and callables are: 

- *Inversion-of-Control*: Letting library code call external code that is not neccesarily trustworthy, valid or correct in mid-computation.
- *Callback-Hell*: Where program flow skips between many decoupled parts of the code that your code becomes extremely hard to understand, reason about and maintain (not to mention any potential performance issues).
- The functions are still *eager* and closed, requiring the creation/processing of the full sequence.

The concept of a function, or ***sub-routine*** goes back to the ENIAC in the late 1940s and the term *sub-routine* is from the early 1950s.

<p align="center"><img src="../../assets/trilobite.png" width="30px"/></p>

If only there was a way to "flip" these iterating functions "inside-out" and iterate over a sequence without pre-committing to, or having to specify a, specific operation...

## Pre-History

### Iterators

The concept of Iterators has been with C++ since the STL was designed by Alex Stepanov and together with the rest of the STL became part of C++98. They are a powerful abstraction for, well, "openly" iterating over elements of a sequence. 

Two related concepts are *Iterator Objects* and *Iterator Adaptors*. These are "stand-alone" iterator types which are often only indirectly or implicitly coupled to a sequence. The C++ standard actually has quite a few of them, both old and new including, for example:

- [`std::istream_iterator`](https://en.cppreference.com/w/cpp/iterator/istream_iterator): a single-pass input iterator object that reads successive objects of type `T` from the `std::basic_istream` object for which it was constructed, by calling the appropriate `operator>>`. The actual read operation is performed when the iterator is incremented, not when it is dereferenced. The first object is read when the iterator is constructed. Dereferencing only returns a copy of the most recently read object.
- [`std::reverse_iterator`](https://en.cppreference.com/w/cpp/iterator/reverse_iterator): an *iterator adaptor* that reverses the direction of a given iterator.
- [`std::recursive_directory_iterator`](https://en.cppreference.com/w/cpp/filesystem/recursive_directory_iterator): an *iterator object* that iterates over the `directory_entry` elements of a directory, and, *recursively*, over the entries of all subdirectories. Here the file system directory sub-tree hierarchy is the "sequence" and not an actual program object with elements (since C++17). 
- Interestingly, the *functions* [`std::prev_permutation()`](https://en.cppreference.com/w/cpp/algorithm/prev_permutation) and [`std::next_permutation()`](https://en.cppreference.com/w/cpp/algorithm/next_permutation) are *not* iterators but do have some iterator-like behavior combined with mutating the underlying sequence.

A useful example of a *user defined* iterator object is OpenCV's [`cv::LineIterator`](https://docs.opencv.org/master/dc/dd2/classcv_1_1LineIterator.html) which is *"used to iterate over all the pixels on the raster line segment connecting two specified points"*. It has a typical iterator object type API (simplified for brevity):

```cpp
class LineIterator
{
public:
    // creates iterator for the line connecting pt1 and pt2 in img 
    // the 8-connected or 4-connected line will be clipped on the image boundaries
    LineIterator( const Mat& img, Point pt1, Point pt2, int connectivity = 8);
    uchar* operator *();           // returns pointer to the current pixel
    LineIterator& operator ++();   // prefix increment operator (++it). shifts iterator to the next pixel
    LineIterator operator ++(int); // postfix increment operator (it++). shifts iterator to the next pixel
    Point pos() const;             // returns coordinates of the current pixel

    // public (!!!) members [ <groan 😩> ]  
    uchar* ptr;
    const uchar* ptr0;
    int step, elemSize;
    int err, count;
    int minusDelta, plusDelta;
    int minusStep, plusStep;
};
```

This iterator does not have an explicit sequence to iterate over. Instead it *lazily* creates the elements of the sequence as the iterator is incremented (using various Bresenham variants).  

`cv::LineIterator` gives us incremental access to all the pixels along a line in the image. We may draw the line by setting color values to the pixels, easily creating e.g. color gradients external to the iteration itself. Alternativly, we may sum the pixel values along the line or mix its value with some alpha channel. The possibilities are endless and need not be known to `cv::LineIterator` itself. 

Here is a usage example from the [docs](https://docs.opencv.org/master/dc/dd2/classcv_1_1LineIterator.html#details):

```cpp
cv::LineIterator it(img, pt1, pt2, 8);
std::vector<cv::Vec3b> buf(it.count);
for(int i = 0; i < it.count; i++, ++it) // copy pixel values along the line into buf
    buf[i] = *(const cv::Vec3b*)*it;
```

It externalizes the iteration to the user code.

### Imperfect Abstraction

However, there still are a few abstraction-related problems with this code (horrible public members notwithstanding). 

First, the iterator's `operator*()` (i.e. `*it` in the snippet) returns a `uchar*` which must be cast to a pointer of the actual pixel type of the image. This is awkward and could be fixed by e.g. deriving a class template from `cv::LineIterator` when the image pixel type is known at compile time (similar to how `cv::Mat_` is a template matrix class derived from `cv::Mat`.)  
But this is a `cv::LineIterator` specific issue, so we will not dwell on it here (though we shall return to it later).

There are more severe concerns which are, in fact, symptomatic for all iterator objects.

#### The Odd Couple

<!-- ![odd-couple](https://media.giphy.com/media/E22xshq7vPL7q/giphy.gif) -->

How do we know when to stop incrementing the iterator (e.g. `++it`)?  
How do we know the sequence is "done"?

For `cv::LineIterator` we must make sure we iterate at most `it.count` times.  
For `std::istream_iterator`, the default-constructed `std::istream_iterator` is known as the [end-of-stream iterator](https://en.cppreference.com/w/cpp/iterator/istream_iterator). When a valid `std::istream_iterator` reaches the end of the underlying stream, it becomes equal to the end-of-stream iterator.  
A `std::reverse_iterator` must be compared to the corresponding sequence's `rend()` iterator and `std::recursive_directory_iterator` must be compared to the value returned by calling the free function `std::end()` on it.  

Note that the last two examples (`std::reverse_iterator` and `std::recursive_directory_iterator`) demonstrate one of the biggest drawbacks of the iterator abstraction: the end-iterator is tightly coupled, at run-time,  to the begin-iterator creation object. This is a pitfall when the provided end iterator is of the correct *type* but *not* created from the same sequence. It is undefined behavior. The code will compile silently and if you're really lucky you'll get a crash (if not, nasal demons may ensue).

<p align="center"><span style="font-size:2em;">🦖</span></p>





