---
title: "code::dive Trip Report"
categories: [C++]
tags: [C++, Conference, code-dive]
---
Trip report from my first ever C++ conference!

![code-dive](../../assets/code_dive_2017/banner.jpg)    

The **code::dive** conference took place last week in Poland at the beautiful city of Wrocław (pronounced *VROWTS-WAF*). I had seen excellent videos of talks from previous years and this year promised a roster of distinguished speakers.    
I had never been to a C++ conference before and beyond the talks I was looking forward to meeting face-to-face many of my online Slack, Twitter and GitHub friends, followers and followees, as well as many of the speakers whose work I've been following online through blogs, repos and videos.  

## The Venue
The two day conference was held at the New Horizons Cinema theater at the center of town. Each day comprised of 5 sessions with 4 simultaneous tracks per session - that's a total of 40 talks!  
<img src="../../assets/code_dive_2017/ranges.jpg" width="300px" style="float:right;margin: 5px"/> Being modern cinema theaters, the halls were *very* comfortable with the projected presentation slides *dwarfing* the speakers! *No font was too small*. 
Interestingly, the conference does not have any plenary keynote lectures - presumably since there is no single hall that can accommodate all the participants.  

In addition, by passing a preliminary quiz online, you could register to participate in the world's first **Escape Room** designed *"especially for programmers, testers, DevOps, and sysadmins"*!

## Day 1
Despite the large number of attendees, registration was orderly and efficient.  
<img src="../../assets/code_dive_2017/badge.jpg" height="300px" style="float:right;margin: 5px"/>The conference name badge had the whole program including the room numbers printed on one side. I think that's a great idea and very convenient.  
Unfortunately, the participant names were handwritten on the badge using white-board dry-erase markers. By the time of the first break most people were walking around with blank badges. This made introductions more difficult and awkward (though OTOH, it was a kind of silly ice-breaker too). I ended up using a permanent marker from my hotel.    
For next year, I'd recommend a sticker with a printed name or at least an abundant supply of permanent markers.

### Customization Points that Suck Less

#### Michał Dominiak
I actually wanted to attend at least 3 of the 4 talks. Unfortunately, my physical self could only attend one temporal track at a time. I shall be watching out for the videos to fill in the blanks.  
I had watched Michał's CppCon talk on the subject and found it a bit too dense (maybe because I was watching it at x1.8 speed). I hoped that attending it live in real-time (x1.0) would make it more approachable. The talk was very interesting but still quite dense. I am intrigued by non-polymorphic type-erasure and how [Dyno](https://github.com/ldionne/dyno) may be used in the context of customization points. I will probably watch the talk again on video.  

### Faces of Undefined Behavior

