::: info Prerequisites

Before reading this article, you should be familiar with:

- [Prefix Sum Techniques](https://labuladong.online/en/algo/data-structure/prefix-sum/)
- [Binary Search Explained](https://labuladong.online/en/algo/essential-technique/binary-search-framework/)

:::

I got inspired to write this article while playing League of Legends mobile. A friend of mine was complaining about getting terrible teammates in ranked matches. I told him I thought my ranked teammates were pretty solid—they didn't seem that bad to me.

He gave me this knowing look and said: "Usually, when players with high MMR can't find teammates at their skill level, they get matched with... weaker players."

Wait, what? I thought about it for a second. Something didn't add up. Was he saying my MMR was low, or was he calling me the weak player?

I immediately challenged him to duo queue with me to prove I wasn't the weak link—he was. I'll let you guess how that went.

After that game, I came here to write this article because it got me thinking about how matchmaking systems work.

![](../pictures/random-weight/images.png)

**I don't know if "hidden MMR" is actually a thing. Matchmaking is the backbone of any competitive game, so it's probably way more complex than a few simple metrics.**

But if we simplify this whole hidden MMR concept, it becomes an interesting algorithmic problem: how does the system randomly match players with different probabilities?

Or to put it more simply: how do you make weighted random selections?

Don't think it's trivial. Sure, if you have an array of length `n` and need to randomly pick an element with equal probability, that's easy—just generate a random index in `[0, n-1]` and each element has a `1/n` chance of being selected.

But what if each element has a different weight, where the weight determines the probability of selecting that element? How would you write an algorithm to randomly pick elements based on those weights?

LeetCode problem 528, "[Random Pick with Weight](https://leetcode.cn/problems/random-pick-with-weight/)," is exactly this problem:

**LeetCode 528. Random Pick with Weight** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given a **0-indexed** array of positive integers `w` where `w[i]` describes the **weight** of the `i^(th)` index.

You need to implement the function `pickIndex()`, which **randomly** picks an index in the range `[0, w.length - 1]` (**inclusive**) and returns it. The **probability** of picking an index `i` is `w[i] / sum(w)`.

	
- For example, if `w = [1, 3]`, the probability of picking index `0` is `1 / (1 + 3) = 0.25` (i.e., `25%`), and the probability of picking index `1` is `3 / (1 + 3) = 0.75` (i.e., `75%`).

Example 1:**

```

**Input**
["Solution","pickIndex"]
[[[1]],[]]
**Output**
[null,0]

**Explanation**
Solution solution = new Solution([1]);
solution.pickIndex(); // return 0. The only option is to return 0 since there is only one element in w.

```

Example 2:**

```

**Input**
["Solution","pickIndex","pickIndex","pickIndex","pickIndex","pickIndex"]
[[[1,3]],[],[],[],[],[]]
**Output**
[null,1,1,1,1,0]

**Explanation**
Solution solution = new Solution([1, 3]);
solution.pickIndex(); // return 1. It is returning the second element (index = 1) that has a probability of 3/4.
solution.pickIndex(); // return 1
solution.pickIndex(); // return 1
solution.pickIndex(); // return 1
solution.pickIndex(); // return 0. It is returning the first element (index = 0) that has a probability of 1/4.

Since this is a randomization problem, multiple answers are allowed.
All of the following outputs can be considered correct:
[null,1,1,1,1,0]
[null,1,1,1,1,1]
[null,1,1,1,0,0]
[null,1,1,1,0,1]
[null,1,0,1,0,0]
......
and so on.

```

**Constraints:**

	
- `1 <= w.length <= 10^(4)`
	
- `1 <= w[i] <= 10^(5)`
	
- `pickIndex` will be called at most `10^(4)` times.

Let's think through this problem and solve weighted random selection.

<!-- hide -->

## Approach

First, let's recap some previous articles on randomized algorithms:

In [Designing a Data Structure for Random Deletion](https://labuladong.online/en/algo/data-structure/random-set/), we focused on data structure design—moving elements to the end of the array before deleting them to avoid shifting data.

In [Randomized Algorithms in Games](https://labuladong.online/en/algo/frequency-interview/random-algorithm/), we covered the classic reservoir sampling algorithm, which uses simple math to select elements with equal probability from an infinite sequence.

**But neither of those articles solve the problem we're facing here. Instead, it's the combination of [Prefix Sum Techniques](https://labuladong.online/en/algo/data-structure/prefix-sum/) and [Binary Search Explained](https://labuladong.online/en/algo/essential-technique/binary-search-framework/) that unlocks weighted random selection.**

How do random algorithms relate to prefix sums and binary search? Let me explain.

Say the input weight array is `w = [1,3,2,1]`. We want the selection probability to match the weights. Here's a way to visualize it—imagine drawing a colored line segment based on these weights:

![](../pictures/random-weight/1.jpeg)

If we randomly drop a stone onto this line segment, and we pick the index corresponding to whichever color the stone lands on, doesn't that mean each index gets selected with probability proportional to its weight?

**Now take a closer look at this colored line segment. What does it look like? It's just a [prefix sum array](https://labuladong.online/en/algo/data-structure/prefix-sum/)!**

![](../pictures/random-weight/2.jpeg)

Next question: how do we simulate dropping a stone onto the line segment?

With a random number, of course. For the prefix sum array `preSum` above, the range is `[1, 7]`. If we generate a random number `target = 5` in that range, it's like randomly dropping a stone on the line segment:

![](../pictures/random-weight/3.jpeg)

But there's another issue: `preSum` doesn't actually contain the element 5. We need to find the smallest element greater than or equal to 5, which is 6, at index 3 of the `preSum` array:

![](../pictures/random-weight/4.jpeg)

**How do we quickly find the smallest element in an array that's greater than or equal to our target? That's exactly what [binary search](https://labuladong.online/en/algo/essential-technique/binary-search-framework/) is for.**

That's the core idea. Here's the breakdown:

1. Build a prefix sum array `preSum` from the weight array `w`.

2. Generate a random number within the range of `preSum`, then use binary search to find the index of the smallest element greater than or equal to this random number.

3. Subtract one from this index (because the prefix sum array has an index offset), and you get the index in the weight array—your final answer:

![](../pictures/random-weight/5.jpeg)

## Solution Code

The approach itself is pretty straightforward, but the implementation is where things get tricky.

Problems involving open/closed intervals, index offsets, and binary search require extremely precise attention to detail. Otherwise, you'll run into all sorts of hard-to-debug issues.

Let's dig into the details, continuing with our previous example:

![](../pictures/random-weight/3.jpeg)

Take the `preSum` array. What range should the random number `target` be in? The closed interval `[0, 7]` or the half-open interval `[0, 7)`?

Neither. It should be in the closed interval `[1, 7]`, **because the 0 in the prefix sum array is essentially just a placeholder**. Think about it:

![](../pictures/random-weight/6.jpeg)

So you should write the code like this:

```java
int n = preSum.length;
// the range of target is closed interval [1, preSum[n - 1]]
int target = rand.nextInt(preSum[n - 1]) + 1;
```

Next, when searching for the index of the smallest element in `preSum` that's greater than or equal to `target`, which variant of binary search should you use? Left boundary search or right boundary search?

You should use left boundary binary search:

```java
// Binary search for the left boundary
int left_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
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

In [Binary Search Explained](https://labuladong.online/en/algo/essential-technique/binary-search-framework/), I focused on cases where the target element appears multiple times in the array, but didn't go into detail about when the target doesn't exist. Let me clarify that here.

**When the target element `target` doesn't exist in array `nums`, the return value from left boundary binary search can be interpreted in these ways:**

1. It's the index of the smallest element in `nums` that's greater than or equal to `target`.

2. It's the index position where `target` should be inserted into `nums`.

3. It's the count of elements in `nums` that are less than `target`.

For example, if you search for `target = 4` in the sorted array `nums = [2,3,5,7]`, left boundary binary search returns 2. Try plugging that into each of the interpretations above—they all work.

All three interpretations are equivalent, so you can use whichever fits your problem best. Obviously, we need the first interpretation here.

Putting it all together, here's the final solution:

```java
class Solution {
    // prefix sum array
    private int[] preSum;
    private Random rand = new Random();
    
    public Solution(int[] w) {
        int n = w.length;
        // construct the prefix sum array, shifting one position for preSum[0]
        preSum = new int[n + 1];
        preSum[0] = 0;
        // preSum[i] = sum(w[0..i-1])
        for (int i = 1; i <= n; i++) {
            preSum[i] = preSum[i - 1] + w[i - 1];
        }
    }
    
    public int pickIndex() {
        int n = preSum.length;
        // Java's nextInt(n) method generates a random integer in [0, n)
        // adding one makes it randomly select a number in the closed interval [1, preSum[n - 1]]
        int target = rand.nextInt(preSum[n - 1]) + 1;
        // get the index of target in the prefix sum array preSum
        // don't forget the prefix sum array preSum and the original array w have an index offset by one position
        return left_bound(preSum, target) - 1;
    }

    // binary search for the left boundary
    private int left_bound(int[] nums, int target) {
        int left = 0, right = nums.length;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                right = mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else if (nums[mid] > target) {
                right = mid;
            }
        }
        return left;
    }
}
```

With all the groundwork we've laid, you should fully understand this code. That solves the weighted random selection problem.

I often get comments from readers joking that they just "cloud grind" problems by reading my articles—they understand everything without actually coding it themselves.

But here's the thing: many problems are easy to understand conceptually, but once you dig deeper, there are tons of subtle pitfalls. This problem is a perfect example. So I still recommend practicing hands-on and reflecting on what you learn.
