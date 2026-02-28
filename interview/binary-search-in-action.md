::: info Required Knowledge

Before reading this article, you need to learn:

- [Binary Search Framework in Detail](https://labuladong.online/en/algo/essential-technique/binary-search-framework/)

:::

In [Binary Search Framework in Detail](https://labuladong.online/en/algo/essential-technique/binary-search-framework/), we thoroughly examined the details of binary search, exploring three scenarios: "searching for an element", "searching for the left boundary", and "searching for the right boundary", teaching you how to write correct, bug-free binary search algorithms.

**However, the binary search framework summarized in that article is limited to the basic scenario of "searching for a specified element in a sorted array". Real algorithm problems aren't this straightforward—you might not even recognize that a problem can use binary search**.

So this article summarizes a framework for applying binary search algorithms, helping you think through and analyze binary search problems systematically, step by step, to arrive at the solution.

## Original Binary Search Code

The prototype of binary search is searching for an element `target` in a **sorted array** and returning the corresponding index.

If the element doesn't exist, you can return some special value. These details can be implemented with minor adjustments to the algorithm.

There's another important issue: if the **sorted array** contains multiple `target` elements, these elements will definitely be adjacent. This involves whether the algorithm should return the index of the leftmost `target` element or the rightmost one—the so-called "searching for the left boundary" and "searching for the right boundary". This can also be implemented through minor code adjustments.

**We discussed these issues in detail in the previous article [Binary Search Core Framework](https://labuladong.online/en/algo/essential-technique/binary-search-framework/). Readers who are unclear on this should review that article**. If you already understand the basic binary search algorithm, you can continue reading.

**In real algorithm problems, the "search left boundary" and "search right boundary" scenarios are commonly used**, while rarely will you be asked to simply "search for an element".

This is because algorithm problems generally ask you to find optimal values, like finding the "minimum speed" for eating bananas or the "minimum capacity" for a ship. The process of finding optimal values is necessarily a process of searching for a boundary, so we'll analyze these two boundary search binary algorithms in detail.

::: note Note

Note that I'm using the left-closed, right-open notation for binary search. If you prefer both ends closed, you can modify the code accordingly.

:::

The specific code implementation for the "search left boundary" binary search algorithm is as follows:

```java
// search for the left boundary
int left_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            // when target is found, shrink the right boundary
            right = mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    return left;
}
```

Suppose the input array is `nums = [1,2,3,3,3,5,7]` and you want to search for `target = 3`. The algorithm will return index 2.

If we draw a diagram, it looks like this:

![](../pictures/binary-search-in-action/1.jpeg)

The specific code implementation for the "search right boundary" binary search algorithm is as follows:

```java
// search for the right boundary
int right_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            // when target is found, shrink the left boundary
            left = mid + 1;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    return left - 1;
}
```

With the same input, the algorithm will return index 4. If we draw a diagram, it looks like this:

![](../pictures/binary-search-in-action/2.jpeg)

Good, the above content is all review. I expect readers at this point should understand it all. Remember the diagrams above—any problem that can be abstracted into these diagrams can be solved using binary search.

<!-- hide -->

## Generalizing Binary Search Problems

What problems can apply binary search algorithm techniques?

**First, you need to abstract from the problem an independent variable `x`, a function `f(x)` about `x`, and a target value `target`**.

Additionally, `x, f(x), target` must satisfy the following conditions:

**1. `f(x)` must be a monotonic function on `x` (either monotonically increasing or decreasing)**.

**2. The problem asks you to calculate the value of `x` that satisfies the constraint `f(x) == target`**.

These rules sound a bit abstract, so let's use a concrete example:

You're given a sorted array `nums` in ascending order and a target element `target`. Calculate the index position of `target` in the array. If there are multiple target elements, return the smallest index.

This is the "search left boundary" basic problem type. We've written the solution code before, but what are `x, f(x), target` here?

We can think of the array element indices as the independent variable `x`, and define the function relationship `f(x)` like this:

```java
// The function f(x) is a monotonically increasing function of the independent variable x
// The input parameter nums will not change, so it can be ignored and is not considered an independent variable
int f(int x, int[] nums) {
    return nums[x];
}
```

This function `f` is actually just accessing the array `nums`. Since the problem gives us the array `nums` in ascending order, the function `f(x)` is monotonically increasing on `x`.

Finally, what does the problem ask us to find? Doesn't it ask us to calculate the leftmost index of element `target`?

Isn't that equivalent to asking "what is the minimum value of `x` that satisfies `f(x) == target`"?

Draw a diagram like this:

![](../pictures/binary-search-in-action/3.jpeg)

**If you encounter an algorithm problem and can abstract it into this diagram, you can apply binary search to it**.

The algorithm code is as follows:

```java
// Function f is a monotonically increasing function of the independent variable x
int f(int x, int[] nums) {
    return nums[x];
}

int left_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (f(mid, nums) == target) {
            // When target is found, shrink the right boundary
            right = mid;
        } else if (f(mid, nums) < target) {
            left = mid + 1;
        } else if (f(mid, nums) > target) {
            right = mid;
        }
    }
    return left;
}
```

This code is actually redundant—it's the conventional binary search code with a slight modification, wrapping direct access to `nums[mid]` in a function `f`. However, this abstracts the framework of binary search thinking in specific algorithm problems.


## A Framework for Applying Binary Search

When you want to use binary search to solve a specific algorithm problem, start by thinking through this code framework:

```java
// function f is a monotonous function of the independent variable x
int f(int x) {
    // ...
}

// main function, find the extremum of x under the constraint of f(x) == target
int solution(int[] nums, int target) {
    if (nums.length == 0) return -1;
    // ask yourself: what is the minimum value of the independent variable x?
    int left = ...;
    // ask yourself: what is the maximum value of the independent variable x?
    int right = ... + 1;
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (f(mid) == target) {
            // ask yourself: is the problem asking for the left boundary or the right boundary?
            // ...
        } else if (f(mid) < target) {
            // ask yourself: how to make f(x) larger?
            // ...
        } else if (f(mid) > target) {
            // ask yourself: how to make f(x) smaller?
            // ...
        }
    }
    return left;
}
```

Here's how to break it down step by step:

**1. Figure out what `x, f(x), target` represent, then write the code for function `f`.**

**2. Determine the range of `x` as your search interval, and initialize `left` and `right` accordingly.**

**3. Based on what the problem asks for, decide whether you need left-boundary or right-boundary binary search, then write your solution.**

Let's walk through this process with a few examples.

## Example 1: Koko Eating Bananas

This is LeetCode problem 875, "[Koko Eating Bananas](https://leetcode.cn/problems/koko-eating-bananas/)":

**LeetCode 875. Koko Eating Bananas** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Koko loves to eat bananas. There are `n` piles of bananas, the `i^(th)` pile has `piles[i]` bananas. The guards have gone and will come back in `h` hours.

Koko can decide her bananas-per-hour eating speed of `k`. Each hour, she chooses some pile of bananas and eats `k` bananas from that pile. If the pile has less than `k` bananas, she eats all of them instead and will not eat any more bananas during this hour.

Koko likes to eat slowly but still wants to finish eating all the bananas before the guards return.

Return *the minimum integer* `k` *such that she can eat all the bananas within* `h` *hours*.

Example 1:**

```

**Input:** piles = [3,6,7,11], h = 8
**Output:** 4

```

Example 2:**

```

**Input:** piles = [30,11,23,4,20], h = 5
**Output:** 30

```

Example 3:**

```

**Input:** piles = [30,11,23,4,20], h = 6
**Output:** 23

```

**Constraints:**

	
- `1 <= piles.length <= 10^(4)`
	
- `piles.length <= h <= 10^(9)`
	
- `1 <= piles[i] <= 10^(9)`

Koko can eat at most one pile per hour. If she can't finish a pile, she'll continue eating it in the next hour. If she finishes early, she'll just wait until the next hour to start another pile.

She wants to finish all the bananas before the guard returns. We need to find the **minimum eating speed `K`**. Here's the function signature:

```java
int minEatingSpeed(int[] piles, int H);
```

So how do we apply the framework we just outlined to write a binary search solution?

Just follow the steps:

**1. Figure out what `x, f(x), target` represent, then write the code for function `f`.**

What's the independent variable `x`? Think back to those function graphs—binary search is essentially searching over the independent variable.

Whatever the problem asks you to find, that's your `x`. Here, Koko's eating speed is our `x`.

What's the monotonic relationship `f(x)` with respect to `x`?

The faster Koko eats, the less time she needs to finish all the piles. Speed and time have a monotonic relationship.

So we can define `f(x)` like this:

If Koko eats at a speed of `x` bananas/hour, she needs `f(x)` hours to finish all the bananas.

Here's the implementation:

```java
// Definition: When the speed is x, it takes f(x) hours to eat all the bananas
// f(x) monotonically decreases as x increases
long f(int[] piles, int x) {
    long hours = 0;
    for (int i = 0; i < piles.length; i++) {
        hours += piles[i] / x;
        if (piles[i] % x > 0) {
            hours++;
        }
    }
    return hours;
}
```

::: info Info

Why does `f(x)` return a `long`? Look at the data constraints and the logic of `f`. Elements in `piles` can be as large as 10^9, and there can be up to 10^4 elements. When `x` is 1, the `hours` variable could reach 10^13, which exceeds the maximum value of an `int` (roughly 2×10^9). Using `long` prevents potential integer overflow.

:::

The `target` is straightforward—the time limit `H` is naturally the `target`, serving as the maximum constraint on `f(x)`'s return value.

**2. Determine the range of `x` as your search interval, and initialize `left` and `right` accordingly.**

What's Koko's minimum eating speed? What's the maximum?

The minimum speed is obviously 1. The maximum is the largest element in `piles`, since she can eat at most one pile per hour—having a bigger appetite won't help.

You have two options here: either loop through `piles` to find the maximum value, or check the problem's constraints for the range of elements in `piles` and initialize `right` to a value just outside that range.

I'll go with the second approach. The problem states `1 <= piles[i] <= 10^9`, so we can set our search boundaries like this:

```java
public int minEatingSpeed(int[] piles, int H) {
    int left = 1;
    // Note that I choose to write the binary search with the left closed and right open, right is an open interval, so add one
    int right = 1000000000 + 1;

    // ...
}
```

Since binary search has logarithmic complexity, even if `right` is a huge number, the algorithm remains efficient.

**3. Based on what the problem asks for, decide whether you need left-boundary or right-boundary binary search, then write your solution.**

Now we know: `x` is the eating speed, `f(x)` is monotonically decreasing, and `target` is the time limit `H`. The problem asks for the minimum speed, meaning we want `x` to be as small as possible:

![](../pictures/binary-search-in-action/4.jpeg)

This calls for left-boundary binary search. But watch out—`f(x)` is monotonically decreasing, so don't blindly apply the template. Think it through with the diagram above:

```java
class Solution {
    public int minEatingSpeed(int[] piles, int H) {
        int left = 1;
        int right = 1000000000 + 1;
        
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (f(piles, mid) == H) {
                // searching for the left boundary, thus we need to shrink the right boundary
                right = mid;
            } else if (f(piles, mid) < H) {
                // need to make the return value of f(x) larger
                right = mid;
            } else if (f(piles, mid) > H) {
                // need to make the return value of f(x) smaller
                left = mid + 1;
            }
        }
        return left;
    }
}
```

<visual slug='koko-eating-bananas'/>

::: tip Tip

I'm using the left-closed, right-open binary search style here. If you prefer the both-ends-closed style, just modify how you initialize `right` and update it:

```java
// both-ends-closed binary search style
int minEatingSpeed(int[] piles, int H) {
    int left = 1;
    // right is a closed interval, so use the actual max value
    int right = 1000000000;

    // right is a closed interval, so use <=
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (f(piles, mid) <= H) {
            // right is a closed interval, so use mid - 1
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return left;
}
```

For a deep dive into the details of this algorithm, check out [Binary Search Explained](https://labuladong.online/en/algo/essential-technique/binary-search-framework/). I won't go into it here.

:::

And that solves the problem. The extra if-branches in our framework are mainly there to help you understand what's happening. Once you've got a working solution, I'd recommend merging redundant branches to improve runtime efficiency:

```java
class Solution {
    public int minEatingSpeed(int[] piles, int H) {
        int left = 1;
        int right = 1000000000 + 1;

        while (left < right) {
            int mid = left + (right - left) / 2;
            if (f(piles, mid) <= H) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        return left;
    }

    // definition: when the speed is x, it takes f(x) hours to eat all the bananas
    // f(x) is monotonically decreasing as x increases
    long f(int[] piles, int x) {
        long hours = 0;
        for (int i = 0; i < piles.length; i++) {
            hours += piles[i] / x;
            if (piles[i] % x > 0) {
                hours++;
            }
        }
        return hours;
    }
}
```


## Problem 2: Shipping Packages

Let's look at LeetCode problem 1011, [Capacity To Ship Packages Within D Days](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/):

**LeetCode 1011. Capacity To Ship Packages Within D Days** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

A conveyor belt has packages that must be shipped from one port to another within `days` days.

The `i^(th)` package on the conveyor belt has a weight of `weights[i]`. Each day, we load the ship with packages on the conveyor belt (in the order given by `weights`). We may not load more weight than the maximum weight capacity of the ship.

Return the least weight capacity of the ship that will result in all the packages on the conveyor belt being shipped within `days` days.

Example 1:**

```

**Input:** weights = [1,2,3,4,5,6,7,8,9,10], days = 5
**Output:** 15
**Explanation:** A ship capacity of 15 is the minimum to ship all the packages in 5 days like this:
1st day: 1, 2, 3, 4, 5
2nd day: 6, 7
3rd day: 8
4th day: 9
5th day: 10

Note that the cargo must be shipped in the order given, so using a ship of capacity 14 and splitting the packages into parts like (2, 3, 4, 5), (1, 6, 7), (8), (9), (10) is not allowed.

```

Example 2:**

```

**Input:** weights = [3,2,2,4,1,4], days = 3
**Output:** 6
**Explanation:** A ship capacity of 6 is the minimum to ship all the packages in 3 days like this:
1st day: 3, 2
2nd day: 2, 4
3rd day: 1, 4

```

Example 3:**

```

**Input:** weights = [1,2,3,1,1], days = 4
**Output:** 3
**Explanation:**
1st day: 1
2nd day: 2
3rd day: 3
4th day: 1, 1

```

**Constraints:**

	
- `1 <= days <= weights.length <= 5 * 10^(4)`
	
- `1 <= weights[i] <= 500`

You need to ship all packages in order within `D` days, and packages can't be split. What's the minimum capacity the ship needs?

Here's the function signature:

```java
int shipWithinDays(int[] weights, int days);
```

Same as before—just follow the process:

**1. Figure out what `x, f(x), target` represent, then implement the function `f`**.

The question asks about ship capacity, so that's our variable `x`.

Shipping days and capacity are inversely related—higher capacity means fewer days. So we'll make `f(x)` calculate how many days we need given capacity `x`. This makes `f(x)` monotonically decreasing.

Here's how to implement `f(x)`:

```java
// Definition: when the carrying capacity is x, it takes f(x) days to transport all the goods
// f(x) decreases monotonically with the increase of x
int f(int[] weights, int x) {
    int days = 0;
    for (int i = 0; i < weights.length; ) {
        // load as much cargo as possible
        int cap = x;
        while (i < weights.length) {
            if (cap < weights[i]) break;
            else cap -= weights[i];
            i++;
        }
        days++;
    }
    return days;
}
```

For this problem, `target` is clearly the number of shipping days `D`. We need to find the minimum capacity where `f(x) == D`.

**2. Determine the range of `x` to use as the binary search interval, and initialize `left` and `right`**.

What's the minimum capacity? What's the maximum?

The minimum capacity should be the heaviest single package in `weights`—you've got to be able to carry at least one package per trip.

The maximum capacity would be the sum of all packages—just carry everything in one trip.

That gives us our search interval `[left, right)`:

```java
int shipWithinDays(int[] weights, int days) {
    int left = 0;
    // note that right is an open interval, so add one more
    int right = 1;
    for (int w : weights) {
        left = Math.max(left, w);
        right += w;
    }
    
    // ...
}
```

**3. Based on what the problem asks for, decide whether to use left-boundary or right-boundary binary search, then write the solution**.

Now we know: `x` is the ship's capacity, `f(x)` is a monotonically decreasing function, and `target` is the day limit `D`. The problem wants the minimum capacity, which means we want the smallest possible `x`:

![](../pictures/binary-search-in-action/5.jpeg)

This is a classic left-boundary binary search. Using the diagram above, we can write the code:

```java
public int shipWithinDays(int[] weights, int days) {
    int left = 0;
    // note that right is an open interval, so add one more
    int right = 1;
    for (int w : weights) {
        left = Math.max(left, w);
        right += w;
    }
    
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (f(weights, mid) == days) {
            // searching for the left boundary, we need to shrink the right boundary
            right = mid;
        } else if (f(weights, mid) < days) {
            // need to make the return value of f(x) larger
            right = mid;
        } else if (f(weights, mid) > days) {
            // need to make the return value of f(x) smaller
            left = mid + 1;
        }
    }
    
    return left;
}
```

That's the solution! We can clean up the redundant if branches to make it faster. Here's the final code:

```java
class Solution {
    public int shipWithinDays(int[] weights, int days) {
        int left = 0;
        int right = 1;
        for (int w : weights) {
            left = Math.max(left, w);
            right += w;
        }

        while (left < right) {
            int mid = left + (right - left) / 2;
            if (f(weights, mid) <= days) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }

        return left;
    }

    // definition: when the carrying capacity is x, it takes f(x) days to transport all the goods
    // f(x) decreases monotonically as x increases
    int f(int[] weights, int x) {
        int days = 0;
        for (int i = 0; i < weights.length; ) {
            // load as many goods as possible
            int cap = x;
            while (i < weights.length) {
                if (cap < weights[i]) break;
                else cap -= weights[i];
                i++;
            }
            days++;
        }
        return days;
    }
}
```

<visual slug='capacity-to-ship-packages-within-d-days'/>

## Problem 3: Split Array

Let's try LeetCode problem 410, [Split Array Largest Sum](https://leetcode.cn/problems/split-array-largest-sum/), marked as Hard:

**LeetCode 410. Split Array Largest Sum** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

Given an integer array `nums` and an integer `k`, split `nums` into `k` non-empty subarrays such that the largest sum of any subarray is **minimized**.

Return *the minimized largest sum of the split*.

A **subarray** is a contiguous part of the array.

Example 1:**

```

**Input:** nums = [7,2,5,10,8], k = 2
**Output:** 18
**Explanation:** There are four ways to split nums into two subarrays.
The best way is to split it into [7,2,5] and [10,8], where the largest sum among the two subarrays is only 18.

```

Example 2:**

```

**Input:** nums = [1,2,3,4,5], k = 2
**Output:** 9
**Explanation:** There are four ways to split nums into two subarrays.
The best way is to split it into [1,2,3] and [4,5], where the largest sum among the two subarrays is only 9.

```

**Constraints:**

	
- `1 <= nums.length <= 1000`
	
- `0 <= nums[i] <= 10^(6)`
	
- `1 <= k <= min(50, nums.length)`

Here's the function signature:

```java
int splitArray(int[] nums, int m);
```

This problem is pretty confusing with all the max and min talk. Take a close look at the examples to understand what it's really asking.

Here's the gist: you're given an array `nums` and a number `m`, and you need to split `nums` into `m` subarrays.

There are multiple ways to split it. Each way creates `m` subarrays, and in each split, one of those subarrays will have the largest sum, right?

You want to find the split where that largest subarray sum is as small as possible compared to all other splits.

Your algorithm should return that minimized largest subarray sum.

Yikes, this sounds super hard and confusing. How do we even apply our binary search pattern here?

**Here's the thing: this problem is actually identical to the shipping problem we just solved. Don't believe me? Let me rephrase it**:

You have one cargo ship. You have several packages, each weighing `nums[i]`. You need to ship all of them within `m` days. What's the minimum capacity your ship needs?

That's exactly LeetCode 1011, [Capacity To Ship Packages Within D Days](https://leetcode.cn/problems/capacity-to-ship-packages-within-d-days/), which we just solved!

Each day's cargo is one subarray of `nums`. Shipping everything in `m` days means splitting `nums` into `m` subarrays. Minimizing the ship's capacity means minimizing the largest subarray sum.

So you can literally copy-paste the shipping problem's solution:

```java
class Solution {
     public int splitArray(int[] nums, int m) {
        return shipWithinDays(nums, m);
    }

    int shipWithinDays(int[] weights, int days) {
        // see above
    }

    int f(int[] weights, int x) {
        // see above
    }
}
```

That wraps it up! To summarize: if you spot a monotonic relationship in a problem, try using binary search. Once you understand the monotonicity and figure out which type of binary search to use, you can sketch it out and write the final code.
