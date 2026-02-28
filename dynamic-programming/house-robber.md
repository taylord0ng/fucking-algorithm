::: info Prerequisites

Before reading this article, you should first learn:

- [Binary Tree Algorithms (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)
- [Core Framework of Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

Today, let's talk about the "House Robber" series of problems. This is a classic and skillful dynamic programming problem.

The House Robber series has three problems, and the difficulty increases step by step. The first is a standard dynamic programming problem. The second adds a circular array condition. The third combines bottom-up and top-down dynamic programming with binary trees, which is very inspiring.

Let's start with the first problem.

## House Robber I

LeetCode 198 "[House Robber](https://leetcode.com/problems/house-robber/)" is described as follows:

There is a row of houses. Each house has some cash, given as a non-negative integer array `nums`, where `nums[i]` is the amount in the i-th house. You are a professional thief. You want to steal as much cash as possible, but **you cannot steal from two adjacent houses**. Otherwise, the alarm will go off.

Write an algorithm to calculate the maximum amount of cash you can steal without triggering the alarm. The function signature is:

```java
int rob(int[] nums);
```

For example, if the input is `nums = [2,1,7,9,3,1]`, the algorithm should return 12. The thief can steal from `nums[0]`, `nums[3]`, and `nums[5]`, getting 2 + 9 + 1 = 12, which is the best choice.

The problem is easy to understand, and it is clearly a dynamic programming problem. As summarized in [Dynamic Programming Details](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), **to solve a dynamic programming problem, you just need to find the "state" and the "choices".**


<!-- hide -->

Imagine you are a professional robber. You walk from left to right along a row of houses. At each house, you have two choices: rob it or skip it.

If you rob this house, you cannot rob the next house. You must start making choices again from the house after the next.

If you skip this house, you can go to the next house and keep making choices.

When you have passed the last house, there is nothing left to rob, so the money you can get is 0 (this is the base case).

This logic is simple. It shows the "state" and the "choices": the index of the house in front of you is the state, and robbing or skipping is your choice.

![](../pictures/robber/1.jpg)

Each time, pick the bigger result between the two choices. In the end, you get the maximum amount of money you can rob:

```java
class Solution {
    // main function
    public int rob(int[] nums) {
        return dp(nums, 0);
    }

    // definition: return the maximum value that can be robbed from nums[start..]
    private int dp(int[] nums, int start) {
        if (start >= nums.length) {
            return 0;
        }
        
        int res = Math.max(
                // do not rob, go to the next house
                dp(nums, start + 1), 
                // rob, go to the house after the next
                nums[start] + dp(nums, start + 2)
            );
        return res;
    }
}
```

After understanding the state transitions, you can see there are overlapping subproblems for the same `start` position. For example:

![](../pictures/robber/2.jpg)

There are many ways for the robber to reach this position. If you enter recursion every time, you waste time. So, there are overlapping subproblems, and you can use memoization to optimize it:

```java
class Solution {

    private int[] memo;
    // main function
    public int rob(int[] nums) {
        // initialize the memoization array
        memo = new int[nums.length];
        Arrays.fill(memo, -1);
        // the robber starts robbing from the 0th house
        return dp(nums, 0);
    }

    // definition: return the maximum amount that can be robbed from dp[start..]
    private int dp(int[] nums, int start) {
        if (start >= nums.length) {
            return 0;
        }
        // avoid repeated calculation
        if (memo[start] != -1) return memo[start];
        
        int res = Math.max(
            dp(nums, start + 1), 
            dp(nums, start + 2) + nums[start]
        );
        // record in the memoization array
        memo[start] = res;
        return res;
    }
}
```

<visual slug='house-robber'/>

This is the top-down dynamic programming solution. We can also make some changes to write a **bottom-up** solution:

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        // dp[i] = x means:
        // starting from the i-th house, the maximum amount of money that can be robbed is x
        // base case: dp[n] = 0
        int[] dp = new int[n + 2];
        for (int i = n - 1; i >= 0; i--) {
            dp[i] = Math.max(dp[i + 1], nums[i] + dp[i + 2]);
        }
        return dp[0];
    }
}
```

We can see that the state transition only depends on the last two states `dp[i]`. So, we can further optimize and reduce the space complexity to O(1):

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        // record dp[i+1] and dp[i+2]
        int dp_i_1 = 0, dp_i_2 = 0;
        // record dp[i]
        int dp_i = 0; 
        for (int i = n - 1; i >= 0; i--) {
            dp_i = Math.max(dp_i_1, nums[i] + dp_i_2);
            dp_i_2 = dp_i_1;
            dp_i_1 = dp_i;
        }
        return dp_i;
    }
}
```

This process is explained in detail in our [Dynamic Programming Detailed Explanation](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/). I believe you can master it easily. What is interesting is the follow-up to this problem, which needs some clever changes based on our current thinking.


## House Robber II

LeetCode Problem 213 "[House Robber II](https://leetcode.com/problems/house-robber-ii/)" is almost the same as the previous problem. The thief still cannot rob two adjacent houses, and the input is still an array. But now, the houses are arranged in a circle, not in a straight line.

