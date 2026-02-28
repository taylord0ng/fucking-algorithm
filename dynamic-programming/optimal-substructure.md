::: info Prerequisites

Before reading this article, you need to learn:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

This article is a full Q&A after [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/). It will explain these questions:

1. What is “optimal substructure”, and how is it related to dynamic programming?
2. How to tell whether a problem is a dynamic programming problem, meaning how to see if there are overlapping subproblems.
3. Why we often set the `dp` array size to `n + 1` instead of `n`.
4. Why there are many ways to traverse the `dp` array: forward, backward, or even diagonally.

<!-- hide -->

## 1. Optimal Substructure Explained

“Optimal substructure” is a property of some problems. It is not only for dynamic programming. Many problems have optimal substructure, but most of them do not have overlapping subproblems, so we do not call them dynamic programming problems.

Here is an easy example: suppose your school has 10 classes. You already know the highest exam score in each class. Now I ask you to find the highest score in the whole school. Can you do it? Of course. You do not need to scan every student again. You just take the max among these 10 highest scores.

This problem **has optimal substructure**: from the best result of subproblems, you can get the best result of a bigger problem. The best score of **each class** is a subproblem. After you know all subproblem answers, you can get the answer for the bigger problem: the best score of the **whole school**.

Even this simple problem has optimal substructure. But it clearly has no overlapping subproblems, so dynamic programming is not needed.

Now another example: suppose your school has 10 classes, and you know the maximum score gap in each class (highest score minus lowest score in that class). Now I ask you to find the maximum score gap in the whole school. You can compute it, but you cannot get it from those 10 class gaps. Because the max gap in the whole school may be between the top score in class 3 and the lowest score in class 6.

This problem **does not have optimal substructure**. You cannot use the best value of each class to get the best value for the whole school. As said in [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), to have optimal substructure, subproblems must be independent. Here the max gap can happen across two classes, so subproblems are not independent. So the problem does not have optimal substructure.

**So what if optimal substructure fails? The strategy is: change the problem.** For the max gap problem, since we cannot use the known class gaps, we may only write brute-force code:

```java
int result = 0;
for (Student a : school) {
    for (Student b : school) {
        if (a is b) continue;
        result = max(result, |a.score - b.score|);
    }
}
return result;
```

Changing the problem means turning it into an equal problem: the maximum score gap is the same as (highest score - lowest score). So we just need the highest and lowest score. That is exactly the first problem, and it has optimal substructure. Now we use optimal substructure to get the extremes, and then solve the max gap. This is much faster.

This example is very simple. But think about dynamic programming problems: we are often finding some kind of max or min. It is the same idea, just with overlapping subproblems.

