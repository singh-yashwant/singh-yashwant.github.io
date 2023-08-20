---
title: "Low Latency Cpp"
date: 2023-08-20T22:54:07+05:30
draft: true
---

Let's have a look today at what low latency C++ means and how is it actually written. The content of this post are basically my interpretation/summary of the 2 part talk given by Timur Doumler which you can watch [here] and [here]. I highly recommend watching those if you are interested in low latecny c++.

First what is latency for a C++ program? Latency is we know is just a synonym for delaly for c++ programmers like us that will the time taken for the instrution(s) to be executed by the processor from the time it was loaded in the processor. So a low latency program will mean a code that takes less time to perform a certain task. But, how is it important?

Any program/software can be programmed to be more optimized either for low latency or high throuhput. That does not mean only 1 of the 2 can be done rather it's more optimal to design a program with one goal in mind either skewed towards low latency or high throughput. Latency and throughput can be seen as the length and cross section of a pipe respectively hence the efficiency of the program can be liquid flowing through that pipe.

So who in the industry care about low latency? We can map certain industries on a line graph whose extremeties denote optimizing for latency and optimzing for througput. Let's have a look where certain industries will end up on the line graph.

[[img here]]

- HFT : They need to be spot on with as low latency as possbile to carry out that trade faster than anyone else.
- Audio processing: Audio industry also need very low latency program to process the audio signals coming in real time any miss will make the final output glitchy.
- Servers: You'll will find them somewhre in the middle balancing both latency and throughput and neither can be too low.
- Gaming: They also try to mainting a balace but somewhat skewed towards latency as we can always render less stuff rather than outputting less frames per second.
- HPC: They usually don't care about the low level stuff, it dosen't matter if some part of the program is slow or fast what matters is the entire dataset should be consumed to provide the desidered results as fast as possible.

Why do HFT and Audio industry carry care about latency so much? For HFTs they might not have inputs coming for them in regular intervals unlike audio signals but whenever they do they have to act in nanoseconds or they might end up loosing money. And for audio as already mentioned they don't need to be as fast as HFTs but fast enough to process all the signals in real time and not ending up adding discontinunity in the output signals. For gaming industry disregard of latency will end up making their game looking glitchy. 

It looks like even the low latency industry can be subdiveded into 
- Ultra low latency: Do everything as fast as possible, there is no metric to beat (eg HFTs)
- Real time: Do everything fast enough to always beat the pre defined metric (eg gaming, audio/video processing)

**Hot path** So every program has that code path that is most critical to the application performance. For HFTs it might the algorithims that are executed when they recive certain market feedbacks, etc. We might end up talking in terms of hot paths later in the article.

Why is C++ the dominant in the low latency world?
Well we can't have languages with automatic garbage collectors slowing us down, C doesn't have all these fancy features that are needed for abstraction and scaling (templates, algorithms, etc) and lack of frameworks such as unreal engine rules out Rust.

## C++ techiniques for low latency programming
This can be subdivide in 2 parts
- Efficient programming
- Programming for deterministic execution time

### Efficient programming
For progamming a softare efficiently we need 2 kinds of information

**Info about our Softare:**
How does the compiler works?
What kind of optimizations take place under the hood?
Details about the programming language?
Details about the efficiency of standard libraries supported by the langugage? Etc.

We can have a look at some C++ code snippets to have an idea about what we have got here

[[assume]]
Example and code

[[restrict]]
Example and code

[[unsequenced]]
Example and code

Timur covered lot more techiniques in his talk and the some features that will end up seeing in future c++ standards (c++23 and 26).
  
**Info about our hardware:**
Architeure of the CPU/GPU being prgrammed.
Cache hierchy and lookup details.
Support for SIMD/ SIMR / vector programming.
  
SIMR is basically SIMD on registers, instead of memory you have registers aligned together and a single instruction executes on them together.

Things that usually end up slowing us down
- Branch hazards
- Data hazards
- Hardware hazards
