::: info Prerequisite

Before reading this article, you need to learn:

- [Binary Tree Algorithms (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)
- [Backtracking Algorithm Core Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/)

:::

You learned permutations, combinations, and subsets in high school math. But it is still hard to write algorithms to solve them. This article will teach you the key ideas to solve these problems with code. After you learn this, you can easily handle different variants later.

For permutation, combination, or subset problems, the task is always: pick some elements from an array `nums` under certain rules. There are three basic forms:

**Form 1: Elements are unique and cannot be reused. That is, all elements in `nums` are different, and each element can be used at most once. This is the basic form.**

For example, for combinations, if `nums = [2,3,6,7]`, the combinations whose sum is 7 should only be `[7]`.

**Form 2: Elements may repeat but still cannot be reused. That is, `nums` may contain duplicates, but each element can be used at most once.**

For example, for combinations, if `nums = [2,5,2,1,2]`, the combinations whose sum is 7 should be `[2,2,2,1]` and `[5,2]`.

**Form 3: Elements are unique and can be reused. That is, all elements in `nums` are different, and each element can be used many times.**

For example, for combinations, if `nums = [2,3,6,7]`, the combinations whose sum is 7 should be `[2,2,3]` and `[7]`.

You may think there is a fourth form: elements may repeat and can be reused. But if an element can be reused, why keep duplicates in `nums`? After removing duplicates, this case becomes the same as Form 3. So we do not treat it as a separate case.

The examples above use combination problems, but permutation, combination, and subset problems can all appear in these three basic forms. So in total, there are 9 variants.

On top of that, the problem can add more conditions. For example, “find combinations whose sum is `target` and have exactly `k` elements.” From here you can create many more variants. No wonder these topics show up so often in coding interviews.

**But no matter how the form changes, the essence is to brute-force all solutions. These solutions form a tree structure. So if you use the backtracking framework correctly, you can solve all these problems by slightly changing the same code framework.**

More concretely, you should first read and understand [Backtracking Algorithm Core Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/). Then remember the backtracking trees for the subset problem and the permutation problem below. With these two trees, you can solve all permutation/combination/subset problems:

![](../pictures/permutation/1.jpeg)

![](../pictures/permutation/2.jpeg)

Why can these two tree structures solve all related problems?

**First, combination problems and subset problems are actually equivalent. We will explain this later. As for the three forms we talked about, they just mean cutting off or adding some branches on these two trees.**

Next, we will start brute-forcing. We will go through all 9 forms of permutation/combination/subset problems and learn how to solve them with backtracking.

::: info Tip

Some of you may have seen other solutions for permutations/subsets/combinations before. The code you saw may be different from what I show in this article. That is because backtracking has two ways to view the brute-force process. I will explain them step by step in [Ball-and-Box Model: Two Views of Backtracking](https://labuladong.online/en/algo/practice-in-action/two-views-of-backtrack/). It is not the right time to explain those other solutions now. Just follow the approach in this article.

:::


<!-- hide -->

## Subsets (no duplicates, no reuse)

LeetCode 78 “[Subsets](https://leetcode.com/problems/subsets/)” is this problem:

You are given an array `nums` with no duplicate elements. Each element can be used at most once. Return all subsets of `nums`.

Function signature:

```java
List<List<Integer>> subsets(int[] nums)
```

For example, if the input is `nums = [1,2,3]`, the algorithm should return:

```java
[ [],[1],[2],[3],[1,2],[1,3],[2,3],[1,2,3] ]
```

First, ignore the code. Think about high-school math: how do we list all subsets by hand?

First, generate the subset with 0 elements, the empty set `[]`. To make it simple, call it `S_0`.

Then, based on `S_0`, generate all subsets with 1 element. Call this `S_1`:

![](../pictures/permutation/3.jpeg)

Next, from `S_1`, we can derive `S_2`, the subsets with 2 elements:

![](../pictures/permutation/4.jpeg)

Why do we only add `3` to `[2]`, but not add `1` in front?

Because in a set, order does not matter. In `[1,2,3]`, after `2` there is only `3`. If you add `1` before `2`, you get `[2,1]`, which is the same as `[1,2]` that we already have.

**In other words, we avoid duplicate subsets by keeping the relative order of elements unchanged**.

Then, from `S_2` we can get `S_3`. In fact, `S_3` has only one subset `[1,2,3]`, which comes from `[1,2]`.

The whole process forms a tree:

![](../pictures/permutation/5.jpeg)

Notice this property of the tree:

**If we treat the root as level 0, and define a node’s “value” as all the elements on the path from the root to that node, then all nodes on level `n` represent all subsets of size `n`**.

For example, all subsets of size 2 are the values of the nodes on this level:

![](../pictures/permutation/6.jpeg)

::: info 

**As shown above, in the rest of this article, “node value” always means the elements on the path from the root to that node. The root is level 0**.

:::

Now, if we want all subsets, we just need to traverse this tree and collect the value of every node.

Here is the code:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    // record the recursive path of backtracking algorithm
    LinkedList<Integer> track = new LinkedList<>();

    // main function
    public List<List<Integer>> subsets(int[] nums) {
        backtrack(nums, 0);
        return res;
    }

    // the core function of backtracking algorithm, traversing the backtracking tree of subset problems
    void backtrack(int[] nums, int start) {

        // the pre-order position, the value of each node is a subset
        res.add(new LinkedList<>(track));
        
        // the standard framework of backtracking algorithm
        for (int i = start; i < nums.length; i++) {
            // make a choice
            track.addLast(nums[i]);
            // control the traversal of branches through the start parameter to avoid generating duplicate subsets
            backtrack(nums, i + 1);
            // undo the choice
            track.removeLast();
        }
    }
}
```

If you have read the earlier article “[Backtracking Algorithm Core Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/)”, this code should be easy to understand. We use the parameter `start` to control how the branches grow so there are no duplicate subsets. We use `track` to record the path from the root to the current node. At each node (in pre-order), we collect the path into the result. After the backtracking tree is fully traversed, we have all subsets:

![](../pictures/permutation/5.jpeg)

At the start of `backtrack`, it looks like there is no base case. Will it recurse forever?

No. When `start == nums.length`, the leaf node’s value is already added to `res`, but the for loop does not run, so the recursion ends.

<visual slug='subsets' />


## Combinations (no duplicate elements, no reuse)

If you can generate all unique subsets, then with small changes, you can also generate all unique combinations.

For example, you are asked to take 2 elements from `nums = [1,2,3]` and form all combinations. How would you do it?

Think a bit: all combinations of size 2 are just all subsets of size 2.

**So combinations and subsets are the same: a combination of size `k` is a subset of size `k`.**

For example, LeetCode 77 “[Combinations](https://leetcode.com/problems/combinations/)”:

Given two integers `n` and `k`, return all possible combinations of `k` numbers chosen from the range `[1, n]`.

The function signature is:

```java
List<List<Integer>> combine(int n, int k)
```

For example, the return value of `combine(3, 2)` should be:

```java
[ [1,2],[1,3],[2,3] ]
```

This is a standard combination problem, but we can translate it into a subset problem:

**You are given an array `nums = [1,2..,n]` and a positive integer `k`. Generate all subsets of size `k`.**

Still use `nums = [1,2,3]` as an example. Earlier, when we asked for all subsets, we collected the values of all nodes. **Now you only need to collect the nodes at level 2 (root is level 0). Those are all combinations of size 2.**

![](../pictures/permutation/6.jpeg)

In code, you only need to change the base case a bit, so the algorithm only collects the values of nodes at level `k`:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    // record the recursive path of backtracking algorithm
    LinkedList<Integer> track = new LinkedList<>();

    // main function
    public List<List<Integer>> combine(int n, int k) {
        backtrack(1, n, k);
        return res;
    }

    void backtrack(int start, int n, int k) {
        // base case
        if (k == track.size()) {
            // reached the k-th level, collect the current node's value
            res.add(new LinkedList<>(track));
            return;
        }
        
        // standard framework of backtracking algorithm
        for (int i = start; i <= n; i++) {
            // make a choice
            track.addLast(i);
            // control the traversal of branches through the start parameter to avoid duplicate subsets
            backtrack(i + 1, n, k);
            // undo the choice
            track.removeLast();
        }
    }
}
```

In this way, the standard combination problem is also solved.

<visual slug='combinations' />


## Permutations (no duplicates, no reuse)

We already talked about permutation problems in the article [Backtracking Algorithm Core Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/), so here we will just do a quick review.

LeetCode 46 “[Permutations](https://leetcode.com/problems/permutations/)” is the standard permutation problem:

Given an array `nums` that **has no duplicate numbers**, return all possible **permutations**.

Function signature:

```java
List<List<Integer>> permute(int[] nums)
```

For example, if the input is `nums = [1,2,3]`, the return value should be:

```java
[
    [1,2,3],[1,3,2],
    [2,1,3],[2,3,1],
    [3,1,2],[3,2,1]
]
```

For the combination/subset problems we just talked about, we used a `start` variable to make sure that after `nums[start]`, we only use elements in `nums[start+1..]`. By fixing the relative order of elements, we avoid duplicate subsets.

**But for permutations, the goal is to try all possible positions of each element. After `nums[i]`, elements on the left of `nums[i]` can also appear. So the previous method does not work. We need an extra `used` array to mark which elements are still available.**

A standard permutation problem can be seen as the following multi-way tree:

![](../pictures/permutation/7.jpeg)

We use the `used` array to mark the elements already in the current path to avoid choosing them again. Then we collect the values at all the leaf nodes. These are all the permutations:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    // record the recursive path of backtracking algorithm
    LinkedList<Integer> track = new LinkedList<>();
    // elements in track will be marked as true
    boolean[] used;

    // main function, input a set of non-repeating numbers, return all permutations
    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        backtrack(nums);
        return res;
    }

    // core function of backtracking algorithm
    void backtrack(int[] nums) {
        // base case, reach the leaf node
        if (track.size() == nums.length) {
            // collect the value at the leaf node
            res.add(new LinkedList(track));
            return;
        }

        // standard framework of backtracking algorithm
        for (int i = 0; i < nums.length; i++) {
            // elements already in track, cannot be selected repeatedly
            if (used[i]) {
                continue;
            }
            // make a choice
            used[i] = true;
            track.addLast(nums[i]);
            // enter the next level of backtracking tree
            backtrack(nums);
            // cancel the choice
            track.removeLast();
            used[i] = false;
        }
    }
}
```

<visual slug='permutations' />

Now the permutation problem is solved.

But what if the problem does not ask for all permutations, and only asks for permutations of length `k`? How should we do that?

It is also simple. Just change the base case of the `backtrack` function and only collect nodes at level `k`:

```java
// core function of backtracking algorithm
void backtrack(int[] nums, int k) {
    // base case, reach the k-th level, collect the value of the node
    if (track.size() == k) {
        // the value of the k-th level node is a permutation of size k
        res.add(new LinkedList(track));
        return;
    }

    // standard framework of backtracking algorithm
    for (int i = 0; i < nums.length; i++) {
        // ...
        backtrack(nums, k);
        // ...
    }
}
```


## Subsets / Combinations (elements can repeat, cannot reuse)

In the standard subset problem we just talked about, the input array `nums` has no duplicate elements. What if `nums` contains duplicates?

LeetCode 90: [Subsets II](https://leetcode.com/problems/subsets-ii/) is this kind of problem:

You are given an integer array `nums`, which may contain duplicates. Return all possible subsets of this array.

Function signature:

```java
List<List<Integer>> subsetsWithDup(int[] nums)
```

For example, if `nums = [1,2,2]`, you should output:

```java
[ [],[1],[2],[1,2],[2,2],[1,2,2] ]
```

Strictly speaking, a “set” should not contain duplicate elements. But since the problem is defined like this, we ignore this detail and just focus on how to solve it.

Still use `nums = [1,2,2]` as an example. To tell the two `2`s apart, we write `nums = [1,2,2']`.

If we draw the subset tree as before, it is clear that two adjacent branches with the same value will create duplicates:

![](../pictures/permutation/8.jpeg)

```txt
[ 
    [],
    [1],[2],[2'],
    [1,2],[1,2'],[2,2'],
    [1,2,2']
]
```

You can see `[2]` and `[1,2]` appear as duplicates, so we need pruning. If from one node there are multiple adjacent branches with the same value, we only traverse the first branch and prune the rest:

![](../pictures/permutation/9.jpeg)

**In code, we first sort `nums` to put equal elements together. If we find `nums[i] == nums[i-1]`, we skip this element**:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();

    public List<List<Integer>> subsetsWithDup(int[] nums) {
        // sort first, let the same elements be together
        Arrays.sort(nums);
        backtrack(nums, 0);
        return res;
    }

    void backtrack(int[] nums, int start) {
        // preorder position, the value of each node is a subset
        res.add(new LinkedList<>(track));
        
        for (int i = start; i < nums.length; i++) {
            // pruning logic, only traverse the first branch of adjacent branches with the same value
            if (i > start && nums[i] == nums[i - 1]) {
                continue;
            }
            track.addLast(nums[i]);
            backtrack(nums, i + 1);
            track.removeLast();
        }
    }
}
```

<visual slug='subsets-ii' />

This code is almost the same as the code for the standard subset problem. We only added sorting and pruning.

Why does this pruning work? If you compare it with the tree drawing above, it should be clear. Now the subset problem with duplicates is solved.

**We said combination problems and subset problems are equivalent**, so let’s look directly at a combination problem: LeetCode 40: [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/):

You are given `candidates` and a target sum `target`. Find all combinations in `candidates` where the numbers sum to `target`.

`candidates` may contain duplicates, and each number can be used at most once.

We call it a combination problem, but if you change the wording, it becomes a subset problem: find all subsets of `candidates` whose sum is `target`.

How do we solve it?

Compare with the subset solution. We just add one extra variable `trackSum` to record the sum of elements in the current path, then change the base case a bit, and we can solve this problem:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    // record the backtracking path
    LinkedList<Integer> track = new LinkedList<>();
    // record the sum of elements in track
    int trackSum = 0;

    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        if (candidates.length == 0) {
            return res;
        }
        // sort first, let the same elements be together
        Arrays.sort(candidates);
        backtrack(candidates, 0, target);
        return res;
    }

    // main function of backtracking algorithm
    void backtrack(int[] nums, int start, int target) {
        // base case, reach the target sum, find a qualified combination
        if (trackSum == target) {
            res.add(new LinkedList<>(track));
            return;
        }
        // base case, exceed the target sum, end directly
        if (trackSum > target) {
            return;
        }

        // standard framework of backtracking algorithm
        for (int i = start; i < nums.length; i++) {
            // pruning logic, only traverse the first branch with the same value
            if (i > start && nums[i] == nums[i - 1]) {
                continue;
            }
            // make a choice
            track.add(nums[i]);
            trackSum += nums[i];
            // recursively traverse the next level of backtracking tree
            backtrack(nums, i + 1, target);
            // cancel the choice
            track.removeLast();
            trackSum -= nums[i];
        }
    }
}
```

<visual slug='combination-sum-ii' />


## Permutations (elements can repeat, no reuse within one permutation)

When the input of a permutation problem has duplicate elements, it is a bit more complex than subset/combination problems. Look at LeetCode 47: [Permutations II](https://leetcode.com/problems/permutations-ii/):

You are given an array `nums` that may contain duplicates. Write an algorithm to return all possible unique permutations. The function signature is:

```java
List<List<Integer>> permuteUnique(int[] nums)
```

For example, if `nums = [1,2,2]`, the function should return:

```java
[ [1,2,2],[2,1,2],[2,2,1] ]
```

Here is the solution code:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();
    boolean[] used;

    public List<List<Integer>> permuteUnique(int[] nums) {
        // sort first, let the same elements be together
        Arrays.sort(nums);
        used = new boolean[nums.length];
        backtrack(nums);
        return res;
    }

    void backtrack(int[] nums) {
        if (track.size() == nums.length) {
            res.add(new LinkedList(track));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            }
            // newly added pruning logic, fix the relative positions of the same elements in the permutation
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            track.add(nums[i]);
            used[i] = true;
            backtrack(nums);
            track.removeLast();
            used[i] = false;
        }
    }
}
```

<visual slug='permutations-ii' />

Compare this code with the standard permutation solution you saw earlier. There are only two differences:

1. We sort `nums`.
2. We add one extra pruning condition.

Like the subset/combination problems with duplicates, this is also used to avoid duplicate results.

But note: the pruning logic for permutations is a bit different from that for subsets/combinations. We add this condition: `!used[i - 1]`.

This part is a bit tricky. To make it easier to understand, we still use subscripts to tell equal elements apart.

Assume the input is `nums = [1,2,2']`. The standard permutation algorithm will produce:

```
[
    [1,2,2'],[1,2',2],
    [2,1,2'],[2,2',1],
    [2',1,2],[2',2,1]
]
```

Clearly, this result has duplicates. For example, `[1,2,2']` and `[1,2',2]` should be treated as the same permutation, but they are counted as two.

So the key point is: how do we design a pruning rule to remove such duplicates?

**The answer: keep the relative order of equal elements the same in every permutation.**

For example, in `nums = [1,2,2']`, we always keep `2` before `2'` in any permutation.

Then, among the 6 permutations above, only these 3 meet the rule:

```
[ [1,2,2'],[2,1,2'],[2,2',1] ]
```

These 3 are the correct result.

Further, if `nums = [1,2,2',2'']`, as long as we fix the order of the repeated `2`s, for example `2 -> 2' -> 2''`, we will get all unique permutations.

If you think about it, the idea is simple:

**The standard permutation algorithm creates duplicates because it treats permutations that only differ by swapping equal elements as different, while they are actually the same. If we fix the order of equal elements, we avoid this.**

Now look at the pruning code:

```java
// New pruning rule: fix the relative order of equal elements
if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
    // If the previous equal element has not been used, skip this one
    continue;
}
// choose nums[i]
```

**When there are duplicates, for example `nums = [1,2,2',2'']`, `2'` can only be chosen after `2` is already used; `2''` can only be chosen after `2'` is used. This fixes the relative order of equal elements in every permutation.**

Now, what if you change `!used[i - 1]` to `used[i - 1]`? The code still passes all tests, but it is slower. Why?

This version is still correct because it just enforces the reverse order `2'' -> 2' -> 2`. That also removes duplicates.

But why is it slower? Because it prunes fewer branches.

For example, for input `nums = [2,2',2'']`, the backtracking tree looks like this:

![](../pictures/permutation/12.jpeg)

If we mark visited paths of `backtrack` in green, and pruned branches in red, then with `!used[i - 1]` the tree looks like this:

![](../pictures/permutation/13.jpeg)

And with `used[i - 1]` it looks like this:

![](../pictures/permutation/14.jpeg)

You can see that `!used[i - 1]` prunes more branches, so it does less useless work and is more efficient. The `used[i - 1]` version still avoids duplicates, but it leaves more useless branches, so it is slower.

You can click the "Edit" button in the visualization panel to change the code and compare the two pruning methods:

<visual slug='permutations-ii' />

There is also another pruning idea mentioned by some readers:

```java
void backtrack(int[] nums, LinkedList<Integer> track) {
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }

    // record the value of the previous branch element
    // the problem statement says -10 <= nums[i] <= 10, so initialize to a special value
    int prevNum = -666;
    for (int i = 0; i < nums.length; i++) {
        // exclude illegal choices
        if (used[i]) {
            continue;
        }
        if (nums[i] == prevNum) {
            continue;
        }

        track.add(nums[i]);
        used[i] = true;
        // record the value on this branch
        prevNum = nums[i];

        backtrack(nums, track);

        track.removeLast();
        used[i] = false;
    }
}
```

This idea is also correct. Imagine a node has several outgoing edges (choices) with the same value:

![](../pictures/permutation/11.jpeg)

If we do nothing, the subtrees under these equal edges will be exactly the same, so we will get duplicate permutations.

Because the array is sorted, equal elements are next to each other. So we can use a `prevNum` variable to record the value of the previous edge. If the next edge has the same value, we skip it. This avoids exploring identical subtrees and thus avoids duplicate permutations.

Now the permutation problem with duplicate elements is also solved.


## Subsets / Combinations (no duplicate elements, can reuse)

Now we come to the last type: the input array has no duplicate elements, but each element can be used unlimited times.

Look at LeetCode 39: [Combination Sum](https://leetcode.com/problems/combination-sum/):

You are given an integer array `candidates` without duplicates and a target integer `target`. Find all combinations of `candidates` where the chosen numbers sum to `target`. You may use each number in `candidates` an unlimited number of times.

The function signature:

```java
List<List<Integer>> combinationSum(int[] candidates, int target)
```

For example, if `candidates = [1,2,3], target = 3`, the algorithm should return:

```
[ [1,1,1], [1,2], [3] ]
```

This problem is described as a combination problem, but it is actually also a subset problem: which subsets of `candidates` sum to `target`?

To solve this type of problem, we still look at the backtracking tree. First, think about this question: **in the standard subset / combination problem, how do we make sure each element is not reused**?

The answer is in the `start` parameter of the `backtrack` function:

```java
// Backtracking algorithm framework without repetition
void backtrack(int[] nums, int start) {
    for (int i = start; i < nums.length; i++) {
        // ...
        // Recursively traverse the next level of the backtracking tree, pay attention to the parameters
        backtrack(nums, i + 1);
        // ...
    }
}
```

Here `i` starts from `start`, so the next level of recursion will start from `start + 1`. This makes sure that the element `nums[start]` will not be used again:

![](../pictures/permutation/1.jpeg)

Now, if we want to allow each element to be reused, we just change `i + 1` to `i`:

```java
// Backtracking algorithm framework with reusable combinations
void backtrack(int[] nums, int start) {
    for (int i = start; i < nums.length; i++) {
        // ...
        // Recursively traverse the next level of backtracking tree, note the parameters
        backtrack(nums, i);
        // ...
    }
}
```

This is like adding one more branch to the old backtracking tree. When we traverse this tree, one element can be used any number of times:

![](../pictures/permutation/10.jpeg)

Of course, this tree would grow forever if we are not careful, so our recursive function must have a good base case to stop the algorithm. When the path sum is greater than `target`, we should stop going deeper.

Here is the solution code:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    // record the backtracking path
    LinkedList<Integer> track = new LinkedList<>();
    // record the sum of the path in track
    int trackSum = 0;

    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        if (candidates.length == 0) {
            return res;
        }
        backtrack(candidates, 0, target);
        return res;
    }

    // main function of backtracking algorithm
    void backtrack(int[] nums, int start, int target) {
        // base case, find the target sum, record the result
        if (trackSum == target) {
            res.add(new LinkedList<>(track));
            return;
        }
        // base case, exceed the target sum, stop traversing downwards
        if (trackSum > target) {
            return;
        }
        // standard framework of backtracking algorithm
        for (int i = start; i < nums.length; i++) {
            // choose nums[i]
            trackSum += nums[i];
            track.add(nums[i]);
            // recursively traverse the next level of backtracking tree
            backtrack(nums, i, target);
            // the same element can be used repeatedly, note the parameter
            // undo the choice of nums[i]
            trackSum -= nums[i];
            track.removeLast();
        }
    }
}
```

<visual slug='combination-sum' />


## Permutations (no duplicates, elements can be reused)

There is no LeetCode problem that directly tests this case, so let’s think about it ourselves.  
If `nums` has no duplicate elements, and each element can be chosen many times, what permutations do we have?

For example, if `nums = [1,2,3]`, then all permutations under this rule are `3^3 = 27` in total:

```java
[
  [1,1,1],[1,1,2],[1,1,3],[1,2,1],[1,2,2],[1,2,3],[1,3,1],[1,3,2],[1,3,3],
  [2,1,1],[2,1,2],[2,1,3],[2,2,1],[2,2,2],[2,2,3],[2,3,1],[2,3,2],[2,3,3],
  [3,1,1],[3,1,2],[3,1,3],[3,2,1],[3,2,2],[3,2,3],[3,3,1],[3,3,2],[3,3,3]
]
```

The standard permutation algorithm uses a `used` array to prune, so that we do not pick the same element twice in one permutation.  
If we allow reusing elements, we just remove all pruning logic related to `used`.

Then this problem becomes easy. Code is as follows:

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

<visual slug='permutation-repeated' />

Now we have finished all nine variants of permutation/combination/subset problems.

## Final Summary

Let’s review how the three forms of permutation/combination/subset problems differ in code.

Since subset and combination problems are basically the same (only the base case is a bit different), we put them together.

**Form 1: No duplicates, cannot reuse.**  
All elements in `nums` are unique, and each element can be used at most once.  
The core `backtrack` code is:

```java
// Backtracking algorithm framework for combination/subset problems
void backtrack(int[] nums, int start) {
    // Standard backtracking algorithm framework
    for (int i = start; i < nums.length; i++) {
        // Make a choice
        track.addLast(nums[i]);
        // Note the parameter
        backtrack(nums, i + 1);
        // Undo the choice
        track.removeLast();
    }
}

