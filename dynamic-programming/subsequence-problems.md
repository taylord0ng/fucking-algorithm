::: info Prerequisites

Before reading this article, you need to learn:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

Subsequence problems are common, and they are not easy.

First, subsequences are harder than substrings and subarrays. A subsequence is not continuous, but a substring/subarray is continuous. Even if you try brute-force, you may still not know how to do it, not to mention using it to solve algorithm problems.

Also, subsequence problems often involve two strings, like [Longest Common Subsequence](https://labuladong.online/en/algo/dynamic-programming/longest-common-subsequence/). If you don’t have enough experience, it is hard to come up with the solution. So this article will show the common patterns for subsequence problems. There are only two templates. If you think in these two ways, you are very likely to get it right.

In most cases, these problems ask for the **longest subsequence**. The shortest subsequence is just one character, so it is not interesting. Once a problem involves subsequences and max/min, it almost always tests **dynamic programming**, and the time complexity is usually **O(n^2)**.

The reason is simple: how many subsequences can a string have? At least exponential. Without dynamic programming, what else can you do?

Since we use dynamic programming, we need to define a `dp` array and find the state transition. The two templates are two ways to define the `dp` array. Different problems may need different `dp` definitions.

## 1. Two templates

<!-- hide -->

**1) Template 1: a 1D `dp` array**:

```java
int n = array.length;
int[] dp = new int[n];

for (int i = 1; i < n; i++) {
    for (int j = 0; j < i; j++) {
        dp[i] = best(dp[i], dp[j] + ...)
    }
}
```

For example, [Longest Increasing Subsequence](https://labuladong.online/en/algo/dynamic-programming/longest-increasing-subsequence/) and [Maximum Subarray Sum](https://labuladong.online/en/algo/dynamic-programming/maximum-subarray/) use this idea.

In this idea, the definition of `dp` is:

**In the subarray `arr[0..i]`, the length of the subsequence that ends with `arr[i]` is `dp[i]`.**

Why does LIS need this? Because it matches induction, so we can find the state transition. I won’t expand it here.

**2) Template 2: a 2D `dp` array**:

```java
int n = arr.length;
int[][] dp = new dp[n][n];

for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        if (arr[i] == arr[j]) 
            dp[i][j] = dp[i][j] + ...
        else
            // calculate min or max
            dp[i][j] = min(...)
    }
}
```

