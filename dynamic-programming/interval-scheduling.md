What is a greedy algorithm? You can think of a greedy algorithm as a special case of dynamic programming. Compared to dynamic programming, a greedy algorithm needs more conditions to work (the greedy choice property), but if it works, it is faster.

For example, suppose a problem needs exponential time with a brute-force solution. If you can use dynamic programming to remove overlapping subproblems, the time can drop to polynomial. If the problem also satisfies the greedy choice property, you can further reduce the time complexity to linear.

What is the greedy choice property? Simply put: at each step, if you make the best local choice, the final result is also the global best. Note that this is a special property, and only some problems have it.

For example, you have 100 RMB bills in front of you, and you can only pick 10. How do you get the largest total amount? Clearly, each time you should pick the bill with the largest value among the remaining ones. In the end, your choice will be optimal.

However, most problems do not have the greedy choice property. For example, in the card game Dou Dizhu, if your opponent plays a pair of 3s, a greedy strategy would be to play the smallest pair that can beat it. But in real situations, it depends on the opponent’s cards and strategy; you might even play the strongest combination “rocket” (the two jokers). In such cases, you cannot use a greedy algorithm and instead need a more complex brute-force algorithm to find the optimal solution.

## 1. Problem Overview

Now back to the main topic. This article solves a classic greedy algorithm problem, Interval Scheduling, which is LeetCode Problem 435 “[Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)”:

You are given many closed intervals of the form `[start, end]`. Please design an algorithm to **find the maximum number of intervals that are mutually non-overlapping**.

```java
int intervalSchedule(int[][] intvs);
```

For example, `intvs = [[1,3], [2,4], [3,6]]`. Among these intervals, the maximum number of non-overlapping intervals is 2, that is `[[1,3], [3,6]]`. Your algorithm should return 2. Note that intervals that only touch at the boundary are not considered overlapping.

This problem has many real-world uses. For example, you have several events today. Each event can be represented as an interval `[start, end]` for its start time and end time. **What is the maximum number of events you can attend today?** Clearly, you cannot attend two events at the same time. So this problem is to find the largest subset of these time intervals that do not overlap.


<!-- hide -->

## 2. Greedy Solution

There are many greedy ideas for this problem that look good but do not give the correct answer. For example:

Maybe we can always choose the interval with the earliest start time? But some intervals may start early but are very long, causing us to miss some short intervals. Or maybe choose the shortest interval each time? Or the interval with the fewest conflicts? All of these ideas have counterexamples and are not correct.

The correct solution is actually simple and has three steps:

1. From the set of intervals `intvs`, choose one interval `x` which ends the earliest (the smallest `end` value).

2. Remove all intervals that overlap with `x` from `intvs`.

3. Repeat step 1 and 2 until `intvs` is empty. All the selected `x` intervals form the largest set of non-overlapping intervals.

To implement this idea, we can sort all intervals by their `end` value in ascending order. After sorting, both step 1 and step 2 are easy to do, as shown in the GIF below:

![](../pictures/interval/1.gif)

Now let's implement the algorithm. For step 1, since we already sorted by `end`, it is easy to choose `x`. The key point is how to remove all intervals that overlap with `x` and select the next `x` for the next round.

**Because we sorted the intervals**, all intervals that overlap with `x` must have a `start` value less than `x`'s `end`. If an interval does not overlap with `x`, its `start` must be greater than or equal to `x`'s `end`:

![](../pictures/interval/2.jpg)

Here is the code:

```java
class Solution {
    public int intervalSchedule(int[][] intvs) {
        if (intvs.length == 0) return 0;
        // sort by end in ascending order
        Arrays.sort(intvs, (a, b) -> Integer.compare(a[1], b[1]));
        // at least one interval does not intersect
        int count = 1;
        // after sorting, the first interval is x
        int x_end = intvs[0][1];
        for (int[] interval : intvs) {
            int start = interval[0];
            if (start >= x_end) {
                // found the next selected interval
                count++;
                x_end = interval[1];
            }
        }
        return count;
    }
}
```


## 3. Application Examples

Let's look at some examples of how to use the interval scheduling algorithm.

First is LeetCode 435 "[Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)":

You are given a set of intervals. Find out the minimum number of intervals you need to remove so that the rest do not overlap. The function signature is:

```java
int eraseOverlapIntervals(int[][] intvs);
```

You can assume that the end point of each interval is always greater than the start point. If two intervals only touch at the boundary, they are not considered overlapping.

For example, if the input is `intvs = [[1,2],[2,3],[3,4],[1,3]]`, the algorithm should return 1. This is because if you remove `[1,3]`, the remaining intervals do not overlap.

We already know how to find the maximum number of non-overlapping intervals. The rest are the intervals we need to remove.

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intvs) {
        int n = intvs.length;
        return n - intervalSchedule(intvs);
    }

    private int intervalSchedule(int[][] intvs) {
        // see above
    }
}
```

<visual slug='non-overlapping-intervals'/>

Next, let's talk about LeetCode 452 "[Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)". Here is the problem:

Imagine there are many round balloons on a 2D plane. If you look at their shadow on the x-axis, you get some intervals. You are given these intervals. As you move along the x-axis, you can shoot an arrow straight up. What is the minimum number of arrows needed to burst all the balloons?

The function signature is:

```java
int findMinArrowShots(int[][] intvs);
```

For example, if the input is `[[10,16],[2,8],[1,6],[7,12]]`, the algorithm should return 2. You can shoot one arrow at x = 6, which bursts `[2,8]` and `[1,6]`. Then shoot another arrow at x = 10, 11, or 12 to burst `[10,16]` and `[7,12]`.

If you think about it, this problem is exactly the same as the interval scheduling problem! If there are at most `n` non-overlapping intervals, you need at least `n` arrows to burst all balloons:

![](../pictures/interval/3.jpg)

There is one small difference: In the `intervalSchedule` algorithm, intervals that only touch at the boundary do not overlap. But in this problem, if an arrow touches the edge of a balloon, the balloon will burst. So, intervals that touch at the boundary are considered overlapping:

![](../pictures/interval/4.jpg)

So, by making a small change to our previous algorithm, we can solve this problem:

```java
class Solution {
    public int findMinArrowShots(int[][] intvs) {
        if (intvs.length == 0) return 0;
        Arrays.sort(intvs, (a, b) -> Integer.compare(a[1], b[1]));
        int count = 1;
        int x_end = intvs[0][1];
        
        for (int[] interval : intvs) {
            int start = interval[0];
            // just change >= to >
            if (start > x_end) {
                count++;
                x_end = interval[1];
            }
        }
        return count;
    }
}
```

<visual slug='minimum-number-of-arrows-to-burst-balloons'/>
