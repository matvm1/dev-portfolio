---
title: "Amortized Analysis"
description: "Amortized Analysis"
pubDate: 2025-04-16
category: "DSA"
draft: false
---

# Introduction
I began taking a [Data Structures and Algorithms course](https://www.coursera.org/learn/algorithms-part1/) a couple of weeks ago, and have learned some very interesting patterns and optimization techniques used when solving problems.
This post is about one of this week's challenges, the implementation of a randomized queue under **amortized runtime constraints**.

# The Task at Hand
The lecture on stacks and queues followed up with this task:  
> *A randomized queue is similar to a stack or queue, except that the item removed is chosen uniformly at random among items in the data structure.
> Create a generic data type RandomizedQueue that implements the following API:*
```java
public class RandomizedQueue<Item> implements Iterable<Item> {

    // construct an empty randomized queue
    public RandomizedQueue()

    // is the randomized queue empty?
    public boolean isEmpty()

    // return the number of items on the randomized queue
    public int size()

    // add the item
    public void enqueue(Item item)

    // remove and return a random item
    public Item dequeue()

    // return a random item (but do not remove it)
    public Item sample()

    // return an independent iterator over items in random order
    public Iterator<Item> iterator()

    // unit testing (required)
    public static void main(String[] args)

}
```
> *Performance requirements: Your randomized queue implementation must support each randomized queue operation (besides creating an iterator) in constant amortized time.*  
> *That is, any intermixed sequence of m randomized queue operations (starting from an empty queue) must take at most cm steps in the worst case, for some constant c.*


# My Approach
`isEmpty()`, `size()`, and `enqueue()` had straightforward implementations, with O(1) access time regardless of whether the data is stored in a linked-list or array.  
`dequeue()` and `sample()` required a bit more thought. In order to implement the randomized selection of an element in O(1) amortized time, the underlying data storage needs to support random access to avoid traversing potentially *n* elements in a queue of size *n*. So I decided to explore if an array-backed implementation could fit the constraint of O(1) amortized time.  

`enqueue()`: A "resizing" array would mean that the operation could take O(n) time if the array were to resize. However, as the array doubles in size, this worse case event should become more and more rare, amortizing the operation to O(1) time.  

`sample()`: Generate a random index from 0 to the size of the queue, then return the element at that index.  

`dequeue()`: This is where the problem got interesting. I was reminded of percolation systems, where the concept of "opening", at random, all sites in a grid that is initially entirely closed approximates a worst case O(n log n) runtime. I circled around this idea for a while thinking - what else can be done if the user were to dequeue until nothing was left? They'd be opening *n* sites at random, which exceeds the O(1) constraint. It then occurred to me - what if the array were to halve whenever it is at 20% capacity? Each of the first 80% selections would take at most 5 attempts on average. The array would then resize, where the odds would shoot back up to 1 attempt on average, then up to 5 attempts once the array is back to 20% capacity and so on.  
So at worst, any dequeue operation in *m* operations would take on average 5 attempts (constant amortized time).  

The risk of thrashing still remained - but would seem irrational/improbable that the client would repeatedly enqueue then dequeue when the stack is at full capacity.  

# Optimization
Something about leaving gaps in the array wasn't sitting well with me, perhaps some edge case I hadn't though of could come back to bite. So I decided to do some research on list randomization. I came across [this Medium article on shuffling lists and arrays](https://medium.com/intuition/shuffling-a-list-or-array-597da1f1a32e). It mentions the inefficent solution of going through each element and selecting a random index to swap with.  

While this may be inefficient for shuffling an entire list, it helped me realize that the order in which elements were stored in the queue did not matter. This is not a queue in the traditional sense, i.e. the client does not care to preserve order - they expect randomness. This opened up the idea of moving elements around, particularly the last non-null element in the array.  

Generate the random index, create a pointer to that object, move the last object into that index, and return the pointer. This preserves the contiguity of the queue, ensuring that there is always an element at any index from 0 to the size of the queue. By preventing these gaps, any dequeue operation takes at most O(1) time (with exception of the halving event).  

## Thoughts
Amortized analysis is one of those areas in technology that isn't a simple yes/no answer or some formula that derives an answer. Will the client use the solution appropriately? What defines "rarity"? What is considered a misuse of the solution? Perhaps I need to learn more on the subject (probably), but at this point it seems like the approach is an even more rounded guess than what operational runtime analysis already is.  

Regarding the lecture on queues, it was very interesting. I had never considered using an array for stack/queue implementations, and have learned that they enable randomization in collections. From random sampling, lotteries, to random playlists, these are some of the applications enabled by this data structure. The problem was effective at demonstrating the complexity of amortized analysis, and allowed me to see that a solution can be "good enough", with room for improvement via optimization.