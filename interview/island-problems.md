::: info Prerequisite Knowledge

Before reading this article, you should first learn:

- [Binary Tree Algorithms (Overview)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)
- [Backtracking Algorithm Core Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/)
- [Common Questions About Backtracking/DFS](https://labuladong.online/en/algo/essential-technique/backtrack-vs-dfs/)

:::

The island problems are classic and very common in interviews. The basic versions are not hard, but there are many interesting variations, like counting sub‑islands, or counting islands with different shapes. This article will cover them all.

**The core of island problems is using DFS/BFS to traverse a 2D grid.**

In this article, we focus on how to use DFS to quickly solve island problems. The key idea for BFS is exactly the same; you just change DFS to BFS.

How do we use DFS in a 2D grid? If you treat each cell in the grid as a node, then its up, down, left, and right neighbors are its adjacent nodes. In this way, the whole grid can be seen as a mesh‑like “graph”.

Based on the ideas from [A Framework for Learning Data Structures and Algorithms](https://labuladong.online/en/algo/essential-technique/algorithm-summary/), we can rewrite the binary tree traversal template into a DFS template for a 2D grid:

```java
// Binary tree traversal framework
void traverse(TreeNode root) {
    traverse(root.left);
    traverse(root.right);
}

// Two-dimensional matrix traversal framework
void dfs(int[][] grid, int i, int j, boolean[][] visited) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        // out of index bounds
        return;
    }
    if (visited[i][j]) {
        // already visited (i, j)
        return;
    }

    // enter the current node (i, j)
    visited[i][j] = true;

    // enter adjacent nodes (quadtree)
    // up
    dfs(grid, i - 1, j, visited);
    // down
    dfs(grid, i + 1, j, visited);
    // left
    dfs(grid, i, j - 1, visited);
    // right
    dfs(grid, i, j + 1, visited);
}
```

Because a 2D grid is essentially a “graph”, we need a `visited` boolean array during traversal to avoid going back to the same cell again. If you understand the code above, then all island problems become easy.

Here is a common trick when working with 2D arrays. You will sometimes see a “direction array” to handle moving up, down, left, and right. This is very similar to the code in [Union-Find Algorithm Explained](https://labuladong.online/en/algo/data-structure/union-find/):

```java
// Direction array, representing up, down, left, right respectively
int[][] dirs = new int[][]{{-1,0}, {1,0}, {0,-1}, {0,1}};

void dfs(int[][] grid, int i, int j, boolean[][] visited) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        // out of index bounds
        return;
    }
    if (visited[i][j]) {
        // already visited (i, j)
        return;
    }

    // enter node (i, j)
    visited[i][j] = true;
    // recursively visit the nodes above, below, left and right
    for (int[] d : dirs) {
        int next_i = i + d[0];
        int next_j = j + d[1];
        dfs(grid, next_i, next_j, visited);
    }
    // leave node (i, j)
}
```

This style just uses a for loop to handle the four directions. You can choose whichever style you prefer. Next, we will solve problems using this framework together with the visual panel.


<!-- hide -->

## Number of Islands

This is LeetCode 200: [Number of Islands](https://leetcode.com/problems/number-of-islands/). This is the simplest and most classic island problem.

The input is a 2D array `grid` that contains only `0` and `1`. `0` means water, `1` means land. Assume the whole grid is surrounded by water.

Connected `1`s form an island. You need to write an algorithm to count how many islands are in `grid`. The function signature is:

```java
int numIslands(char[][] grid);
```

For example, if the input `grid` is as below, there are 4 islands, so the algorithm should return 4:

![](../pictures/island/1.jpg)

The idea is simple. The key is how to find and mark islands. Here we use DFS. Let’s look at the code:

```java
class Solution {
    // main function, calculate the number of islands
    int numIslands(char[][] grid) {
        int res = 0;
        int m = grid.length, n = grid[0].length;
        // traverse the grid
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '1') {
                    // each time an island is found, increment the island count
                    res++;
                    // then use DFS to flood the island
                    dfs(grid, i, j);
                }
            }
        }
        return res;
    }

    // starting from (i, j), turn all adjacent land into water
    void dfs(char[][] grid, int i, int j) {
        int m = grid.length, n = grid[0].length;
        if (i < 0 || j < 0 || i >= m || j >= n) {
            // out of index bounds
            return;
        }
        if (grid[i][j] == '0') {
            // already water
            return;
        }
        // turn (i, j) into water
        grid[i][j] = '0';
        // flood the land above, below, left, and right
        dfs(grid, i + 1, j);
        dfs(grid, i, j + 1);
        dfs(grid, i - 1, j);
        dfs(grid, i, j - 1);
    }
}
```

<visual slug='number-of-islands'/>

**Why do we use DFS to “flood” an island every time we find one? The main reason is to avoid using a `visited` array.**

The `dfs` function stops when it reaches a `0`. So as long as we turn every visited cell into `0`, we will not go back to it again.

::: tip Tip

This kind of DFS is also called the FloodFill algorithm. Now you can see why this name fits well.

:::

That is all for the most basic island problem. Next, we look at some variations.

## Number of Closed Islands

In the previous problem, we assume the whole grid is surrounded by water, so land on the border also counts as islands.

LeetCode 1254: [Number of Closed Islands](https://leetcode.com/problems/number-of-closed-islands/) is different from the previous one in two ways:

1. `0` means land, `1` means water.
2. You need to count the number of “closed islands”. A “closed island” means a group of `0`s that are completely surrounded by `1`s in four directions. So **land that touches the border is not a closed island**.

The function signature is:

```java
int closedIsland(int[][] grid)
```

For example, for the following grid:

![](../pictures/island/2.png)

The algorithm should return 2. Only the gray `0`s are land that is surrounded by water in all four directions, so they are “closed islands”.

**How do we find “closed islands”? It is simple: if we remove all islands that touch the border, the remaining islands are exactly the closed islands.**

With this idea, we can go straight to the code. Note that in this problem `0` is land and `1` is water:

```java
class Solution {
    // main function: calculate the number of closed islands
    public int closedIsland(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        for (int j = 0; j < n; j++) {
            // flood the island on the top edge
            dfs(grid, 0, j);
            // flood the island on the bottom edge
            dfs(grid, m - 1, j);
        }
        for (int i = 0; i < m; i++) {
            // flood the island on the left edge
            dfs(grid, i, 0);
            // flood the island on the right edge
            dfs(grid, i, n - 1);
        }
        // traverse the grid, the remaining islands are all closed islands
        int res = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 0) {
                    res++;
                    dfs(grid, i, j);
                }
            }
        }
        return res;
    }

    // starting from (i, j), turn all adjacent land into water
    void dfs(int[][] grid, int i, int j) {
        int m = grid.length, n = grid[0].length;
        if (i < 0 || j < 0 || i >= m || j >= n) {
            return;
        }
        if (grid[i][j] == 1) {
            // it is already water
            return;
        }
        // turn (i, j) into water
        grid[i][j] = 1;
        // flood the land above, below, left and right
        dfs(grid, i + 1, j);
        dfs(grid, i, j + 1);
        dfs(grid, i - 1, j);
        dfs(grid, i, j - 1);
    }
}
```

<visual slug='number-of-closed-islands'/>

We just flood all land connected to the border first. Then, any island we count after that must be a closed island.

::: tip Tip

Besides DFS/BFS, you can also use the Union Find algorithm for island problems. In the article [Union Find In Practice](https://labuladong.online/en/algo/data-structure/union-find/), we used Union Find to solve a similar problem.

:::

With a small change, this solution also works for LeetCode 1020: [Number of Enclaves](https://leetcode.com/problems/number-of-enclaves/). That problem does not ask for the number of closed islands, but for the total area of all closed islands.

The idea is the same: first flood all land connected to the border, then count the remaining land cells. Very simple. Note that in LeetCode 1020, `1` is land and `0` is water.

Due to space, we skip the full code and move on to the next island problem.

## Max Area of Island

This is LeetCode 695: [Max Area of Island](https://leetcode.com/problems/max-area-of-island/). Here `0` is water and `1` is land. This time you are not asked to count islands, but to find the maximum area of an island. The function signature is:

```java
int maxAreaOfIsland(int[][] grid)
```

For example, for the grid below:

![](../pictures/island/3.jpg)

The orange island has the largest area. The algorithm should return 6.

**The main idea is the same as before. But while the `dfs` function is flooding an island, it should also record the area of this island.**

We can let `dfs` return the number of land cells it floods each time. Let’s look at the code:

```java
class Solution {
    public int maxAreaOfIsland(int[][] grid) {
        // record the maximum area of the island
        int res = 0;
        int m = grid.length, n = grid[0].length;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    // flood the island and update the maximum island area
                    res = Math.max(res, dfs(grid, i, j));
                }
            }
        }
        return res;
    }

    // flood the land adjacent to (i, j) and return the flooded land area
    int dfs(int[][] grid, int i, int j) {
        int m = grid.length, n = grid[0].length;
        if (i < 0 || j < 0 || i >= m || j >= n) {
            // out of index boundary
            return 0;
        }
        if (grid[i][j] == 0) {
            // already sea water
            return 0;
        }
        // turn (i, j) into sea water
        grid[i][j] = 0;

        return dfs(grid, i + 1, j)
            + dfs(grid, i, j + 1)
            + dfs(grid, i - 1, j)
            + dfs(grid, i, j - 1) + 1;
    }
}
```

<visual slug='max-area-of-island'/>

The solution is almost the same as before, so we will not repeat more here. The next two island problems are more tricky; we will focus on them.


## Number of Sub-Islands

If the previous problems are standard template problems, then LeetCode 1905 “[Count Sub Islands](https://leetcode.com/problems/count-sub-islands/)” needs a bit more thinking:

**LeetCode 1905. Count Sub Islands** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given two `m x n` binary matrices `grid1` and `grid2` containing only `0`'s (representing water) and `1`'s (representing land). An **island** is a group of `1`'s connected **4-directionally** (horizontal or vertical). Any cells outside of the grid are considered water cells.

An island in `grid2` is considered a **sub-island **if there is an island in `grid1` that contains **all** the cells that make up **this** island in `grid2`.

Return the ***number** of islands in *`grid2` *that are considered **sub-islands***.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/06/10/test1.png)

```

**Input:** grid1 = [[1,1,1,0,0],[0,1,1,1,1],[0,0,0,0,0],[1,0,0,0,0],[1,1,0,1,1]], grid2 = [[1,1,1,0,0],[0,0,1,1,1],[0,1,0,0,0],[1,0,1,1,0],[0,1,0,1,0]]
**Output:** 3
**Explanation: **In the picture above, the grid on the left is grid1 and the grid on the right is grid2.
The 1s colored red in grid2 are those considered to be part of a sub-island. There are three sub-islands.

```

Example 2:**

![](https://assets.leetcode.com/uploads/2021/06/03/testcasex2.png)

```

**Input:** grid1 = [[1,0,1,0,1],[1,1,1,1,1],[0,0,0,0,0],[1,1,1,1,1],[1,0,1,0,1]], grid2 = [[0,0,0,0,0],[1,1,1,1,1],[0,1,0,1,0],[0,1,0,1,0],[1,0,0,0,1]]
**Output:** 2 
**Explanation: **In the picture above, the grid on the left is grid1 and the grid on the right is grid2.
The 1s colored red in grid2 are those considered to be part of a sub-island. There are two sub-islands.

```

**Constraints:**

	
- `m == grid1.length == grid2.length`
	
- `n == grid1[i].length == grid2[i].length`
	
- `1 <= m, n <= 500`
	
- `grid1[i][j]` and `grid2[i][j]` are either `0` or `1`.

**The key is: how to quickly check whether an island is a sub-island**? We can use [Union Find](https://labuladong.online/en/algo/data-structure/union-find/) to solve it, but this article focuses on DFS, so we will not talk about Union Find here.

When is an island `B` in `grid2` a sub-island of some island `A` in `grid1`?

When every land cell in island `B` is also land in island `A` at the same position, then `B` is a sub-island of `A`.

**In other words, if there is any land in island `B` whose corresponding cell in island `A` is water, then `B` is not a sub-island of `A`**.

So we just need to traverse all islands in `grid2`, remove those islands that cannot be sub-islands, and the remaining ones are sub-islands.

Following this idea, we can write the code directly:

```java
class Solution {
    public int countSubIslands(int[][] grid1, int[][] grid2) {
        int m = grid1.length, n = grid1[0].length;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid1[i][j] == 0 && grid2[i][j] == 1) {
                    // this island is definitely not a sub-island, flood it
                    dfs(grid2, i, j);
                }
            }
        }
        // now the remaining islands in grid2 are all sub-islands, calculate the number of islands
        int res = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid2[i][j] == 1) {
                    res++;
                    dfs(grid2, i, j);
                }
            }
        }
        return res;
    }

    // starting from (i, j), turn all adjacent land into water
    void dfs(int[][] grid, int i, int j) {
        int m = grid.length, n = grid[0].length;
        if (i < 0 || j < 0 || i >= m || j >= n) {
            return;
        }
        if (grid[i][j] == 0) {
            return;
        }

        grid[i][j] = 0;
        dfs(grid, i + 1, j);
        dfs(grid, i, j + 1);
        dfs(grid, i - 1, j);
        dfs(grid, i, j - 1);
    }
}
```

<visual slug='count-sub-islands'/>

This problem is similar to counting the number of “closed islands”. In that problem, we remove islands that touch the border. In this problem, we remove islands that cannot be sub-islands.

## Number of Distinct Islands

This is the last island problem in this article. As the final problem, it is of course the most interesting one.

LeetCode 694 “[Number of Distinct Islands](https://leetcode.com/problems/number-of-distinct-islands/)” still gives you a 2D grid. `0` means water, `1` means land. This time you need to count the number of **distinct** islands. The function signature is:

```java
int numDistinctIslands(int[][] grid)
```

For example, suppose the input grid is:

![](../pictures/island/5.jpg)

There are four islands, but the island in the bottom-left and the island in the top-right have the same shape. So there are only three distinct islands, and the algorithm should return 3.

We clearly need to convert each island in the grid to some form, like a string, then use a data structure like a HashSet to remove duplicates and get the number of distinct islands.

To convert an island to a string is basically serialization. Serialization is basically a traversal. In a previous article, “[Serialize and Deserialize Binary Trees](https://labuladong.online/en/algo/data-structure/serialize-and-deserialize-binary-tree/)”, we talked about converting between trees and strings. It is similar here.

**First, for islands with the same shape, if we start DFS from the same relative starting point, the traversal order of the `dfs` function will be the same**.

Because the traversal order is hard-coded in your recursive function and does not change dynamically:

```java
void dfs(int[][] grid, int i, int j) {
    // recursion order:
    // up
    dfs(grid, i - 1, j);
    // down
    dfs(grid, i + 1, j);
    // left
    dfs(grid, i, j - 1);
    // right
    dfs(grid, i, j + 1);
}
```

So in some sense, the traversal order can describe the shape of an island. For example, look at these two islands:

![](../pictures/island/6.png)

Assume their DFS traversal order is:

```
down, right, up, backtrack up, backtrack right, backtrack down
```

If we use `1, 2, 3, 4` for up, down, left, right, and `-1, -2, -3, -4` for backtracking up, down, left, right, then we can write the traversal as:

```
2, 4, 1, -1, -4, -2
```

**This sequence is like the serialized result of the island. As long as we generate this sequence for each island during DFS and compare them, we can count how many distinct islands there are**.

::: info Do we have to record “backtrack” moves?

Some careful readers may ask: why must we record backtrack moves to uniquely represent the traversal order? It seems okay to skip them? No, in fact we must record them.

For example, “down, right, backtrack right, backtrack down” and “down, backtrack down, right, backtrack right” are clearly two different traversal orders. But if we do not record backtracks, both become “down, right”. They look the same, which is wrong.

:::

So we need to slightly change the `dfs` function and add parameters to record the traversal order:

```java
void dfs(int[][] grid, int i, int j, StringBuilder sb, int dir) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n 
        || grid[i][j] == 0) {
        return;
    }
    // preorder traversal position: entering (i, j)
    grid[i][j] = 0;
    sb.append(dir).append(',');
    
    // up
    dfs(grid, i - 1, j, sb, 1);
    // down
    dfs(grid, i + 1, j, sb, 2);
    // left
    dfs(grid, i, j - 1, sb, 3);
    // right
    dfs(grid, i, j + 1, sb, 4);
    
    // postorder traversal position: leaving (i, j)
    sb.append(-dir).append(',');
}
```

::: note Note

Look carefully at this code. It makes a choice before recursion and undoes the choice after recursion. Does it look like the [backtracking framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/)? It actually is a backtracking algorithm, because it focuses on the “branches” (the traversal path of the island), not the “nodes” (each cell of the island).

You can rewrite this function into the standard backtracking form.

:::

`dir` records the direction. After `dfs` finishes, `sb` stores the whole traversal sequence. With this `dfs` function, we can now write the final solution:

```java
class Solution {
    public int numDistinctIslands(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        // record the serialized results of all islands
        HashSet<String> islands = new HashSet<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    // flood this island and store the serialized result of the island
                    StringBuilder sb = new StringBuilder();
                    // the initial direction can be written arbitrarily, it does not affect correctness
                    dfs(grid, i, j, sb, 666);
                    islands.add(sb.toString());
                    /**<extend up -200>
                    ![](../pictures/island/6.png)
                    */
                }
            }
        }
        // the number of distinct islands
        return islands.size();
    }

    private void dfs(int[][] grid, int i, int j, StringBuilder sb, int dir) {
        // see above
    }
}
```

<visual slug='number-of-distinct-islands'/>

That solves this problem. As for why the initial `dir` parameter in the first `dfs` call can be anything, it is because this `dfs` function is actually a backtracking algorithm. It focuses on the “branches” instead of the “nodes”. The difference is explained in detail in “[Graph Basics](https://labuladong.online/en/algo/data-structure-basic/graph-basic/)”, so we will not repeat it here.

These are all the ideas for the island problems in this series. Many people may be able to solve the earlier problems, but the last two are more clever. I hope this article is helpful to you.
