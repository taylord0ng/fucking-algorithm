::: info Prerequisites

Before reading this article, you should first study:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

The main forms of dynamic programming algorithms are recursive solutions with memoization and iterative solutions using state transition. These two approaches are essentially the same and have similar efficiency.

This article will introduce one advantage of the iterative approach in dynamic programming: the ability to optimize the space usage of the `dp` array (commonly known as the rolling array technique), reducing space complexity.

Simply put, in some cases, if the state transition equation only depends on adjacent states, there is no need to maintain the entire `dp` array. You only need to keep the necessary states, which can reduce space complexity.

In my opinion, mastering the space optimization technique is not mandatory, for the following reasons:

1. In most coding tests, space constraints are not very strict. Usually, you can pass all test cases even without this optimization.

2. Using space optimization can make the code less readable, making it harder to understand and debug.

Therefore, this article is for those who are interested in further study. Feel free to learn and understand it in depth.


<!-- hide -->

## When can we use space compression

We usually use space compression on 2D `dp` problems. **If to compute `dp[i][j]` you only need states that are “next to” `dp[i][j]`, then you can compress the 2D `dp` array into 1D**. This reduces space from O(N^2) to O(N).

What does “states next to `dp[i][j]`” mean? For example, in the previous article [Longest Palindromic Subsequence](https://labuladong.online/en/algo/dynamic-programming/subsequence-problem/), the final code is:

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int n = s.length();
        // initialize the dp array to all zeros
        int[][] dp = new int[n][n];
        // base case
        for (int i = 0; i < n; i++) {
            dp[i][i] = 1;
        }
        // traverse in reverse to ensure correct state transitions
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i + 1; j < n; j++) {
                // state transition equation
                if (s.charAt(i) == s.charAt(j)) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        // length of the longest palindromic subsequence in the entire s
        return dp[0][n - 1];
    }
}
```

::: tip Tip

In this article, we won’t talk about how to derive the state transition formula. We only talk about how to compress space for 2D DP. The skill is general, so even if you did not read the previous article or do not understand the code, it will not stop you from learning space compression.

:::

Look at how we update `dp[i][j]`. It only depends on three states: `dp[i+1][j-1], dp[i][j-1], dp[i+1][j]`:

![](../pictures/space-optimal/1.jpeg)

These are the states “next to” `dp[i][j]`. Since we only need these three neighbors to compute `dp[i][j]`, we actually do not need the whole big 2D `dp` table, right? **The core idea of space compression is to “project” the 2D array into a 1D array**:

![](../pictures/space-optimal/2.jpeg)

The word “project” is just to help you imagine it: we want one 1D array to play the role of the original 2D array.

This idea is simple, but there is a clear problem. In the picture, `dp[i][j-1]` and `dp[i+1][j-1]` are in the same column, but a 1D array can only hold one value at that position. If we project them into 1D, one will overwrite the other. Then how can we still compute `dp[i][j]`?

This is the hard part of space compression. Let’s solve it using the “Longest Palindromic Subsequence” example. Its main transition logic is this code:

```java
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        // 状态转移方程
        if (s.charAt(i) == s.charAt(j)) {
            dp[i][j] = dp[i + 1][j - 1] + 2;
        } else {
            dp[i][j] = Math.max(dp[i + 1][j], dp[i][j - 1]);
        }
    }
}
```

## Rolling array optimization

First, we do a simple **rolling array optimization**. Since `dp[i][j]` only depends on `dp[i+1][..]` and `dp[i][..]`, we do not need the whole `N * N` `dp` array. We only need two rows.

We can use an array `dp[2][n]`, and use modulo `% 2` to switch between these two rows:

```java
// 滚动数组优化
int[][] dp = new int[2][n];
dp[0][0] = 1;

for (int i = n - 2; i >= 0; i--) {
    int curr = i % 2;
    int next = (i + 1) % 2;
    // need to handle base case when using rolling array
    dp[curr][i] = 1;
    for (int j = i + 1; j < n; j++) {
        if (s.charAt(i) == s.charAt(j)) {
            dp[curr][j] = dp[next][j - 1] + 2;
        } else {
            dp[curr][j] = Math.max(dp[next][j], dp[curr][j - 1]);
        }
    }
}