This idea is used more often, especially when the subsequence involves two strings/arrays, like [Longest Common Subsequence](https://labuladong.online/en/algo/dynamic-programming/longest-common-subsequence/) and [Edit Distance](https://labuladong.online/en/algo/dynamic-programming/edit-distance/). It can also be used for one string/array, like the palindrome subsequence problem in this article.

**2.1 Two strings/arrays**: the `dp` definition is:

**In subarray `arr1[0..i]` and subarray `arr2[0..j]`, the subsequence length we want is `dp[i][j]`.**

**2.2 One string/array**: the `dp` definition is:

**In subarray `array[i..j]`, the subsequence length we want is `dp[i][j]`.**

Next, we use the longest palindromic subsequence problem to explain how to use dynamic programming in the second case.

## 2. Longest Palindromic Subsequence

We previously solved [Longest Palindromic Substring](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/). Now let’s level up: LeetCode 516, “[Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)”. We need the length of the longest palindromic subsequence.

Given a string `s`, find the length of the longest palindromic subsequence in `s`. The function signature is:

```java
int longestPalindromeSubseq(String s);
```

Example: `s = "aecda"`, the answer is 3, because the longest palindromic subsequence is `"aca"`, length 3.

We define the `dp` array like this: **in substring `s[i..j]`, the length of the longest palindromic subsequence is `dp[i][j]`**. Remember this definition, or you won’t understand the algorithm.

Why define `dp` this way? To find the state transition, we need induction: how to use known results to get unknown results. With this definition, induction is natural, and the state transition is easy to see.

More specifically, to compute `dp[i][j]`, assume you already know `dp[i+1][j-1]` (the answer for `s[i+1..j-1]`). Can you compute `dp[i][j]` (the answer for `s[i..j]`)?

![](../pictures/lps/1.jpg)

Yes. It depends on whether `s[i]` equals `s[j]`:

**If they are equal**, then they must be part of the longest palindromic subsequence, so:

![](../pictures/lps/2.jpg)

**If they are not equal**, then they **cannot both** be in the longest palindromic subsequence. We try removing one side, and take the larger one:

![](../pictures/lps/3.jpg)

In code:

```java
if (s[i] == s[j])
    // both must be in the longest palindromic subsequence
    dp[i][j] = dp[i + 1][j - 1] + 2;
else
    // compare s[i+1..j] and s[i..j-1]
    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
```

So we get the transition. By the `dp` definition, the answer is `dp[0][n - 1]`, which is the longest palindromic subsequence length of the whole string `s`.

## 3. Code implementation

First, the base case: if there is only one character, the longest palindromic subsequence length is 1, so `dp[i][j] = 1` when `i == j`.

Since `i <= j`, for `i > j`, there is no substring, so we should set it to 0.

Now look at the transition: to compute `dp[i][j]`, we need `dp[i+1][j-1]`, `dp[i+1][j]`, and `dp[i][j-1]`. After filling the base case, the table looks like this:

![](../pictures/lps/4.jpg)

**To make sure these needed cells are already computed, we must traverse diagonally or traverse backward**:

![](../pictures/lps/5.jpg)

::: tip Tip

For more about the traversal order of the `dp` array, see [Dynamic Programming Q&A](https://labuladong.online/en/algo/dynamic-programming/faq-summary/).

:::

I choose backward traversal. The code is:

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

<visual slug='longest-palindromic-subsequence'/>

Now the longest palindromic subsequence problem is solved.

## 4. Extension

Palindrome problems are not used everywhere, but after you can solve longest palindromic subsequence, you can also solve similar problems.

For example, LeetCode 1312, “[Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/)”:

Given a string `s`, you can insert any character at any position. To make `s` a palindrome, find the minimum number of insertions.

The function signature is:

```java
int minInsertions(String s);
```

Example: `s = "abcea"`, the answer is 2, because you can insert 2 characters to get `"abeceba"` or `"aebcbea"`. If `s = "aba"`, the answer is 0, because `s` is already a palindrome.

This is also a subsequence problem on a single string, so we can use a 2D `dp` array. Define `dp[i][j]` like this:

**For substring `s[i..j]`, the minimum insertions needed to make it a palindrome is `dp[i][j]`.**

By this definition, the base case is `dp[i][i] = 0`, because one character is already a palindrome.

Then use induction. Assume you already know `dp[i+1][j-1]`. How do you compute `dp[i][j]`?

![](../pictures/palindrome-insert/1.jpeg)

It is very similar to the longest palindromic subsequence transition:

```java
if (s[i] == s[j]) {
    // no insertion needed
    dp[i][j] = dp[i + 1][j - 1];
} else {
    // make s[i+1..j] or s[i..j-1] a palindrome, pick the smaller one
    // then insert one more char to match s[i] or s[j]
    dp[i][j] = min(dp[i + 1][j], dp[i][j - 1]) + 1;
}
```

Finally, we still traverse the `dp` table backward and write the code:

```java
class Solution {
    public int minInsertions(String s) {
        int n = s.length();
        // dp[i][j] represents the minimum number of insertions needed to make the substring s[i..j] a palindrome
        // initialize all elements of the dp array to 0
        int[][] dp = new int[n][n];
        // traverse in reverse order to ensure correct state transitions
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i + 1; j < n; j++) {
                // state transition equation
                if (s.charAt(i) == s.charAt(j)) {
                    dp[i][j] = dp[i + 1][j - 1];
                } else {
                    dp[i][j] = Math.min(dp[i + 1][j], dp[i][j - 1]) + 1;
                }
            }
        }
        // the minimum number of insertions for the entire string s
        return dp[0][n - 1];
    }
}
```

Now this problem is also solved with the subsequence template. Since it is so similar to the longest palindromic subsequence problem, can we reuse that solution directly?

Yes. We can even avoid writing the transition if you think about it:

**First compute the longest palindromic subsequence of `s`. Then all characters not in it are exactly the characters we need to insert.**

So we can reuse the previous `longestPalindromeSubseq` function:

```java
class Solution {
    // calculate the minimum number of insertions to make s a palindrome
    public int minInsertions(String s) {
        return s.length() - longestPalindromeSubseq(s);
    }

    // calculate the length of the longest palindromic subsequence in s
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

That’s all for subsequence algorithms. Hope it helps.
