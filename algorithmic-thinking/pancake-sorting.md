LeetCode Problem 969 ["Pancake Sorting"](https://leetcode.com/problems/pancake-sorting/) is an interesting real-world problem: Imagine you have `n` pancakes of different sizes on a plate. Using a spatula, how can you flip the pancakes a few times to sort them by size (smallest on top, largest on bottom)?

![](../pictures/pancakeSort/1.jpg)

Think of flipping a stack of pancakes with a spatula. There is a rule: each time, you can only flip the top few pancakes.

![](../pictures/pancakeSort/2.png)

Our task is: **How can we use an algorithm to find a sequence of flips to make the pancake stack sorted?**

First, we need to model the problem. We use an array to represent the pancake stack:

**LeetCode 969. Pancake Sorting** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given an array of integers `arr`, sort the array by performing a series of **pancake flips**.

In one pancake flip we do the following steps:

	
- Choose an integer `k` where `1 <= k <= arr.length`.
	
- Reverse the sub-array `arr[0...k-1]` (**0-indexed**).

For example, if `arr = [3,2,1,4]` and we performed a pancake flip choosing `k = 3`, we reverse the sub-array `[3,2,1]`, so `arr = [1,2,3,4]` after the pancake flip at `k = 3`.

Return *an array of the *`k`*-values corresponding to a sequence of pancake flips that sort *`arr`. Any valid answer that sorts the array within `10 * arr.length` flips will be judged as correct.

Example 1:**

```

**Input:** arr = [3,2,4,1]
**Output:** [4,2,4,3]
**Explanation: **
We perform 4 pancake flips, with k values 4, 2, 4, and 3.
Starting state: arr = [3, 2, 4, 1]
After 1st flip (k = 4): arr = [1, 4, 2, 3]
After 2nd flip (k = 2): arr = [4, 1, 2, 3]
After 3rd flip (k = 4): arr = [3, 2, 1, 4]
After 4th flip (k = 3): arr = [1, 2, 3, 4], which is sorted.

```

Example 2:**

```

**Input:** arr = [1,2,3]
**Output:** []
**Explanation: **The input is already sorted, so there is no need to flip anything.
Note that other answers, such as [3, 3], would also be accepted.

```

**Constraints:**

	
- `1 <= arr.length <= 100`
	
- `1 <= arr[i] <= arr.length`
	
- All integers in `arr` are unique (i.e. `arr` is a permutation of the integers from `1` to `arr.length`).

How do we solve this problem? Like in the article [Recursively Reverse Part of a Linked List](https://labuladong.online/en/algo/data-structure/reverse-linked-list-recursion/), we need to use recursion.

### 1. Idea Analysis

Why does this problem have the property of recursion? For example, we can make a function like this:

```java
// cakes is a pile of pancakes, the function will sort the first n pancakes
void sort(int[] cakes, int n);
```

If we find the largest pancake among the first `n` pancakes, and flip it to the bottom:

![](../pictures/pancakeSort/3.jpg)

Now the problem size is smaller, so we can recursively call `pancakeSort(A, n-1)`:

![](../pictures/pancakeSort/4.jpg)

Next, for these `n - 1` pancakes, how to sort them? Again, find the largest, put it at the bottom, and call `pancakeSort(A, n-1-1)`...

You see, this is recursion. To summarize:

1. Find the largest pancake among the `n` pancakes.

2. Move this largest pancake to the bottom.

3. Recursively call `pancakeSort(A, n - 1)`.

Base case: When `n == 1`, there is only one pancake, so no flips are needed.

Now, one last question: **How do we move a certain pancake to the bottom?**

It's simple. For example, if the 3rd pancake is the largest and you want to move it to the bottom (the `n`th position):

1. Use the spatula to flip the top 3 pancakes. Now the largest is at the top.

2. Flip the top `n` pancakes. Now the largest is at the bottom.

With these two steps, you can write the solution. The problem also asks us to return the flip operation sequence. We just need to record each flip.

### 2. Code Implementation

Just write the above idea in code. Note: array index starts from 0, but the answer should use 1-based indices.

```java
class Solution {
    // record the sequence of flip operations
    LinkedList<Integer> res = new LinkedList<>();

    public List<Integer> pancakeSort(int[] cakes) {
        sort(cakes, cakes.length);
        return res;
    }

    void sort(int[] cakes, int n) {
        // base case
        if (n == 1) return;

        // find the index of the largest pancake
        int maxCake = 0;
        int maxCakeIndex = 0;
        for (int i = 0; i < n; i++)
            if (cakes[i] > maxCake) {
                maxCakeIndex = i;
                maxCake = cakes[i];
            }

        // first flip, move the largest pancake to the top
        reverse(cakes, 0, maxCakeIndex);
        res.add(maxCakeIndex + 1);
        // second flip, move the largest pancake to the bottom
        reverse(cakes, 0, n - 1);
        res.add(n);
        /**<extend up -150>
        ![](../pictures/pancakeSort/3.jpg)
        */
        // recursive call
        sort(cakes, n - 1);
        /**<extend up -150>
        ![](../pictures/pancakeSort/4.jpg)
        */
    }

    void reverse(int[] arr, int i, int j) {
        while (i < j) {
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
            i++;
            j--;
        }
    }
}
```

<visual slug='pancake-sorting'/>

With the explanation above, the code should be clear.

The time complexity is easy to calculate. There are `n` recursive calls, and each call has a for loop of O(n), so total time complexity is O(n^2).

**Finally, let's think about a question:** In this solution, the length of the flip sequence is `2(n - 1)`, because each recursion does 2 flips, and there are `n` layers of recursion. But the base case does not flip, so the total number is `2(n - 1)`.

Clearly, this is not the optimal (shortest) sequence. For example, with pancakes `[3,2,4,1]`, our algorithm gives flips `[3,4,2,3,1,2]`. But the shortest flip sequence is `[2,3,4]`:

```
Start:        [3,2,4,1]
Flip top 2:   [2,3,4,1]
Flip top 3:   [4,3,2,1]
Flip top 4:   [1,2,3,4]
```

If you are asked to find the **shortest** flip sequence to sort the pancakes, how would you solve it? What is the core idea to find the optimal solution? What algorithm should you use? Feel free to share your thoughts.
