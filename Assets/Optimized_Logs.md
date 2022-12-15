# Particle Optimization Logs for *CSC-461*
---
## Original
``` c
LoopTime: update: 37.563000 ms  draw: 158.891388 ms  tot: 196.454391
LoopTime: update: 36.613503 ms  draw: 166.713913 ms  tot: 203.327423
LoopTime: update: 37.656200 ms  draw: 165.157410 ms  tot: 202.813614
```

---
## Changed doubles to float.
ChangeList : `94556`

Date: Monday Early Morning, 3:40 AM, March 15, 2021. Weather: Warm inside.

Comment: Update is way faster than I expected and I missed some float values in.

``` c
LoopTime: update:26.191599 ms  draw:162.610504 ms  tot:188.802094
LoopTime: update:27.647900 ms  draw:171.844101 ms  tot:199.491989
LoopTime: update:26.639101 ms  draw:157.839401 ms  tot:184.478500
```

---
## Loops optimization
Removed unnecessary values set in Matrix.cpp

ChangeList : `94575`

Date: Monday Morning, 8:17 AM , March 15, 2021. Weather: Nice.

Comment: Object pooling was also in but wasn't for STL, it was still deleting.
```c
LoopTime: update:27.143801 ms  draw:174.166901 ms  tot:201.310715
LoopTime: update:28.348402 ms  draw:166.265198 ms  tot:194.613602
LoopTime: update:28.176800 ms  draw:159.682907 ms  tot:187.859695
```
---
## Removed STL List
ChangeList : `94577`

Date: Monday Morning, 9:20 AM , March 15, 2021. Weather: Sunny.

Comment: Some Improvement. Also implemented object pooling. with bugs.
```c
LoopTime: update:14.251000 ms  draw:159.802200 ms  tot:174.053192
LoopTime: update:15.534900 ms  draw:150.666000 ms  tot:166.200912
LoopTime: update:13.652801 ms  draw:146.718506 ms  tot:160.371307
```
---
## Added SIMD and removed switch cases in get/set function
used memset and memcpy for copy Constructor and constructor in matrix.
TODO: make these functions work on matrixes.

ChangeList : `94674`

Date: Monday Evening, 4 PM , March 15, 2021. Weather: Great

Comment: Had some hope from SIMD. not much improvements.
```c
LoopTime: update:15.284201 ms  draw:148.086807 ms  tot:163.371002
LoopTime: update:15.316400 ms  draw:147.350510 ms  tot:162.666901
LoopTime: update:14.827000 ms  draw:147.245789 ms  tot:162.072800
```
---
## Added Big 4s
ChangeList : `94683`

Date: Monday Evening, 5:10 PM , March 15, 2021. Weather: Still Great.

Comment: Respect for Big 4s. 4s is strong with this one indeed.

```c
LoopTime: update:9.381101 ms  draw:149.947296 ms  tot:159.328400
LoopTime: update:10.228200 ms  draw:138.032501 ms  tot:148.260696
LoopTime: update:9.417900 ms  draw:139.260101 ms  tot:148.678009
```
---
## Replaced Vects to Martix
Added SIMD for Cross product in vector

ChangeList : `94744`

Date: Monday Evening, 8:09 PM , March 15, 2021. Weather: Still Great.

Comment: SIMD please don't disappoint.
```c
LoopTime: update:10.710200 ms  draw:141.377808 ms  tot:152.087997
LoopTime: update:10.766700 ms  draw:139.538010 ms  tot:150.304703
LoopTime: update:10.841599 ms  draw:140.770813 ms  tot:151.612396
```
---
## Went on a goose chase.
ChangeList : `94887`

Date: Monday(Tuesday) Late Night, 3:15 AM , March 16, 2021. Weather: Dark.

Comment: Followed stack and it overflowed my time out of window.

---
## Added consts and pass by reference
ChangeList : `95087`

Date: Tuesday Evening, 7:10 PM , March 16, 2021. Weather: Didn't care to check.

Comment: some SIMD some RVO and stable state for revert back. 
```c
LoopTime: update:11.089300 ms  draw:138.996902 ms  tot:150.086197
LoopTime: update:11.263300 ms  draw:137.433792 ms  tot:148.697098
LoopTime: update:10.725599 ms  draw:138.866104 ms  tot:149.591705
```
---
## Hot n Cold

