---
title: Store Multiple Booleans Using Single Variable
author: MijazzChan
date: 2021-04-18 17:14:49 +0800
categories: [Personal_Notes]
tags: [notes]
---


## Intro

Inspired by a stackoverflow answer which is more than a decade old. [StackOverflow-Answer1](https://stackoverflow.com/questions/229886/size-of-a-byte-in-memory-java), [StackOverflow - Answer2](https://stackoverflow.com/questions/383551/what-is-the-size-of-a-boolean-variable-in-java)

[@Jon Skeet](https://stackoverflow.com/users/22656/jon-skeet) there provide a way of measuring how much memory do `byte`, `int` and  `boolean` variables takes. 

To simply clarify my point here, I modified some of his code in order to reproduce the situation.

> He pointed out that the actual size of those variables was VM-dependent, and  his VM was `Sun's JDK build 1.6.0_11`.
>
> My testcase VM  is `AdoptOpenJDK (build 11.0.10+9)` - hotspot VM. 

Also, in this  [Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html) about `Primitive Data Types`. 

> **boolean**: The `boolean` data type has only two possible values: `true` and `false`. Use this data type for simple flags that track true/false conditions. This data type represents one bit of information, but its "size" isn't something that's precisely defined. 

### Reproduce Code

```java
    class LotsOfBytes {
        byte a0, a1, a2, a3, a4, a5, a6, a7, a8, a9, aa, ab, ac, ad, ae, af;
        byte b0, b1, b2, b3, b4, b5, b6, b7, b8, b9, ba, bb, bc, bd, be, bf;
        byte c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, ca, cb, cc, cd, ce, cf;
        byte d0, d1, d2, d3, d4, d5, d6, d7, d8, d9, da, db, dc, dd, de, df;
        byte e0, e1, e2, e3, e4, e5, e6, e7, e8, e9, ea, eb, ec, ed, ee, ef;
    }

    class LotsOfInts {
        int a0, a1, a2, a3, a4, a5, a6, a7, a8, a9, aa, ab, ac, ad, ae, af;
        int b0, b1, b2, b3, b4, b5, b6, b7, b8, b9, ba, bb, bc, bd, be, bf;
        int c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, ca, cb, cc, cd, ce, cf;
        int d0, d1, d2, d3, d4, d5, d6, d7, d8, d9, da, db, dc, dd, de, df;
        int e0, e1, e2, e3, e4, e5, e6, e7, e8, e9, ea, eb, ec, ed, ee, ef;
    }

    class LotsOfBooleans {
        boolean a0, a1, a2, a3, a4, a5, a6, a7, a8, a9, aa, ab, ac, ad, ae, af;
        boolean b0, b1, b2, b3, b4, b5, b6, b7, b8, b9, ba, bb, bc, bd, be, bf;
        boolean c0, c1, c2, c3, c4, c5, c6, c7, c8, c9, ca, cb, cc, cd, ce, cf;
        boolean d0, d1, d2, d3, d4, d5, d6, d7, d8, d9, da, db, dc, dd, de, df;
        boolean e0, e1, e2, e3, e4, e5, e6, e7, e8, e9, ea, eb, ec, ed, ee, ef;
    }
```

each of  `LotsOfXXX` has 80 variables.

```java
private static final int SIZE = 1000000;

    @Test
    void test() {
        LotsOfBytes[] first = new LotsOfBytes[SIZE];
        LotsOfInts[] second = new LotsOfInts[SIZE];
        LotsOfBooleans[] third = new LotsOfBooleans[SIZE];

        System.gc();
        long startMem = getMemory();

        for (int i = 0; i < SIZE; i++) {
            first[i] = new LotsOfBytes();
        }

        System.gc();
        long endMem = getMemory();

        System.out.println("Size for LotsOfBytes: " + (endMem - startMem));
        System.out.println("Average size: " + ((endMem - startMem) / ((double) SIZE)));
        System.out.println("Appro. size for each Byte: " + ((endMem - startMem) / ((double) SIZE)) / 80);
        System.out.println();

        System.gc();
        startMem = getMemory();
        for (int i = 0; i < SIZE; i++) {
            second[i] = new LotsOfInts();
        }
        System.gc();
        endMem = getMemory();

        System.out.println("Size for LotsOfInts: " + (endMem - startMem));
        System.out.println("Average size: " + ((endMem - startMem) / ((double) SIZE)));
        System.out.println("Appro. size for each Int: " + ((endMem - startMem) / ((double) SIZE)) / 80);
        System.out.println();


        System.gc();
        startMem = getMemory();

        for (int i = 0; i < SIZE; i++) {
            third[i] = new LotsOfBooleans();
        }

        System.gc();
        endMem = getMemory();

        System.out.println("Size for LotsOfBooleans: " + (endMem - startMem));
        System.out.println("Average size: " + ((endMem - startMem) / ((double) SIZE)));
        System.out.println("Appro. size for each Boolean: " + ((endMem - startMem) / ((double) SIZE)) / 80);
        System.out.println();


        // Make sure nothing gets collected
        long total = 0;
        for (int i = 0; i < SIZE; i++) {
            total += first[i].a0 + second[i].a0 + ((!third[i].a0) ? 0 : 1);
        }
        System.out.println(total);
    }

    private static long getMemory() {
        Runtime runtime = Runtime.getRuntime();
        return runtime.totalMemory() - runtime.freeMemory();
    }
```

### Result

```
Size for LotsOfBytes: 95671792
Average size: 95.671792
Appro. size for each Byte: 1.1958974

Size for LotsOfInts: 336440976
Average size: 336.440976
Appro. size for each Int: 4.205512199999999

Size for LotsOfBooleans: 95999768
Average size: 95.999768
Appro. size for each Boolean: 1.1999971
```

Note that `Appro. size for each XXX` is not precisely the actual size of each `XXX`. There was memory reserved for class/obj definition or other kind of stuff. 

As it generates, `byte`  and `boolean` , each of them, takes 1 Byte memory. `int` takes 4 Byte each.

**What if you can store 8 `boolean` inside a single `byte`**. In that way, 1 `boolean` takes nearly 1 bit. 

## Method

### How this works?

Suppose that you need to store a `boolean` array just like

```java
boolean[] someFlags = new boolean[8];
```

you will have a array of 8 `boolean`, all in false state, which is

```java
[false, false, false, false, false, false, false, false]
```

Given the situation discussed above, 8 Byte here taken.

What if you can using a `byte` ‘s bit to represent them.

```
127(dec) -> 0111 1111 (binary)
  0(dec) -> 0000 0000 (binary)
```

 Each bit holds a positional boolean status inside one Byte.

Here is an example: 

```
[false, false, true, false, false, false, true, true]
           boolean[] <=> binary
[  0  ,  0  ,   1 ,   0  ,   0  ,   0  ,   1 ,   1 ]
              binary <=> dec
35(dec)
```

### Set positional boolean flag to true

> `byte flags = 0;`

```java
flags |= 1 << position;
// which is equivalent to 
// flag = flag | 1 << position
```

Take a deep look in how that works.

+ Suppose we need to `someFlags[2] = true`

  ```java
  flags = 0000 0000 | (1 << 2);
  flags = 0000 0000 | 0000 0100;
  flags = 0000 0100;
  ```
```



+ Then`someFlags[4] = true`:

  ```java
  flags = 0000 0100 | (1 << 4);
  flags = 0000 0100 | 0001 0000;
flags = 0001 0100;
```

  

  as long as the position arg not exceeding 7, flags will be well preserved in this variable.

### Set positional boolean flag to false

> flags will remain untouched.(`0001 0100`)
>
> Additional info: `~`operator is `Complement Operator`. Flipping bits in binary will do the work.

```java
flags &= ~(1 << position);
// which is equivalent to 
// flag = flag & ~(1 << position)
```

+ Suppose we need to toggle `someFlags[2] = false`

  ```java
  flags = 0001 0100 & ~(1 << 2);
  flags = 0001 0100 & ~(0000 0100);
  // "~" operator => flip bits
  flags = 0001 0100 &   1111 1011;
  flags = 0001 0000;
  ```

  Now the someFlags[2] is set to false;

### Retrieve flag

> flags will remain untouched.(`0001 0000`)

 ```java
return (flags & (1 << position)) == (1 << position);
 ```

+ Suppose we need to check whether `someFlags[4] == true`

  ```java
  return (0001 0000 & (1 << 4)) == (1 << 4);
  return (0001 0000 & 0001 0000) == 0001 0000;
  return  0001 0000 == 0001 0000;
  return true;
  ```



## Furthermore

`int`, which consists of 4 Byte(32 bit), is totally viable theoretically, and it can hold 32 boolean variables, but consumes memory just as 4 `boolean` do.

In modern computer we may not need this kind of ~~twisted~~ way to saving memory. Just consider it a way of leetcode technique. However, working on some embedded devices with little tiny memory size available, you may find it useful.


## Keyword

+ Leetcode save memory
+ memory saving tweaks
+ multiple booleans in 1 variable