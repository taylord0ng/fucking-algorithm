::: info Prerequisite

Before reading this article, you need to learn:

- [Two Pointers Techniques for Arrays](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/)

:::

LeetCode Problem 42 "[Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)" is interesting and often appears in interviews. This article will explain how to optimize the solution step by step.

Here is the problem:

**LeetCode 42. Trapping Rain Water** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

```

**Input:** height = [0,1,0,2,1,0,1,3,2,1,2,1]
**Output:** 6
**Explanation:** The above elevation map (black section) is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped.

```

Example 2:**

```

**Input:** height = [4,2,0,3,2,5]
**Output:** 9

```

**Constraints:**

	
- `n == height.length`
	
- `1 <= n <= 2 * 10^(4)`
	
- `0 <= height[i] <= 10^(5)`

You are given an array representing a bar chart. The question asks how much water can be trapped by the bars after raining.

```java
int trap(int[] height);
```

Next, I will introduce three methods from simple to advanced: brute-force solution -> memoization solution -> two pointers solution. We will solve this problem in O(N) time and O(1) space.

## 1. Core Idea

::: tip Quick Tip

When solving algorithm problems, if you have no idea how to start, try to simplify the problem. Think about a small part first and write the simplest brute-force solution. You might find a key point and then optimize step by step to reach the best solution.

:::

For this problem, let's not think about the whole bar chart first. Just think about how much water can be trapped at position `i`?

![](../pictures/rain-water/0.jpg)

At position `i`, we can trap 2 units of water. This is because `height[i]` is 0, and the highest possible water level here is 2. So, 2 - 0 = 2.

Why can position `i` hold up to 2 units of water? It depends on the tallest bar to the left and the tallest bar to the right of position `i`. Let's call these heights `l_max` and `r_max`. **The highest water level at position `i` is `min(l_max, r_max)`.**

So, for position `i`, the amount of water trapped is:


<!-- hide -->

```python
water[i] = min(
    # Highest bar on the left
    max(height[0..i]),  
    # Highest bar on the right
    max(height[i..end]) 
) - height[i]
```

![](../pictures/rain-water/1.jpg)

![](../pictures/rain-water/2.jpg)

This is the main idea of this problem. We can write a simple brute-force algorithm:

```java
class Solution {
    public int trap(int[] height) {
        int n = height.length;
        int res = 0;
        for (int i = 1; i < n - 1; i++) {
            int l_max = 0, r_max = 0;
            // find the tallest pillar on the right
            for (int j = i; j < n; j++)
                r_max = Math.max(r_max, height[j]);
            // find the tallest pillar on the left
            for (int j = i; j >= 0; j--)
                l_max = Math.max(l_max, height[j]);
            // if itself is the tallest,
            // l_max == r_max == height[i]
            res += Math.min(l_max, r_max) - height[i];
        }
        return res;
    }
}
```

This solution is very straightforward. The time complexity is O(N^2), and space complexity is O(1). But calculating `r_max` and `l_max` this way is not smart, because we have to loop every time. Can we make this better?

## 2. Memoization Optimization

In the brute-force solution, at each position `i`, we need to calculate `r_max` and `l_max`. Instead of calculating them every time, we can calculate the results in advance and store them. This will reduce the time complexity.

**We can use two arrays, `r_max` and `l_max`, as memoization. `l_max[i]` means the highest bar on the left of position `i`, and `r_max[i]` means the highest bar on the right of position `i`.** We calculate these two arrays first, so we don't repeat work:

```java
class Solution {
    public int trap(int[] height) {
        if (height.length == 0) {
            return 0;
        }
        int n = height.length;
        int res = 0;
        // array serves as a memo
        int[] l_max = new int[n];
        int[] r_max = new int[n];
        // initialize base case
        l_max[0] = height[0];
        r_max[n - 1] = height[n - 1];
        // calculate l_max from left to right
        for (int i = 1; i < n; i++)
            l_max[i] = Math.max(height[i], l_max[i - 1]);
        // calculate r_max from right to left
        for (int i = n - 2; i >= 0; i--)
            r_max[i] = Math.max(height[i], r_max[i + 1]);
        // calculate the answer
        for (int i = 1; i < n - 1; i++)
            res += Math.min(l_max[i], r_max[i]) - height[i];
            /**<extend up -300>
            ![](../pictures/rain-water/1.jpg)
            */
        return res;
    }
}
```

<visual slug='trapping-rain-water'/>

This optimization is similar to the brute-force method, but it avoids repeated calculation. The time complexity becomes O(N), which is the best possible. But the space complexity is O(N). Next, let's look at a clever solution that reduces space complexity to O(1).


## 3. Two Pointer Solution

::: note My Advice

This solution is for expanding your thinking. You don't have to focus too much on finding the best solution. For most people, in real interviews or tests, it is enough to use simple and clear methods like the one above. Although it uses a bit more space, most online judges will still accept it.

Unless you cannot pass all test cases, and you have finished other questions with time left, you can then spend time improving the above solution.

:::

The idea of this solution is the same, but the implementation is clever. This time, we do not use extra arrays to pre-calculate the values. Instead, we use two pointers to calculate while moving, which saves space.

First, take a look at this piece of code:

