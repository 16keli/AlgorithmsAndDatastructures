## runningMedian

### Time per insertion/calculation: O(log n) expected, O(n) worst case ---- Space O(n) expected, O(n log n) worst case

> Given a continuous stream of numbers, devise an efficient way to compute the running median after every insertion

This may not be that common of a question, but I find it interesting nonetheless. And no, this is not the intended solution for 281 project 2, it's merely an alternative method of computing the running median. This is a Streaming algorithm, which means the input will only be presented **once**, so we must proceed with caution.

A basic naive solution to the problem might be maintaining a vector of all values seen. To get the value, we would sort and compute the median that way. Unfortunately, we will be computing the median after every number is inserted. With unsorted data, we require an expensive O(n log n) sort. Even if we start with mostly-sorted data, we must still move the data to make room for the new number, a O(n) operation. That's still not good enough for us. We can achieve O(log n) time a few different ways. We will be implementing a solution using skip lists here.

I assume you have some familiarity with skip lists already. I recommend reading up on skip lists (coming to this repo soom TM) before proceeding.

The ADT of choice (for this implementation, anyways) is a somewhat unholy marriage of the set and list implementations of the skip list. It is a multiset, which allows for continued insertion (and removal) of the same elements, but the data within the skip list is also accessible by index.

Basically, the interface of the indexable multiset will consist of (at minimum):

```cpp
// Size of skip list (number of elements)
size_t size() const;

// Insert a number into the list
void insert(double val);

// Access the given index (overloaded operator[] just like vector, map, etc.)
const double & operator[] (size_t index);
```

One may question the practicality of this method of computing running medians. Compared to other methods, such as the dual heaps, this is *much* more complicated. You have to implement an entire data structure!!!!!!! Why do this?

For fun, basically.

### Skip List > Two Heaps?

Using a skip list provides a few more advantages over dual heaps:
*	Skip lists are probabalistic, so unlike deterministic methods like the heaps, an adversary can never guarantee worst case runtime with a certain input. Basically one of the few reasons to actually use a skip list (covered in 376 thank you based Ilya)
*	An insertion into a skip list requires just one (expected) O(log n) operation, while dual heaps will (often enough) require two O(log n) operations
*	Skip lists allow for efficient computation of a **moving** median as well, while dual heaps will require quite a bit of tinkering to support. More on this later
*	Skip lists will never require the expensive reallocations which will result in half-empty space usage for heaps
	*	Why is this? Well, since heaps are based on vectors (at least in C++), and vectors must reallocate their space when full, this will sometimes result in quite unfortunate happenings.
	*	That is, consider a full heap of n elements, which means all n slots are occupied. If we wish to add one, then we must expand to 2n (typically, and also unfortunately) slots, but only n + 1 are used!

### Skip List < Two Heaps?

But we must also look at the disadvantages compared to dual heaps:
*	Skip lists are probabalistic, so there's a chance you could get a really shitty internal structure. Yes, it's very rare that something bad like that will happen (for the same reason that worst-case runtime of quicksort probability also decreases with size), but it **could** still happen.
*	Arrays are much more cache-friendly than pointers
*	Well, the fact that you're kinda implementing an entire new data structure. Heaps are very likely defined already and easy to use right off the bat.
*	Computation of the median will require O(log n) time in a skip list, while it requires constant time with dual heaps. If we repeatedly compute the median without adding any data, we will see the skip list fall far behind
	*	We can assuage this by caching the computed value of the median, and only recomputing if needed. Still, this doesn't change the fact that computation is one O(log n) operation in a skip list, while in dual heaps it's O(1)

### TL;DR?

From a pure performance perspective:
*	Skip List > Two Heap on:
	*	insertion
	*	moving median
*	Skip List < Two Heap on:
	*	calculation
	*	cache friendliness
It's up to you to decide which approach is better, based on the context of your problem

### Small Details

Enough about comparing the two methods. Let's get on to some thinking that might prove useful.

We must consider possible cases, and what tools we have at our disposal. Recall that we can use `size()` and `operator[]`. Then, it seems pretty obvious what to do.
*	Odd Size: access the median element and return it
*	Even Size: access the lower of median element, then the higher of median element and return their average

adasdasd I don't know what else to write ahhhhh