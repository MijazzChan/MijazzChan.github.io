---
title: Some Utility Functions While Leetcoding
author: MijazzChan
date: 2021-04-13 14:52:54 +0800
categories: [Personal_Notes]
tags: [notes, leetcode, tools]
---

## Array Generator

> `Secure.Random` / `ThreadLocalRandom`

```java
public static final ThreadLocalRandom randStream = ThreadLocalRandom.current();

    public static int randomInt(int from, int to) {
        return randStream.nextInt(Math.min(from, to), Math.max(from, to));
    }

    public static double randomDouble(double from, double to) {
        return randStream.nextDouble(Math.min(from, to), Math.max(from, to));
    }

    public static int[] randomIntArray(int size, int from, int to) {
        return randStream.ints(size, from, to).toArray();
    }

    public static int[] randomIntArray(int size){
        return randStream.ints(size, 0, 10_000).toArray();
    }

    public static double[] randomDoubleArray(int size, double from, double to) {
        return randStream.doubles(size, from, to).toArray();
    }

    public static double[] randomDoubleArray(int size){
        return randStream.doubles(size, 0, 100).toArray();
    }
```

## `ListNode` with `toString()` method implement

+ `toString()` method is really essential and useful during unit/problem testing/testcase.
+ Build ListNode from an array: The combination of `buildListNodeFrom()` method and `randomIntArray()` mentioned above can save a shit-load of time while building `ListNode` with plain code. 

```java
public class ListNode {

    public int val;

    public ListNode next;

    // Constructor
    public ListNode() {
    }

    public ListNode(int val) {
        this.val = val;
    }

    public ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
	// ListNode toString method
    // ListNode Print Method
    @Override
    public String toString() {
        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append(this.val).append("=>");
        while (this.next != null) {
            stringBuilder.append(this.next.val).append("=>");
            this.next = this.next.next;
        }
        return stringBuilder.append("NULL").toString();
    }
    // Build ListNode from an array
    public static ListNode buildListNodeFrom(int[] values) {
        ListNode head = new ListNode(0, new ListNode());
        ListNode listNode = head.next;
        if (null == values || values.length == 0) {
            throw new IllegalArgumentException("Values[] cannot be null");
        }
        for (int i = 1; i < values.length; i++) {
            listNode.val = values[i - 1];
            listNode.next = new ListNode(values[i]);
            listNode = listNode.next;
        }
        return head.next;
    }
}
```



**To be updated.**

