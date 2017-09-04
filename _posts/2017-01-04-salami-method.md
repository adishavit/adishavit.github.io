---
title:  "The Salami Method"
categories: [C++, Architecture]
tags: [C++, architecture, cross-platform]
---
C and C++ are probably the only viable languages for true cross-platform development. 

![img](../../assets/salami-banner.jpg)

* Table of Contents
{:toc}

## Introduction  

Over the past several years, I worked on several core C++ libraries that needed to be integrated into multiple platforms. The same C++ code needed to run on mobile devices (iOS, Android), desktops (Windows DLLs, Linux shared objects),  cloud services (Linux and .NET integration) and web-browsers (emscripten, [FireBreath](http://www.firebreath.org/)).  

That is quite a list of target platforms, each with its own constraints, its own "native" environment, language, run-time, type-system, UI-system and OS models. In this context, "native" means the "natural" way to develop on the platform, e.g. Java on Android, JavaScript in the browser and Objective-C/Swift on iOS. This is contrary to C++ being a "native" language in the ["Going Native"](https://channel9.msdn.com/Shows/C9-GoingNative), bare-metal, sense. 

Along the way, I developed my own way of structuring such cross-platform (core) C++ code.  

<p style="text-align: center;"><img src="../../assets/salami_stub_top.svg" width="40px"/></p>

## The Salami Method

> Like all good architectures, the Salami Method tries to cleanly separate concerns.  
> Nevertheless, it lacks the greasy, heart-attack-inducing goodness of a good salami.

Be it creating DLLs, Android NDK/JNI, C++ on iOS or even GUI-based desktop apps, many samples, articles, tutorials and examples frequently mix platform specific code with core functionality. While this might serve to demonstrate specific API usage and techniques, it often leads to spaghetti code.
The result is a refactoring and maintenance nightmare that is non-portable, untestable with ample nooks and crannies for bugs to hide in. 
To make things worse, module boundaries (e.g. DLL APIs), are dangerous places where error handling must be considered and addressed carefully as things like exceptions might not be able to percolate further causing program termination or undefined-behavior.

The salami method finely distinguishes between the different aspects and layers required for exposing platform-*independent* C++ on different "specific" platforms. At its extreme it strives to create a *single*, *thin*, *transparent* layer for each such aspect so that each layer is more easily built, tested, debugged, managed and maintained.

<p style="text-align: center;">
  <img src="http://i.giphy.com/ZOPQqUVKtUOI0.gif" width="250px"/>
</p>

The benefits of thinly slicing our API include:

- **The DRY principle**: Sharing as much code as possible between platforms, avoids duplication and reimplementation.
- **Single Responsibility and Testability**: Each layer has its own single purpose and can be debugged and tested independently.
- **Consistency**: Business logic remains isolated in the deepest layers and is shared among all target platforms. This ensures consistent behavior across platforms.
- **New Platforms**: Code is future ready for targeting new platforms. 
- **Developer Skills**: Leverage skills of different developers independently at relevant parts of the code. No mixing of concerns. 
- **Refactoring**: Well separated concerns allow for easier refactoring.

We identify the following conceptual layers:

![img](../../assets/salami_method.svg)

Often you will find that not all of these layers make sense as distinct code layers. In some situations it may be more practical to merge two layers into a single layer - *thicker slices*, if you will.  
Similarly, some platforms may not require all the layers described here. For example, C++ code can be integrated into iOS and Objective-C code more painlessly than e.g. creating an Android JNI. On these platforms the top layers can be left unused.  

Even when not actually implementing all these layers as separate code, it is important to realize that they might still be conceptually in the code.

<p style="text-align: center;"><img src="../../assets/slice.svg"/></p>

### The Cross Platform Core
At the heart of our system lies the cross-platform C++ core. The apple of our eye, the IP, the business logic, the *raison d'être* of this exercise. This is your well written, idiomatic C++ code base. Since it is cross-platform, this code, along with its dependencies, ought to build on all the target platforms using their tool chains. It is recommended that this core is buildable as one or more (static [[^1]]) libraries. 
 

> **Testing**: Use your favorite unit and functional test framework at this level. 

[^1]: Prefer static libs because they can be integrated directly into the DLL further down and enabling delivery of a single DLL file (per platform).


<p style="text-align: center;"><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/></p>

### The Cross Platform Public C++ Interface (XAPI)
At this layer we expose only the public API of our C++ core.  
Here we consider the codebase from the user or client usage perspective as opposed to the architectural design perspective we may have used in the core layer. We should apply good API design principles as usual, thinking about how the API is to-be used, considering things like initialization and shutdown, lifetime management, sessions, configuration, serialization etc.

It is a kind of SDK that can be delivered and built by other teams without the need for access to, or familiarity with, the full codebase, the build tools or any dependent libraries.

This is the opportunity for a compilation firewall (e.g. via the [pImpl Idiom](https://en.wikipedia.org/wiki/Opaque_pointer)). Keep private headers, types and repos private.   
We might want to remove certain types from the public API and use various alternatives instead. For example, perhaps our codebase passes and manipulates `std::filesystem::path` objects. It is often easier for external users to use `std::string` instead (see my ["Emscriptened!"](https://adishavit.github.io/2016/emscipten-fs-path/) post for an exactly this use case).  
Another example I came across was a team that did not <del>know how</del> want to include Boost in their build chain. They asked that any Boost used in the core is contained within the code and does not leak through the API. In fact, we eventually delivered only headers and binary libraries for this particular team.

The Cross Platform C++ Public API layer is possibly a poster child candidate for C++ Modules when they finally land.  

> **Testing**: At this level, mock and unit test the API/SDK itself.  
> **Naming Convention**: Given a core file `core.cpp`, I would often have a corresponding file `core_api.cpp`

<p style="text-align: center; line-height:0px;"><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><br><img src="../../assets/slice.svg"/></p>

For smaller projects, or for ones where the core architecture already provides a natural API, you may be able to skip this layer and go directly to the next one.

> ### ⚠️ WARNING!
> From this point on, it is *essential*, ***not*** to insert any functional or business logic into any subsequent layer since some target platforms might not actually be using them. Introducing functional logic henceforth will cause duplication of effort (DRY-violation) or worse, inconsistent behavior between platforms.


### The Cross Platform Public C Interface (XCAPI)
Although some platforms may be able to directly assimilate C++ APIs (e.g. iOS, emscripten-embind (see warning above)) the *real* truly cross-platform *lingua-franca* of computing is C. The main reason for this is ABI compatibility (no nasty mangling issues) and a simple, sufficiently well defined, binding/linking model supported practically by anything vaguely resembling a computing platform . This means that even though you can cross *some* module boundaries with C++ objects (e.g. with COM), doing it with a C API is much simpler (once you have it of course) and much more widely supported. 

At this layer, we define a C-style interface to our C++ public API defined in the previous layer. A C-style interface means an `extern "C"` interface based on global standalone functions without C++ classes. These functions will use the C++ API inside their `.cpp` implementations. 
Sometimes, the function based API will pass C++ types as arguments, e.g. `std::string` is a common example - in these cases additional conversions would be required later on (at the cost of thickening that layer - more about this in the footnote later on).

This layer should be as thin as possible and only be concerned with the "usual" aspects of wrapping a C++ API in C. Often this would simply be an `extern "C"` function calling a C++ function. More complex APIs may need managing of object lifetimes. Some APIs can get away with one global "Singleton" object which is all that is needed for using the API. In other cases, you might need to pass around opaque handles to objects into and out of the C API.  

Note that C wrapper friendliness in general and object lifetime management via a C API in particular *should* be a significant concern to take into account when designing the public C++ interface described in the previous section.

Whether you may or may not use C structs in the API or must limit yourself only to primitive C types will ultimately be determined by the type richness or paucity of the target platforms. For example, you can pass C structs through DLL functions, but Android JNI is less forthcoming in allowing them through. You will have to decide at which layer to make such a decision, e.g. keep the C-struct on all other platforms and only break it up on JNI.  

This layer is also the place to resolve overloaded C++ functions by creating separate differently named C functions to call the overloaded C++ function.

> **Testing**: At this level, mock and unit test the C API/SDK itself verifying conversions, overloads and object lifetimes are properly done.  
> **Naming Convention**: Given a public API file `core_api.h`, the corresponding file would be `core_c_api.cpp`

A recent blog post, [Generate C interface from C++ source code using Clang libtooling](http://samanbarghi.com/blog/2016/12/06/generate-c-interface-from-c-source-code-using-clang-libtooling/), describes some of the issues involved in generating a C interface from C++ source code, and how an automated tool might do this... automatically.  
If and when such a tool is available, it will make the previous C++ API layer even *more* important as the automated tool will just convert arguments and function calls without any "design" related considerations.

Our layer files might look something like this:  

```cpp
// foo_session_c_api.h
extern "C"
{
   bool initFromFileName(std::string const& fileName);
   bool initFromCount(int count);
   bool processBuffer(uint8_t* buffer, int size);
   bool isReady(); 
}
```
- Use `extern "C"` in header for C linking;
- Break overloads with longer names e.g. `FooSession::init()`.

```cpp
// foo_session_c_api.cpp
#include <foo_session_api.hpp> // C++ API
#include "foo_session_c_api.h" // header for this file

Foo::Session the_session; // use a single global "singleton".

bool initFromFileName(std::string const& fileName) 
{  return the_session.init(fileName); }

bool initFromCount(int count)
{  return the_session.init(count); }

bool processBuffer(uint8_t* buffer, int size)
{  return the_session.process(gsl::span<uint8_t>(buffer, buffer+size)); } // Use gsl::span<>

bool isReady()
{  return the_session.isReady() }
```
- For brevity, this example uses a single global session object as the backend of the API. 
- Typically, all the functions are simple thin wrappers for method calls. 
- Use type helpers like [`gsl::span<>`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#bounds1-dont-use-pointer-arithmetic-use-span-instead) or [`std::string_view`](http://en.cppreference.com/w/cpp/string/basic_string_view) to lift weaker low-level types to stronger safer types as early as possible [[^2]].  

[^2]: The sharp eyed reader would have noticed that the first function takes an argument of type `std::string` which is a C++ type and would certainly not get proper C ABI (and require the user to include `<string>` of a potentially different STL implementation. There are several reasons I put it in the example that I wanted to demonstrate: (1) The other C-style abstractions, e.g. lifetime management, overloaded name resolutions etc. are just as  important at this point; (2) As mentioned above, we can often do the remaining type conversions at the next layer (3) Some tools like emscripten embind can automatically consume standard C++ types like `std::string` so at this layer there is no gain from making the API even more primitive.

<p style="text-align: center;line-height:0px;"><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><br><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/></p>

Alas, sometimes it is not practical to create C-style APIs and the next layer may have to work directly with the C++ API. For example, I once had a continuation-based asynchronous C++ core that would return results asynchronously on different threads. This worked fine on iOS, but these multi-threaded responses had to be manually injected into JVM threads (on Android). Adding an additional C-style API in-between was an added complexity (to a complex enough flow). Instead, I had the Android JNI interface use the C++ API directly. The cost of this was that, had I needed to target a third platform, I might have needed to re-implement the threading logic injection on that platform too.

### The Platform-Specific Boundary Interface Layer (BIL)
Up until now, everything we wrote was cross-platform C/C++. From this point onward, everything we do is platform-specific. This layer needs to be implemented independently for *each* target platform. The role of this layer is to have one clear place in the code where the core interfaces the target platform.  
This is where platform-specific conventions, constraints and conversions are enforced. It is here that we must perform bidirectional data type and value conversions between the "native" target platform and our platform agnostic code from the previous layers. The required conversions are dictated by each particular target.

For many platforms, this layer is the *module boundary*. The final frontier. Anything that happens beyond it is happening in a different environment, run-time, language and universe. Exceptions are not welcome across this boundary. Unhandled exceptions percolating to this layer, will cause severe havoc, undefined behavior and most likely a program or process crash.  
This layer is the final stand for handling any exceptions that have made it thus far. It is also the natural place to insert any logging logic since it is here that we have access to the platform's logging facilities.  
Combined with logging, exceptions can be caught, logged and reported. Furthermore, on some platforms, like the JVM, it is possible to throw a new JVM Java exception with the C++ exception info - simulating passing the exception through the module boundary.

> **Testing**: Unfortunately, I have yet to come by a good general solution for testing code written at this layer. I have a long standing, as-yet unanswered, [StackOverflow question](http://stackoverflow.com/questions/23268158/unit-testing-jni-calls) about this. Yet another reason to keep this layer as thin as possible.  
> **Naming Convention**: Since this is platform specific code, each target platform might have a different name. Given a public C API file `core_c_api.h`, the corresponding files might be called `core_c_api_jni.cpp` or `core_c_api_dll.cpp` for Android JNI or Windows DLLs respectively.

Windows/Linux DLL files typically look something like this:  

```cpp
// foo_session_c_api_dll.h
extern "C"
{
   bool DLL_EXPORT FooSession_initFromFileName(LPCSTR fileName);
   bool DLL_EXPORT FooSession_initFromCount(int count);
   bool DLL_EXPORT FooSession_processBuffer(unsigned char* buffer, int size);
   bool DLL_EXPORT FooSession_isReady(); 
}
```
- Prefix a "namespace" `FooSession` to the function name to avoid export name collisions. We could have done this at the previous level as well - in that case we would have needed different names here to avoid ambiguity.
- The `DLL_EXPORT` macro resolves to the proper platform specific attribute based on build and include configurations, e.g. `__declspec(dllexport)` on Windows or `__attribute__((visibility("hidden")))` on Linux. I usually have CMake generate this macro for me for all builds. 
- Uses e.g. Windows-specific `LPCSTR` on Windows to pass strings. 

```cpp
// foo_session_c_api_dll.cpp
#include <foo_session_c_api.h>     // C API
#include "foo_session_c_api_dll.h" // header for this file

bool DLL_EXPORT FooSession_initFromFileName(LPCSTR fileName) try
{  return ::initFromFileName(fileName); } // automatic LPCSTR conversion to std::string
catch (...) { return false; }

bool DLL_EXPORT FooSession_initFromCount(int count) try
{  return ::initFromCount(count); }
catch (...) { return false; }

>bool DLL_EXPORT FooSession_processBuffer(uint8_t* buffer, int size) try
{  return ::processBuffer(buffer, size); }
catch (...) { return false; }

bool DLL_EXPORT FooSession_isReady() try
{  return ::isReady() }
catch (...) { return false; }
```
- Catch and handle all exceptions. I find the [function-try-block syntax](http://en.cppreference.com/w/cpp/language/function-try-block) more concise here.
- Typically, all the functions are simple thin wrappers for C API calls. 
	
On Android we typically have something like this:

```cpp
// foo_session_c_api_jni.cpp
#include <jni.h>               // JNI headers#include <android/log.h>       // Android loggin facilities#include <foo_session_c_api.h> // C API#include "jni_utils.h"         // For JNIByteArrayAdapter and exceptionHandlerJNIEXPORT jboolean JNICALL Java_initFromFileName(JNIEnv* env, jobject thiz, jstring fileName) try
{  return ::initFromFileName(jni_utils::getString(env, fileName)); } // JNI string helper
catch(...) { return exceptionHandler(); }

JNIEXPORT jboolean JNICALL Java_initFromCount(JNIEnv* env, jobject thiz, jint count) try
{  return ::initFromCount(count); }
catch(...) { return exceptionHandler(); }

JNIEXPORT jboolean JNICALL Java_processBuffer(JNIEnv* env, jobject thiz, jbyteArray buffer) try
{  
   jni_utils::JNIByteArrayAdapter buffer_span(env, buffer); // JNI helper wrapper
   return ::processBuffer(buffer_span.ptr(), buffer_span.size()); 
}
catch(...) { return exceptionHandler(); }

JNIEXPORT jboolean JNICALL Java_isReady(JNIEnv* env, jobject thiz) try
{  return ::isReady() }
catch(...) { return exceptionHandler(); }
```
- Function signature, types and names must conform to JNI conventions, e.g. must prefix with `Java_`.
- No need for header, the NDK compiler needs only this `.cpp`.
- Data cannot always be passed directly to the C API and must be converted via the JVM (and taking care to free resources when done). The `jni_utils::JNIByteArrayAdapter` class is an RAII wrapper over the JVM calls that adapts a JNI array to a C-style array. How it works is beyond the scope of this post. Similarly, `jni_utils::getString` provides access and RAII facilities for JNI strings.
- For exception handling, my JNI utils also includes `exceptionHandler()` which intercepts any caught exceptions, logs them using `__android_log_print()` and generates a JVM exception using the `JNIEnv::ThrowNew()` API. Again, the details are beyond the scope of this post.

<p style="text-align: center;; line-height:0px;"><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><br><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/></p>

Having reached the final frontier, it is time to boldly go to infinity and beyond.

### The Native Import Layer (NIMP)
Every target platform has a different way of importing, loading and consuming our service. At this end of the universe we may be on a different device and hardware, running a different OS and writing a different language altogether.

Once the service is loaded it would typically be available through what is commonly called a "native" interface. Ironically, "native" in *this* context refers to our C/C++ bare-metal service as opposed to the managed or interpreted code running on a VM such as the CLR, JS or the JVM.

The "native" interface is an exact match to the boundary interface we exposed above. There is not much to say since the syntax and types are completely dictated by the importing target platform.

> **Testing**: You should run Integration Tests for/on the target platform using whatever facilities are available there.     
> **Naming Convention**: Since this is platform specific code, each target platform might have a different name. Given `core_c_api_jni.cpp` the corresponding Java file might be `core_native.java` or `core_native_dll_wrapper.cs`.

 For example, our Java JNI interface might look something like this:

```java
// foo_session_native.java
// imports ...
public class FooSession 
{
   static { System.loadLibrary("native_foosession"); } // load the DLL
   
   public static native boolean initFromFileName(String fileName); 
   public static native boolean initFromCount(int count); 
   public static native boolean processBuffer(byte[] buffer); 
   public static native boolean isReady();
}
```
- This is a direct mapping of the JNI function names and type to the [Java type system](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/types.html).
- The code for managed C# is almost identical.  

For C/C++ DLLs, changing the macro `DLL_EXPORT` to e.g. `__declspec(dllimport)` (on Windows) at build time, we can use the same header `foo_session_c_api_dll.h` as for exporting (assuming we did not `#include` any unnecessary, non-deliverable, header into it.

<p style="text-align: center; line-height:0px;"><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><br><img src="../../assets/slice.svg"/><img src="../../assets/slice.svg"/><br><img src="../../assets/slice.svg"/></p>

The prototypes and types allowed by the platform's export/import facilities often create ugly, low-level APIs which are verbose, inconvenient and unnatural to use on the target platform. As responsible library writers, we'd like to facilitate a more natural interaction interface for our users. This is the role of the last layer.

### The Native Interface Wrappers (NIW)
Given the low-level interface of the previous layer, it is frequently very effective to wrap it with a higher level interface more suitable for the target environment. This wrapper would simply wrap the `native` calls, but provide a more natural and familiar syntax and higher level types for the users. This will allow e.g. the mobile developers to seamlessly work with our code without knowing too much about loading native libraries or native data types.

> **Testing**: If you use such a wrapper, it should be part of the Integration Tests mentioned above.     
> **Naming Convention**: This is platform specific code so each target may have a different name. The interface can enrich the existing native class in the same file or done as a standalone API.

Let's say we have a face detector that returns the 2D position of the center of a face in an image. The native result is returned as an array of two `float`s since this is the only way to pass such data through the native interface. We can enrich that to return an Android Java type: 

```java
// face_detector_native.java
import android.graphics.PointF; // Android point typepublic class FaceDetector {   static { System.loadLibrary("native_facedetector"); } // load the DLL      // native import function/method, returns a float array   public static native float[] getFaceCenterPoint();      // Java-ized wrapper: return proper 2D point type   public static PointF GetFaceCenterPoint()   {      float[] centerPt = getFaceCenterPoint();     // call native function      return new PointF(centerPt[0], centerPt[1]); // return as Android Java type: PointF   }}
```
- The wrapper here is just another *non*-`native` method: `GetFaceCenterPoint()` for the same class. Similar wrappers can be made for .NET managed code as well.

## Beautiful Symmetry

In the XCAPI and BIL layers, we created C-style wrappers for the C++ API. This entailed going to lower-level code with less expressive syntax and weaker types. At the NIW layer we have the opportunity to undo that and restore order. We wrap the low level code with higher level functions and types. We get a kind of mirror symmetry between our cross-platform code and the platform-specific SDK/API running on the target platform.

<p style="text-align: center;"><img src="../../assets/symmetry.svg" width="400px"/></p>
<p style="text-align: center;"><img src="http://i.giphy.com/8Z8rBD9Oyn744.gif" width="340px"/></p>
<p style="text-align: center;"><img src="../../assets/salami_stub.svg" width="40px"/></p>

## Write Once, Run Anywhere... Not!

The Salami Method is quite removed from the ideal of *"write once, run anywhere"* proclaimed by some languages and platforms. On the other hand, it does allow supporting a *very* wide variety of platforms while keeping the high-performance profile of C/C++ (in as much as the platform allows).

As [Coldplay said](https://youtu.be/8iYdJH1i4rc): *Nobody said it was easy!*  
But what's the alternative?  
Keep separate code-bases with multiple teams for maintenance?  
Spaghetti-code mixing all the above in single monolithic structures? 

From my experience, most projects do not implement *all* these layers as distinct interfaces. Depending on the target platforms and practical considerations, some of these layers might be merged, though it is important to keep in mind that the role of each such layer still holds.

<p style="text-align: center;">
  <img src="http://i.giphy.com/G42OC0n4TqV0Y.gif" width="250px"/>
</p>

> ## Meta 
A first post for the new year! And my longest one yet!   
Although this post has been in planning for several years, a colleague prodded me to finally write it.  
I hope you enjoy it. **Happy New Year 2017** 🎉 
>
> I'd like to thank [@galsh83](https://twitter.com/galsh83), [@MaximRaskin](@MaximRaskin) and [@orens](https://github.com/orens) for their valuable feedback on this post.  
If you found this post helpful, or you have more thoughts or horror stories on this subject, please leave a message in the comments below, on Twitter or Reddit.

 
*Credits: 
[banner](https://flic.kr/p/7RJ5RA) ::
[giphy](http://gph.is/1dHbVr5) :: 
[giphy](http://gph.is/YZIdpy) ::
[giphy](http://gph.is/2cu98bo) :: 
[Starving Donald Duck Scene](https://youtu.be/KqEVYbPw9lI)*