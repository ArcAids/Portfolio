# **Space Invaders Design documentation** 
## *SE-456 Architecture of Real-Time Systems*
---
---

[![Submission video](https://img.youtube.com/vi/yVpaRq7ckIg/0.jpg)](https://www.youtube.com/watch?v=yVpaRq7ckIg)


## `Overview`
Since the Space invaders is a feature rich already, not to count in all the engine features needed to be added to make it from the ground up such as Lists, iterators, Adapters and so on, If not made with proper planning of architecture things can become a mess in no time.

To make it work properly with minimum bugs, maintainable code, and meet up all the deadlines, the project must be architected accordingly.

## A. Game Scenes

Although playable part is basically the *game* but Its not complete without a Main menu to let player choose Game modes and maybe check some tutorials. There is also end screen to reward the player once he is done playing. The engine code on the other hand is common for all the scenes and we need to abstract those functions avaiable to all the different scenes with their unique functionalities. It is easily achievable by using **_State pattern_**.

![](https://github.com/ArcAids/SPACE_UML/blob/main/SceneStates.png?raw=true)

### 1. **State Pattern**
Whenever an object needs different implementations at different times or under different conditions, it can hold onto an instance of abstracted base class which can be swapped with instances of that base class, implementing different functionalities as required.
The states can be handles easily with common abstracted functions and they can switch to next state easily.

With different states of the game engine functions *(Render and Update etc.)* needed and only one running at a time, It only makes sense if implementations were separated in different places for cleaner and decoupled code.

![](https://github.com/ArcAids/SPACE_UML/blob/main/StateMachine.png?raw=true)

its easy to abstract their common functionalities in one abstract base class `SceneState`. While another class(`SceneContext`) can hold onto instance of a state which could be swapped with other instances of the `SceneState`. 
`SceneContext` is being used to manage the whole state machine by switching states, getting current state and initializing all the states in beginning.
It is easy to add more functions to each state with their custom implementation without worrying about other states because one must only need to access abstracted functions: `Draw()`, `Handle()`, `Update()` and `Transition()`, through `SceneContext` *Singleton* class.

*Some of the functions are virtual with some common functionality abstracted up the heirarchy*

State machines usually also have this `Handle()` function which has logic to decide what State to enter after this one according to current variables or maybe just manual state. for example, `CreditScene` will always transition to `MainMenuScene` while GameScene can either switch to `GameScene2`(`poScenePlay2` which is another instance of the same game state for player 2) or `CreditsScene` in case game ends. we can also add in helper functions to be called on each state at certain points such as `Transition` or `*OnLeave*`

---

But who runs all these states?

## B. Game Class
`SceneContext` class has multiple SceneStates and it manages switching between them and initializing but who initialized the big State machines of the game? Yes, thats Game Class. which is managed by abstracted engine code but that is not important.

Game Class does not only intialize but also calls all the state's `Render()` and `Update()` functions. It can easily get the current state and call the appropriate function without knowing the current state.

But not everything in the game is supposed to be part of one of the big states. There are some things that are shared among all states and needs to be accessed all over. For example, We need text appearing in all the states and the data must be initialized and stored somewhere accessible to all the states and classes.

### 2. **Singleton**
The classes that only require just one instance through out the software can be created to hold their own private instance, which is used to work upon the data of the class while being accessible from anywhere through public static functions. These functions gets the current *and only*(sometimes not the only one) instance from the class using another private static function and work upon the data members.

![](https://github.com/ArcAids/SPACE_UML/blob/main/Singleton.png?raw=true)

Since we have Sprites, Textures, Images and Glyphs needed in almost every scene, we can tackle the problem by making their managers all Singletons. With singleton Image/Texture/Sprite Managers its super easy, barely an inconvenience, to access and alter this data from anywhere and make sure we never have more than 1 instance of the sprites that we will be using.
This all can be done in Game Class once before even initializing the state pattern so they can all work and access the sprites and other data.

### __2.1 Doubletons. *(Bonus)*__
  Not exactly a thing and not limited to just 2 instances, but it is possible for a singleton to have multiple instances by making it have `SetInstance()` function and swap its static instances to a different one. which changes the data on what the singleton class has been currently working on.

  *It seems similar to a State pattern at this point but it keeps the functionality Same and changes the data, which is not state pattern at all.
  This also needs to have public constructor so more instances can be created to swap the active one.*

![](https://github.com/ArcAids/SPACE_UML/blob/main/Doubleton.png?raw=true)

  As double player mode requires another instance of `GameScene` we need all the singleton instances of player 1 data to be saved somewhere before we move to next player, which is being done by caching the singleton instances in `GameScene` for each player and swapping the Singleton's instance to current scene's cached copy before we start working on it.

  Before moving to states, there is another problem to tackle first. 
  #### Data that we are making singletons for.
  Azul Engine provides us with basic tools to render shapes sprites from tga files using rects but the engine is limiting with its data and functionality. Accessing the engine code and working on it sounds promising but that is not easy to do when the engine code is abstracted and readonly. How would this be solved?

---
### **3. Adapter Pattern**
Adapter pattern is adding a layer to a class to add or alter its functionality without having any access to its code. Simply working on a reference to the original object, new classes can be written to provide with new tailored needs. It adds a layor of abstraction to the original adapted class.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Adapter.png?raw=true)

Azul.Sprite, Azul.Texture classes have needed functionality but these can be adapted by new Sprite and Texture classes which not only calls apropriate functions in original object but also are derived from DLink class, which lets them to be added into a doubled linked list.

*These also provides benefit of updating local data any number of times before it is passed down to original object, reducing the overhead to do calculations everytime any change is made. (If required. totally upto us on how we implement it)*

> `Image` class is not adapting any Azul class but my understanding says it is adapting the Texture class by adding Texture and Rect to it so Sprite does not directly needs to be aware of Textures.

There isn't only 1 object of each type we need in a game so holding a list of these is essential and for that, Other patterns are needed.

---
## B.1 Lists.

To have Lists of objects of any type, we need multiple things working, which are `ListBase`, `IteratorBase`, `ListManager` and their Derived classes since these 3 must be abstract.
To make these work, we need these patterns:
1. Template 
1. Iterator,
and additionally,
1. Object Pooling

SLinks and DLinks are being used for the project and these are easy to implement on their own by making managers to handle the lists. Thier job as a list managers is extracted by deriving from a common abstract class called `ListBase` to provide possibilties to extend functionalities for classes accessing the `DLinkManager` and `SLinkManager` by just accessing `ListBase`, which avoids strong coupling.

### **4. Template Pattern**
For a class do some common work on different type of objects, it lets the derived classes implement its abstract functions which are needed for the template class to fulfil its job. Derived classes then can provide with different types and ways to fulfil the same functionality that template class was made for.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Template.png?raw=true)

Lists needs to be managed by a class and there is multiple different types of objects which needed to be added to a list. `DLink` and `SLink` provides objects with data to be added in list; and `DLinkManager` and `SLinkManager` creates a list of said `Nodes` **But these can add any type of objects to the List, making it a hetrogenous List** to avoid that, Template pattern can be used by Creating a Manager(`ManagerBase`) for the Manager(`ListBase`).

`ManagerBase` lets its derived classes to create new Objects which are added to the list, since each derived class is handling single type of object(*deriving from NodeBase*), only 1 type of object is added to that list, making it homogenous. 

> `ManagerClass` is an overkil for most of the lists since some doesn't need double linked list(*FlyWeight*) and some doesn't need Object Pooling(*Flyweight again.*). But its easier to reuse this than writing new class with less features.

`CreateNewNode()` function is implemented in derived but is used in base class, allowing Template to do its job and add whatever(derived from NodeBase) returned by the function to be added to its lists(yup lists.)

---

There is a list of objects now but they must be iterated through so it's easy to find objects and just print them all.

### **5. Iterator Pattern**

Iteratorating through a list isn't always the same, there are different ways to iterate and Iterator pattern makes it easier to not get stuck with only one way to iterate and not to be tightly coupled with list. 

**Iterator provides with some abstract functions that can be used to iterate through the list that iterator is for.**


![](https://github.com/ArcAids/SPACE_UML/blob/main/Iterator.png?raw=true)

Having an iterator each for SLink and DLink is needed to avoid accessing the nodes to get their *pNext* everytime we iterate(which would be tight coupling since it assumes there is a pNext)

There is also `ForwardIterator` for `Composite` classes which iterate through a given composite class and its children as a tree.

See Object Pooling later.

---

Now that there are 4 to 5 states with some pre intitialized and globally accessible lists of data to work on, The game states can use them freely.

## C. Main Menu and Credits Scene
Main Menus of a game usually has the Title and instruction shown on, and There is just that in Space Invaders. Being the first scene It has a lot of texts written all over.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Font%20System.png?raw=true)

The *Font* and *Glyphs* of text also needs to be stored somewhere, in a list perhaps. We could just add them all up in a list like we did for Sprites, Images and Textures but since there are so many Letters and symbols in Alphabet, its possible to add some extra that won't even be used through out the game. 

---

### **6. FlyWeight Pattern**
Flyweight pattern is a storage pattern which is very similar to `Object Pooling`(talked later) but with a big cache. It doesn't remove the objects ever. Flyweight keeps adding the data and saving it in some data structure and is accessible to anywhere it is needed.
Only thing common in this and ObjectPooling is that they both store objects.

Flyweight also has this another feature, that it creates new data as they are needed instead of adding them all in the beginning.


``` c#
public Glyph GetGlyph(int key)
        {
            Glyph g = GlyphManager.Find(glyphBatch,key);
            if (g == null)
                g = GlyphManager.AddGlyphFromXml(glyphBatch, assetName, key);
            return g;
        }
```
Since the Glyphs/Letters in the game are needed throughout without them having to have multiple instances, they can be shared and added to a list in flyweight pattern(only added when needed). If its already added, it gets them from list, otherwise it is added and returned.

> Since this is a great pattern for data sharing it would be great if we used this for Sprite, Texture and Images as well. they are never removed either and are only used in `GameScene`.

> It would also be nice to have a SLink + no Object Pooling ManagerBase for this.

---

## D. Sprite Rendering
Having access to all the Adapted Sprites loaded from Textures file and added to a list, are easy to render by calling their `Render()` Function. Creating multiple Sprites and render them all is easy but that is not very efficient way to do this, since each sprite(with same image) points to same `Azul.Sprite, Image` and `Texture`, making it redundant to have these values saved in every single one of them. That problem can be solved with 

### **7. Proxy Pattern**
Proxy pattern is caching the part of original class that isn't common for a type of object and then only using shared original object which has its value updated from the cached proxy values before being Executed.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Proxy.png?raw=true)

For a Sprite that means, One Sprite can be rendered multiple times at different positions and colors(since we are only using position and color in proxy). 
Each proxy has access to 1 original sprite which it renders after updating original's data to locally stored one.

> I say Proxies are very similar to adapter, but with proxy, a single original instance(Sprite) is shared among multiple Instances of proxy class(SpriteProxy)

Having Proxy classes for Sprites and Sprite Boxes makes the system more efficient and saves memory.

---
Now that rendering multiple Sprites is efficient, Adding and Removing them again and again should also be considered. deleting and creating new objects is heavy, specially in C# where all class instances are in Heap(structs in stack.). To avoid creating and deleting new objects during gameplay, Object pooling is used.

### **8. Object Pooling**
Object Pooling is a pattern that pools an object, instead of deleting it, for it to be reused when a new object of same type is needed.
pool can be another list, dictionary or any data structure which stores deleted objects without deleting them from the memory and provide new objects from this list if there is one available, or it creates a nwe one.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Object%20Pooling.png?raw=true)

`ManagerBase` class has reference to 2 `ListBase`s, 1 of them is for active objects in `active list`, and other one for removed uninitialized ones.
Everytime we remove or add a new object to the list, object pooling is handled by the Base `ManagerBase` Template class which makes sure not to create *new object* calls if there is already something in reserve list.
Reserve list pools a certain number beforehand so it does not have to create new during gameplay unless reserve gets emptied, then it goes on and adds more objects to the `reserve list` for future expansion.

> We don't need object pooling for flyweight and pure Singletons(Sprite, Texture and Images etc that we never delete). but again, its easier to reuse classes.

>DLink is great with this since we remove objects from the list often.
---
## E. GameObjects
Not a Pattern but with all these patterns so far, Its possible to create `GameObjects` for the game. Each GameObject would have a `spriteProxy`, `Collision Box`(Collision sprite and Collision Rect) and can hold even other `GameObjects`in a tree like structure.

Creating multiple `GameObjects` and render them is possible with all these patterns but having to type in same thing over and over for `55 Aliens` and `70+ shield cells` per shield makes the code a mess and really hard to deal with since making 1 single change might mean having to edit 100s of lines of code.
Better way to create new similar objects is

### **9. Factory Pattern**

Factory class is used to create instances of a certain type of objects with common variable data by simply calling a Factory Function.
It helps with doing needed tasks to create specific object it is asked for, without manually having to do them every time we need the object.

![](https://github.com/ArcAids/SPACE_UML/blob/main/Factories.png?raw=true)

For Shields and Aliens where there exists different types of aliens and shield cells(edges, columns and cells), Factory classes is used.
The `ShieldFactory`'s `CreateShieldAssembly()` function takes in what type of(an enum defining the type) and position of the shield part to create. It also adds them to `GameObjectNodeManager` list and activates their sprite and collision box before returning a ready to use object.
`AlienFactory` is also similar.

---
`Bombs` are one of the GameObjects that is dropped by the aliens randomly. It needs to be animated but there are 3 different ways it supposed to be animated. having 3 different implementations for rendering it but rest of the functionality stays the same can be achieved with Strategy Pattern!

### **10. Strategy Pattern**
This pattern is very similar to State Pattern since State pattern is basically specialized Strategy Pattern.
It also is about an instance of abstracted class which have multiple implementations based on derived class of the abstracted instance.
Difference is, it doens't have to be swapable. **the strategy pattern allows different algorithm in different derived classes to be chosen by just passing the instance of needed algorithm to a class**


![](https://github.com/ArcAids/SPACE_UML/blob/main/Strategy.png?raw=true)

Using this pattern, `Bomb` class can use a strategy to implement its render method which alters the gameobject into showing different behavious. each bomb having different instance of `BombAnimationMode`'s derived classes, `BasicBombAnimation`, `ZigZagBombAnimation` and `DaggerBombAnimation`. which all have different implementation of same `Animate` Function.

---

Aliens are needed to be in columns and a Grid, It would be hard to upadte each one individually. It also helps with collision system if they are all in a Hierarchy, to GameObjects have a Hierarchy system, `Composite` pattern is used.

### **11. Composite**
This one is a little bit tricky pattern with multiple parts. In Composite pattern, each abstract class `Component`(called GameObejct in this case), is divided into 2 categories, which Inherits from the `Component` class:
  1. Leaf
  1. Composite


![](https://github.com/ArcAids/SPACE_UML/blob/main/Composite.png?raw=true)

#### **11.1. Leaf**
Leaf is a Components that are stored in ends of the hierarchy and they are usually the objects being managed. They **can NOT hold onto other `Components`**

#### **11.2. Composite**
Composites are also Components but they have a list of `Components` that are considered its children. They create the hierarchy with a Tree Like Structure holding onto other `Composites` or `Leafs`. They **holds onto other `Components`**

with both of these, Composite Pattern creates a Tree Like structure with HetroGenous Objects. Each Composite can hold onto any type deriving from `Component` which is called `GameObject` class for this project.

---
Not All objects are rendered on the screen since there are some `Composites` as well, that means they does not need a `SpriteProxy` which is data member of `GameObject` class. To ensure it works without having to check for null pointers we can add NullObjects.

### **12. NullObject**
NullObjects are simply place holder classes of a certain type that we want to not set to null(in this case SpriteProxy). It implements all the abstract and virtual function and leaves them empty so it avoids doing anything but still lets other classes to consider it a non null value.


![](https://github.com/ArcAids/SPACE_UML/blob/main/NullObject.png?raw=true)

`SpriteProxyNull` Object can be passed to `GameObjects` of `Composite` Type so Composite does not render but still have a sprite which is not *null*.

---

## F. Timer
Game needs a lot events happening after certain time or delay, to create that havinga  Priority Queue works with all the events sorted and then called accordingly.
With them being added in sorted position, avoiding going through the whole list is easy, as only beginning of the Queue needs to be checked every time.

What Event does this Timer calls? It can call a Command pattern event.

### **13. Command Pattern**
Command Pattern is an interface pattern with a contract of having functions as abstract functions in base class and these functions can be called on the derived class. replacing the abstracted class with implemented instances with different code.


![](https://github.com/ArcAids/SPACE_UML/blob/main/Command.png?raw=true)

`Timer` have command attached to its events which triggers its `Execute()` when the event time is reached. Each class deriving from `Command` can have its unique functionality like Printing Text, Dropping Bombs, Changing scene etc.

---

## G. Collisions
To make a complete game `Collisions` are necessary. Its easy to test all the collisions with each other by calculating their boxes and with maths. to make it efficient having collision checked only for Root nodes first and then going down the hierarchy one step at a time is much more efficient since we avoid checking each `bomb` vs each `shield cell` and only checks for each `bomb` against 4 `shield Root`.

![](https://github.com/ArcAids/SPACE_UML/blob/main/Collisiion.png?raw=true)

but keeping track of what happens when two things collide is a challenge.
and having switch or if else is expensive, easier way to tackle this problem is using Visitor Pattern.

### **14. Visitor Pattern**
Visitor patterns is having interactions between classes saved in the classes itself and those interaction functions are called by the other class its interacting with. How?
Each class:
1. must derive from Common base class `Visitor`
1. must implment abstract `Accept(Visitor v)` method from base class.


`Visitor` class contains a list of all the methods of interaction from each class, so 1 function per class and each function has the object reference passed in, so the class interacting can work on the instance.
Accept function is called externally which calls in its abstract method in `Visitor` class on object `v`(passed in as arguement in Accept function).

![](https://github.com/ArcAids/SPACE_UML/blob/main/Visitor.png?raw=true)

For collision system, this can be used by calling Accept function on colliding objects and passing other collided object as argument. Which leads object 1 to call appropriate function that must be overriden by object 2.

For example, Missile can have 
``` c
void VisitBomb(Bomb m)
```
which is called by Bomb's Accept function on missile when they collide each other. this way `Missile` class can implement what to do when bomb collides, without ever need to use switch or if to figure out what function to call.

---
But there are common things that happens on different collisions, like playing a sound, which not only happens when bomb collides with missile but also with shield and top wall.

to extract this functionality to be reused without having to copy paste code around, we use Observer Pattern.

### **15. Observer Pattern**
Observer Pattern, much like command but is different.
This pattern is also like command in implementation since there is a contract with abstract functions, which are overriden by base classes and are called from somewhere else when needed.

It allows us to keep a list of these Observers/Listeners and execute all of them when the event(that they had been listening to) triggers.

![](https://github.com/ArcAids/SPACE_UML/blob/main/Observer.png?raw=true)

It waits for some event to happen, and it usually is stored in a list.

When a collision happened, a list of observers can be stored and triggered for each `colPair` when the collision happens.

---
---
## H. Conclusion

And That is all the `Design Patterns` needed for Making `Space Invaders`. There are more classes that are used to complete the project but these are the major systems needed to artchitect this project.

