::: info Prerequisites

Before reading this article, you need to study:

- [Binary Tree Algorithms Series (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)
- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

In the earlier article [Step-by-Step Guide to Binary Trees (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/), we divided recursive enumeration into two approaches: "Traversal" and "Decomposing Problems". The "Traversal" approach can be extended to [Backtracking Algorithms](https://labuladong.online/en/algo/essential-technique/backtrack-framework/), and the "Decomposing Problems" approach can be extended to [Dynamic Programming Algorithms](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/).

This shift in thinking is not limited to binary tree-related algorithms. In this article, we will step outside the realm of binary tree problems to see how to abstract problems into a tree structure in actual algorithm questions, and then optimize step-by-step through "Traversal" and "Decomposing Problems" approaches, smoothly transitioning from backtracking algorithms to dynamic programming algorithms.

As a quick aside, the previous article [Detailed Explanation of the Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/) stated that **standard dynamic programming problems always aim to find the optimal solution**. This is because dynamic programming problems have a property called "optimal substructure", meaning that the optimal solution to the overall problem can be derived from the optimal solutions of its subproblems.

However, in common parlance, even if a problem does not seek the optimal solution, as long as it uses a memoization technique to eliminate overlapping subproblems, we often call it a dynamic programming algorithm. Strictly speaking, this does not fit the definition of a dynamic programming problem. It might be more accurate to call such solutions "DFS algorithms with memoization". But we don't need to be too hung up on terminology. Since everyone is comfortable with the term, we can call it dynamic programming.

The two problems discussed in this article do not seek the optimal solution, but we will still refer to their solutions as dynamic programming solutions. This explanation is to prevent confusion for those who prefer precision. Without further ado, let's dive into the problems.


<!-- hide -->

## Word Break I

Let's first look at Leetcode problem 139, "[Word Break](https://leetcode.com/problems/word-break/)":

**LeetCode 139. Word Break** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given a string `s` and a dictionary of strings `wordDict`, return `true` if `s` can be segmented into a space-separated sequence of one or more dictionary words.

**Note** that the same word in the dictionary may be reused multiple times in the segmentation.

Example 1:**

```

**Input:** s = "leetcode", wordDict = ["leet","code"]
**Output:** true
**Explanation:** Return true because "leetcode" can be segmented as "leet code".

```

Example 2:**

```

**Input:** s = "applepenapple", wordDict = ["apple","pen"]
**Output:** true
**Explanation:** Return true because "applepenapple" can be segmented as "apple pen apple".
Note that you are allowed to reuse a dictionary word.

```

Example 3:**

```

**Input:** s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
**Output:** false

```

**Constraints:**

	
- `1 <= s.length <= 300`
	
- `1 <= wordDict.length <= 1000`
	
- `1 <= wordDict[i].length <= 20`
	
- `s` and `wordDict[i]` consist of only lowercase English letters.
	
- All the strings of `wordDict` are **unique**.

The function signature is as follows:

```java
boolean wordBreak(String s, List<String> wordDict);
```

This is a very frequently asked interview question. Let's think about how to solve it using "traversal" and "problem decomposition" strategies.

### Traversal Strategy (Backtracking Solution)

**First, let's discuss the "traversal" strategy, specifically using backtracking to solve this problem.** The most classic application of backtracking algorithms is in permutation and combination problems. It is not hard to see that this problem can be transformed into a permutation problem:

Now, given a word list `wordDict` with no duplicate words and a string `s`, determine whether you can select and arrange some words (words can be selected repeatedly) from `wordDict` to form the string `s`.

This is the last variant discussed in the previous article [Nine Variants of Permutation and Combination Problems Solved by Backtracking Algorithms](https://labuladong.online/en/algo/essential-technique/permutation-combination-subset-all-in-one/): the permutation problem with distinct elements that can be selected repeatedly. In the previous article, I wrote a `permuteRepeat` function, the code is as follows:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();

    public List<List<Integer>> permuteRepeat(int[] nums) {
        backtrack(nums);
        return res;
    }

    // core function of backtracking algorithm
    void backtrack(int[] nums) {
        // base case, reaching the leaf node
        if (track.size() == nums.length) {
            // collect the values at the leaf node
            res.add(new LinkedList(track));
            return;
        }

        // standard framework of backtracking algorithm
        for (int i = 0; i < nums.length; i++) {
            // make a choice
            track.add(nums[i]);
            // enter the next level of backtracking tree
            backtrack(nums);
            // undo the choice
            track.removeLast();
        }
    }
}
```

When you input `nums = [1,2,3]` to this function, the output is 3^3 = 27 possible combinations:


```java
[
  [1,1,1],[1,1,2],[1,1,3],[1,2,1],[1,2,2],[1,2,3],[1,3,1],[1,3,2],[1,3,3],
  [2,1,1],[2,1,2],[2,1,3],[2,2,1],[2,2,2],[2,2,3],[2,3,1],[2,3,2],[2,3,3],
  [3,1,1],[3,1,2],[3,1,3],[3,2,1],[3,2,2],[3,2,3],[3,3,1],[3,3,2],[3,3,3]
]
```

This code is actually traversing a full N-ary tree of height `N + 1` (`N` is the length of `nums`). Each path from the root to a leaf is a permutation result:

![](../pictures/word-break/1.jpeg)

Similarly, this problem is quite the same. Suppose `wordDict = ["a", "aa", "ab"], s = "aaab"`. If you want to use the words in `wordDict` to form `s`, it is like facing an M-ary tree, where `M` is the number of words in `wordDict`. **At each node of the backtracking tree, you need to check which word matches the prefix of `s[i..]`, and decide which branch to take:**

![](../pictures/word-break/2.jpeg)

As mentioned before in [Backtracking Algorithm Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/), you can think of the `backtrack` function as a pointer walking through the backtracking tree. It keeps track of the variable `i` at each node, so you can traverse the whole tree and find all combinations that match `s`.

Here is the backtracking solution code:

```java
class Solution {
    List<String> wordDict;
    // record whether a valid answer is found
    boolean found = false;
    // record the path of the backtracking algorithm
    LinkedList<String> track = new LinkedList<>();

    // main function
    public boolean wordBreak(String s, List<String> wordDict) {
        this.wordDict = wordDict;
        // execute the backtracking algorithm to enumerate all possible combinations
        backtrack(s, 0);
        return found;
    }

    // backtracking algorithm framework
    void backtrack(String s, int i) {
        // base case
        if (found) {
            // if an answer has been found, do not continue recursive search
            return;
        }
        if (i == s.length()) {
            // the entire s has been matched, a valid answer is found
            found = true;
            return;
        }

        // backtracking algorithm framework
        for (String word : wordDict) {
            // check which word can match the prefix of s[i..]
            int len = word.length();
            if (i + len <= s.length()
                && s.substring(i, i + len).equals(word)) {
                // found a word that matches s[i..i+len)
                // make a choice
                track.addLast(word);
                // enter the next level of the backtracking tree, continue matching s[i+len..]
                backtrack(s, i + len);
                // undo the choice
                track.removeLast();
            }
        }
    }
}
```

This code follows the backtracking framework strictly. It should be easy to understand. But this code cannot pass all test cases. Let's analyze its time complexity using the method from [Time Complexity Guide](https://labuladong.online/en/algo/essential-technique/complexity-analysis/).

A rough way to estimate the time complexity of a recursive function is: number of recursive calls (number of nodes in the recursion tree) x complexity of the function itself. In this problem, each node in the recursion tree is one way to cut `s`. In the worst case, how many ways to cut `s`? A string `s` of length `N` has `N - 1` "gaps" to cut. Each gap can be "cut" or "not cut", so there are up to $O(2^N)$ ways, meaning up to $O(2^N)$ nodes in the recursion tree.

Of course, in practice, things are better because there is pruning logic. But in terms of worst-case complexity, the number of nodes is exponential.

How about the complexity of the `backtrack` function itself? The main cost is looping over `wordDict` to find a word that matches the prefix of `s[i..]`:

```java
// Traverse all words in wordDict
for (String word : wordDict) {
    // Check which word can match the prefix of s[i..]
    int len = word.length();
    if (i + len <= s.length()
        && s.substring(i, i + len).equals(word)) {
        // Found a word that matches s[i..i+len)
        // ...
    }
}
```

Let `M` be the length of `wordDict`, and `N` the length of `s`. The worst-case time complexity here is $O(MN)$ (for loop is $O(M)$, Java's `substring` is $O(N)$). So in total, the time complexity is $O(2^N * MN)$.

Here is a small optimization: you can also enumerate prefixes of `s[i..]` and check if `wordDict` contains them:

```java
// Note that we need to convert it into a HashSet to improve the efficiency of the contains method
HashSet<String> wordDict = new HashSet<>(wordDict);

// Traverse all prefixes of s[i..]
for (int len = 1; i + len <= s.length(); len++) {
    // Check if there is a word in wordDict that can match the prefix of s[i..]
    String prefix = s.substring(i, i + len);
    if (wordDict.contains(prefix)) {
        // Found a word that matches s[i..i+len)
        // ...
    }
}
```

This code gives the same result as before, but its time complexity is $O(N^2)$, which is different from the previous code.

Which is better? It depends on the problem's data range. Here, `1 <= s.length <= 300, 1 <= wordDict.length <= 1000`, so $O(N^2)$ is smaller. This code should run a bit faster. You can try it yourself; I won't write it out here.

But even if you optimize this part, the total time complexity is still exponential, $O(2^N * N^2)$, so this approach cannot pass all test cases. Where is the problem?

For example, input `wordDict = ["a", "aa", "b"], s = "aaab"`. Notice that the backtracking algorithm will have repeated cases:

![](../pictures/word-break/3.jpeg)

The red-marked parts are different cuts but end up with the same `"aab"`. So the subtrees below these two nodes repeat, which means redundant computation. This is why the complexity is exponential.


### Optimizing with Postorder Position

For any algorithm, the way to remove redundant calculations is to use a memo. Backtracking can also use a memo, which we call "pruning". This means we cut off the redundant subtrees.

For example, when facing the substring `"aab"`, we want the memo to tell us if `"aab"` can be split successfully. If we already tried and found it cannot be split, we can skip it and do not need to explore its subtrees, which improves efficiency. If it can be split, we do not need to care about the memo, because `found == true` is a base case, and the whole recursion will stop.

As explained in [General Rules for Binary Tree/Recursion Algorithms](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/), if we want the memo to work this way, we need to update the memo at the postorder position, because `"aab"` is actually a subtree. **You need to record in the memo whether the current subtree can be split successfully after visiting all its children.**

The backtracking function `backtrack` is used for traversal, and does not return a value, so there is no information passed back from the subtree. But for this problem, we have a way, because we have an external variable `found`. This variable can tell us if the subtree can be split successfully:

**If `found` is false, it means we have not found a successful split, which also means the current subtree cannot be split.** Now we can write this result in the memo, so next time we can skip redundant brute-force searching.

The actual code only needs a small change to add the memo:

```java
class Solution {
    // memoization, store substrings (subtrees) that cannot be split, to avoid redundant calculations
    HashSet<String> memo = new HashSet<>();

    // ...

    void backtrack(String s, int i) {
        if (found) {
            return;
        }
        if (i == s.length()) {
            found = true;
            return;
        }

        // added pruning logic, check if the substring (subtree) has been calculated
        String suffix = s.substring(i);
        if (memo.contains(suffix)) {
            // the current substring (subtree) cannot be split, no need to continue recursion
            return;
        }

        for (String word : wordDict) {
            // ...
        }

        // post-order position, record substrings (subtrees) that cannot be split in the memo
        if (!found) {
            memo.add(suffix);
        }
    }
}
```


### Thinking of Problem Decomposition (Dynamic Programming)

The last solution used the backtracking algorithm because the problem is simple. We used a `found` variable to update the memoization at the postorder position. But for more complex problems, like Word Break II, this method will not work.

To update the memoization with the answer of a subtree, it is better to use the return value of the recursive function. This is the "problem decomposition" way of thinking.

Before, we tried to solve this problem from a permutation and combination perspective. Now, let's see if we can break the original problem into smaller, similar subproblems, and use their answers to solve the original problem.

For the input string `s`, if we can find a word from `wordDict` that matches the prefix `s[0..k]`, then as long as we can form `s[k+1..]`, we can form the whole string `s`. In other words, we have broken the big problem `wordBreak(s[0..])` into a smaller subproblem `wordBreak(s[k+1..])`, and then use the answer of the subproblem to get the answer of the original problem.

With this idea, we can define a `dp` function as follows:

```java
// Definition: return whether s[i..] can be composed
int dp(String s, int i);

// Calculate whether the entire s can be composed, call dp(s, 0)
```

With this function definition, we can translate the above logic into pseudocode:

```java
List<String> wordDict;

// Definition: return whether s[i..] can be formed
int dp(String s, int i) {
    // base case, s[i..] is an empty string
    if (i == s.length()) {
        return true;
    }
    // traverse wordDict to see which words are prefixes of s[i..]
    for (Strnig word : wordDict) {
        // word is a prefix of s[i..]
        if (s.substring(i).startsWith(word)) {
            int len = word.length();
            // as long as s[i+len..] can be formed, s[i..] can be formed
            if (dp(s, i + len) == true) {
                return true;
            }
        }
    }
    // all words have been tried, cannot form the entire s
    return false;
}
```

Just like in the backtracking solution, the for loop in the `dp` function can also be optimized:

```java
// Note that we use a hash set to quickly determine if an element exists
HashSet<String> wordDict;

// Definition: return whether s[i..] can be formed
int dp(String s, int i) {
    // base case, s[i..] is an empty string
    if (i == s.length()) {
        return true;
    }
    
    // Traverse all prefixes of s[i..] to see which prefixes exist in wordDict
    for (int len = 1; i + len <= s.length(); len++) {
        // s[i..len) exists in wordDict
        if (wordDict.contains(s.substring(i, i + len))) {
            // As long as s[i+len..] can be formed, s[i..] can be formed
            if (dp(s, i + len) == true) {
                return true;
            }
        }
    }
    // All words have been tried, and the entire s cannot be formed
    return false;
}
```

For this `dp` function, the position of the pointer `i` is the "state". So we can use a memoization table to avoid recalculating the same subproblems, making the solution faster. Here is the final code:

```java
class Solution {
    // Use a hash set to facilitate quick existence checks
    HashSet<String> wordDict;
    // Memoization, -1 represents uncomputed, 0 represents cannot form, 1 represents can form
    int[] memo;

    // Main function
    public boolean wordBreak(String s, List<String> wordDict) {
        // Convert to a hash set for quick element existence checks
        this.wordDict = new HashSet<>(wordDict);
        // Initialize memoization array to -1
        this.memo = new int[s.length()];
        Arrays.fill(memo, -1);
        return dp(s, 0);
    }

    // Definition: whether s[i..] can be formed
    boolean dp(String s, int i) {
        // base case
        if (i == s.length()) {
            return true;
        }
        // Prevent redundant calculations
        if (memo[i] != -1) {
            return memo[i] == 0 ? false : true;
        }

        // Traverse all prefixes of s[i..]
        for (int len = 1; i + len <= s.length(); len++) {
            // Check which prefixes exist in wordDict
            String prefix = s.substring(i, i + len);
            if (wordDict.contains(prefix)) {
                // Found a word that matches s[i..i+len)
                // As long as s[i+len..] can be formed, s[i..] can be formed
                boolean subProblem = dp(s, i + len);
                if (subProblem == true) {
                    memo[i] = 1;
                    return true;
                }
            }
        }
        // s[i..] cannot be formed
        memo[i] = 0;
        return false;
    }
}
```

::: tip More Optimization

Notice when we compute the `prefix`, we use the substring function provided by the language, which has time complexity $O(N)$. The start index is always `i`, and the end index increases with `j`. We can manually maintain the `prefix` substring to avoid calling the substring function, which will make it a bit faster. You can try this small optimization yourself.

:::

<visual slug='word-break'/>

This solution can pass all test cases. According to the [Algorithm Time and Space Complexity Guide](https://labuladong.online/en/algo/essential-technique/complexity-analysis/), let's calculate its time complexity:

With memoization, duplicate nodes in the recursion tree are removed. The number of function calls drops from exponential to $O(N)$ (number of states). Each function call has complexity $O(N^2)$, so the total time complexity is $O(N^3)$, which is much better than backtracking.


## Word Break II

With the last problem as a foundation, LeetCode 140 "[Word Break II](https://leetcode.com/problems/word-break-ii/)" becomes much easier. Let's look at the problem:

**LeetCode 140. Word Break II** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

Given a string `s` and a dictionary of strings `wordDict`, add spaces in `s` to construct a sentence where each word is a valid dictionary word. Return all such possible sentences in **any order**.

**Note** that the same word in the dictionary may be reused multiple times in the segmentation.

Example 1:**

```

**Input:** s = "catsanddog", wordDict = ["cat","cats","and","sand","dog"]
**Output:** ["cats and dog","cat sand dog"]

```

Example 2:**

```

**Input:** s = "pineapplepenapple", wordDict = ["apple","pen","applepen","pine","pineapple"]
**Output:** ["pine apple pen apple","pineapple pen apple","pine applepen apple"]
**Explanation:** Note that you are allowed to reuse a dictionary word.

```

Example 3:**

```

**Input:** s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
**Output:** []

```

**Constraints:**

	
- `1 <= s.length <= 20`
	
- `1 <= wordDict.length <= 1000`
	
- `1 <= wordDict[i].length <= 10`
	
- `s` and `wordDict[i]` consist of only lowercase English letters.
	
- All the strings of `wordDict` are **unique**.
	
- Input is generated in a way that the length of the answer doesn't exceed 10^(5).

Compared to the last problem, this question not only asks if `s` can be broken up, but also how to break it up. In fact, you just need to change the previous solution a little to solve this problem.

### Traversal Approach (Backtracking)

In the last problem, the backtracking algorithm used a `found` variable. Once it found a way to break up the string, it stopped searching. For this problem, we should not stop early. Instead, we need to collect all possible ways to break up the string as answers:

```java
class Solution {
    // record the results
    List<String> res = new LinkedList<>();
    // record the path of the backtracking algorithm
    LinkedList<String> track = new LinkedList<>();
    List<String> wordDict;

    // main function
    public List<String> wordBreak(String s, List<String> wordDict) {
        this.wordDict = wordDict;
        // execute the backtracking algorithm to enumerate all possible combinations
        backtrack(s, 0);
        return res;
    }

    // backtracking algorithm framework
    void backtrack(String s, int i) {
        // base case
        if (i == s.length()) {
            // find a valid combination that spells out the entire s, convert to string
            res.add(String.join(" ", track));
            return;
        }

        // backtracking algorithm framework
        for (String word : wordDict) {
            // see which word can match the prefix of s[i..]
            int len = word.length();
            if (i + len <= s.length()
                && s.substring(i, i + len).equals(word)) {
                // find a word that matches s[i..i+len)
                // make a choice
                track.addLast(word);
                // enter the next level of the backtracking tree, continue matching s[i+len..]
                backtrack(s, i + len);
                // undo the choice
                track.removeLast();
            }
        }
    }
}
```

The time complexity of this solution is similar to the previous one: $O(2^N * MN)$. But since the data size for this problem is small, it can still pass all test cases.

### Can We Optimize with Postorder Position?

Like before, this solution still has room for optimization:

![](../pictures/word-break/3.jpeg)

For repeated subtrees, there will still be unnecessary repeated calculations. We can use a memo to cache the split results of substring `"aab"` to avoid this.

However, it is hard to add memoization in a backtracking algorithm, because the `track` variable only keeps track of the path from the root to the current node, and does not record information about subtrees.

So, to remove overlapping subproblems, we usually use a "divide problem" approach and update the memo with the function's return value.

### Divide Problem Approach (Dynamic Programming)

This problem can also be solved by dividing the problem. We just need to slightly change the `dp` function from the previous problem:

```java
class Solution {
    HashSet<String> wordDict;
    // memoization
    List<String>[] memo;

    public List<String> wordBreak(String s, List<String> wordDict) {
        this.wordDict = new HashSet<>(wordDict);
        memo = new List[s.length()];
        return dp(s, 0);
    }

    // definition: returns all possible constructions of s[i..] using wordDict
    List<String> dp(String s, int i) {
        List<String> res = new LinkedList<>();
        if (i == s.length()) {
            res.add("");
            return res;
        }
        // prevent redundant calculations
        if (memo[i] != null) {
            return memo[i];
        }
        
        // traverse all prefixes of s[i..]
        for (int len = 1; i + len <= s.length(); len++) {
            // check which prefixes exist in wordDict
            String prefix = s.substring(i, i + len);
            if (wordDict.contains(prefix)) {
                // found a word matching s[i..i+len)
                List<String> subProblem = dp(s, i + len);
                // all combinations of s[i+len..] plus prefix
                // are all combinations of s[i]
                for (String sub : subProblem) {
                    if (sub.isEmpty()) {
                        // prevent extra spaces
                        res.add(prefix);
                    } else {
                        res.add(prefix + " " + sub);
                    }
                }
            }
        }
        // store in memo
        memo[i] = res;
        
        return res;
    }
}
```

<visual slug='word-break-ii'/>

This solution also uses a memo to remove overlapping subproblems, so the number of recursive calls for the `dp` function is reduced to $O(N)$. But the complexity of the `dp` function itself increases, because `subProblem` is a list of subsets and its length is exponential.

Also, string concatenation is not very efficient, and memo needs to store all subproblem results. So, from a Big O point of view, the time complexity of this algorithm is not lower than the backtracking algorithm. It is still exponential. But this solution does remove overlapping subproblems, so it is a bit better than backtracking.

In summary, when we solve permutation and combination problems, we usually use backtracking to "traverse" the tree, not the divide problem approach. That's because saving the results of subproblems takes a lot of time and space. Unless there are many overlapping subproblems, it's not worth it.

That’s all for this article. I hope you now have a better understanding of the backtracking approach and the divide problem approach.
