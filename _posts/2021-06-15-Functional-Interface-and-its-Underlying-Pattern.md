---
title: Functional Interface and its Underlying Pattern - Effective Java reading notes
author: MijazzChan
date: 2021-06-15 20:47:41 +0800
categories: [Personal_Notes, Effective_Java]
tags: [java, notes]
---

> Just like I said in this [post](/posts/Some-Neat-Design-Patterns-Effective-Java-reading-notes/), These patterns are pretty easy/simple, but it really helps me a lot especially when managing to understanding Java’s underlying design pattern through reading Java source code. Following these patterns also helps producing code which is developer-friendly. 

Keyword: Functional Interface, `Map::computeIfPresent`, PECS mnemonic, Consumer, Predicate, Supplier, BinaryOperator, UnaryOperator.

## Functional Interface Introduction

Annotated widely across `java.util.function`, Functional Interface provide a way to represent a function that accept one/multiple argument(s) by creating a interface then implementing it with lambda expressions, method references, or constructors.

However, before I started writing this post, crawling over blogs and posts, I still can't find a vivid example that can explain or express how flexible it can be. Thus, here is a example I came up with.

```java
    @Test
    void functionalInterface_usage_Stream() {
        LongStream longStream = LongStream.range(1L, 200L);
        // LongToIntFunction
        LongToIntFunction mapPositiveLongToInt = (
                longNumber -> (longNumber > Integer.MAX_VALUE)
                        ? Integer.MAX_VALUE
                        : (int) longNumber
        );
        IntStream intStream = longStream.mapToInt(mapPositiveLongToInt);
        // IntPredicate
        IntPredicate isPowerOf2 = (
                num -> (num != 0) && ((num & (num - 1)) == 0)
        );
        List<Integer> powerOf2Under200 = intStream
                .filter(isPowerOf2)
                .boxed()
                .collect(Collectors.toUnmodifiableList());
        Assertions.assertEquals(powerOf2Under200, List.of(1, 2, 4, 8, 16, 32, 64, 128));

        class Circle {
            double radius;
            double area;

            public Circle(double radius, double area) {
                this.radius = radius;
                this.area = area;
            }
        }
        // takes Integer -> returns Circle
        Function<Integer, Circle> radiusToCircle = (
                radius -> new Circle(radius, StrictMath.PI * radius * radius)
        );

        List<Circle> circleList = powerOf2Under200
                .stream()
                .map(radiusToCircle)
                .collect(Collectors.toList());
        // Takes Circle -> returns boolean
        Predicate<Circle> areaBetween800and5000 = (
                circle -> circle.area > 800 && circle.area < 5000
        );
        // Takes Circle -> void, consumes it.
        Consumer<Circle> circleAreaPrinter = (
                circle -> System.out.printf("%-10.5f", circle.area)
        );

        circleList.stream()
                .filter(areaBetween800and5000)
                .forEach(circleAreaPrinter);
        // console: 804.24772 3216.99088
    }
```

## Functional Interfaces  in `java.util.function`

```java
/**
 * Represents a function that accepts one argument and produces a result.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
    
    // ... compose(), andThen(), identity()
}
```

Dig deeper into said package, you will find 43 functional interfaces. There are interfaces with specific type declaration, such as `IntConsumer`, `LongToDoubleFunction`. Let's set aside those with type declaration, with simple classification, we can derive 6 basic functional interfaces. 

| Interface           | Function Signature    | How it perform                                | Example               |
| ------------------- | --------------------- | --------------------------------------------- | --------------------- |
| `Function<T, R>`    | `R apply(T t)`        | Functions which take T but return R           | `Arrays::asList`      |
| `Supplier<T>`       | `T get()`             | ... take no arg and return T                  | `LocalDate::now`      |
| `Comsumer<T>`       | `void accept(T t)`    | ... take T as arg but return nothing          | `System.out::println` |
| `Predicate<T>`      | `boolean test(T t)`   | ... take T as arg and return a condition bool | `Collection::isEmpty` |
| `UnaryOperator<T>`  | `T apply(T t)`        | ... take 1 T as arg and also return T         | `String::toLowerCase` |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | ... take 2 T as arg and also return T         | `BigInteger::add`     |

With all this method only accepting certain type or returning certain type, despite 8 primitive types also have corresponding boxed primitives which fits the design pattern,  additional variants of `Function` interfaces are provided, for use when the argument/result type is primitive.

**Mentioned in Effective Java**, Do NOT use basic functional interface with boxed primitives instead of primitive functional interface. Although with the auto-boxing and auto-unboxing mechanisms, it will still work but with the consequences of bad performance.

## Pattern Usage in Java's Design

> `Map::compute`, `Map::computeIfabsent`, `Map::computeIfpresent` have similar design pattern. We will discuss `computeIfPresent` here.

