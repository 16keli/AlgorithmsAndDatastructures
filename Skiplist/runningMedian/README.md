## runningMedian

### Time per insertion/calculation: O(log n) expected, O(n) worst case ---- Space O(n) expected, O(n log n) worst case

> Given a continuous stream of numbers, devise an efficient way to compute the running median after every insertion

This may not be that common of a question, but I find it interesting nonetheless. And no, this is not the intended solution for 281 project 2, it's merely an alternative method of computing the running median that I found interesting.

The data structure of choice is a somewhat unholy marriage of the set and list implementations of the skip list. The multiset allows for continued insertion (and removal) of the same elements, but the data within the skip list is also accessible by index.

Basically, the interface of the indexable multiset will consist of (at minimum):

```cpp
// Size of skip list (number of elements)
size_t size() const;

// Insert a number into the list
void insert(double val);

// Access the given index (overloaded operator[] just like vector, map, etc.)
double & operator[] (size_t index);
```

One may question the practicality of this method of computing running medians. Compared to other methods, such as the dual heaps, this is *much* more complicated. You have to implement an entire data structure!!!!!!! Why do this?

For fun, basically.

### Skip List > Two Heaps?

Also, using a skip list provides a few more advantages over dual heaps:
*	Skip lists are probabalistic, so unlike deterministic methods like the heaps, an adversary can never guarantee worst case runtime with a certain input. Basically one of the few reasons to actually use a skip list (covered in 376 thank you based Ilya)
*	An insertion into a skip list requires just one (expected) O(log n) operation, while dual heaps will (often enough) require two O(log n) operations
*	Skip lists allow for efficient computation of a **moving** median as well, while dual heaps will require quite a bit of tinkering to support. More on this later
*	Skip lists will never require the expensive reallocations which will result in half-empty space usage for heaps
	*	Why is this? Well, since heaps are based on vectors (at least in C++), and vectors must reallocate their space when full, this will sometimes result in quite unfortunate happenings.
	*	That is, consider a full heap of n elements, which means all n slots are occupied. If we wish to add one, then we must expand to 2n (typically, and also unfortunately) slots, but only n + 1 are used!

### Skip List < Two Heaps?

But we must also look at the disadvantages compared to dual heaps:
*	Skip lists are probabalistic, so there's a chance you could get a really shitty internal structure. Yes, it's very rare that something bad like that will happen (for the same reason that worst-case runtime of quicksort probability also decreases with size), but it **could** still happen.
*	Well, the fact that you're kinda implementing an entire new data structure. Heaps are very likely defined already and easy to use right off the bat.
*	Computation of the median will require O(log n) time in a skip list, while it requires constant time with dual heaps. If we repeatedly compute the median without adding any data, we will see the skip list fall far behind
	*	We can assuage this by caching the computed value of the median, and only recomputing if needed. Still, this doesn't change the fact that computation is one O(log n) operation in a skip list, while in dual heaps it's O(1)

### Small Details

Enough about comparing the two methods. Let's get on to some thinking that might prove useful.

We must consider possible cases, and what tools we have at our disposal. Recall that we can use `size()` and `operator[]`. Then, it seems pretty obvious what to do.
*	Odd Size: access the median element and return it
*	Even Size: access the lower of median element, then the higher of median element and return their average

adasdasd I don't know what else to write ahhhhh