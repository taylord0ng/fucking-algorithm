::: info Prerequisites

Before reading this article, you should first learn:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

This article discusses a classic algorithm problem: given several floors and several eggs, you need to determine the minimum number of attempts required to find the highest floor from which an egg can be dropped without breaking. This problem is frequently asked in interviews at major Chinese companies as well as Google and Facebook. However, they often change the context to throwing cups or bowls instead of eggs to avoid wastage.

We'll get to the specific problem shortly. This problem has numerous solution techniques, including several different dynamic programming approaches with varying efficiencies, and finally, an extremely efficient mathematical solution. Consistent with the style of this book, we avoid overly complex techniques that are not broadly applicable, as they are not worth the effort to learn.

Let's now use the general dynamic programming approach we've emphasized to analyze this problem.


## 1. Understanding the Problem

This is LeetCode problem 887: [Super Egg Drop](https://leetcode.com/problems/super-egg-drop/). I will explain the problem:

You have a building with floors numbered from 1 to `N`. You are given `K` eggs (`K` is at least 1). There is a floor `F` (where `0 <= F <= N`). If you drop an egg from floor `F`, it will **not break**. If you drop an egg from any floor higher than `F`, it will break. If you drop an egg from any floor lower than `F`, it will not break. If the egg does not break, you can pick it up and use it again.

Now, the question is: In the **worst case**, what is the **minimum** number of tries needed to find floor `F`?

In other words, you need to find the highest floor `F` where the egg does not break. But what does "minimum number of tries in the worst case" mean? Let’s look at some examples.

First, let’s ignore the limit on the number of eggs. If there are 7 floors, how do you find the floor where the egg just breaks?

The most basic way is linear search: drop the egg from the 1st floor. If it does not break, try the 2nd floor. If it does not break, try the 3rd floor, and so on.

With this method, the **worst case** is when you reach the 7th floor and the egg still does not break (`F = 7`). So you need to drop the egg 7 times.

Now you should understand what "worst case" means. **The egg only breaks when you have tried all possible floors**. If the egg breaks on the 1st floor, you are just lucky; that is not the worst case.

Now, what does "minimum number of tries" mean? Still ignoring the egg limit, if there are 7 floors, we can use a better strategy.

The best idea is to use binary search. First, drop the egg from the middle floor: `(1 + 7) / 2 = 4`.

If the egg breaks, then `F` is less than 4. Try the middle of 1 to 3.

If the egg does not break, then `F` is at least 4. Try the middle of 5 to 7.

With this strategy, the **worst case** is either when `F = 7` and the egg never breaks, or when `F = 0` and the egg always breaks. But in both worst cases, you only need to try `log7` rounded up, which is 3. This is fewer than 7, so this is the **minimum** number of tries.

Actually, if there is no limit on the number of eggs, binary search is clearly the best strategy. But in this problem, you **have a limit on the number of eggs `K`, so you cannot always use binary search**.

For example, if you have only 1 egg and 7 floors, can you use binary search? If you drop the egg from the 4th floor and it does not break, you can try higher floors. But if it breaks, you have no eggs left and cannot test any more floors, so you cannot find the exact floor `F`.

In this case, you can only use linear search, try floors one by one from the bottom. In the worst case, you need 7 tries, so the answer is 7.

Some readers may think: Binary search eliminates floors the fastest. What if you use binary search first, and only switch to linear search when there is 1 egg left? Is this the best way?

Unfortunately, no. For example, if there are 100 floors and 2 eggs, you drop the first egg at floor 50. If it breaks, you must do linear search from floor 1 to 49. In the worst case, you need 50 tries.

If you don’t do "binary search", but do "divide by 5" or "divide by 10", you can reduce the number of tries. For example, drop the first egg every 10 floors. When it breaks, use the second egg to check the floors one by one. In this way, you need at most 20 tries. The best answer is actually 14. There are many possible good strategies, but there is no simple rule.

After all this explanation, the goal is to make sure you understand the problem. This problem is really complex, even for humans. So how can we solve it with an algorithm?


<!-- hide -->

## 2. Idea Analysis

For dynamic programming problems, we can use the framework we mentioned before: figure out the "state" and "choice", then try all possibilities.

The "state" here is simple. It is the number of eggs `K` you have now and the number of floors `N` you need to test. As you throw eggs, the number of eggs may go down, and the range of floors will get smaller. That is how the state changes.

The "choice" is which floor you choose to throw the egg from. In the linear scan method, you try each floor one by one; in the binary search idea, you pick the middle floor each time. Different choices will change the state.

Now that we know the "state" and the "choice", the basic idea of dynamic programming is clear: use a 2D `dp` array or a `dp` function with two parameters to represent the state. Add a for loop to try all choices and update the answer with the best choice:

```java
// Definition: With K eggs and N floors,
// return the minimum number of throws needed in this state
int dp(int K, int N) {
    int res;
    for (int i = 1; i <= N; i++) {
        res = Math.min(res, throw egg from the i-th floor);
    }
    return res;
}
```

This pseudocode does not show the recursion and state transition yet, but the main structure is done.

When you throw the egg from the i-th floor, there are two possible results: the egg breaks, or it doesn't break. Now the state transitions:

If the egg breaks, you have one less egg (`K-1`), and you only need to check the floors below (from `1` to `i-1`), which is `i-1` floors.

If the egg does not break, the number of eggs stays the same, but you only need to check the floors above (from `i+1` to `N`), which is `N-i` floors.

![](../pictures/drop-egg/1.jpg)

::: note Note

Some readers may ask: If the egg does not break on the i-th floor, should we include the i-th floor in the next search? The answer is no, because we already checked it. At the start, we said `F` can be 0, and after the recursion, the i-th floor is actually like the 0-th floor, so it is already included.

:::

Because we want the minimum number of throws in the worst case, we need to consider the bigger result of the two situations (egg breaks or not):

```java
int dp(int K, int N):
    for 1 <= i <= N:
        // Minimum throws in the worst case
        res = min(res, max(
                // Egg breaks
                dp(K - 1, i - 1),
                // Egg does not break
                dp(K, N - i),
            ) + 1
            // Add one because we threw an egg from the i-th floor
        )
    return res
```

The recursive base cases are easy. If there are 0 floors (`N == 0`), you need 0 throws. If you have only 1 egg (`K == 1`), you must check every floor one by one:

```java
int dp(int K, int N) {
    // base case
    if (K == 1) return N;
    if (N == 0) return 0;
    // ...
}
```

That solves the problem! Just add a memo table to remove repeated subproblems:

```java
class Solution {
    // memoization
    int[][] memo;

    public int superEggDrop(int K, int N) {
        // m will not exceed N times (linear scan)
        memo = new int[K + 1][N + 1];
        for (int[] row : memo) {
            Arrays.fill(row, -666);
        }
        return dp(K, N);
    }

    // definition: holding K eggs, facing N floors, the minimum number of egg drops is dp(K, N)
    int dp(int K, int N) {
        // base case
        if (K == 1) return N;
        if (N == 0) return 0;

        // check memo to avoid redundant calculations
        if (memo[K][N] != -666) {
            return memo[K][N];
        }
        // state transition equation
        int res = Integer.MAX_VALUE;
        for (int i = 1; i <= N; i++) {
            // try on all floors, take the minimum number of egg drops
            res = Math.min(
                res,
                // take the worst case whether it breaks or not
                Math.max(dp(K, N - i), dp(K - 1, i - 1)) + 1
            );
        }
        // store the result in memo
        memo[K][N] = res;
        return res;
    }
}
```

What is the time complexity of this algorithm? For dynamic programming, the time complexity is the number of subproblems times the cost of the function.

The cost of the function itself is O(N), since there is a for loop in the `dp` function.

The number of subproblems is the number of different states, which is O(KN).

So the total time complexity is O(K*N^2), and the space complexity is O(KN).

This problem is complicated, but the code is simple. That is the power of dynamic programming: try all possibilities, use a memo table to speed up, and get the answer.

Some readers may not understand why we use a for loop to go through all floors `[1..N]`. You may think this is like linear scan, but it is not. We are just making a "choice" at each step.

For example, if you have 2 eggs and 10 floors, which floor will you throw the egg from this time? We do not know, so we try all 10 floors. The recursion will handle the next steps and pick the best choice for us.

There are better solutions to this problem. For example, you can use binary search in the for loop to make it O(K*N*logN). With more advanced dynamic programming, you can bring it down to O(KN). With math, you can reach O(K*logN) time and O(1) space.

The binary search solution might confuse you. It has nothing to do with the binary search idea we discussed before. Here, binary search works because the function is monotonic, so we can quickly find the best value.

Next, let's see how to optimize this.


## 3. Binary Search Optimization

The key to optimizing binary search is the monotonicity of the state transition equation. Let's first briefly review the original dynamic programming idea:

1. Use brute-force to try dropping eggs on every floor `1 <= i <= N`. Each time, choose the floor that gives the **fewest** attempts.

2. Each time you drop an egg, there are two possibilities: the egg breaks or it does not break.

3. If the egg breaks, then `F` is below floor `i`. If it does not break, then `F` is above floor `i`.

4. Whether the egg breaks or not, we care about the **worst case**. So, we take the higher number of attempts between the two situations.

The key state transition code is this:

```java
// Current state: K eggs, N floors
// Returns the best result for this state
int dp(int K, int N):
    for 1 <= i <= N:
        // Fewest number of attempts in the worst case
        res = min(res, max(
                    // Egg breaks
                    dp(K - 1, i - 1),
                    // Egg does not break
                    dp(K, N - i),
                ) + 1
                // Dropped an egg at floor i, so add one
            )
    return res
```

This for loop is the code implementation of the following state transition formula:

![](../pictures/drop-egg/formula1.png)

If you can understand this state transition formula, it will be easy to understand how binary search can be used to optimize it.

According to the definition of the `dp(K, N)` array (with `K` eggs and `N` floors, the minimum number of attempts needed), **when `K` is fixed, this function increases as `N` increases**. No matter how smart your strategy is, if the number of floors increases, the number of attempts must increase.

Now, look at the two functions `dp(K - 1, i - 1)` and `dp(K, N - i)`, where `i` goes from 1 to `N`. If we fix `K` and `N`, **treat these two functions as functions of `i`**: the first one increases as `i` increases, and the second one decreases as `i` increases:

![](../pictures/drop-egg/2.jpg)

We want the minimum of the maximum value of these two functions. In other words, we are looking for the lowest point where the two curves cross, which is shown in red in the picture above.

In the previous article [Binary Search in Real World](https://labuladong.online/en/algo/frequency-interview/binary-search-in-action/), we discussed that binary search is widely used. As long as you can find a function with monotonicity, you can often use binary search to optimize a linear search. Looking at the curves of these two `dp` functions, what we want is the lowest point, which looks like this:

```java
for (int i = 1; i <= N; i++) {
    if (dp(K - 1, i - 1) == dp(K, N - i))
        return dp(K, N - i);
}
```

If you are familiar with binary search, you may notice that this is just finding a "valley" point. We can use binary search to quickly find this point. Here is the improved code. The linear search in the `dp` function is changed to binary search, making it much faster:

```java
class Solution {
    // memoization
    int[][] memo;

    public int superEggDrop(int K, int N) {
        // m will not exceed N times (linear scan)
        memo = new int[K + 1][N + 1];
        for (int[] row : memo) {
            Arrays.fill(row, -666);
        }
        return dp(K, N);
    }

    // definition: holding K eggs, facing N floors, the minimum number of egg drops is dp(K, N)
    int dp(int K, int N) {
        // base case
        if (K == 1) return N;
        if (N == 0) return 0;

        // check memo to avoid redundant calculations
        if (memo[K][N] != -666) {
            return memo[K][N];
        }

        // for (int i = 1; i <= N; i++) {
        //     res = Math.min(
        //         res,
        //         Math.max(dp(K, N - i), dp(K - 1, i - 1)) + 1
        //     );
        // }

        // use binary search instead of linear search
        int res = Integer.MAX_VALUE;
        int lo = 1, hi = N;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            // two cases: the egg breaks at the mid floor or it does not break
            int broken = dp(K - 1, mid - 1);
            int not_broken = dp(K, N - mid);
            // res = min(max(broken, not_broken) + 1)
            if (broken > not_broken) {
                hi = mid - 1;
                res = Math.min(res, broken + 1);
            } else {
                lo = mid + 1;
                res = Math.min(res, not_broken + 1);
            }
        }
        memo[K][N] = res;
        return res;
    }
}
```

What is the time complexity of this algorithm? **The time complexity of a dynamic programming algorithm is the number of subproblems times the complexity of the function itself.**

The complexity of the function itself ignores the recursive part. Here, there is a binary search inside the `dp` function, so the function's complexity is O(logN).

The number of subproblems is the total number of different states, which is O(KN).

So, the total time complexity of the algorithm is O(KNlogN), and the space complexity is O(KN). This is more efficient than the previous O(KN^2) algorithm.


## 4. Redefining State Transition

Finding the state transition for dynamic programming is not always clear. Different ways to define the state can lead to very different solutions and complexity. This is a good example.

Let’s review our previous definition of the `dp` array:

```java
int dp(int k, int n)
// Current state: k eggs, facing n floors
// Return the minimum number of throws needed in this state
```

Or using an array:

```java
dp[k][n] = m
// Current state: k eggs, facing n floors
// The minimum number of throws needed in this state is m
```

With this definition, if you know the number of eggs and the number of floors, you can know the minimum number of throws needed. The final answer is `dp(K, N)`.

In this way, you must try all possible ways to throw the eggs. Even if you use binary search to optimize, you are just pruning the search space, but the overall idea is still brute-force.

Now, let's change the definition of the `dp` array a bit. **If you know the number of eggs and the maximum number of throws allowed, you can know the highest floor you can test for `F`.** This is what it means:

```java
dp[k][m] = n
// You have k eggs, and can throw them m times
// In the worst case, you can test up to n floors

// For example, dp[1][7] = 7 means:
// You have 1 egg, and you are allowed 7 throws;
// In this state, you can test up to 7 floors,
// so you can find the floor F where the egg just doesn’t break
// (You check one floor at a time)
```

This is like a "reverse" version of our original idea. Let's not worry about how to write the state transition for now. First, think about what answer we want under this definition.

What we want to find is the minimum number of throws `m`. But now, `m` is part of the state, not the result. We can handle it like this:

```java
int superEggDrop(int K, int N) {

    int m = 0;
    while (dp[K][m] < N) {
        m++;
        // state transition...
    }
    return m;
}
```

The problem asks: **Given `K` eggs and `N` floors, what is the minimum number of throws needed in the worst case?** The `while` loop ends when `dp[K][m] == N`, which means: **Given `K` eggs and `m` throws, you can test up to `N` floors in the worst case.**

Look at these two statements, they are exactly the same! So organizing the code this way is correct. The key is how to find the state transition equation. Let's start from our original idea. Before, we used a picture to help understand the state transition:

![](../pictures/drop-egg/1.jpg)

This picture only shows one floor `i`. The original method needs to scan all floors linearly or with binary search to find min and max values. But with this new `dp` definition, you don’t need that anymore, because of these two facts:

**1. No matter which floor you throw the egg from, the egg either breaks or does not break. If it breaks, you check the floors below. If it does not break, you check the floors above.**

**2. The total number of floors = floors above + floors below + 1 (the current floor).**

With this, we can write the state transition as:

```python
dp[k][m] = dp[k][m - 1] + dp[k - 1][m - 1] + 1
```

**`dp[k][m - 1]` is the number of floors above,** because the number of eggs `k` does not change (egg didn’t break), but the throws decrease by one.

**`dp[k - 1][m - 1]` is the number of floors below,** because you have one less egg (egg broke), and the throws decrease by one.

::: note Note

Why do we subtract one from `m`? The definition says `m` is the maximum number of throws allowed, not the number of throws used.

:::

![](../pictures/drop-egg/3.jpg)

Now the whole idea is ready. Just put the state transition into the code framework:

```java
class Solution {
    public int superEggDrop(int K, int N) {
        // m will not exceed N times (linear scan)
        int[][] dp = new int[K + 1][N + 1];
        // base case:
        // dp[0][..] = 0
        // dp[..][0] = 0
        // Java initializes arrays to 0 by default
        int m = 0;
        while (dp[K][m] < N) {
            m++;
            for (int k = 1; k <= K; k++)
                dp[k][m] = dp[k][m - 1] + dp[k - 1][m - 1] + 1;
        }
        return m;
    }
}
```

<visual slug='super-egg-drop'/>

If this code is still hard to understand, it’s actually the same as this:

```java
for (int m = 1; dp[K][m] < N; m++)
    for (int k = 1; k <= K; k++)
        dp[k][m] = dp[k][m - 1] + dp[k - 1][m - 1] + 1;
```

This code is easier to follow. We are not looking for a value in the `dp` array, but for an index `m` that meets the condition, so we use a `while` loop to find it.

What is the time complexity of this algorithm? It's clearly O(KN), two nested loops.

Also, note that `dp[k][m]` only depends on the left and upper-left states. You can use the space optimization trick from [Dynamic Programming Space Compression](https://labuladong.online/en/algo/dynamic-programming/space-optimization/) to make it a one-dimensional `dp` array. We won’t write that here.


## 5. Further Optimization

We can optimize further, but I will just briefly mention the idea here.

Based on the previous idea, **note that the function `dp(m, k)` increases as `m` increases, because with the same number of eggs `k`, more attempts allow you to test more floors**.

Here, you can use binary search to quickly find when `dp[K][m] == N`. This will reduce the time complexity to O(KlogN). But I think writing an O(K\*N\*logN) binary search algorithm is enough. The later optimizations are just for expanding your thinking, and you just need to know about them.

One thing is clear: you can replace the linear scan of `m` with binary search. The main code framework is to change the brute-force `while` loop over `m` to binary search:

```java
// change linear search to binary search
// for (int m = 1; dp[K][m] < N; m++)
int lo = 1, hi = N;
while (lo < hi) {
    int mid = (lo + hi) / 2;
    if (... < N) {
        lo = ...;
    } else {
        hi = ...;
    }
    
    for (int k = 1; k <= K; k++) {
        // state transition equation
    }
}
```

To sum up, the first binary search optimization uses the monotonicity of the `dp` function to quickly search for the answer. The second optimization changes the state transition equation to simplify the process, but the logic is harder to come up with. You can also use some math methods and binary search to optimize the second solution, but it is not really necessary to learn them.
