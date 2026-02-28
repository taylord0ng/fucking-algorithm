::: info Prerequisite

Before reading this article, you should first learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)

:::

The prefix sum technique is used to quickly and repeatedly calculate the sum of elements in a range of indices.

## Prefix Sum in a 1D Array

Let’s look at an example: LeetCode 303 “[Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/)”. You need to calculate the sum of elements in a subarray. This is a standard prefix sum problem:

**LeetCode 303. Range Sum Query - Immutable** <span class="inline-block w-2 h-2 rounded-full bg-green-500"></span>

Given an integer array `nums`, handle multiple queries of the following type:

	
- Calculate the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** where `left <= right`.

Implement the `NumArray` class:

	
- `NumArray(int[] nums)` Initializes the object with the integer array `nums`.
	
- `int sumRange(int left, int right)` Returns the **sum** of the elements of `nums` between indices `left` and `right` **inclusive** (i.e. `nums[left] + nums[left + 1] + ... + nums[right]`).

Example 1:**

```

**Input**
["NumArray", "sumRange", "sumRange", "sumRange"]
[[[-2, 0, 3, -5, 2, -1]], [0, 2], [2, 5], [0, 5]]
**Output**
[null, 1, -1, -3]

**Explanation**
NumArray numArray = new NumArray([-2, 0, 3, -5, 2, -1]);
numArray.sumRange(0, 2); // return (-2) + 0 + 3 = 1
numArray.sumRange(2, 5); // return 3 + (-5) + 2 + (-1) = -1
numArray.sumRange(0, 5); // return (-2) + 0 + 3 + (-5) + 2 + (-1) = -3

```

**Constraints:**

	
- `1 <= nums.length <= 10^(4)`
	
- `-10^(5) <= nums[i] <= 10^(5)`
	
- `0 <= left <= right < nums.length`
	
- At most `10^(4)` calls will be made to `sumRange`.

```java
// The problem requires you to implement such a class
class NumArray {

    public NumArray(int[] nums) {}
    
    // Query the cumulative sum of the closed interval [left, right]
    public int sumRange(int left, int right) {}
}
```

The `sumRange` function needs to return the sum of elements in a given index range. Without prefix sum, people may write code like this:

```java
class NumArray {
	private int[] nums;
	public NumArray(int[] nums) {
		this.nums = nums;
	}

	public int sumRange(int left, int right) {
		// use a for loop to compute the sum
		int sum = 0;
		for (int i = left; i <= right; i++) {
			sum += nums[i];
		}
		return sum;
	}
}
```

This solution runs a for loop every time `sumRange` is called. The time complexity is $O(N)$. Since `sumRange` may be called very often, this is not efficient.

The correct way is to use prefix sum to optimize it, so the time complexity of `sumRange` becomes $O(1)$:

```java
class NumArray {
    // prefix sum array
    private int[] preSum;

    // input an array to construct the prefix sum
    public NumArray(int[] nums) {
        // preSum[0] = 0, to facilitate the calculation of accumulated sums
        preSum = new int[nums.length + 1];
        // calculate the accumulated sums of nums
        for (int i = 1; i < preSum.length; i++) {
            preSum[i] = preSum[i - 1] + nums[i - 1];
        }
    }

    // query the sum of the closed interval [left, right]
    public int sumRange(int left, int right) {
        return preSum[right + 1] - preSum[left];
    }
}
```

The key idea is to create a new array `preSum`, where `preSum[i]` stores the sum of `nums[0..i-1]`. See the picture, where $10 = 3 + 5 + 2$:

![](../pictures/difference/1.jpeg)

With this `preSum` array, if we want the sum of the elements in the index range `[1, 4]`, we can compute `preSum[5] - preSum[1]`.

In this way, `sumRange` only needs one subtraction, no loop, so the worst-case time complexity is constant $O(1)$.

<visual slug='range-sum-query-immutable' >

Open the visualization below and click the line <code type="click">preSum[i] = preSum[i - 1] + nums[i - 1]</code> to see how the `preSum` array is built. Click the line <code type="click">console.log</code> multiple times to see calls to `sumRange`:

</visual>

This technique also appears in real life. For example, your class has several students, each with a final exam score (full score is 100). You need to design an API: given a score range, return how many students have scores in this range.

You can first use counting sort to count how many students got each score, then use prefix sum to build the score-range query API:

```java
// scores of all students
int[] scores = new int[]{...};
// full score is 100
int[] count = new int[100 + 1];

// count how many students got each score
for (int score : scores) {
    count[score]++;
}
// build the prefix sum array
for (int i = 1; i < count.length; i++) {
    count[i] = count[i] + count[i-1];
}

// use the prefix sum array `count` to do range queries

// query how many students scored in [80, 90]
int result = count[90] - count[79];
```

Next, we will see how the prefix sum idea works in 2D arrays.


## Prefix Sum in 2D Matrix

This is LeetCode 304: [Range Sum Query 2D – Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/). It is very similar to the previous problem. The previous one asked you to compute the sum of a subarray. This one asks you to compute the sum of a submatrix in a 2D matrix:

**LeetCode 304. Range Sum Query 2D - Immutable** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given a 2D matrix `matrix`, handle multiple queries of the following type:

	
- Calculate the **sum** of the elements of `matrix` inside the rectangle defined by its **upper left corner** `(row1, col1)` and **lower right corner** `(row2, col2)`.

Implement the `NumMatrix` class:

	
- `NumMatrix(int[][] matrix)` Initializes the object with the integer matrix `matrix`.
	