```java
int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int l_max = 0, r_max = 0;
    
    while (left < right) {
        l_max = Math.max(l_max, height[left]);
        r_max = Math.max(r_max, height[right]);
        // At this point, what do l_max and r_max represent respectively?
        left++; right--;
    }
}
```

For this code, what do `l_max` and `r_max` mean?

It is easy to understand. **`l_max` means the highest bar from `height[0..left]`, and `r_max` means the highest bar from `height[right..end]`.**

With this in mind, let's look at the solution:

```java
class Solution {
    public int trap(int[] height) {
        int left = 0, right = height.length - 1;
        int l_max = 0, r_max = 0;

        int res = 0;
        while (left < right) {
            l_max = Math.max(l_max, height[left]);
            r_max = Math.max(r_max, height[right]);

            // res += min(l_max, r_max) - height[i]
            if (l_max < r_max) {
                res += l_max - height[left];
                /**<extend up -250>
                ![](../pictures/rain-water/5.jpg)
                */
                left++;
            } else {
                res += r_max - height[right];
                right--;
            }
        }
        return res;
    }
}
```

You see, the core idea is exactly the same as before. But careful readers might notice a small difference:

In the previous memoization solution, `l_max[i]` and `r_max[i]` mean the highest bar from `height[0..i]` and `height[i..end]`.

```java
res += Math.min(l_max[i], r_max[i]) - height[i];
```

![](../pictures/rain-water/3.jpg)

But in the two pointer solution, `l_max` and `r_max` mean the highest bars from `height[0..left]` and `height[right..end]`. For example, look at this code:

```java
if (l_max < r_max) {
    res += l_max - height[left];
    left++; 
} 
```

![](../pictures/rain-water/4.jpg)

Here, `l_max` is the highest bar to the left of the `left` pointer. But `r_max` is not always the highest bar to the right of `left`. Can this still give the right answer?

The key is, we only care about `min(l_max, r_max)`. **In the picture above, we already know `l_max < r_max`. It does not matter if `r_max` is the largest on the right. What matters is the water at `height[i]` only depends on the lower value, which is `l_max`.**

![](../pictures/rain-water/5.jpg)

In this way, the trapping rain water problem is solved.


## Extension: Container With Most Water

Now let's look at a problem very similar to the "Trapping Rain Water" problem. It's LeetCode problem 11: ["Container With Most Water"](https://leetcode.com/problems/container-with-most-water/):

**LeetCode 11. Container With Most Water** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `i^(th)` line are `(i, 0)` and `(i, height[i])`.

Find two lines that together with the x-axis form a container, such that the container contains the most water.

Return *the maximum amount of water a container can store*.

**Notice** that you may not slant the container.

Example 1:**

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

```

**Input:** height = [1,8,6,2,5,4,8,3,7]
**Output:** 49
**Explanation:** The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.

```

Example 2:**

```

**Input:** height = [1,1]
**Output:** 1

```

**Constraints:**

	
- `n == height.length`
	
- `2 <= n <= 10^(5)`
	
- `0 <= height[i] <= 10^(4)`

```java
// The function signature is as follows
int maxArea(int[] height);
```

This problem is similar to the rain water problem, and you can use the same method. In fact, it is even more simple. The difference is:

**In the rain water problem, each x-coordinate has a bar with width. In this problem, each x-coordinate has only a vertical line, no width.**

Earlier, we talked a lot about `l_max` and `r_max`. They are used to figure out how much water `height[i]` can hold. But in this problem, since `height[i]` has no width, it becomes much easier.

For example, in the rain water problem, if you know the heights of `height[left]` and `height[right]`, can you calculate how much water is between `left` and `right`?

You can't, because you don't know how much water each bar in between can hold. You need the `l_max` and `r_max` for each bar to get the answer.

But in this problem, if you know the heights of `height[left]` and `height[right]`, can you work out the water held between `left` and `right`?

Yes, because in this problem the lines have no width. The water between `left` and `right` is:

```python
min(height[left], height[right]) * (right - left)
```

Just like in the rain water problem, the height is decided by the smaller of `height[left]` and `height[right]`.

The way to solve this problem is still the two-pointer technique:

**Use two pointers, `left` and `right`, starting from both ends and moving towards the center. For each pair `[left, right]`, calculate the area, and keep the biggest area as the answer.**

Let's look at the code:

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0, right = height.length - 1;
        int res = 0;
        while (left < right) {
            // the area of the rectangle between [left, right]
            int cur_area = Math.min(height[left], height[right]) * (right - left);
            res = Math.max(res, cur_area);
            // two-pointer technique, move the shorter side
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }
        return res;
    }
}
```

<visual slug='container-with-most-water'/>

The code is almost the same as the rain water problem. But you may wonder, why do we move the lower side in this if statement:

```java
// Two-pointer technique, move the lower side
if (height[left] < height[right]) {
    left++;
} else {
    right--;
}
```

**It is easy to understand, because the height of the rectangle is decided by the smaller of `height[left]` and `height[right]`.**

If you move the lower side, that side may get higher, and the rectangle height may get bigger, so the area might also get bigger. If you move the higher side, the height will never get bigger, so the area cannot get bigger.

That's it. This problem is solved.
