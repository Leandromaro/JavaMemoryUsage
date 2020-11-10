# Java Memory Usage
A bunch of annotations about memory usage

## Intro
You might think that if you are programming in Java, what do you need to know about how memory works? Java has automatic memory management, a nice and quiet garbage collector that works in the background to clean up the unused objects and free up some memory.

Therefore, you as a Java programmer do not need to bother yourself with problems like destroying objects, as they are not used anymore. However, even if this process is automatic in Java, it does not guarantee anything. By not knowing how the garbage collector and Java memory is designed, you could have objects that are not eligible for garbage collecting, even if you are no longer using them.

So knowing how memory actually works in Java is important, as it gives you the advantage of writing high-performance and optimized applications that will never ever crash with an OutOfMemoryError. On the other hand, when you find yourself in a bad situation, you will be able  to quickly find the memory leak.

![memory](https://dzone.com/storage/temp/7590038-javamemory12.jpg)

## The Stack
Stack memory is responsible for holding references to heap objects and for storing value types (also known in Java as primitive types), which hold the value itself rather than a reference to an object from the heap.

In addition, variables on the stack have a certain visibility, also called scope. Only objects from the active scope are used. For example, assuming that we do not have any global scope variables (fields), and only local variables, if the compiler executes a method’s body, it can access only objects from the stack that are within the method’s body. It cannot access other local variables, as those are out of scope. Once the method completes and returns, the top of the stack pops out, and the active scope changes.

Maybe you noticed that in the picture above, there are multiple stack memories displayed. This due to the fact that the stack memory in Java is allocated per Thread. Therefore, each time a Thread is created and started, it has its own stack memory — and cannot access another thread’s stack memory.

## The Heap
This part of memory stores the actual object in memory. Those are referenced by the variables from the stack. For example, let’s analyze what happens in the following line of code:

```
StringBuilder builder = new StringBuilder();
```

The new keyword is responsible for ensuring that there is enough free space on heap, creating an object of the StringBuilder type in memory and referring to it via the “builder” reference, which goes on the stack.

There exists only one heap memory for each running JVM process. Therefore, this is a shared part of memory regardless of how many threads are running. Actually, the heap structure is a bit different than it is shown in the picture above. The heap itself is divided into a few parts, which facilitates the process of garbage collection.

The maximum stack and the heap sizes are not predefined — this depends on the running machine. However, later in this article, we will look into some JVM configurations that will allow us to specify their size explicitly for a running application.
