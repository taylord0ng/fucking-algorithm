::: info Prerequisites

Before reading this article, you should learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)

:::

Some readers say that after reading many articles on this site, they learned “framework thinking” and can solve most problems that follow a fixed pattern.

But framework thinking is not magic. Some special tricks are like this: if you know them, they are easy; if you don’t, they feel very hard. You can only learn them by doing more problems and summarizing.

In this article, I will share some smart operations on 2D arrays. Just remember the idea. Next time you see a similar problem, you won’t get stuck.

## Rotate a Matrix Clockwise / Counterclockwise

Rotating a 2D array is a common interview test question. LeetCode 48, “[Rotate Image](https://leetcode.com/problems/rotate-image/)”, is a classic one:

**LeetCode 48. Rotate Image** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an `n x n` 2D `matrix` representing an image, rotate the image by **90** degrees (clockwise).

You have to rotate the image [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm), which means you have to modify the input 2D matrix directly. **DO NOT** allocate another 2D matrix and do the rotation.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/08/28/mat1.jpg)

```

**Input:** matrix = [[1,2,3],[4,5,6],[7,8,9]]
**Output:** [[7,4,1],[8,5,2],[9,6,3]]

```

Example 2:**

![](https://assets.leetcode.com/uploads/2020/08/28/mat2.jpg)

```

**Input:** matrix = [[5,1,9,11],[2,4,8,10],[13,3,6,7],[15,14,12,16]]
**Output:** [[15,13,2,5],[14,3,4,1],[12,6,8,9],[16,7,10,11]]

```

**Constraints:**

	
- `n == matrix.length == matrix[i].length`
	
- `1 <= n <= 20`
	
- `-1000 <= matrix[i][j] <= 1000`

The problem is easy to understand: rotate a 2D matrix 90 degrees clockwise. The hard part is: **you must modify it in-place**. The function signature is:

```java
void rotate(int[][] matrix)
```

How do we rotate a 2D matrix in-place? If you think a bit, it feels complicated. You may think you need to rotate it “layer by layer”:

![](../pictures/2d-array/1.png)

**But for this problem, you need a different idea.** Before we talk about the smart solution, let’s warm up with another problem that Google once asked:

You are given a string `s` with words and spaces. Write an algorithm to reverse the order of words **in-place**.

For example:

```shell
s = "hello world labuladong"
```

You need to make it:

```shell
s = "labuladong world hello"
```

A common way is: split by spaces, reverse the word list, then join. But that uses extra space, so it is not “in-place”.

**The correct trick is: first reverse the whole string `s`:**

```shell
s = "gnodalubal dlrow olleh"
```

**Then reverse each word by itself:**

```shell
s = "labuladong world hello"
```

Now you reversed the word order in-place. LeetCode 151, “[Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)”, is a similar problem.

This trick can be used in other problems too. For example, LeetCode 61, “[Rotate List](https://leetcode.com/problems/rotate-list/)”: given a singly linked list, rotate it so every node moves right by `k` positions.

Example: `1 -> 2 -> 3 -> 4 -> 5`, `k = 2`. The result is `4 -> 5 -> 1 -> 2 -> 3`.

For this problem, don’t move nodes one by one. Let me translate it: it really means “move the last `k` nodes to the head”, right?

Still not clear? Moving the last `k` nodes to the head means you split the list into the first `n - k` nodes and the last `k` nodes, then reverse them in-place, right?

This is the same idea as reversing words in-place. You first reverse the whole list, then reverse the first `n - k` nodes and the last `k` nodes.

There are some details. For example, `k` can be larger than the list length. So you should compute the length `n` first, then do `k = k % n`. Then `k` will be valid and the result will be correct.

If you have time, try this problem yourself. It is not hard, so I won’t show the code here.

Why did I talk about these two problems?

**The point is: our “natural” idea is not always the best for a computer; and the computer’s clean idea is not always natural for us.** That may be the fun part of algorithms.

<!-- hide -->

Back to rotating an `n x n` matrix clockwise. The common idea is to find a mapping from old coordinates to new coordinates. But we can try a jump in thinking: do reverse or mirror operations. That can give a new way.

**First, mirror the `n x n` matrix `matrix` across the main diagonal (top-left to bottom-right):**

![](../pictures/2d-array/2.jpeg)

**Then reverse each row:**

![](../pictures/2d-array/3.jpeg)

**You will get the matrix rotated 90 degrees clockwise:**

![](../pictures/2d-array/4.jpeg)

Turn this idea into code and you can solve the problem:

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        // first, reverse the 2D matrix along the diagonal
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                // swap(matrix[i][j], matrix[j][i]);
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        // then, reverse each row of the 2D matrix
        for (int[] row : matrix) {
            reverse(row);
        }
    }

    // reverse a 1D array
    void reverse(int[] arr) {
        int i = 0, j = arr.length - 1;
        while (j > i) {
            // swap(arr[i], arr[j]);
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
            i++;
            j--;
        }
    }
}
```

<visual slug='rotate-image'>

You can open the panel below. Click the line <code type="click">let temp = matrix[i][j]</code> many times to see the diagonal mirror step. Then click <code type="click">reverse(row)</code> many times to see each row reversed and get the final answer:

</visual>

Some readers may ask: if I never did this problem before, how could I think of this?

Yes, if you never saw this type of problem, it’s hard to think of. But now you have seen it. If you know it, it’s easy; if you don’t, it’s hard. You will probably never forget it.

**Now let’s extend it: how do we rotate the matrix 90 degrees counterclockwise?**

The idea is similar: mirror the matrix across the other diagonal, then reverse each row. That gives the counterclockwise result:

![](../pictures/2d-array/5.jpeg)

The code is:

```java
class Solution {

