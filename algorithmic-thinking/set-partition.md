::: info Prerequisite Knowledge

Before reading this article, you should first learn:

- [N-ary Tree Structure and Traversal Framework](https://labuladong.online/en/algo/data-structure-basic/n-ary-tree-traverse-basic/)
- [Binary Tree Algorithms (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)
- [Backtracking Algorithm Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/)
- [Ball-and-Box Model: Two Brute-Force Views of Backtracking](https://labuladong.online/en/algo/practice-in-action/two-views-of-backtrack/)

:::

I said before that backtracking is the most useful algorithm in written tests. When you have no idea, just use backtracking to do a brute-force search. Even if you cannot pass all test cases, you can still pass some of them. The basic skill of backtracking is not hard: you brute-force a decision tree, and for each step, you "make a choice" before recursion and "undo the choice" after recursion.

**However, even for brute-force search, some ideas are better than others.** In this article, we look at a classic backtracking problem: LeetCode 698, “[Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/)”. This problem can help you understand the backtracking mindset more deeply and write backtracking functions more easily.

The problem is very simple:

You are given an array `nums` and a positive integer `k`. Please check whether `nums` can be divided into `k` subsets such that the sum of elements in each subset is the same.

The function signature is:

```java
boolean canPartitionKSubsets(int[] nums, int k);
```

::: info Thinking Question

Earlier, in [Subset Partition as a Knapsack Problem](https://labuladong.online/en/algo/dynamic-programming/knapsack2/), we solved a subset partition problem. But that problem only asked us to split the set into two equal subsets, and we could turn it into a knapsack problem and solve it with dynamic programming.

Why can partitioning into two equal subsets be transformed into a knapsack problem and solved with dynamic programming, but partitioning into `k` equal subsets cannot be transformed this way and must be solved by brute-force backtracking? Please think about this yourself first.

:::

::: details Answer to the Thinking Question

Why can partitioning into two equal subsets be transformed into a knapsack problem?

In the setting of [Subset Partition as a Knapsack Problem](https://labuladong.online/en/algo/dynamic-programming/knapsack2/), we have one knapsack and several items. Each item has **two choices**: "put it into the knapsack" or "do not put it into the knapsack".  

When we split the original set `S` into two equal subsets `S_1` and `S_2`, each element in `S` also has **two choices**: "put it into `S_1`" or "do not put it into `S_1` (put it into `S_2`)". The brute-force idea here is actually the same as in the knapsack problem.

But if you want to split `S` into `k` equal subsets, then each element in `S` has **`k` choices**. This is essentially different from the standard knapsack setting, so you cannot directly use the knapsack DP approach. You have to use backtracking and brute-force search.

:::


<!-- hide -->

## Problem Idea

Now back to this problem. We need to partition the array into subsets. Subset problems are different from permutation and combination problems, but we can still use the “balls and boxes model” to think about it from two different views.

We want to split an array `nums` with `n` numbers into `k` subsets with the same sum. You can imagine putting `n` numbers into `k` “buckets”, and at the end, the sum of numbers in each of the `k` buckets must be the same.

In the earlier article [Understanding Backtracking with the Balls and Boxes Model](https://labuladong.online/en/algo/practice-in-action/two-views-of-backtrack/), we said the key of backtracking is:

You must know how to “make choices”, then you can use recursion to do brute-force search.

If we imitate the way we derive the permutation formula, and assign `n` numbers into `k` buckets, we can also have two views:

**First view: from the `n` numbers’ perspective, each number must choose one of the `k` buckets to go into.**

![](../pictures/set-split/5.jpeg)

**Second view: from the `k` buckets’ perspective, for each bucket, you go through all `n` numbers in `nums`, and decide whether to put the current number into this bucket.**

![](../pictures/set-split/6.jpeg)

You may ask, what is the difference between these two views?

Just like in the permutation and subset problems, using different views to do brute-force search gives the same result, but the code logic is different. The concrete implementation is different, and the time and space complexity may also be different. We want to choose the solution with lower complexity.

## Number-based view

Everyone knows how to iterate through the `nums` array with a for loop:

```java
for (int index = 0; index < nums.length; index++) {
    print(nums[index]);
}
```

Can you traverse the array with recursion? It is also simple:

```java
void traverse(int[] nums, int index) {
    if (index == nums.length) {
        return;
    }
    print(nums[index]);
    traverse(nums, index + 1);
}
```

If you call `traverse(nums, 0)`, it has the same effect as the for loop.

Now back to this problem. From the numbers’ view, each number chooses one of `k` buckets. The for-loop version looks like this:

```java
// k buckets (collections), record the sum of numbers in each bucket
int[] bucket = new int[k];

// enumerate each number in nums
for (int index = 0; index < nums.length; index++) {
    // enumerate each bucket
    for (int i = 0; i < k; i++) {
        // decide whether nums[index] should go into the i-th bucket
        // ...
    }
}
```

If we change it to a recursive version, the logic becomes:

```java
// k buckets (collections), record the sum of numbers in each bucket
int[] bucket = new int[k];

// enumerate each number in nums
void backtrack(int[] nums, int index) {
    // base case
    if (index == nums.length) {
        return;
    }
    // enumerate each bucket
    for (int i = 0; i < bucket.length; i++) {
        // choose to put it into the i-th bucket
        bucket[i] += nums[index];
        // recursively enumerate the next number's choice
        backtrack(nums, index + 1);
        // undo the choice
        bucket[i] -= nums[index];
    }
}
```

The above code is only the brute-force structure, it cannot solve the problem yet. But we can fix it with some small changes:

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        // exclude some base cases
        if (k > nums.length) return false;
        int sum = 0;
        for (int v : nums) sum += v;
        if (sum % k != 0) return false;

        // k buckets (sets), record the sum of numbers in each bucket
        int[] bucket = new int[k];
        // the theoretical sum of numbers in each bucket
        int target = sum / k;
        // try all possibilities, to see if nums can be divided into k subsets with sum of target
        return backtrack(nums, 0, bucket, target);
    }

    // recursively try each number in nums
    boolean backtrack(
        int[] nums, int index, int[] bucket, int target) {

        if (index == nums.length) {
            // check if the sum of numbers in all buckets is target
            for (int i = 0; i < bucket.length; i++) {
                if (bucket[i] != target) {
                    return false;
                }
            }
            // successfully divide nums into k subsets
            return true;
        }
        
        // try all possible buckets for nums[index]
        for (int i = 0; i < bucket.length; i++) {
            // pruning, the bucket is full
            if (bucket[i] + nums[index] > target) {
                continue;
            }
            // put nums[index] into bucket[i]
            bucket[i] += nums[index];
            /**<extend down -200>
            ![](../pictures/set-split/5.jpeg)
            */
            // recursively try the next number's choices
            if (backtrack(nums, index + 1, bucket, target)) {
                return true;
            }
            // undo the choice
            bucket[i] -= nums[index];
        }

        // nums[index] cannot be put into any bucket
        return false;
    }
}
```

With the earlier explanation, this code should be easy to understand. In fact, we can still do one more optimization.

Look at the recursive part of the `backtrack` function:

```java
for (int i = 0; i < bucket.length; i++) {
    // pruning
    if (bucket[i] + nums[index] > target) {
        continue;
    }

    if (backtrack(nums, index + 1, bucket, target)) {
        return true;
    }
}
```

**If we can make more cases hit that pruning `if` branch, we can reduce the number of recursive calls, and thus reduce the time complexity to some extent.**

How can we make more cases hit this `if` branch? Note that our `index` parameter increases from 0, which means we go through the `nums` array from 0 in the recursion.

If we sort the `nums` array in advance and put larger numbers in front, then larger numbers will be put into the `bucket` first. After that, for the remaining numbers, `bucket[i] + nums[index]` will be larger, so the pruning `if` condition is easier to trigger.

So we can add some code to the previous solution:

```java
boolean canPartitionKSubsets(int[] nums, int k) {
    // other code remains unchanged
    // ...
    // sort the nums array in descending order
    Arrays.sort(nums);
    // reverse the array to get the descending order array
    for (i = 0, j = nums.length - 1; i < j; i++, j--) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    // *****************
    return backtrack(nums, 0, bucket, target);
}
```

This solution is correct, but it is still slow and cannot pass all test cases. Next we will look at the solution from the other view.


## The Bucket Perspective

At the beginning of the article, we said: **from the bucket perspective, we use brute-force. For each bucket, we scan all numbers in `nums` and decide whether to put the current number into this bucket. After one bucket is full, we continue to fill the next bucket, until all buckets are full**.

We can express this idea with the following code:

```java
// fill all the buckets until
while (k > 0) {
    // record the sum of numbers in the current bucket
    int bucket = 0;
    for (int i = 0; i < nums.length; i++) {
        // decide whether to put nums[i] into the current bucket
        if (canAdd(bucket, num[i])) {
            bucket += nums[i];
        }
        if (bucket == target) {
            // filled one bucket, move on to the next one
            k--;
            break;
        }
    }
}
```

We can also rewrite this `while` loop as a recursive function. It is a bit more complex than before. First, we write a `backtrack` function:

```java
boolean backtrack(int k, int bucket, int[] nums, int start, boolean[] used, int target);
```

Do not be scared by so many parameters. I will explain them one by one. If you fully understood the earlier article [Understanding Backtracking with the Ball-and-Box Model](https://labuladong.online/en/algo/practice-in-action/two-views-of-backtrack/), you can also write such a backtracking function easily.

The parameters of this `backtrack` function can be understood like this:

Now bucket `k` is deciding whether it should take the element `nums[start]`. The current sum of numbers already in bucket `k` is `bucket`. `used` marks whether an element has already been put into some bucket. `target` is the required sum for each bucket.

With this function definition, we can call `backtrack` like this:

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        // exclude some base cases
        if (k > nums.length) return false;
        int sum = 0;
        for (int v : nums) sum += v;
        if (sum % k != 0) return false;
        
        boolean[] used = new boolean[nums.length];
        int target = sum / k;
        // the k-th bucket is initially empty, start making choices from nums[0]
        return backtrack(k, 0, nums, 0, used, target);
    }
}
```

Before we implement the logic of `backtrack`, repeat the bucket perspective again:

1. We must scan all numbers in `nums` and decide which numbers should be put into the current bucket.

2. If the current bucket is full (the sum in this bucket reaches `target`), we move on to the next bucket and repeat step 1.

The code below implements this logic:

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        // see the above text
    }

    boolean backtrack(int k, int bucket, 
        int[] nums, int start, boolean[] used, int target) {
        // base case
        if (k == 0) {
            // all buckets are full, and nums must all be used up
            // because target == sum / k
            return true;
        }
        if (bucket == target) {
            /**<extend down -200>
            ![](../pictures/set-split/6.jpeg)
            */
            // the current bucket is full, recursively enumerate the choices for the next bucket
            // let the next bucket start selecting numbers from nums[0]
            return backtrack(k - 1, 0 ,nums, 0, used, target);
        }

        // start from start to look back for valid nums[i] to put into the current bucket
        for (int i = start; i < nums.length; i++) {
            // pruning
            if (used[i]) {
                // nums[i] has already been placed in another bucket
                continue;
            }
            if (nums[i] + bucket > target) {
                // the current bucket cannot hold nums[i]
                continue;
            }
            // make a choice, put nums[i] into the current bucket
            used[i] = true;
            bucket += nums[i];
            // recursively enumerate whether the next number should be placed in the current bucket
            if (backtrack(k, bucket, nums, i + 1, used, target)) {
                return true;
            }
            // cancel the choice
            used[i] = false;
            bucket -= nums[i];
        }
        // having exhausted all numbers, none can fill the current bucket
        return false;
    }
}
```

**This code can produce the correct answer, but it is very slow. We should think about how to optimize it.**

First, in this solution each bucket is actually identical, but our backtracking algorithm treats them as different. This causes repeated work.

What does this mean? Our backtracking algorithm is, in the end, brute-forcing all possible combinations, and checking whether we can form `k` buckets (subsets) each with sum `target`.

For example, in the case below, `target = 5`. The algorithm may first fill the first bucket with `1, 4`:

![](../pictures/set-split/1.jpeg)

Now the first bucket is full, so we fill the second bucket. The algorithm may put `2, 3` in it:

![](../pictures/set-split/2.jpeg)

Then it continues with the later elements, using brute-force to try to build more buckets with sum 5.

But if, in the end, we cannot form `k` subsets with sum `target`, what will the algorithm do?

Backtracking will go back to the first bucket and try again. Now it knows `{1, 4}` in the first bucket does not work, so it will try `{2, 3}` in the first bucket:

![](../pictures/set-split/3.jpeg)

Now the first bucket is full again. Then it starts to fill the second bucket with `{1, 4}`:

![](../pictures/set-split/4.jpeg)

Here you should see the problem. This situation is actually the same as the previous one. That means we already know there is no solution. We do not need to brute-force again.

But our algorithm will still keep searching, because it thinks: “The first and second buckets hold different elements now, so these are two different cases.”

How can we make the algorithm smarter, so it can recognize this and avoid redundant work?

Notice that the `used` array must be the same in these two situations. So we can treat the `used` array as the “state” of the backtracking process.

**So we can use a `memo` map. When we fill one bucket, we record the current `used` state. If the same `used` state appears again later, we know we have already tried this situation, so we can stop and prune the search.**

You may ask: `used` is a boolean array, how can we use it as a key? This is easy. We can convert the array to a string and then use the string as the key in a hash map.

Look at the code. We only need to slightly change the `backtrack` function:

```java
class Solution {

    // Memoization, storing the state of the used array
    HashMap<String, Boolean> memo = new HashMap<>();

    public boolean canPartitionKSubsets(int[] nums, int k) {
        // See above
    }

    boolean backtrack(int k, int bucket, int[] nums, int start, boolean[] used, int target) {        
        // base case
        if (k == 0) {
            return true;
        }
        // Convert the state of used into a string like [true, false, ...]
        // for easy storage in the HashMap
        String state = Arrays.toString(used);

        if (bucket == target) {
            // The current bucket is filled, recursively enumerate the choices for the next bucket
            boolean res = backtrack(k - 1, 0, nums, 0, used, target);
            // Store the current state and result in the memoization
            memo.put(state, res);
            return res;
        }
        
        if (memo.containsKey(state)) {
            // If the current state has been calculated before, return directly without further recursion
            return memo.get(state);
        }

        // Other logic remains unchanged...
    }
}
```

If we submit this version, we will still find it slow. This time, the problem is not the algorithm logic, but the implementation.

**Because in each recursion we convert the `used` array to a string, which is costly for the language runtime. So we can optimize further.**

Notice the constraint: `nums.length <= 16`. That means `used` will never have more than 16 elements. So we can use a bit-mask trick: use one integer variable `used` instead of a boolean array.

More precisely, we can use the `i`-th bit of integer `used` (`(used >> i) & 1`) to represent `used[i]` being true or false.

With this, we both save space and can use the integer `used` directly as the key in a HashMap, without converting arrays to strings.

Here is the final solution:

```java
class Solution {
    public boolean canPartitionKSubsets(int[] nums, int k) {
        // exclude some basic cases
        if (k > nums.length) return false;
        int sum = 0;
        for (int v : nums) sum += v;
        if (sum % k != 0) return false;
        
        // use bit manipulation technique
        int used = 0;
        int target = sum / k;
        // the k-th bucket initially has nothing, start choosing from nums[0]
        return backtrack(k, 0, nums, 0, used, target);
    }

    HashMap<Integer, Boolean> memo = new HashMap<>();

    boolean backtrack(int k, int bucket,
                    int[] nums, int start, int used, int target) {        
        // base case
        if (k == 0) {
            // all buckets are filled, and all nums must have been used
            return true;
        }
        if (bucket == target) {
            // current bucket is filled, recursively enumerate the choices for the next bucket
            // let the next bucket start choosing from nums[0]
            boolean res = backtrack(k - 1, 0, nums, 0, used, target);
            // cache the result
            memo.put(used, res);
            return res;
        }
        
        if (memo.containsKey(used)) {
            // avoid redundant calculations
            return memo.get(used);
        }

        for (int i = start; i < nums.length; i++) {
            // pruning
            // check if the i-th bit is 1
            if (((used >> i) & 1) == 1) {
                // nums[i] has already been used in another bucket
                continue;
            }
            if (nums[i] + bucket > target) {
                continue;
            }
            // make a choice
            // set the i-th bit to 1
            used |= 1 << i;
            bucket += nums[i];
            // recursively enumerate whether the next number goes into the current bucket
            if (backtrack(k, bucket, nums, i + 1, used, target)) {
                return true;
            }
            // undo the choice
            // use XOR operation to reset the i-th bit to 0
            used ^= 1 << i;
            bucket -= nums[i];
        }

        return false;
    }
}
```

<visual slug='partition-to-k-equal-sum-subsets'/>

Now the second idea for this problem is done.

## Final Summary

Both ideas in this article can produce the correct answer. But even with sorting optimization, the first solution is clearly slower than the second one. Why?

Let’s analyze the time complexity. Assume `n` is the number of elements in `nums`.

For the first solution (the number perspective): for each of the `n` numbers, we have `k` choices (which bucket to put it in). So the number of combinations is `k^n`, and the time complexity is $O(k^n)$.

For the second solution (the bucket perspective): each bucket scans `n` numbers, and for each number we have two choices: “put in” or “not put in”. So there are `2^n` combinations per bucket. We have `k` buckets, so the time complexity is $O(k * 2^n)$.

**Of course, these are rough upper bounds for the worst case. In practice the complexity is much better thanks to all our pruning. But from the upper bounds we can already see that the first idea is much slower.**

So, who says backtracking has no technique? Backtracking is brute-force, but brute-force can be smart or stupid. The key is: from which “perspective” do you brute-force?

In simple words, we should try to “do more small steps” instead of “one huge step”. That is, it is better to have more choices with small branching (multiplicative growth), instead of having a huge branching factor (exponential growth). Doing `n` times of “k choices once” giving $O(k^n)$ is often worse than doing `n` times of “2 choices” repeated `k` times giving $O(k * 2^n)$.

In this problem, we tried brute-force from two perspectives. The code looks long, but the core logic is similar. After reading this article, you should have a deeper understanding of backtracking.