This means the first and last houses are also neighbors, so they cannot both be robbed. For example, if the input array is `nums=[2,3,2]`, the answer should be 3, not 4, because you cannot rob both the first and last house.

This new rule is not hard to handle. In our previous article [Monotonic Stack Problems](https://labuladong.online/en/algo/data-structure/monotonic-stack/), we talked about how to deal with circular arrays. So how do we solve this here?

Since the first and last houses cannot be robbed together, there are only three cases:
1. Rob neither the first nor the last house;
2. Rob the first house, but not the last;
3. Rob the last house, but not the first.

![](../pictures/robber/3.jpg)

It's simple. Just find the maximum result among these three cases. Actually, we only need to compare case 2 and case 3. **These two cases give us more choices than case 1. Since all house values are non-negative, more choices mean the result will not be worse.**

So we only need to slightly change the previous solution:

```java
class Solution {
    public int rob(int[] nums) {
        int n = nums.length;
        if (n == 1) return nums[0];
        return Math.max(robRange(nums, 0, n - 2), 
                        robRange(nums, 1, n - 1));
    }

    // Definition: return the maximum value that can be robbed in the closed interval [start, end]
    int robRange(int[] nums, int start, int end) {
        int n = nums.length;
        int dp_i_1 = 0, dp_i_2 = 0;
        int dp_i = 0;
        for (int i = end; i >= start; i--) {
            dp_i = Math.max(dp_i_1, nums[i] + dp_i_2);
            dp_i_2 = dp_i_1;
            dp_i_1 = dp_i;
        }
        return dp_i;
    }
}
```

Now, the second problem is solved.


## House Robber III

LeetCode Problem 337 ["House Robber III"](https://leetcode.com/problems/house-robber-iii/) gives a new twist to the problem. The robber now faces houses arranged as a binary tree, not in a row or a circle. The houses are the nodes of the tree. Two connected houses cannot be robbed at the same time. This really is a smart thief! The function signature is as follows:

```java
int rob(TreeNode root);
```

For example, if the input is the following binary tree:

```yaml BinaryTree
root:
  value: 3
  left:
    value: 2
    right:
      value: 3
  right:
    value: 3
    right:
      value: 1
```

The algorithm should return 7, because robbing the first and third levels gives the highest amount: 3 + 3 + 1 = 7.

If the input is this binary tree:

```yaml BinaryTree
root:
  value: 3
  left:
    value: 4
    left:
      value: 1
    right:
      value: 3
  right:
    value: 5
    right:
      value: 1
```

The algorithm should return 9, because robbing the second level gets the most money: 4 + 5 = 9.

The main idea is still the same: at each node, we choose to rob it or not, and pick the better option. We can write the code directly based on this idea:

```java
class Solution {
    Map<TreeNode, Integer> memo = new HashMap<>();
    public int rob(TreeNode root) {
        if (root == null) return 0;
        // use memoization to eliminate overlapping subproblems
        if (memo.containsKey(root)) 
            return memo.get(root);
        // rob this house, then go to the next next house
        int do_it = root.val
            + (root.left == null ? 
                0 : rob(root.left.left) + rob(root.left.right))
            + (root.right == null ? 
                0 : rob(root.right.left) + rob(root.right.right));
        // do not rob this house, then go to the next house
        int not_do = rob(root.left) + rob(root.right);
        
        int res = Math.max(do_it, not_do);
        memo.put(root, res);
        return res;
    }
}
```

<visual slug='house-robber-iii'/>

Let's look at the time complexity. Although the recursion looks like a four-way tree, with memoization, each node is only visited once. So the time complexity is $O(N)$, where $N$ is the number of nodes in the tree. The space complexity is also $O(N)$ because of the memoization.

If you are confused about time or space complexity, you can read [A Practical Guide to Time and Space Complexity](https://labuladong.online/en/algo/essential-technique/complexity-analysis/).

But there is an even better solution. For example, one reader commented with this solution:

```java
class Solution {
    int rob(TreeNode root) {
        int[] res = dp(root);
        return Math.max(res[0], res[1]);
    }

    // return an array of size 2, arr
    // arr[0] represents the maximum amount of money obtained without robbing root
    // arr[1] represents the maximum amount of money obtained by robbing root
    int[] dp(TreeNode root) {
        if (root == null)
            return new int[]{0, 0};
        int[] left = dp(root.left);
        int[] right = dp(root.right);
        // if we rob, the next house cannot be robbed
        int rob = root.val + left[0] + right[0];
        // if we do not rob, the next house can either be robbed or not, depending on the profit
        int not_rob = Math.max(left[0], left[1])
                    + Math.max(right[0], right[1]);
        
        return new int[]{not_rob, rob};
    }
}
```

The time complexity is still $O(N)$, but the space complexity is only the stack space for the recursion, which is the height of the tree $O(H)$. No extra space for memoization is needed.

This solution is a bit different. It changes the definition of the recursive function and adjusts the logic, but still gets the right answer with cleaner code. This is a good example of using post-order traversal, which we discussed in [Binary Tree Thinking (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/).

In fact, this solution is much faster in practice, even though the time complexity is the same. This is because it does not use extra memoization, so it has less overhead and is more efficient.
