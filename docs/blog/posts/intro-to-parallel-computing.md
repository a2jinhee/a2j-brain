---
date: 2024-12-31
categories:
  - parallel-computing
tags:
  - notes
---

# Intro. Why parallel computing? 

<!-- more -->

Before we get our feet wet with parallel computing, I would like to preface with a simple question- *why parallel computing*? The answer is fairly simple: transistor scaling is plateau-ing. The following excerpt from New York Times (2004!) sheds more light on this pivotal moment:

> ***Intel's Big Shift After Hitting Technical Wall:***
> 
> ...Then two weeks ago, Intel, the world's largest chip maker, publicly acknowledged that it had hit a "thermal wall" on its microprocessor line. As a result, the company is changing its product strategy and disbanding one of its most advanced design groups. Intel also said that it would abandon two advanced chip development projects, code- named Tejas and Jayhawk. Now, Intel is embarked on a course already adopted by some of its major rivals: **obtaining more computing power by stamping multiple processors on a single chip rather than straining to increase the speed of a single processor.**...
> 
> *- John Markoff, New York Times, May 17, 2004*

The ever-famous **Moore's Law** set a standard when reducing CMOS transistors size, pushing fabrication factories to halve the size of its chip every year. Smaller and faster transistors led to higher clock rate, lower power consumption, and economical use of chip area. Advanced compilers and vendor-independent OS (e.g., Linux) also lowered the cost in bringing out new architectures. With single-threaded CPU performance doubling every 18 months, working to parallelize program code was often not worth the time.

However, the glorious era of CPUs come to an end when limitations surface from the two driving forces of CPU performance improvement.

  - Diminishing gains with instruction-level parallelism (ILP): It is difficult to increase issue width (i.e., the number of instructions a processor can execute simultaneously), and benefits gleaned from this declines. 
  - Power wall: It is difficult to push SOTA clock frequency due to high temperature (Frequency $\propto$ Power consumption $\propto$ Temperature).

Thus began the era of *parallel computing*. 