    // rotate the two-dimensional matrix 90 degrees counterclockwise in place
    public void rotate2(int[][] matrix) {
        int n = matrix.length;
        // mirror the two-dimensional matrix along the diagonal from bottom left to top right
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n - i; j++) {
                // swap(matrix[i][j], matrix[n-j-1][n-i-1])
                int temp = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][n - i - 1];
                matrix[n - j - 1][n - i - 1] = temp;
            }
        }
        // then reverse each row of the two-dimensional matrix
        for (int[] row : matrix) {
            reverse(row);
        }
    }

    void reverse(int[] arr) {
        // see above
    }
}
```

Now the rotation problem is solved.

## Spiral Traversal of a Matrix

Next, let’s look at LeetCode 54, “[Spiral Matrix](https://leetcode.com/problems/spiral-matrix/)”, and see a common way to traverse a 2D matrix:

**LeetCode 54. Spiral Matrix** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given an `m x n` `matrix`, return *all elements of the* `matrix` *in spiral order*.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/13/spiral1.jpg)

```

**Input:** matrix = [[1,2,3],[4,5,6],[7,8,9]]
**Output:** [1,2,3,6,9,8,7,4,5]

```

Example 2:**

![](https://assets.leetcode.com/uploads/2020/11/13/spiral.jpg)

```

**Input:** matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
**Output:** [1,2,3,4,8,12,11,10,9,5,6,7]