### `Map::computeIfPresent`

This method has been added since 1.8. You can have a glance at the source code(`Map.java`:1074). Its code is really straight-forward. The basic idea of this method is to

+ Accept a `key`, and a `BiFunction`
+ If the `key` exists in the `Map`
  - use `BiFunction` with `key` and its `oldValue` as arguments to derive a new Value
  - If the new Value is not null
    + Update the value by `map.put(key, newValue);`
  - If the new Value is null
    + Remove the entry of the `key`.
+ If the `key` not exists in the `Map`
  - Do nothing.

```java
// java.util.Map.java : 1074    
default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }
```

learn from above, a `BiFunction` is simply a Function Interface which takes 3 type parameters, first and second are the types of function argument, the third one is the type of returning obj.

```java
// java.util.function.BiFunction.java : 44
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);
    // .......
}
```

Focusing on its `remappingFunction` argument type `BiFunction<? super K, ? super V, ? extends V>`, it is also obviously a **PECS pattern**. 

> Didn't heard of them? Check out my last post about [PECS Mnemonic](/posts/PECS-Mnemonic-Bounded-wildcard-Type-Effective-Java-reading-notes/). 

From the aspect of PECS Mnemonic, in the scope of `BiFunction`

+ `<? super K>` is a consumer, it consumes `K key` from this argument for `apply()` 's remapping usage.
+ `<? super V>` is a consumer, it consumes `V oldValue` from this argument for `apply()` 's remapping usage.
+  `<? extends V>` is a producer, it produces  a newly generated  `V newValue` and return it.

From the aspect of Functional Interface,

+ first type argument is Map's key type (as arg being passed in)

+ second type argument is Map's value type (as arg being passed in)

+ third type argument **should** be Map's value type `V` or `V`'s sub-type. (as obj being returned)

  Otherwise, returning type cannot be put inside the Map because of type mis-matching.

Theory and explanation without practice or example are always hard to ~~swallow~~(follow).

```java
// Map.compute
// Map.computeIfPresent
    @Test
    void computeIfPresent_FunctionalInterface() {
        // Suppose you have a piecewise-defined function
        /*      { x*2,   0<x<3  }
         *  y = { x*3,  3<=x<5 } x is N{0, 1, 2, 3, 4...}
         *      {  0 ,  others  }
         */
        // This map is to store function value from 0~10
        Map<Integer, Double> yValMap = new HashMap<>();
        // Create a functional interface to calculate the value of y
        Function<Integer, Double> calY = (
                x -> {
                    if (x > 0 && x < 3) return (double) (x * 2);
                    else if (x >= 3 && x < 5) return (double) (x * 3);
                    else return null;
                }
        );
        // for x in range(1, 5)
        // because yValMap has no k-v, initialize it with x from 1 to 10
        // calY fits => Function<? super Integer, ? extends Number>
        IntStream.range(1, 5).forEach(
                x -> yValMap.computeIfAbsent(x, calY)
        );
        printMyFunctionMap(yValMap);

        // Suppose a z, where z = x + y*1.6
        // z's equation contains both x and y. inside yValMap, you have
        // both x and y as K and V. use x as key, y as oldValue compute z as newValue
        // You just alter yValMap to fit zValMap's logic.
        BiFunction<Integer, Double, Double> updateZFromY = (
                (x, y) -> {
                    return x + 1.6 * y;
                }
        );

        // for x in range(1, 5)
        // x from 1, 10 is present in the map, invoke computeIfPresent
        // will pass (key, oldValue) which is (x, y) as argument
        // to updateZFromY to perform compute z's value as newValue.
        IntStream.range(1, 5).forEach(
                x -> yValMap.computeIfPresent(x, updateZFromY)
        );
        // After computing, yValMap is zValMap now.
        Map<Integer, Double> zValMap = yValMap;
        printMyFunctionMap(zValMap);

    }

    // parameter PECS Mnemonic
    void printMyFunctionMap(Map<? extends Integer, ? extends Double> map) {
        for (Map.Entry<? extends Integer, ? extends Double> entry : map.entrySet()) {
            int x = entry.getKey();
            double y = entry.getValue();
            System.out.printf("%d -> %.2f\n", x, y);
        }
        System.out.println("*****");
    }
```

```java
// Console
1 -> 2.00
2 -> 4.00
3 -> 9.00
4 -> 12.00
*****
1 -> 4.20
2 -> 8.40
3 -> 17.40
4 -> 23.20
*****
```

Similar pattern can be seen wildly across `java.util.Stream`. Especially on `map()` and `flatMap()` method, it help abstraction on the data/object flow, generalize it like a data/object pipe. `<R> Stream<R> map(Function<? super T, ? extends R> mapper)`, `IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper)`....

