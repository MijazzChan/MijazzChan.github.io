---
title: Notes of PECS Mnemonic, Bounded wildcard type - Effective Java reading notes 
author: MijazzChan
date: 2021-06-10 20:37:19 +0800
categories: [Personal_Notes, Effective_Java]
tags: [java, notes]
---

## Intro

PECS stands for Producer extend Consumer super. The PECS mnemonic captures the fundamental principle that guides the use of wildcard types.

Although the way it works may seem a little bit confusing. [An answer from Stack Overflow by Julien](https://stackoverflow.com/questions/4343202/difference-between-super-t-and-extends-t-in-java) perfectly illustrate how that actually affects the way of adding object to `List` and List assigning.

### Bounded Wildcard Type with `super` and `extends`

Suppose you have had following Classes, and their hierarchies => 

```java
class A {}
class B extends A {}
class C extends B {}
```

+ A wildcard of `List<? extends T>` 

```java
|-------------------------|-------------------|---------------------------------|
|         wildcard        |        get        |              assign             |
|-------------------------|-------------------|---------------------------------|
|    List<? extends C>    |    A    B    C    |                       List<C>   |
|-------------------------|-------------------|---------------------------------|
|    List<? extends B>    |    A    B         |             List<B>   List<C>   |
|-------------------------|-------------------|---------------------------------|
|    List<? extends A>    |    A              |   List<A>   List<B>   List<C>   |
|-------------------------|-------------------|---------------------------------|
```

+ A wildcard of `List<? super T>`

```java
|-------------------------|-------------------|-------------------------------------------|
|         wildcard        |        add        |                   assign                  |
|-------------------------|-------------------|-------------------------------------------|
|     List<? super C>     |              C    |  List<Object>  List<A>  List<B>  List<C>  |
|-------------------------|-------------------|-------------------------------------------|
|     List<? super B>     |         B    C    |  List<Object>  List<A>  List<B>           |
|-------------------------|-------------------|-------------------------------------------|
|     List<? super A>     |    A    B    C    |  List<Object>  List<A>                    |
|-------------------------|-------------------|-------------------------------------------|
```

In all of the cases:

- you can always **get** `Object` from a list regardless of the wildcard.
- you can always **add** `null` to a mutable list regardless of the wildcard.

## Producer or Consumer in PECS Mnemonic

Don’t be scared by the table above. Producer and Consumer pattern is often/normally used in function param.

Code example below is mentioned in Effective Java Chapter5-ITEM31. (with minor modifications)

```java
// suppose you have a Stack Class
public class Stack<E> {
    // Constructor
    // other methods...size(), isEmpty()..etc.
    public void push(E e) {
        // push a element
    }
    public void pop(E e) {
        // pop a element
    }
}
```

### Producer

Then you would have a `pushAll()` method which handles a incoming instance of `Iterable` and push it all into the stack. 

```java
public void pushAll(Iterable<E> from) {
    for (E element: from){
        this.push(element);
    }
}
```

However, without wildcard support, `pushAll(Iterable<E> from)` can only support the exact class `E` of `Iterable<>` being passed to the method. Normally you would expect `pushAll(Iterable<E> from)` can handle a `Iterable` containing not only E but also E’s sub-classes. Just like the example provided in Effective Java.

```java
Stack<Number> numberStack = new Stack<>(); // <> is called Diamond operator.
Iterable<Number> numbers = ...;
Iterable<Integer> integers = ...;

// Works, because E => Number is a exact match
numberStack.pushAll(numbers);

// Error, but normally you would expect this to work.
// Because Integer is a sub-class of E=Number.
// Stack<Number> can definitely contain Number's sub-classes.
numberStack.pushAll(integers);
```

But with the help of bounded wildcard, `pushAll` can accept `Iterable` of E and E’s subtype instead of E itself.

> `Subtype` is a little bit mis-leading, since subtype => every type is a subtyope of itself, even though it does not explicitly extend itself. 
>
> But for readability, I will mention X and X’s subtype.

```java
public void pushAll(Iterable<? extends E> from) {
    for (E element: from){
        this.push(element);
    }
}
```

You don’t have to worry about whether `from`'s element can be casted to E, because the bounded wildcard ensures `from`‘s elements are and only are E’s type or E’s subtype.

**In the scope of `Stack<E>`, the `from`object from `pushAll` is considered/regarded as the producer of `E`.** Since it produces E, and its products are being added into the stack.

 ### Consumer

With the `pushAll` method implemented above, you may think of adding a `popAll` method.

Let’s start without that bounded wildcard support.

```java
public void popAll(Collection<E> to) {
    while (this.size() != 0) {
        to.add(this.pop());
    }
}
```

Again, that `to` from `popAll` is considered/regarded as the consumer of `E` inside this scope. Since it consumes E from the `Stack<E>` then passing them out.

However, If you pass a `Collection<Object>` object as `to` into this method, it will not compile. But normally you would expect, a `Collection<Object>` should have no trouble being added with Object and Object’s subtype. In other words, `Collection<Object>` should be able to add `E`, since `E` is a subtype of `Object`.

Once again, Bounded Wildcard types provide a way out.

We want to ensure that `Collection<E> to` is a `Collection` of `E` and `E`'s super-type. That’s how it comes into handy.

```java
public void popAll(Collection<? super E> to){
    while (this.size() != 0) {
        to.add(this.pop());
    }
}
```

## Conclusion

Proper usage of bounded wildcard type is nearly invisible to the users of a class.

Similar pattern can be seen in Java’s source code. It’s wildly used.

For instance, in `java.util.Collections`

```java

    /**
     * Copies all of the elements from one list into another.  After the
     * operation, the index of each copied element in the destination list
     * will be identical to its index in the source list.  The destination
     * list's size must be greater than or equal to the source list's size.
     * If it is greater, the remaining elements in the destination list are
     * unaffected. <p>
     *
     * This method runs in linear time.
     *
     * @param  <T> the class of the objects in the lists
     * @param  dest The destination list.
     * @param  src The source list.
     * @throws IndexOutOfBoundsException if the destination list is too small
     *         to contain the entire source List.
     * @throws UnsupportedOperationException if the destination list's
     *         list-iterator does not support the {@code set} operation.
     */
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }

```

In the view of `T` in `Collections.copy(List<? super T> dest, List<? extends T> src)`. `src` produces `T`, `dest` consumes `T`. Hence the bounded wildcard is fit in this case.

At the same time, suggestions from Effective Java.

+  If a type parammeter appears only once in a method declaration, replace it with a wildcard.

+ All comparables and comparators are consumers.

+ DO NOT use bounded wildcard types as return types.

```java
// Not recommended but readability is nice for me.  
public static <T extends Comparable<T>> T someMethod(List<T> list)

// Replace it with
public static <T extends Comparable<? super T>> someMethod(List<? extend T> list)
```