- `int sumRegion(int row1, int col1, int row2, int col2)` Returns the **sum** of the elements of `matrix` inside the rectangle defined by its **upper left corner** `(row1, col1)` and **lower right corner** `(row2, col2)`.

You must design an algorithm where `sumRegion` works on `O(1)` time complexity.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/14/sum-grid.jpg)

```

**Input**
["NumMatrix", "sumRegion", "sumRegion", "sumRegion"]
[[[[3, 0, 1, 4, 2], [5, 6, 3, 2, 1], [1, 2, 0, 1, 5], [4, 1, 0, 1, 7], [1, 0, 3, 0, 5]]], [2, 1, 4, 3], [1, 1, 2, 2], [1, 2, 2, 4]]
**Output**
[null, 8, 11, 12]

**Explanation**
NumMatrix numMatrix = new NumMatrix([[3, 0, 1, 4, 2], [5, 6, 3, 2, 1], [1, 2, 0, 1, 5], [4, 1, 0, 1, 7], [1, 0, 3, 0, 5]]);
numMatrix.sumRegion(2, 1, 4, 3); // return 8 (i.e sum of the red rectangle)
numMatrix.sumRegion(1, 1, 2, 2); // return 11 (i.e sum of the green rectangle)
numMatrix.sumRegion(1, 2, 2, 4); // return 12 (i.e sum of the blue rectangle)

```

**Constraints:**

	
- `m == matrix.length`
	
- `n == matrix[i].length`
	
- `1 <= m, n <= 200`
	
- `-10^(4) <= matrix[i][j] <= 10^(4)`
	
- `0 <= row1 <= row2 < m`
	
- `0 <= col1 <= col2 < n`
	
- At most `10^(4)` calls will be made to `sumRegion`.

Of course, you can use nested for loops to scan the matrix. But then the time complexity of the `sumRegion` function will be high, and your algorithm will look naive.

Notice that the sum of any submatrix can be turned into a combination of sums of several larger rectangles:

![](../pictures/presum/5.jpeg)

These four large rectangles share a key feature: their top-left corner is always the origin `(0, 0)`.

So the better idea for this problem is very similar to prefix sum in a 1D array. We can keep a 2D `preSum` array. `preSum[i][j]` records the sum of all elements in the matrix with top-left at the origin and bottom-right at `(i, j)`. Then we can use a few additions and subtractions to get the sum of any submatrix:

```java
class NumMatrix {
    // preSum[i][j] records the sum of elements in the matrix [0, 0, i-1, j-1]
    private int[][] preSum;

    public NumMatrix(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        if (m == 0 || n == 0) return;
        // construct the prefix sum matrix
        preSum = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // calculate the sum of elements for each matrix [0, 0, i, j]
                preSum[i][j] = preSum[i-1][j] + preSum[i][j-1] + matrix[i - 1][j - 1] - preSum[i-1][j-1];
            }
        }
    }

    // calculate the sum of elements in the submatrix [x1, y1, x2, y2]
    public int sumRegion(int x1, int y1, int x2, int y2) {
        // the sum of the target matrix is obtained by operations on four adjacent matrices
        return preSum[x2+1][y2+1] - preSum[x1][y2+1] - preSum[x2+1][y1] + preSum[x1][y1];
    }
}
```

In this way, the time complexity of the `sumRegion` function is optimized to $O(1)$ with the prefix sum trick. This is a classic “space-for-time” method.

<visual slug='range-sum-query-2d-immutable' >

You can open the visualization below. Click the line <code type="click">preSum[i][j] = ...</code> many times to see how the <code type="click">preSum</code> array is computed. Click the <code type="click">console.log</code> line many times to see how the <code type="click">sumRegion</code> function is called:

</visual>

This is all for the prefix sum technique. You can say this skill is easy once you get it, but hard if you do not. In real problems, you should train your flexibility in thinking, so you can quickly see that a problem is a prefix sum problem.


## Further Expansion

The prefix sum technique explained in this article uses a precomputed `preSum` array to quickly calculate the sum of elements within a given index range. However, it is not limited to summation; it can also be used for quickly computing products and other scenarios.

Moreover, prefix sum arrays are often combined with other data structures or algorithmic techniques. I will explain these combinations in [High-Frequency Prefix Sum Exercises](https://labuladong.online/en/algo/problem-set/perfix-sum/) along with relevant exercises.

However, the prefix sum technique has several limitations.

**First Limitation: The prerequisite for using the prefix sum technique is that the original array `nums` does not change.**

If an element in the original array changes, the values in the `preSum` array after that element become invalid, requiring $O(n)$ time to recalculate the `preSum` array, which is similar to the brute-force method.

**Second Limitation: The prefix sum technique is only applicable in scenarios with inverse operations.**

For example, in summation scenarios, if you know $x + 6 = 10$, you can deduce $x = 10 - 6 = 4$. Similarly, in product scenarios, if you know $x * 6 = 12$, you can deduce $x = 12 / 6 = 2$. This is known as having an inverse operation, which allows the use of the prefix sum technique.

However, some scenarios do not have inverse operations. For example, in maximum value scenarios, if you know $max(x, 8) = 8$, you cannot deduce the value of $x$.

To address both issues simultaneously, more advanced data structures are required. The most general solution is the [Segment Tree](https://labuladong.online/en/algo/data-structure-basic/segment-tree-basic/), which will be explained in detail in the data structure design chapter.