The earlier article [Egg Dropping Problem](https://labuladong.online/en/algo/dynamic-programming/egg-drop/) shows how to change a problem. Different optimal substructure may lead to different solutions and different speed.

Another common and simple example: find the maximum value in a binary tree (for simplicity, assume all node values are non-negative):

```java
int maxVal(TreeNode root) {
    if (root == null)
        return -1;
    int left = maxVal(root.left);
    int right = maxVal(root.right);
    return max(root.val, left, right);
}
```

This also has optimal substructure. The max value of the tree rooted at `root` can be derived from the max values of the left and right subtrees (subproblems). This is easy to understand if you remember the school/class examples.

Still, this is not a dynamic programming problem. The goal is to show: optimal substructure is not special to dynamic programming. Most “find max/min” problems have it.

But the other way is important: as a necessary condition for dynamic programming, **optimal substructure usually means you are asked to find a max/min**. So when you see an annoying max/min problem, you should think about dynamic programming. This is a common pattern.

Dynamic programming is pushing forward from the simplest base case. You can think of it like a chain reaction: small answers build bigger answers. Only problems with optimal substructure can work like this.

Finding optimal substructure is basically proving the state transition is correct. If your transition follows optimal substructure, you can write a brute-force recursive solution. After you have the brute-force solution, you can check if there are overlapping subproblems. If yes, optimize; if no, it is fine. This is also a common pattern.

I will not list “real” dynamic programming examples here. You can read older articles to see how transitions follow optimal substructure. Let’s move on to other confusing dynamic programming behaviors.

## 2. How to See Overlapping Subproblems Quickly

Many readers say:

After reading [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), I know how to optimize a dynamic programming solution step by step;

After reading [Dynamic Programming Design: Mathematical Induction](https://labuladong.online/en/algo/dynamic-programming/longest-increasing-subsequence/), I know how to write a brute-force solution (the state transition).

**But even if I have a brute-force solution, it is hard to tell whether it has overlapping subproblems**, so I cannot know if I should use memoization to speed it up.

I have mentioned this in several dynamic programming articles. Here I will summarize it.

**First, the simplest way is to draw the recursion tree and check if there are repeated nodes.**

For example, the Fibonacci recursion tree in [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/):

![](../pictures/dynamic-programming/1.jpg)

This tree clearly has repeated nodes, so we can use memoization to avoid repeated work.

But Fibonacci is too simple. In real problems, dynamic programming can be more complex, like 2D or 3D DP. You can still draw the recursion tree, but it becomes messy.

For example, in [Minimum Path Sum](https://labuladong.online/en/algo/dynamic-programming/minimum-path-sum/), we have this brute-force solution:

```java
int dp(int[][] grid, int i, int j) {
    if (i == 0 && j == 0) {
        return grid[0][0];
    }
    if (i < 0 || j < 0) {
        return Integer.MAX_VALUE;
    }

    return Math.min(
            dp(grid, i - 1, j), 
            dp(grid, i, j - 1)
        ) + grid[i][j];
}
```

Even if you never read the earlier article, you can see from the code that the parameters `i, j` keep changing during recursion. So the “state” is the value `(i, j)`. Can you tell whether there are overlapping subproblems?

If the input is `i = 8, j = 7`, the recursion tree for this 2D state looks like this, and we can see overlap:

![](../pictures/optimal/2.jpeg)

**But if you think a bit more, you will see you do not need to draw the tree. You can tell from the recursion structure.**

The trick is: remove code details, and keep only the recursion framework:

```java
int dp(int[][] grid, int i, int j) {
    dp(grid, i - 1, j), // #1
    dp(grid, i, j - 1)  // #2
}
```

We can see `i, j` keep decreasing. Now a question: if we want to move from state `(i, j)` to `(i-1, j-1)`, how many paths are there?

There are clearly two paths: `(i, j) -> #1 -> #2` or `(i, j) -> #2 -> #1`. More than one path means `(i-1, j-1)` will be computed many times, so overlapping subproblems must exist.

One more slightly more complex example: the brute-force code for [Regular Expression Matching](https://labuladong.online/en/algo/dynamic-programming/regular-expression-matching/):

```java
boolean dp(String s, int i, String p, int j) {
    int m = s.length(), n = p.length();
    // base case
    if (j == n) {
        return i == m;
    }
    if (i == m) {
        if ((n - j) % 2 == 1) {
            return false;
        }
        for (; j + 1 < n; j += 2) {
            if (p.charAt(j + 1) != '*') {
                return false;
            }
        }
        return true;
    }

    boolean res = false;

    if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') {
        if (j < n - 1 && p.charAt(j + 1) == '*') {
            res = dp(s, i, p, j + 2) || dp(s, i + 1, p, j);
        } else {
            res = dp(s, i + 1, p, j + 1);
        }
    } else {
        if (j < n - 1 && p.charAt(j + 1) == '*') {
            res = dp(s, i, p, j + 2);
        } else {
            res = false;
        }
    }
    
    return res;
}
```

The code looks complex. Drawing a tree is painful. But we can ignore all details and branches, and only keep the recursion framework:

```java
boolean dp(String s, int i, String p, int j) {
    dp(s, i, p, j + 2);     // #1
    dp(s, i + 1, p, j);     // #2
    dp(s, i + 1, p, j + 1); // #3
}
```

Same as the previous problem, the “state” is still `(i, j)`. Now another question: if we want to move from `(i, j)` to `(i+2, j+2)`, how many paths are there?

Clearly, there are at least two paths: `(i, j) -> #1 -> #2 -> #2` and `(i, j) -> #3 -> #3`. This means there are a huge number of overlapping subproblems.

So without drawing anything, we already know this solution needs memoization to speed it up.


## 3. Choosing the size of the dp array

For example, in the earlier article [Edit Distance](https://labuladong.online/en/algo/dynamic-programming/edit-distance/), I first showed a top-down recursive solution, with a `dp` function like this:

```java
class Solution {
    public int minDistance(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // According to the definition of the dp function, calculate the minimum edit distance between s1 and s2
        return dp(s1, m - 1, s2, n - 1);
    }

    // Definition: the minimum edit distance between s1[0..i] and s2[0..j] is dp(s1, i, s2, j)
    int dp(String s1, int i, String s2, int j) {
        // Handle base case
        if (i == -1) {
            return j + 1;
        }
        if (j == -1) {
            return i + 1;
        }

        // Perform state transition
        if (s1.charAt(i) == s2.charAt(j)) {
            return dp(s1, i - 1, s2, j - 1);
        } else {
            return min(
                dp(s1, i, s2, j - 1) + 1,
                dp(s1, i - 1, s2, j) + 1,
                dp(s1, i - 1, s2, j - 1) + 1
            );
        }
    }

    int min(int a, int b, int c) {
        return Math.min(a, Math.min(b, c));
    }
}
```

Then I changed it into a bottom-up iterative solution:

```java
class Solution {
    public int minDistance(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // Definition: the minimum edit distance between s1[0..i] and s2[0..j] is dp[i+1][j+1]
        int[][] dp = new int[m + 1][n + 1];
        // Initialize base case
        for (int i = 1; i <= m; i++)
            dp[i][0] = i;
        for (int j = 1; j <= n; j++)
            dp[0][j] = j;
        
        // Solve from bottom to top
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // Perform state transition
                if (s1.charAt(i-1) == s2.charAt(j-1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = min(
                        dp[i - 1][j] + 1,
                        dp[i][j - 1] + 1,
                        dp[i - 1][j - 1] + 1
                    );
                }
            }
        }
        // According to the definition of the dp array, store the minimum edit distance between s1 and s2
        return dp[m][n];
    }
}
```

These two solutions use the same idea. But some readers asked: why does the iterative solution create the `dp` array as `int[m+1][n+1]`? Why is the edit distance of `s1[0..i]` and `s2[0..j]` stored in `dp[i+1][j+1]` with an index shift?

Can we copy the definition of the `dp` function, create `dp` as `int[m][n]`, and store the answer for `s1[0..i]` and `s2[0..j]` in `dp[i][j]`?

**In theory, you can define it in any way, as long as you handle the base case correctly.**

Look at the definition of the `dp` function: `dp(s1, i, s2, j)` computes the edit distance of `s1[0..i]` and `s2[0..j]`. When `i` or `j` is `-1`, it means an empty string, which is the base case. So the function handles these special cases at the beginning.

Now look at the `dp` array. You can define `dp[i][j]` as the edit distance of `s1[0..i]` and `s2[0..j]`. But then how do you handle the base case? Array indexes cannot be `-1`.

So we create the `dp` array as `int[m+1][n+1]` and shift all indexes by 1. We keep index 0 for the base case (empty string). Then we define `dp[i+1][j+1]` to store the edit distance of `s1[0..i]` and `s2[0..j]`.

## 4. The traversal direction of the dp array

When solving dynamic programming problems, many people feel unsure about the traversal order of the `dp` array. Using a 2D `dp` array as an example, sometimes we traverse forward:

```java
int[][] dp = new int[m][n];
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        // compute dp[i][j]
```

Sometimes we traverse backward:

```java
for (int i = m - 1; i >= 0; i--)
    for (int j = n - 1; j >= 0; j--)
        // compute dp[i][j]
```

Sometimes we traverse diagonally:

```java
// traverse the array diagonally
for (int l = 2; l <= n; l++) {
    for (int i = 0; i <= n - l; i++) {
        int j = l + i - 1;
        // compute dp[i][j]
    }
}
```

Even more confusing, sometimes both forward and backward traversal work. For example, in some parts of [Stock Problems](https://labuladong.online/en/algo/dynamic-programming/stock-problem-summary/), both directions are fine.

If you look closely, you only need to remember two rules:

**1. During traversal, the states you need must already be computed.**

**2. When traversal ends, the cell that stores the final answer must have been computed.**

Now let’s explain these two rules.

Take the classic problem [Edit Distance](https://labuladong.online/en/algo/dynamic-programming/edit-distance/). From the definition of `dp`, the base cases are `dp[..][0]` and `dp[0][..]`, and the final answer is `dp[m][n]`. From the transition, `dp[i][j]` depends on `dp[i-1][j]`, `dp[i][j-1]`, and `dp[i-1][j-1]`, like this:

![](../pictures/optimal/1.jpg)

So how should you traverse the `dp` array? Based on the two rules, you should traverse forward:

```java
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        // use dp[i-1][j], dp[i][j - 1], dp[i-1][j-1]
        // to compute dp[i][j]
```

Because in each step, the left, upper, and upper-left cells are base cases or have already been computed. And when the loops end, you reach the answer `dp[m][n]`.

Here is another example: the palindromic subsequence problem. See [Subsequence Problem Template](https://labuladong.online/en/algo/dynamic-programming/subsequence-problem/). From the definition of `dp`, the base case is on the diagonal in the middle. `dp[i][j]` depends on `dp[i+1][j]`, `dp[i][j-1]`, and `dp[i+1][j-1]`. The final answer is `dp[0][n-1]`, like this:

![](../pictures/lps/4.jpg)

In this case, based on the two rules, there are two correct traversal orders:

![](../pictures/lps/5.jpg)

You can traverse diagonally from top-left to bottom-right, or traverse from bottom to top and left to right. This way, when you compute `dp[i][j]`, the left, lower, and lower-left cells have already been computed, so you get the correct result.

Now you should understand these two rules. Just look at where the base case is and where the final result is stored. Make sure every value you use during traversal is already computed. Sometimes there are multiple correct ways, and you can choose the one you like.
