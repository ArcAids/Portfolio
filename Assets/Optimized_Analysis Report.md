# Particles Optimization Analysis: 


The particle project being very inefficient with the code, could use several tweaks to drastically improve the performance.  

For now the current state timings of the system on my PC are as following:
```c
LoopTime: update: 37.563000 ms  draw: 158.891388 ms  tot: 196.454391
LoopTime: update: 36.613503 ms  draw: 166.713913 ms  tot: 203.327423
LoopTime: update: 37.656200 ms  draw: 165.157410 ms  tot: 202.813614
```
Average Update: `37.2775676667 ms`
Approx : `37 ms`
Approx Draw: `163 ms`
Total = `200 ms`
Here are some major changes and improvements and my estimates on the performance boost they might provide:

1. Changing `doubles` to `float`
1. Implementing Object Pooling.
1. SIMD Matrix and Vector maths
1. Custom Big 4s
1. sentinals out of the loop.
1. Data alignment
1. Data Structure fix.
1. Hot n Cold
1. pass by reference and RVO
1. Removing useless Code.
1. getting rid of conditionals/switches

## 1. `double` to `float`.
According to my google (re)search floats are around 30% faster then double. Although not as precise but since we are not calculating size of quantum particles with minimum error, we are safe to use floats.
Although they are 30% faster, they would still not give us this big boost since we are not just doing float/double calculations. A lot more is going on.

```c
10% faster. so new times would be around `33.3 ms` : `150 ms
```

---

 ## 2. Object Pooling
 Analyser and basic C++ knowledge both says that most of the time is being spent in allocating memory to all the 100k particles in the beginning and then they gets destroyed and recreated once their life is ended.

 allocating new memory for particles isn't a good idea anyways, since particles are supposed to be in a large numbers.
 with proper pooling and we can avoid redundant new and delete calls during the simulatation. My estimation is it can run around 2 times faster with just object pooling in.

 Update is creating new particles and Draw is just iterating through a list so update should be the only one getting help from this.
 
```c
100% faster. so new times would be around ` 16.6 ms` : `150 ms
```

---

## 3. SIMD Maths
Since The matrix multiplication is being called for every particle multiple times on every frame, it could be faster.
since SIMD MxM ran nearly twice as fast in PA 6, it means it has potential. but we are only doing matrix multiplication at fraction of time so it shouldn't be drastic as twice. lets say

Since we are doing a lot Matrix multiplications in Draw I believe draw will benefit more from it. 

```c
15% faster. so new update time would be around ` 13 ms` : `120 ms
```

---


## 3. Data Alignment
With proper caching we can add more data to cache at once without wasting bytes in unnecessary padding. It will be more helpful for simd and caching though.

```c
3% faster. so new update time would be around `12.5 ms` : `118 ms
```

---

## 4. Sentinels out of loop.
we are looping through all the particles in both draw and update so I believe it will make it run 10-15% faster if we remove all the unnecessory work done inside loops.

Temporary Matrices are being initialized in Draw while I noticed extra loop in update so it will make both draw and update faster.

```c
10-15% faster. so new update time would be around `10.5 ms` : `105 ms
```

---

## Big 4s
Implementing efficient constructors can make it faster since there are so many temporary objects being made inside and outside the loops. having their constructors defaulted can be expensive.

```c
2% faster. new times would be: approx `10 ms` : `102 ms
```

---

## Data structure improvement
We are using a linked list to keep track of all the pointers in memory. although linked lists are efficient, but them being spread out in memory makes them slower than sequential arrays.
with a Queue instead of linkedlist, we could easily handle dead particles since they will be dead in same sequence they were instantiated.

Major change a queue would do is `not having to copy paste data into drawBuffer` but Draw can `render directly from original Particle List` from `start` to `end`.

won't have to check and change pointers around as well to remove or add a node. just change start and end values.
Since we are still doing same amound of work in draw, it shouldn't effect draw much.

```c
30% faster. new times would be: approx `6.66 ms` : `100 ms
```

---

## Hot n Cold
Particle holds a lot of data and it gets iterated through once in Update and Once in Draw.
If we divide the draw and Update list with data required for sepicific iteration, we could improve caching of lists and make it much faster.

Draw needs curr, prev, diff position, rotation, scale and life while Update only needs life, prev, next most of the time.

this way draw buffer would be smaller and easier to cache for Draw iteration.
Update can iterate through particles checking only their next prev and life.

```c
30% faster. new times would be: approx `4.66 ms` : `70 ms
```

---

*Other smaller changes could even improve somewhere between 10-15%*

```c
12% faster. new times would be: approx `4 ms` : `60 ms
```



---
---

# What profiler can tell:

**Instrumental profiler:**

What we learn from profiler is that 60% of time spent is in Draw calls and only 20% is Update. Most speed gain would come from optimizing Draw functions.

Matrix multiplications optimization and removing temporary variables being called in loop should help it a lot.
But most time spent in draw is `delete` which can be easily avoided with `object pooling`. 

Same for update 10% calls out of 20% are just new calls which won't be there after object pooling.

Rand is taking a lot of calls too but we can't change that.

**CPU profiler.**

it doesn't work on my laptop. couldn't use it sadly.

---
---
# Conclusion
It should take around 2-4 days to implement all the changes to get our total times from around `200 ms` to `64 ms` That is more than 2 times faster. I have no Idea how I'm gonna make it 4 times faster. my calculations says its going to be hard.

---
---
---