return dp[0][n - 1];
```

**In practice, this level of optimization is already enough. The `dp` array now has only two rows, so the space complexity is O(N)**, which is almost the same as a 1D array.

Next, we will go further and fully compress `dp` into a 1D array.


## One-Dimensional Array Optimization

Think about the picture above. The “projection” turns many rows into one row. So when we compress a 2D `dp` array into 1D, we usually remove the first dimension `i`, and keep only the `j` dimension. **The 1D `dp` array after compression is just the row `dp[i][..]` from the original 2D `dp` array**.

First, let’s change the code above. Just blindly remove the `i` dimension and make `dp` a 1D array:

```java
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        // What does each number in the 1D dp array mean here?
        if (s.charAt(i) == s.charAt(j)) {
            dp[j] = dp[j - 1] + 2;
        } else {
            dp[j] = Math.max(dp[j], dp[j - 1]);
        }
    }
}
```

In this code, the 1D `dp` array can only express one row `dp[i][..]` of the 2D `dp` array. But when we do state transition, we need the values `dp[i+1][j-1], dp[i][j-1], dp[i+1][j]`.

So we must think about two questions:

1. Before we assign a new value to `dp[j]`, which cell in the 2D `dp` array does `dp[j]` correspond to?

2. Which cell in the 2D `dp` array does `dp[j-1]` correspond to?

**For question 1: before `dp[j]` gets a new value, it is the value from the previous iteration of the outer loop. That is the value at `dp[i+1][j]` in the 2D `dp` array**.

**For question 2: `dp[j-1]` is from the previous iteration of the inner loop. That is the value at `dp[i][j-1]` in the 2D `dp` array**.

So most of the problem is solved. The only remaining state we can’t get directly from the 1D array is `dp[i+1][j-1]`:

```java
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        if (s.charAt(i) == s.charAt(j)) {
            // dp[i][j] = dp[i+1][j-1] + 2;
            dp[j] = ?? + 2;
        } else {
            // dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
            dp[j] = Math.max(dp[j], dp[j - 1]);
        }
    }
}
```

We iterate `i` and `j` from left to right, and from bottom to top. So we can see that when we update the 1D `dp` array, `dp[i+1][j-1]` will be overwritten by `dp[i][j-1]`. The figure below marks the visiting order of these four positions:

![](../pictures/space-optimal/3.jpeg)

**So if we want `dp[i+1][j-1]`, we must store it in a temp variable `temp` before it is overwritten, and keep this value until we compute `dp[i][j]`**. To do this, based on the figure, we can write code like this:

```java
for (int i = n - 2; i >= 0; i--) {
    // variable to store dp[i+1][j-1]
    int pre = 0;
    for (int j = i + 1; j < n; j++) {
        int temp = dp[j];
        if (s.charAt(i) == s.charAt(j)) {
            // dp[i][j] = dp[i+1][j-1] + 2;
            dp[j] = pre + 2;
        } else {
            dp[j] = Math.max(dp[j], dp[j - 1]);
        }
        // in the next round, pre becomes dp[i+1][j-1]
        pre = temp;
    }
}
```

Don’t underestimate this piece of code. This is the most clever part of 1D `dp`. If you get it, it feels easy; if you don’t, it feels impossible. To make it clear, let’s use concrete values to break down the logic:

Assume `i = 5, j = 7` and `s[5] == s[7]`. Then we enter this logic:

```java
for (int i = 5; i--) {
    for (int j = 7; j++) {
        if (s[5] == s[7]) {
            // dp[5][7] = dp[i+1][j-1] + 2;
            dp[7] = pre + 2;
        }
    }
}
```


I ask you, what is this `pre` variable? It is the `temp` value from the previous iteration of the inner for loop.

Now, let me ask you, what was the `temp` value from the previous iteration of the **inner** for loop? It is `dp[j-1]`, which is `dp[6]`. However, note that this is the `dp[6]` from the **previous iteration** of the **outer** for loop, not the current `dp[6]`.

This should be understood in terms of the index of a two-dimensional array. Your current `dp[6]` is `dp[i][6] = dp[5][6]` in the two-dimensional `dp` array, while the `temp` is `dp[i+1][6] = dp[6][6]` in the two-dimensional `dp` array.

In other words, the `pre` variable is `dp[i+1][j-1] = dp[6][6]`, which is the result we are looking for.

Now we have successfully reduced the dimensionality of the state transition equation, tackling the toughest part, but we must still handle the base case:

```java
// initialize all elements of the dp array to 0
int[][] dp = new int[n][n];
// base case
for (int i = 0; i < n; i++) {
    dp[i][i] = 1;
}
```

How do we reduce the base case to one dimension? It’s simple. Remember that space compression is projection. Let's project the base case onto one dimension:

![](../pictures/space-optimal/4.jpeg)

All the base cases in the two-dimensional `dp` array fall into the one-dimensional `dp` array without conflict or overlap, so we can write the code like this:

```java
// base case: initialize the one-dimensional dp array to all 1s
int[] dp = new int[n];
Arrays.fill(dp, 1);
```

At this point, we have reduced both the base case and the state transition equation, effectively writing a complete code:

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int n = s.length();
        // base case: initialize all elements of the one-dimensional dp array to 1
        int[] dp = new int[n];
        Arrays.fill(dp, 1);

        for (int i = n - 2; i >= 0; i--) {
            int pre = 0;
            for (int j = i + 1; j < n; j++) {
                int temp = dp[j];
                // state transition equation
                if (s.charAt(i) == s.charAt(j))
                    dp[j] = pre + 2;
                else
                    dp[j] = Math.max(dp[j], dp[j - 1]);
                pre = temp;
            }
        }
        return dp[n - 1];
    }
}
```

This concludes the article. However, the space compression technique, while impressive, is based on conventional dynamic programming thinking.

As you can see, using space compression techniques to reduce the dimensionality of a two-dimensional `dp` array makes the solution code much less readable. If someone looks only at this solution, it can be quite confusing. Algorithm optimization is such a process: start by writing a highly readable brute-force recursive algorithm, then try to use dynamic programming techniques to optimize overlapping subproblems, and finally attempt to use space compression techniques to optimize space complexity.

This means you should at least be proficient in using the framework discussed in our previous article [Detailed Explanation of Dynamic Programming Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/) to find the state transition equation and write a correct dynamic programming solution. Only then can you analyze the situation of state transitions and consider whether space compression techniques can be applied for optimization.

I hope readers can progress steadily and gradually. For such extreme optimizations, it's okay not to pursue them. After all, having a good grasp of the basics will make you confident in any situation!
