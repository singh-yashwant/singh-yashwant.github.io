---
title: "Loop Optimizations in compilers[Part 1]"
date: 2023-04-02T01:20:52+05:30
draft: false
---

## Introduction ##

Loops are "the" most compute intensive part of any software program making loop optimizations some of the most rewarding optimizations for a compiler. In this post, I'll try to go over some of these optimizations which are covered by LLVM(I'm sure you'll find them in other compilers too). This is not an extensive list you can always find more.

### Loop Invariant Code Motion (Code hoisting)

<p>Let's break down optimization name :)
- Loop Invariant = Constant/unchanging code inside a loop
- Code Motion = Moving that code outside

It's a known coding practice to always keep definitions and constant code outside the loop. What's interesting is even if you don't, the compiler will take care of it. The compiler determines if any instruction is "invariant" and moves it out of the loop. Either inside prehearder(every loop in LLVM has a preheader which has only 1 outgoing edge to the loop header) or loop exit (similarly every loop has an exit block with only 1 incoming edge).</p>

Let us look at an [example](https://godbolt.org/z/8W9qoYs68).

*Before licm*
![img1](/images/licm1.jpg)

*After licm*
![img1](/images/licm2.jpg)

Ignoring we aren't doing anything inside the loop notice how the compiler recognizes that loading and storing %AVal and %BVal has nothing to do inside and hoists them to preheader. This also causes few <em>'phi'</em> instructions to appear in the exit block, what those are is a story for another day.

### Loop deletion 
No points for guessing, this optimization deletes any loop which doesn't have effects on the code, so rest assured all those random loops you write aren't gonna affect your code's performance. 

In the below [example](https://godbolt.org/z/zfdPYxd4s)
'for.loop' represents a loop block, although it's not much of a loop since the exit condition is a constant('false') nonetheless the compiler sees it as a loop and would have behaved similarly for a proper loop if it does not affect rest of the code. This loop gets deleted once we explicitly ask the compiler to perform loop deletion.

*Before loop deletion*

![img1](/images/ld1.jpg)

*After loop deletion*
![img1](/images/ld2.jpg)

### Loop unswitching 
This optimization transforms loops containing branches on loop invariant(independent) conditions to multiple loops by moving the branch outside and a copy of a 
loop in both branches. This might sound like suboptimizing but it isn't. 

```
for (...):
    A
    if (loop invariant cond):
        B
    else:
        C
```

turns into, 

```
if (loop invariant cond):
    for (...):
        A, B
else:
    for (...):
        A, C
```

The gain here is only checking the condition once instead of in every iteration. 

Let us take a look at the compiler in [action](-passes='loop(simple-loop-unswitch),verify<loops>').

*Before loop unswitch*
![img1](/images/lu1.jpg)

'continue' block is the invariant condition guarded code inside the loop. 

*After loop unswitch*
![img1](/images/lu2.jpg)

The conditional instruction is moved to the entry block and the loop now is guarded by it not the other way around. 


---
This is it for this post. I intend to cover more of these optimizations in the future. Thanks for reading!

---

{{< subscribe-convertkit >}}