ChangeList : `95097`

Date: Tuesday Evening, 7:44 PM , March 16, 2021. Weather: Its been snowing!

Comment: At least I attempted to.
```c
LoopTime: update:6.811000 ms  draw:149.935394 ms  tot:156.746384
LoopTime: update:7.350900 ms  draw:145.357300 ms  tot:152.708206
LoopTime: update:9.499600 ms  draw:146.834198 ms  tot:156.333786
```
---
## Cursed Keenan’s Name out loud 
ChangeList : `N/A`

Date: N/A Weather: N/A.

Comment: 
    Rant time:

Tried working on stack with Hot n cold for a day
Had some issue so reverted back.
Realized my particles render upside down and reverted back a lot to find that problem was in Object pooling. wasn't reseting values after getting from pool.
Phew, took so much time.

---
## Did matrix multiplication by hand.
ChangeList : `95216`

Date: Tuesday after MidNight, 1:40 PM , March 17, 2021. Weather: Omnious.

Comment: Did not help much since my draw is bottle necked by GPU.
```c
//Lost times.
```
---
## Improved on multiplication more!
also removed extra code from Draw along with all the temp Matrices

ChangeList : `95472`

Date: Wednesday Evening, 7:49 PM , March 17, 2021. Weather: Raining.

Comment: was hoping some serious improvements from matrix multiplication. but bottleneck is OpenGL
```c
// Lost times..
```
---
## Added continuous data buffer for Particles as well
ChangeList : `95472`

Date: Wednesday Evening, 8:25 PM , March 17, 2021. Weather: Raining.

Comment: The times are with above 2 changes as well. Slight improvement in draw.
A lot of improvement in Update because of removing matrices and some useless calculations.
```c
LoopTime: update:4.551800 ms  draw:143.522400 ms  tot:148.074188
LoopTime: update:4.806700 ms  draw:146.547791 ms  tot:151.354492
LoopTime: update:4.562400 ms  draw:147.543808 ms  tot:152.106201
```
---
## SIMD for vector add and subtract

ChangeList : `95562`

Date: Wednesday Evening, 10:20 PM , March 17, 2021. Weather: Raining.

Comment: Simd proving itself useful!
```c
LoopTime: update:2.331000 ms  draw:141.902100 ms  tot:144.233109
LoopTime: update:2.650800 ms  draw:138.212601 ms  tot:140.863403
LoopTime: update:2.481600 ms  draw:138.595505 ms  tot:141.077103
```
---
## Did Compiler optimization.
ChangeList : `95562`

Date: Wednesday Evening, 10:20 PM , March 17, 2021. Weather: walk worthy.

Comment: also made drawBuffer iterate by pointer instead of [i] but doesn't seem to make much difference since I'm still using for loop anyways.
```c
LoopTime: update:1.837600 ms  draw:132.802399 ms  tot:134.639999
LoopTime: update:2.129000 ms  draw:130.606903 ms  tot:132.735901
LoopTime: update:2.009700 ms  draw:130.827698 ms  tot:132.837402
```
---
## Added *= += and -= to Vector.
ChangeList : `95570`

Date: Wednesday Evening, 10:40 PM , March 17, 2021. Weather: Hungry.

Comment: 
```c
LoopTime: update:1.777300 ms  draw:136.258301 ms  tot:138.035599
LoopTime: update:1.922200 ms  draw:135.551605 ms  tot:137.473801
LoopTime: update:1.922100 ms  draw:135.580200 ms  tot:137.502304
```

---
## Optimized Exectue a little
ChangeList : `95612`

Date: Wednesday Evening, 12:14 AM , March 18, 2021. 

Weather: Cloudy and no chance of meat balls.

Comment: No difference but removing code can't make it worse. and code has memory leaks. fixed later.
```c
LoopTime: update:1.939100 ms  draw:130.741211 ms  tot:132.680313
LoopTime: update:1.837400 ms  draw:140.853104 ms  tot:142.690506
LoopTime: update:1.875500 ms  draw:147.910110 ms  tot:149.785614
```
---
---
---