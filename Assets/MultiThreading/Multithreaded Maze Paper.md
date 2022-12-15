# Multithreaded Maze Solver

## Description:
Optimizing a path finding algorithm to improve traversing speed using Multithreading alone. I reworked the given Single-threaded DFS algorithm to find the solution in 2 parts and then stitch the solutions to find the path from start to end. I avoided making any single thread optimizations so only speed gains would be done by multithreading.

## Multithreaded Solution
While single threaded only look for the destination, multithreading solution divides the work in 2 parts by starting looking the other end from both start and destination. Making it ideally twice as fast. 

Both threads, TopDownSolver and BottomUpSolver are separate classes using slightly altered code. I start an Async task to launch thread for each of them and wait for them to return their part of the solution.

``` c++
auto result=std::async(std::launch::async,
        &(Student_TopDown_STMazeSolverDFS::Solve), &mtStudentSolver);
auto result2 = std::async(std::launch::async,
        &(Student_BottomUp_STMazeSolverDFS::Solve), &mtStudentSolverDown);
```

`MainThread` waits for the results from both the threads in a future and we combine them from there.

![][red]

[red]: Maze%20Solver.png

 ### DFS Thread
picking DFS(Depth First Search) was just an obvious choice since it is much faster than (BFS)Breadth First Search. I avoided making any changes to algorithm since that wouldn't be right for the task. 

After finding the solution, the algorithm needs to recreate the path before passing it back to main thread which is taxing since it adds a O(N) complexity where N is number of steps in solution.
Since this happens in single-threaded solution as well, divinding it into 2 threads helps recreating path efficiently.

Unlike Single-threaded algorithm needs to do some extra work to communicate between both threads. Both threads share a reference to atomic_bool `found` flag and an `atomic<Position>` variable to communicate with each other in case one of them find the solution by colliding with other thread's path.

``` c++
struct CollisionSharedData {
	std::atomic<Position> pos;
	std::atomic_bool found=false;
}; 
```

### Finding Collisions.
Each cell in the maze is an atomic int with most bits empty and ready to be used. 2 of those bits are being used to mark the cell as traversed by a thread or not. 
``` c++
enum Bit {
	APAINT = 0x8,   //A been here
	BPAINT = 0x4,   //B been here
	ABPAINT = 0xC   // not used
};
```
Every step the solver take, it sets the cell Bit to 1, leaving it's mark here in case other thread reaches the same cell. If the thread hits a deadend, it traverses back but it leaves it's footprint(bit mark) there, I avoid taking off paint since there is no way other thread will get to that position without hitting a collision at optimal path first.

In original DFS Algorithm when solver reaches its destination it throws an exception to start building the solution path. Multi-threaded solution does the same thing except it only builds the solution till the collision point.

Collision point is found when either of the thread reaches a cell which other thread has traversed before. Each cell is checked against other thread's bitmask to detect path overlap(collision).

![][ref]

[ref]: MazeCollision.png

When collision occurs, one thread might be way ahead looking into wrong paths and deadends, as shown in diagram above, red thread which is TopDown, is still going while Green(BottomUp) has found the collision, thus solution, so it sets the `CollisionSharedData::found` flag along with the collision position.

threads when receiving the found trigger, throws an exception right away along with collision position. they build the solution until the collision they either detect or receive from other thread, before returning it to Main thread.

 ### Bottom Up vs Top Down
Bottom up also works in the same way with some tiny changes for logic and optimization.
other than starting from end and ending at start of the maze, BottomUp also needs to be biased to go North before south. so adding some additional functions that were North biased did the trick.

moreover, finding the solution at the end adds a couple more passes to reverse it, but it is easily avoidable, since I flip the direction before adding it to the result and instead of reversing the whole BottomUp solution, I iterate over BottomUp solution in reverse to concatinate it to TopDown solution.still it adds a O(N) where N is number of steps in BottomUp result.

 ### Combining results

 Once both threads find the collision and creates their half of the solution, they return the pointer to a vector of solution that is just combined together to find the final path.

```
for (auto it = end(*pSolutionBottomUp); it != begin(*pSolutionBottomUp); )
{
    it--;
    pSolutionTopDown->push_back(*it);
}
```

 ## Data movement.

![][red]

No Data is ever moved around, only references. but flow is as shown in 1st diagram. 
once both threads start, they traverse the maze until they either find the collision themselves or other thread passes them the solution.
Orange thread here detects and passes the collision to Blue. then both threads start working to create their solution to reach the collision point and returns the result back to main.

Main concatinates both results and that's the solution.

 ## Challenges
**Commuinication:** I tried making it work with ThreadBase, with KillerThread and ThreadProxy first but realized really soon that would be too slow. I then tried to communicate between threads with Conditional variable which was slugish, changing things to atomic polling flag made it way faster in no time.
 
 **Base Collision:** if the maze is too small or collision happens right at the start or end, there would be 1 extra step in solution which is copy of the collision(start or end). to tackle this, I added a bit paint on the start point of the thread(either start or end of the maze) at class's constuctor, hence if 1 thread reaches the end, it will collide and exits as if it hit a collision. avoiding doing any extra checks.

 ## Improvements
 - leftout single threaded optimizations: Changing the biases and changing the algorithm could make it faster but wouldn't be fair, so I left it as similar to single threaded solution as I could.

 - More threads: I wanted to add some painting threads to block all the deadends, but got busy with research instead.
 
 - other over engineered way would be starting up a pool of threads and use one of them to check in each direction whenever you reach a choice, for each available path.