```

**Constraints:**

	
- `m == matrix.length`
	
- `n == matrix[i].length`
	
- `1 <= m, n <= 10`
	
- `-100 <= matrix[i][j] <= 100`

```java
// The function signature is as follows
List<Integer> spiralOrder(int[][] matrix)
```

**The key idea is to traverse in the order: right, down, left, up, and use four variables to mark the borders of the unvisited area:**

![](../pictures/2d-array/6.png)

As we traverse in a spiral, the borders shrink, until we finish the whole matrix:

![](../pictures/2d-array/7.png)

With this idea, writing the code is easy:

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        int upper_bound = 0, lower_bound = m - 1;
        int left_bound = 0, right_bound = n - 1;
        List<Integer> res = new LinkedList<>();
        // if res.size() == m * n, the entire array has been traversed
        while (res.size() < m * n) {
            if (upper_bound <= lower_bound) {
                // traverse from left to right at the top
                for (int j = left_bound; j <= right_bound; j++) {
                    res.add(matrix[upper_bound][j]);
                }
                // move the upper boundary down
                upper_bound++;
            }
            
            if (left_bound <= right_bound) {
                // traverse from top to bottom on the right
                for (int i = upper_bound; i <= lower_bound; i++) {
                    res.add(matrix[i][right_bound]);
                }
                // move the right boundary left
                right_bound--;
            }
            
            if (upper_bound <= lower_bound) {
                // traverse from right to left at the bottom
                for (int j = right_bound; j >= left_bound; j--) {
                    res.add(matrix[lower_bound][j]);
                }
                // move the lower boundary up
                lower_bound--;
            }
            
            if (left_bound <= right_bound) {
                // traverse from bottom to top on the left
                for (int i = lower_bound; i >= upper_bound; i--) {
                    res.add(matrix[i][left_bound]);
                }
                // move the left boundary right
                left_bound++;
            }
        }
        return res;
    }
}
```

<visual slug='spiral-matrix' >

You can open the panel below. Click the line <code type="click">while (res.length < m * n)</code> many times to see the spiral traversal from outside to inside:

</visual>

LeetCode 59, “[Spiral Matrix II](https://leetcode.com/problems/spiral-matrix-ii/)”, is very similar, but reversed: it asks you to generate a matrix in spiral order:

**LeetCode 59. Spiral Matrix II** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given a positive integer `n`, generate an `n x n` `matrix` filled with elements from `1` to `n^(2)` in spiral order.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/11/13/spiraln.jpg)

```

**Input:** n = 3
**Output:** [[1,2,3],[8,9,4],[7,6,5]]

```

Example 2:**

```

**Input:** n = 1
**Output:** [[1]]

```

**Constraints:**

	
- `1 <= n <= 20`

```java
// The function signature is as follows
int[][] generateMatrix(int n)
```

With the setup above, you can finish it with small code changes:

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int[][] matrix = new int[n][n];
        int upper_bound = 0, lower_bound = n - 1;
        int left_bound = 0, right_bound = n - 1;
        // the number to be filled into the matrix
        int num = 1;
        
        while (num <= n * n) {
            if (upper_bound <= lower_bound) {
                // traverse from left to right at the top
                for (int j = left_bound; j <= right_bound; j++) {
                    matrix[upper_bound][j] = num++;
                }
                // move the upper bound down
                upper_bound++;
            }
            
            if (left_bound <= right_bound) {
                // traverse from top to bottom on the right
                for (int i = upper_bound; i <= lower_bound; i++) {
                    matrix[i][right_bound] = num++;
                }
                // move the right bound left
                right_bound--;
            }
            
            if (upper_bound <= lower_bound) {
                // traverse from right to left at the bottom
                for (int j = right_bound; j >= left_bound; j--) {
                    matrix[lower_bound][j] = num++;
                }
                // move the lower bound up
                lower_bound--;
            }
            
            if (left_bound <= right_bound) {
                // traverse from bottom to top on the left
                for (int i = lower_bound; i >= upper_bound; i--) {
                    matrix[i][left_bound] = num++;
                }
                // move the left bound right
                left_bound++;
            }
        }
        return matrix;
    }
}
```

<visual slug='spiral-matrix-ii' >

You can open the panel below. Click the line <code type="click">while (num <= n * n)</code> many times to see the spiral matrix being generated:

</visual>

Now both spiral matrix problems are solved.

These are some useful 2D array traversal tricks. For other array tricks, see: [Prefix Sum Array](https://labuladong.online/en/algo/data-structure/prefix-sum/), [Difference Array](https://labuladong.online/en/algo/data-structure/diff-array/), [Two-Pointer Tricks for Arrays](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/). For linked list tricks, see: [Six Key Tricks for Singly Linked Lists](https://labuladong.online/en/algo/essential-technique/linked-list-skills-summary/).
