::: info Prerequisite Knowledge

Before reading this article, you need to study:

- [Core Framework of Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)
- [Classic Dynamic Programming: Edit Distance](https://labuladong.online/en/algo/dynamic-programming/edit-distance/)

:::

Regular expressions are a very powerful tool. In this article, we will look at the basic idea behind regular expressions. LeetCode problem 10: [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) asks us to implement a simple regular matching algorithm, including the "." wildcard and the "*" wildcard.

These two wildcards are used most often. The dot "." can match any single character. The star "*" means the character before it can repeat any number of times (including 0 times).

For example, the pattern `".a*b"` can match the text `"zaaab"` or `"cb"`. The pattern `"a..b"` matches `"amnb"`. The pattern `".*"` can match any text.

The problem gives us two strings `s` and `p`. `s` is the text, and `p` is the pattern. You need to check if the pattern `p` can match the text `s`. You can assume the pattern only contains lowercase letters and the two wildcards above. The pattern will always be valid; there won't be cases like `*a` or `b**`.

The function signature is as follows:

```java
boolean isMatch(string s, string p);
```

What is the hard part of this regular expression we need to implement?

The dot wildcard is easy. Any character in `s` can match a `.` in the pattern. The tricky part is the star wildcard. When you see "*", the character before it can be used any number of times: repeated once, many times, or not at all. What should we do?

The answer is simple: try all possible cases. If any case matches, then `p` matches `s`. Whenever we need to try all cases for two strings, we should think about using dynamic programming.

## 1. Idea Analysis

Let’s think about how `s` and `p` match. We use two pointers `i` and `j` to move along `s` and `p`. If both pointers reach the end, it means the match is successful. Otherwise, the match fails.

**If we ignore the "*" wildcard, when matching characters `s[i]` and `p[j]`, all we need to do is check if they match:**

```java
boolean isMatch(String s, String p) {
    int i = 0, j = 0;
    while (i < s.length() && j < p.length()) {
        // the '.' wildcard is a versatile match
        if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') {
            // match, continue to match s[i+1..] and p[j+1..]
            i++; j++;
        } else {
            // not a match
            return false;
        }
    }
    return i == j;
}
```

Now, if we add the "*" wildcard, things get a bit more complex, but we just need to consider each case separately.

**When `p[j + 1]` is "*", let’s look at the cases:**

1. If `s[i] == p[j]`, there are two possibilities:

1.1 `p[j]` might match multiple characters. For example, `s = "aaa", p = "a*"`, then `p[0]` matches all three `"a"`.

1.2 `p[j]` might match 0 characters. For example, `s = "aa", p = "a*aa"`. Since the rest of the pattern can match `s`, `p[0]` can match 0 times.

2. If `s[i] != p[j]`, there is only one case:

`p[j]` can only match 0 times, and we see if the next part can match `s[i]`. For example, `s = "aa", p = "b*aa"`, `p[0]` can only match 0 times.

So, we can update the code for the "*" wildcard as follows:

```java
if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') {
    // match
    if (j < p.length() - 1 && p.charAt(j + 1) == '*') {
        // there is a * wildcard, which can match 0 or more times
    } else {
        // no * wildcard, match exactly once
        i++; j++;
    }
} else {
    // do not match
    if (j < p.length() - 1 && p.charAt(j + 1) == '*') {
        // there is a * wildcard, can only match 0 times
    } else {
        // no * wildcard, the match cannot proceed
        return false;
    }
}
```

The idea is clear. But when we see the "*" wildcard, should we match 0 times or more? How many times?

This is a "choice" problem. We need to try all possible choices to get the answer. The core of dynamic programming is "state" and "choice". **The "state" is the position of the two pointers `i` and `j`. The "choice" is how many times `p[j]` matches a character.**

## 2. Dynamic Programming Solution

Based on the "state", we can define a `dp` function:

```java
boolean dp(String s, int i, String p, int j);
```


<!-- hide -->

The definition of the `dp` function is as follows:

**If `dp(s, i, p, j) = true`, it means `s[i..]` can match `p[j..]`. If `dp(s, i, p, j) = false`, it means `s[i..]` cannot match `p[j..]`.**

Based on this definition, the answer we want is the result of `dp` when `i = 0` and `j = 0`. So we use the `dp` function like this:

```java
boolean isMatch(String s, String p) {
    // pointers i and j start moving from index 0
    return dp(s, 0, p, 0);
}
```

We can write the main logic of the `dp` function based on the previous code:

```java
// core logic of the dp function (pseudocode)
// definition: the function returns whether s[i..] can match p[j..]
boolean dp(String s, int i, String p, int j) {
    if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') {
        // match
        if (j < p.length() - 1 && p.charAt(j + 1) == '*') {
            // 1.1 wildcard matches 0 times or multiple times
            return dp(s, i, p, j + 2) || dp(s, i + 1, p, j);
        } else {
            // 1.2 regular match 1 time
            return dp(s, i + 1, p, j + 1);
        }
    } else {
        // no match
        if (j < p.length() - 1 && p.charAt(j + 1) == '*') {
            // 2.1 wildcard matches 0 times
            return dp(s, i, p, j + 2);
        } else {
            // 2.2 cannot continue matching
            return false;
        }
    }
}
```

**According to the definition of the `dp` function**, these cases are easy to explain:

**1.1 Wildcard matches 0 or more times**

Add 2 to `j` and keep `i` the same. This means we skip `p[j]` and the wildcard after it, which is the case where the wildcard matches 0 times.

Even if `s[i] == p[j]`, this can still happen, as shown below:

![](../pictures/regex/5.jpeg)

Add 1 to `i` and keep `j` the same. This means `p[j]` matches `s[i]`, but `p[j]` can match more, which is the case where the wildcard matches multiple times:

![](../pictures/regex/2.jpeg)

If either case leads to a match, we return true, so we use "or" to combine both.

**1.2 Normal matching once**

This is the case without a `*` wildcard. If `s[i] == p[j]`, just add 1 to both `i` and `j`:

![](../pictures/regex/3.jpeg)

**2.1 Wildcard matches 0 times**

Similar to case 1.1, add 2 to `j` and keep `i` the same:

![](../pictures/regex/1.jpeg)

**2.2 If there is no `*` wildcard and cannot match, it means match failed**

![](../pictures/regex/4.jpeg)

Looking at the pictures makes it easy to understand. Now, let's think about the base cases of the `dp` function:

**One base case is when `j == p.length()`**. According to the definition of the `dp` function, it means the pattern string `p` has been matched. We need to see if the text string `s` has also been matched. If `s` is also matched, then it's a success:

```java
if (j == p.length()) {
    return i == s.length();
}
```

**Another base case is when `i == s.length()`**. According to the `dp` definition, this means the text string `s` has been completely matched. Can we just check if `p` is also matched?

```java
if (i == s.length()) {
    // Is this enough?
    return j == p.length();
}
```

**This is not correct. At this time, we cannot just check if `j` is equal to `p.length()`. As long as `p[j..]` can match an empty string, it counts as a successful match.** For example, for `s = "a", p = "ab*c*"`, when `i` reaches the end of `s`, `j` hasn't reached the end of `p`, but `p` can still match `s`.

So we can write code like this:

```java
int m = s.length(), n = p.length();

if (i == s.length()) {
    // if it can match an empty string, characters and * must appear in pairs
    if ((n - j) % 2 == 1) {
        return false;
    }
    // check if it is in the form of x*y*z*
    for (; j + 1 < p.length(); j += 2) {
        if (p.charAt(j + 1) != '*') {
            return false;
        }
    }
    return true;
}
```

With this idea, we can write the complete code:

```java
class Solution {
    // memoization
    private int[][] memo;

    public boolean isMatch(String s, String p) {
        int m = s.length(), n = p.length();
        memo = new int[m][n];
        for (int[] row : memo) {
            Arrays.fill(row, -1);
        }
        // pointers i and j start moving from index 0
        return dp(s, 0, p, 0);
    }

    // calculate whether p[j..] matches s[i..]
    private boolean dp(String s, int i, String p, int j) {
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

        // check memo to avoid redundant calculations
        if (memo[i][j] != -1) {
            return memo[i][j] == 1;
        }

        boolean res = false;

        if (s.charAt(i) == p.charAt(j) || p.charAt(j) == '.') {
            if (j < n - 1 && p.charAt(j + 1) == '*') {
                res = dp(s, i, p, j + 2)
                        || dp(s, i + 1, p, j);
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
        // store the current result in the memo
        memo[i][j] = res ? 1 : 0;
        return res;
    }
}
```

<visual slug='regular-expression-matching'/>

The code uses a hash table called `memo` to remove overlapping subproblems. How to spot overlapping subproblems? You can check [Dynamic Programming Q&A](https://labuladong.online/en/algo/dynamic-programming/faq-summary/) for techniques. Let's look at the recursive framework of the regex algorithm:

```java
boolean dp(String s, int i, String p, int j) {
    dp(s, i, p, j + 2);     // 1
    dp(s, i + 1, p, j);     // 2
    dp(s, i + 1, p, j + 1); // 3
    dp(s, i, p, j + 2);     // 4
}
```

If you want to get from `dp(s, i, p, j)` to `dp(s, i+2, p, j+2)`, there are at least two paths: `1 -> 2 -> 2` and `3 -> 3`. This means the state `(i+2, j+2)` will be calculated more than once. This shows there are overlapping subproblems, so we need a memo to remove them and make the code more efficient.

The time complexity of dynamic programming is "number of states" * "time per recursion". In this problem, the number of states is the combinations of `i` and `j`, which is `M * N` (`M` is the length of `s`, `N` is the length of `p`). The `dp` function has no loop (except for the base case, which doesn't happen often), so each recursion takes constant time. Multiply them, and the total time complexity is $O(MN)$.

The space complexity is just the size of the memo, which is $O(MN)$.