#### Andrzej Krzemieński
*-- "Optimizing buggy slow code into buggy fast code."*  
I've been reading and following Andrzej's [blog](https://akrzemi1.wordpress.com/) for a long time and was looking forward to this talk. The essence of the talk is the seemingly simple assertion that the compiler can assume that _undefined behavior_ never happens.
However, a simple reading of this statement is misleading. In today's compilers what it really means is that the compiler *actively* assumes that undefined behavior *never* happens, and actively makes logical assumptions and implications about the code based on this assumption! The implications are that UB can cause the compilers to remove parts of code that the user had never intended.  One example shows the compiler removing a runtime check for UB and its associated error logging statement! Andrzej then continued to discuss how and whether to perform runtime checks for potential UB. These checks themselves can cause static analysis tools to inhibit some important warnings and how Contracts (now being proposed) may help mitigate this. This was the first time I've seen an explanation how language support for Contracts is different from regular runtime `assert`s. 
I wholly recommend this talk - it was one of the most interesting, well planned and well delivered talks in the whole conference.

### Exploring C++17 and Beyond

#### Mark Isaacson
The talk briefly mentioned many of the new C++17 features and then, somewhat arbitrarily, focused on three: `std::string_view`, `if constexpr` and, my favorite - the as-yet _non_-C++ [operator-dot](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0416r1.pdf).  
It was obvious that Mark is an experienced speaker and I'd recommend watching the video if only just for the intriguing `operator.` exposition.  

### Escape Room

The escape-room game was great fun! I was teamed up with 3 other participants and sent in to investigate and uncover cyber-security fraud evidence. The challenges were indeed challenging and fun. Secrecy forbids me from saying more about it - though regex-foo (not-mine) was a definite plus. We were the second team to actually *finish* the game and did so with just 30 seconds to spare!  
The escape-room was a brilliant idea and _very_ well executed. It was obvious that it had taken some serious planning and setting up - and the production was **awesome**! Kudos to the code::dive escape::room team!

Unfortunately, my alloted time-slot was scheduled such that I had to miss most of the last two talks of the day. In the future, it would be great if the escape-room slots were scheduled to overlap at most a *single* talk+break.  

Before the escape-room, I managed to catch the first 30m of Andreas Weis's *Howling at the Moon: Lua for C++ Programmers*. It gave a good overview of Lua's unique features with surprising screenshots from old Monkey-Island adventure games. I will be watching the end of the talk when it comes online. 

### Evening
Following an open dinner invitation in the C++ Slack `#code_dive` channel, I joined a  group of super-friendly Polish C++ computer-vision developers all from [Adaptive Vision](https://www.adaptive-vision.com/en/). Although we had never met before, we had a wonderful evening at [Szynkarnia](http://szynkarnia.com.pl/). We were later joined by many more conference attendees, particularly a large group of Dutch students led by [Wouter van Ooijen](https://www.voti.nl/blog/).       

## Day 2

### The Fastest Template Metaprogramming in the West

#### Odin Holmes
Odin Holmes is a great speaker and I had great expectations for this talk. I was not disappointed! The talk began with a focused and *very* gentle introduction to Template Metaprogramming (TMP) and gradually proceeded into more advanced material.  
<img src="../../assets/code_dive_2017/RuleOfChiel.png" width="400px" style="float:right;margin: 5px"/> Using clever compilation time measurements, Odin and Chiel Douwes came up with [The Rule of Chiel](https://www.reddit.com/r/cpp/comments/6gur2x/the_rule_of_chiel_aka_compiletime_cost_of/) which ranks different template related operations by compilation time. These are a function of the complexity and count of new type instantiations done by the compiler during the compilation process. Guided by these measurements, Odin showed how to choose the lightest alternatives for use in TMP - e.g. prefering template aliases to type instantiations wherever possible. The talk showed the framework for building an optimally packed `tuple` implementation that reorders its types to optimally reduce alignment-related padding. This all happens at compile time and could be made standard conforming.  

### Introducing the Ranges TS

#### Eric Niebler
[Eric Niebler](http://ericniebler.com/) came to the conference directly from the standard committee meetings in Albuquerque, Arizona where the Ranges TS was finally officially inducted into C++20. If code::dive had keynote lectures, this would be one of them. The hall was packed and the expectations were high. Unfortunately, Murphy had other plans, and the talk was plagued with multiple technical glitches including approx. 20 of projector problems after the first slide.  
Fortunately, both Eric and the session host Rafał Motriuk, a veteran radio journalist and a true professional, handled the glitches with grace and an impromptu quiz about Eric's musical taste.  

The parts of the Ranges TS that made it into the standard are currently just a small subset of the [Ranges library](https://github.com/ericniebler/range-v3). Eric is now working on standardizing the true "meat" of the library: range views and adaptors. These lazy concepts are where ranges truely shine and I cannot wait until they become standardized. Eric continued by showing several exciting projects extending the reach of ranges. These include [*Towards Composable GPU Programming: Programming GPUs with Eager Actions and Lazy Views*](http://eprints.gla.ac.uk/146597/7/146597.pdf) and combining ranges, coroutines and asyncrony to build a C++ based reactive programming model.  
Despite the technical glitches - a fantastic talk!

### Developing C++ @ Facebook scale

#### Mark Isaacson
In his second talk, Mark discussed how Facebook engineering helps its developers identify bugs, fix failing tests, avoid integration test noise, reduce false positives and promote informative and relevant reporting using clever filtering and smart UI/UX design. I can only hope that Facebook will one day open these tools and make them available to other teams that do not have the resources to build such a system themselves.  
Mark ended the talk by conducting the audience into an awesome Thunderstorm Orchestra  (you'll have to watch the video to understand what this means).  


### Mix Tests and Production Code with Doctest

#### Viktor Kirilov
*-- "Implementing and using the fastest modern C++ testing framework"*  

I discovered [doctest](https://github.com/onqtam/doctest) a few years ago when looking for a single-header, lightweight, unit-testing library for a small project I was working on. I loved the [informative GitHub readme](https://github.com/onqtam/doctest), the simple usage, the extreme portability and of course the performance. It is still one of my go-to libraries when starting a new project.      
In his talk, Viktor Kirilov, the author of doctest, presented the library, its rich feature set, performance comparisons and a deep-dive into some of its internals. Viktor blasted through quite a lot of material though some slides were a bit too dense for comfort. Viktor also discussed some of the great lengths he goes to, to minimize compilation speeds and not generate any warnings on any compilers and sanitizers even at the highest warning levels.  
If you are interested in what goes into writing a super-strength, widely appealing library or some of the magic behind modern C++ unit testing libraries like doctest or Catch, you should watch this talk. Regardless, I recommend you consider [doctest](https://github.com/onqtam/doctest) the next time you are looking for a unit-testing framework.    

### Intro to Rust

#### Alex Crichton

Alex Crichton is a member of the Rust Core Team and has been working on the Rust programming language for 5 years. He is actually employed by Mozilla to work on Rust and is an excellent speaker. For a C++ programmer, Rust is a very interesting language. It attempts to fix and avoid many (perceived-or-not) shortcomings of C++ (at least in certain areas). I've been meaning to pick up Rust and this was a good opportunity to hear an introduction from one of the core developers. As expected, the talk centered around Rust's model for mutability, "borrowing" and sharing and fine-grained object lifetime control. I enjoyed the fact that Alex used C++ terms like "moving" to explain the Rust model. This made things much clearer in my opinion.  
The talk is a good introduction to Rust and definitely worth watching. 

### Evening

After dinner at [Cafe-Konspira](http://konspira.org/cafe-konspira/), an unconvincing retro restaurant, Denis Bakhvalov and I joined the conference after-party at [Mleczarnia](http://mleczarnia.wroclaw.pl/wroclaw). The music was not too loud to converse and the beer was free as in beer. This was a wonderful and informal opportunity to meet many of the participants and speakers. Set underground in what appears as a dungeon or rennovated bunker, there was no cellphone reception which seemed encourage even more personal, face-to-face interaction. 

Spirits were high when Odin Holmes and Rafał Motriuk on the guitar were joined by the band of Dutch students in a special reification of the Smokie song "Living Next Door to Alice" dedicated to Eric Niebler.  
<p align="center">
<img src="../../assets/code_dive_2017/who_the_f_is_eric.gif"/>
</p>


## Day 3
I had a spare day to spend in Wrocław before my flight back so after a bit of shopping in the morning, I joined the [free Old Town Wrocław walking tour](https://freewalkingtour.com/wroclaw/tours/free/old-town-wroclaw/). The guide was great: he was funny and knowledgeable and spoke excellent English. Eric Niebler joined  half-way through the tour and so we spent the rest of the afternoon sightseeing and trying various local Polish beers.      

<p align="center">
<img src="../../assets/code_dive_2017/wroclaw.jpg"/>
</p>

## Summary

**I had a *fantastic* time at code::dive.** 
Sure, you can watch talks online. But as I said at the beginning, meeting the folks in person is *itself* worth the trip. Everyone I met was friendly and welcoming. I came to the conference without knowing *anyone* there. This might seem intimidating at first, but on the other hand, I avoided the natural tendency to clump together with the people I know. I had tweeted that I'll be at the conference and joined the dedicated Slack channel to encourage people to come forward and say hello. This is how I met Denis Bakhvalov and also Miodrag Milanović whom I've only known via Twitter.  

Although the conference is not explicitly a C++ conference, there was always at least one session focused on C++. I truly hope that this remains in future conferences as well.  
Another plus for code::dive is that November appears to be a slow touristic month in Europe and that means airline tickets are extremely cheap. Another bad excuse not to come next year.    

If I have one regret it is that I did not know of and missed the proposal submission deadline and did not send a talk proposal to actually speak at the conference. Though maybe it's best to attend without speaking on one's first international C++ conference. There is always *next year!*



*I have made this longer than usual because I have not had time to make it shorter.  
-- Blaise Pascal*