// Backtracking algorithm framework for permutation problems
void backtrack(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        // Pruning logic
        if (used[i]) {
            continue;
        }
        // Make a choice
        used[i] = true;
        track.addLast(nums[i]);

        backtrack(nums);
        // Undo the choice
        track.removeLast();
        used[i] = false;
    }
}
```

**Form 2: Duplicates allowed, cannot reuse.**  
`nums` may contain duplicate elements, and each element can be used at most once.  
The key is sorting and pruning. The core `backtrack` code is:

```java
Arrays.sort(nums);
// backtracking algorithm framework for combination/subset problems
void backtrack(int[] nums, int start) {
    // standard backtracking algorithm framework
    for (int i = start; i < nums.length; i++) {
        // pruning logic, skip the adjacent branches with the same value
        if (i > start && nums[i] == nums[i - 1]) {
            continue;
        }
        // make a choice
        track.addLast(nums[i]);
        // note the parameter
        backtrack(nums, i + 1);
        // undo the choice
        track.removeLast();
    }
}


Arrays.sort(nums);
// backtracking algorithm framework for permutation problems
void backtrack(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        // pruning logic
        if (used[i]) {
            continue;
        }
        // pruning logic, fix the relative position of the same elements in the permutation
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
            continue;
        }
        // make a choice
        used[i] = true;
        track.addLast(nums[i]);

        backtrack(nums);
        // undo the choice
        track.removeLast();
        used[i] = false;
    }
}
```

**Form 3: No duplicates, can reuse.**  
All elements in `nums` are unique, and each element can be used many times.  
We just remove the de-duplication logic. The core `backtrack` code is:

```java
// Backtracking algorithm framework for combination/subset problems
void backtrack(int[] nums, int start) {
    // Standard backtracking algorithm framework
    for (int i = start; i < nums.length; i++) {
        // Make a choice
        track.addLast(nums[i]);
        // Note the parameter
        backtrack(nums, i);
        // Undo the choice
        track.removeLast();
    }
}

// Backtracking algorithm framework for permutation problems
void backtrack(int[] nums) {
    for (int i = 0; i < nums.length; i++) {
        // Make a choice
        track.addLast(nums[i]);
        backtrack(nums);
        // Undo the choice
        track.removeLast();
    }
}
```

As long as you think from the view of the recursion tree, these problems look complex but are actually simple. We just change the base case and a little logic.  
This is why I stress the importance of tree problems in [Framework Thinking for Learning Algorithms and Data Structures](https://labuladong.online/en/algo/essential-technique/algorithm-summary/) and [Binary Tree Tutorial (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/).

If you have read this far, you really deserve a round of applause. I believe that when you meet all kinds of strange algorithm problems in the future, you will see through their nature at a glance and solve them with the same stable framework.

Also, because of the length of this article, I did not analyze the time and space complexity of these algorithms. You can try to analyze their complexity yourself using the methods I introduced in [A Practical Guide to Time and Space Complexity](https://labuladong.online/en/algo/essential-technique/complexity-analysis/).
