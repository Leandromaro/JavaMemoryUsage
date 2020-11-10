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

## Reference Types
If you look closely at the Memory Structure picture, you will probably notice that the arrows representing the references to the objects from the heap are actually of different types. That is because, in the Java programming language, we have different types of references: strong, weak, soft, and phantom references. The difference between the types of references is that the objects on the heap they refer to are eligible for garbage collecting under the different criteria. Let’s have a closer look at each of them.

1. Strong Reference
These are the most popular reference types that we all are used to. In the example above with the StringBuilder, we actually hold a strong reference to an object from the heap. The object on the heap it is not garbage collected while there is a strong reference pointing to it, or if it is strongly reachable through a chain of strong references.

2. Weak Reference
In simple terms, a weak reference to an object from the heap is most likely to not survive after the next garbage collection process. A weak reference is created as follows:

```
WeakReference<StringBuilder> reference = new WeakReference<>(new StringBuilder());
```

A nice use case for weak references are caching scenarios. Imagine that you retrieve some data, and you want it to be stored in memory as well — the same data could be requested again. On the other hand, you are not sure when, or if, this data will be requested again. So you can keep a weak reference to it, and in case the garbage collector runs, it could be that it destroys your object on the heap. Therefore, after a while, if you want to retrieve the object you refer to, you might suddenly get back a null value. A nice implementation for caching scenarios is the collection WeakHashMap<K,V>. If we open the WeakHashMap class in the Java API, we see that its entries actually extend the WeakReference class and uses its ref field as the map’s key:

```
/**
 * The entries in this hash table extend WeakReference, using its main ref
 * field as the key.
*/
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>; {

    V value;
```

Once a key from the WeakHashMap is garbage collected, the entire entry is removed from the map.

3. Soft Reference
These types of references are used for more memory-sensitive scenarios, since those are going to be garbage collected only when your application is running low on memory. Therefore, as long as there is no critical need to free up some space, the garbage collector will not touch softly reachable objects. Java guarantees that all soft referenced objects are cleaned up before it throws an OutOfMemoryError. The Javadocs state, “all soft references to softly-reachable objects are guaranteed to have been cleared before the virtual machine throws an OutOfMemoryError.”

Similar to weak references, a soft reference is created as follows:

```
SoftReferenc<StringBuilder> reference = new SoftReference<>(new StringBuilder());
```

4. Phantom Reference
Used to schedule post-mortem cleanup actions, since we know for sure that objects are no longer alive. Used only with a reference queue, since the .get() method of such references will always return null. These types of references are considered preferable to finalizers.

How Strings Are Referenced
The String type in Java is a bit differently treated. Strings are immutable, meaning that each time you do something with a string, another object is actually created on the heap. For strings, Java manages a string pool in memory. This means that Java stores and reuse strings whenever possible. This is mostly true for string literals. For example:

```
String localPrefix = "297"; //1

String prefix = "297";      //2

if (prefix == localPrefix){

    System.out.println("Strings are equal" );

}else{

    System.out.println("Strings are different");

}
```

When running, this prints out the following:

__Strings are equal__

Therefore, it turns out that after comparing the two references of the String type, those actually point to the same objects on the heap. However, this is not valid for Strings that are computed. Let’s assume that we have the following change in line //1 of the above code

```
String localPrefix = new Integer(297).toString(); //1
```

Output:

__Strings are different__

In this case, we actually see that we have two different objects on the heap. If we consider that the computed String will be used quite often, we can force the JVM to add it to the string pool by adding the .intern() method at the end of computed string:

```
String localPrefix = new Integer(297).toString().intern(); //1
```

Adding the above change creates the following output:

__Strings are equal__
