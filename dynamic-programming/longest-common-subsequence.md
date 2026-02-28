::: info Prerequisite Knowledge

Before reading this article, you need to learn:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

I'm not sure how everyone feels about solving algorithm problems, but I've concluded that the technique is to break down a large problem into a small point, study how to solve the problem at this small point, and then expand it to the whole problem through recursion/iteration.

For example, in our previous article [Binary Tree Series Part 3](https://labuladong.online/en/algo/data-structure/binary-tree-part3/), when solving binary tree problems, we break down the entire problem to a specific node, imagine standing at that node, figure out what needs to be done, and then apply the binary tree recursion framework.

The same applies to dynamic programming problems, especially those related to subsequences. **This article starts with the "Longest Common Subsequence Problem," summarizing three subsequence problems**. By carefully discussing this type of problem, you can grasp this way of thinking.

## Longest Common Subsequence

Calculating the Longest Common Subsequence (LCS) is a classic dynamic programming problem, as seen in LeetCode Problem 1143 "[Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)":

Given two input strings `s1` and `s2`, find their longest common subsequence and return its length. The function signature is as follows:

```java
int longestCommonSubsequence(String s1, String s2);
```

For instance, if `s1 = "zabcde", s2 = "acez"`, their longest common subsequence is `lcs = "ace"`, with a length of 3, so the algorithm returns 3.

If you haven't solved this problem before, a simple brute-force algorithm would be to enumerate all subsequences of `s1` and `s2`, check for common ones, and then find the longest among them.

Obviously, this approach has a very high complexity, as enumerating all subsequences is exponential, making it impractical.

The correct approach is not to consider the entire strings, but to focus on each character of `s1` and `s2`. As summarized in the previous article [Subsequence Problem Template](https://labuladong.online/en/algo/dynamic-programming/subsequence-problem/):


<!-- hide -->

**For problems involving finding subsequences of two strings, using two pointers `i` and `j` to move through the strings usually indicates a dynamic programming approach**.

The problem of finding the longest common subsequence can also follow this pattern. We can start by writing a `dp` function:

```java
// Definition: Calculate the length of the longest common subsequence of s1[i..] and s2[j..]
int dp(String s1, int i, String s2, int j)
```

According to the definition of this `dp` function, the answer we seek is `dp(s1, 0, s2, 0)`, with the base case being when `i == len(s1)` or `j == len(s2)`, since at this point, `s1[i..]` or `s2[j..]` is equivalent to an empty string, and the length of the longest common subsequence is obviously 0:

```java
int longestCommonSubsequence(String s1, String s2) {
    return dp(s1, 0, s2, 0);
}

// Definition: Calculate the length of the longest common subsequence of s1[i..] and s2[j..]
int dp(String s1, int i, String s2, int j) {
    // base case
    if (i == s1.length() || j == s2.length()) {
        return 0;
    }
    // ...
}
```

**Next, we should focus not on the entire strings `s1` and `s2`, but on each individual character, considering what each character should do**.

![](../pictures/LCS/1.jpeg)

We look at `s1[i]` and `s2[j]`, **if `s1[i] == s2[j]`, it indicates that this character is definitely in the `lcs`**:

![](../pictures/LCS/2.jpeg)

This way, a character in the `lcs` is found. Based on the definition of the `dp` function, we can refine the code:


```java
// Definition: Calculate the length of the longest common subsequence of s1[i..] and s2[j..]
int dp(String s1, int i, String s2, int j) {
    if (s1.charAt(i) == s2.charAt(j)) {
        // s1[i] and s2[j] must be in the lcs,
        // plus the lcs length of s1[i+1..] and s2[j+1..], which is the answer
        return 1 + dp(s1, i + 1, s2, j + 1);
    } else {
        // ...
    }
}
```

Earlier, we discussed the case of `s1[i] == s2[j]`, but what should we do if `s1[i] != s2[j]`?

**`s1[i] != s2[j]` means at least one character from `s1[i]` or `s2[j]` is not in the `lcs`:**

![](../pictures/LCS/3.jpeg)

As shown above, there are three possible situations. How do we determine which one it is?

In fact, we don't know, so we calculate the answers for all three situations and take the largest result because the problem requires us to find the length of the "longest" common subsequence.

How do we calculate the answers for these three situations? Recall the definition of our `dp` function, which is specifically designed for this purpose!

The code can be further refined:

```java
// Definition: Calculate the length of the longest common subsequence of s1[i..] and s2[j..]
int dp(String s1, int i, String s2, int j) {
    if (s1.charAt(i) == s2.charAt(j)) {
        return 1 + dp(s1, i + 1, s2, j + 1);
    } else {
        // At least one character from s1[i] and s2[j] is not in the lcs,
        // Enumerate the results of the three cases and take the maximum result
        return max(
            /**<extend up -300>
            ![](../pictures/LCS/3.jpeg)
            */
            // Case 1: s1[i] is not in the lcs
            dp(s1, i + 1, s2, j),
            // Case 2: s2[j] is not in the lcs
            dp(s1, i, s2, j + 1),
            // Case 3: neither is in the lcs
            dp(s1, i + 1, s2, j + 1)
        );
    }
}
```

This is already very close to our final answer. **There is a small optimization, the situation where "neither `s1[i]` nor `s2[j]` is in the lcs" can actually be ignored.**

Since we are looking for the maximum value, the length of `lcs` calculated in situation three with `s1[i+1..]` and `s2[j+1..]` is definitely less than or equal to the `lcs` length in situation two with `s1[i..]` and `s2[j+1..]`, because `s1[i+1..]` is shorter than `s1[i..]`, and therefore the `lcs` calculated from it cannot be longer.

Similarly, the result of situation three is definitely less than or equal to situation one. **In short, situation three is covered by situation one and situation two,** so we can directly ignore situation three, and the complete code is as follows:

```java
class Solution {
    // memoization to eliminate overlapping subproblems
    int[][] memo;

    // main function
    public int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // memoization value of -1 represents not yet calculated
        memo = new int[m][n];
        for (int[] row : memo) 
            Arrays.fill(row, -1);
        // calculate the lcs length of s1[0..] and s2[0..]
        return dp(s1, 0, s2, 0);
    }

    // definition: calculate the longest common subsequence length of s1[i..] and s2[j..]
    int dp(String s1, int i, String s2, int j) {
        // base case
        if (i == s1.length() || j == s2.length()) {
            return 0;
        }
        // if calculated before, return the answer from the memoization directly
        if (memo[i][j] != -1) {
            return memo[i][j];
        }
        // make a choice based on the situation of s1[i] and s2[j]
        if (s1.charAt(i) == s2.charAt(j)) {
            // s1[i] and s2[j] must be in the lcs
            memo[i][j] = 1 + dp(s1, i + 1, s2, j + 1);
        } else {
            // at least one of s1[i] and s2[j] is not in the lcs
            memo[i][j] = Math.max(
                dp(s1, i + 1, s2, j),
                dp(s1, i, s2, j + 1)
            );
        }
        return memo[i][j];
    }
}
```

This approach follows our previous popular article [Dynamic Programming Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/) and should be easy to understand. As for why we add a `memo` for memoization, we've written about it many times before. For new readers, here's a brief repetition: first, abstract the recursive framework of our core `dp` function:

```java
int dp(int i, int j) {
    dp(i + 1, j + 1); // #1
    dp(i, j + 1);     // #2
    dp(i + 1, j);     // #3
}
```

You see, assuming I want to transition from `dp(i, j)` to `dp(i+1, j+1)`, there is more than one way to do so. You can go directly via `#1`, or through `#2 -> #3`, or through `#3 -> #2`.

This is the overlapping subproblem. If we do not use `memo` for memoization to eliminate subproblems, then `dp(i+1, j+1)` will be calculated multiple times, which is unnecessary.

At this point, the longest common subsequence problem is completely solved, using a top-down dynamic programming approach with memoization. We can also use a bottom-up iterative dynamic programming approach. The key is how to define the `dp` array, and here is the bottom-up solution:

```java
class Solution {
    public int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // definition: the length of lcs of s1[0..i-1] and s2[0..j-1] is dp[i][j]
        int[][] dp = new int[m + 1][n + 1];
        // goal: the length of lcs of s1[0..m-1] and s2[0..n-1] is dp[m][n]
        // base case: dp[0][..] = dp[..][0] = 0

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // now i and j start from 1, so we need to subtract 1
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    // s1[i-1] and s2[j-1] must be in the LCS
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    // at least one of s1[i-1] and s2[j-1] is not in the lcs
                    dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
                }
            }
        }

        return dp[m][n];
    }
}
```

<visual slug='longest-common-subsequence'/>

In the bottom-up solution, the way the `dp` array is defined differs slightly from our recursive solution, and there is an index offset because array indices start at 0. However, the thought process is entirely the same as our recursive solution. If you understand the recursive solution, this solution should not be difficult to understand.

Additionally, the bottom-up solution can be optimized using the [Dynamic Programming Space Compression Technique](https://labuladong.online/en/algo/dynamic-programming/space-optimization/) we discussed earlier, reducing the space complexity to O(N). Due to space limitations, we won't elaborate here.

Now, let's look at two problems similar to the longest common subsequence.


## Delete Operation for Strings

This is LeetCode Problem 583, "[Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/)". Let's look at the problem:

Given two words `s1` and `s2`, return the minimum number of steps required to make `s1` and `s2` the same. In each step, you can delete one character from either string.

The function signature is as follows:

```java
int minDistance(String s1, String s2);
```

For example, given `s1 = "sea"` and `s2 = "eat"`, the algorithm returns 2. In the first step, transform `"sea"` to `"ea"`, and in the second step, transform `"eat"` to `"ea"`.

The problem asks us to calculate the minimum number of deletions needed to make the two strings identical. We can think about what these two strings will look like after deletions.

The result of deletions is their longest common subsequence!

Therefore, to calculate the number of deletions, we can derive it from the length of the longest common subsequence:

```java
class Solution {
    public int minDistance(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // reuse the previous function to calculate the length of lcs
        int lcs = longestCommonSubsequence(s1, s2);
        return m - lcs + n - lcs;
    }

    // calculate the length of the longest common subsequence
    int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // define: the length of lcs for s1[0..i-1] and s2[0..j-1] as dp[i][j]
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // now i and j start from 1, so we need to subtract one
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    // s1[i-1] and s2[j-1] are definitely in the lcs
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    // at least one of s1[i-1] and s2[j-1] is not in the lcs
                    dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
                }
            }
        }
        return dp[m][n];
    }
}
```

And that's how we solve this problem!


## Minimum ASCII Delete Sum

This is LeetCode problem 712, ["Minimum ASCII Delete Sum for Two Strings"](https://leetcode.com/problems/minimum-ascii-delete-sum-for-two-strings/). The problem is similar to the previous one, except that the previous problem aimed to minimize the number of deletions, while this one aims to minimize the sum of the ASCII values of the deleted characters.

The function signature is as follows:

```java
int minimumDeleteSum(String s1, String s2)
```

For example, given `s1 = "sea", s2 = "eat"`, the algorithm returns 231.

This is because deleting `"s"` from `"sea"` and `"t"` from `"eat"` makes the two strings equal, with the minimum sum of the ASCII values of the deleted characters, i.e., `s(115) + t(116) = 231`.

**This problem cannot directly reuse the function for calculating the longest common subsequence, but by slightly modifying the base case and state transition part, you can directly write the solution code**:

```java
class Solution {

    // memoization
    int memo[][];

    // main function
    public int minimumDeleteSum(String s1, String s2) {
        int m = s1.length(), n = s2.length();
        // memo value of -1 indicates uncalculated
        memo = new int[m][n];
        for (int[] row : memo)
            Arrays.fill(row, -1);

        return dp(s1, 0, s2, 0);
    }

    // definition: delete s1[i..] and s2[j..] to make them the same string,
    // the minimum ascii sum is dp(s1, i, s2, j).
    int dp(String s1, int i, String s2, int j) {
        int res = 0;
        // base case
        if (i == s1.length()) {
            // if s1 is exhausted, delete the remaining characters of s2
            for (; j < s2.length(); j++)
                res += s2.charAt(j);
            return res;
        }
        if (j == s2.length()) {
            // if s2 is exhausted, delete the remaining characters of s1
            for (; i < s1.length(); i++)
                res += s1.charAt(i);
            return res;
        }

        if (memo[i][j] != -1) {
            return memo[i][j];
        }

        if (s1.charAt(i) == s2.charAt(j)) {
            // if s1[i] and s2[j] are in the longest common subsequence (lcs), no deletion needed
            memo[i][j] = dp(s1, i + 1, s2, j + 1);
        } else {
            // at least one of s1[i] and s2[j] is not in the lcs, delete one
            memo[i][j] = Math.min(
                    s1.charAt(i) + dp(s1, i + 1, s2, j),
                    s2.charAt(j) + dp(s1, i, s2, j + 1)
            );
        }
        return memo[i][j];
    }
}
```

<visual slug='minimum-ascii-delete-sum-for-two-strings'/>

The base case has some differences. When calculating the `lcs` length, if one string is empty, the `lcs` length is necessarily 0. However, in this problem, if one string is empty, all characters of the other string must be deleted, so you need to calculate the sum of the ASCII values of all characters in the other string.

Regarding state transition, when `s1[i]` and `s2[j]` are the same, no deletion is needed; when they are different, deletion is needed. Therefore, the `dp` function can be used to calculate both cases and derive the optimal result. The rest is similar, so it won't be elaborated.

Thus, the three subsequence problems are solved. The key is to break down the problem to the character level and determine whether each pair of characters should be included in the result subsequence, thereby avoiding brute-force enumeration of all subsequences.

This is a common approach to finding subsequences in two strings. It is recommended to understand it well and practice more~
