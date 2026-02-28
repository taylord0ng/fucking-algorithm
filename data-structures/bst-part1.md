::: info Prerequisites

Before reading this article, you need to study:

- [Binary Tree Basics](https://labuladong.online/en/algo/data-structure-basic/binary-tree-basic/)
- [DFS/BFS Traversal of Binary Trees](https://labuladong.online/en/algo/data-structure-basic/binary-tree-traverse-basic/)

:::

In previous articles, we covered binary trees step by step. We wrote [Thinking](https://labuladong.online/en/algo/data-structure/binary-tree-part1/), [Construction](https://labuladong.online/en/algo/data-structure/binary-tree-part2/), [Postorder](https://labuladong.online/en/algo/data-structure/binary-tree-part3/), and [Serialization](https://labuladong.online/en/algo/data-structure/serialize-and-deserialize-binary-tree/) articles.

Today, we start a series on Binary Search Tree (BST). I will guide you through solving BST problems step by step.

First, you should be familiar with the properties of BST (see [Binary Tree Basics](https://labuladong.online/en/algo/data-structure-basic/binary-tree-basic/)):

1. For any node `node` in a BST, all nodes in the left subtree are smaller than `node`, and all nodes in the right subtree are larger than `node`.

2. For any node `node`, its left and right subtrees are also BSTs.

Binary search trees are not very complex, but they are very important in data structures. Data structures like AVL tree and Red-Black tree are based on BST and have self-balancing properties. These provide logN time for insertion, deletion, search, and update. B+ trees, segment trees, and other structures are also designed with BST ideas.

**From the perspective of solving algorithm problems, besides its definition, BST has another key property: the result of its inorder traversal is sorted in ascending order.**

This means, if you have a BST, the following code can print all node values in ascending order:

```java
void traverse(TreeNode root) {
    if (root == null) return;
    traverse(root.left);
    // inorder traversal code position
    print(root.val);
    traverse(root.right);
}
```

Now, based on this property, let's solve two algorithm problems.


<!-- hide -->

## Find the Kth Smallest Element

This is LeetCode Problem 230: [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/). Let's look at the problem:

**LeetCode 230. Kth Smallest Element in a BST** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given the `root` of a binary search tree, and an integer `k`, return *the* `k^(th)` *smallest value (**1-indexed**) of all the values of the nodes in the tree*.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/28/kthtree1.jpg)

```

**Input:** root = [3,1,4,null,2], k = 1
**Output:** 1

```

Example 2:**

![](https://assets.leetcode.com/uploads/2021/01/28/kthtree2.jpg)

```

**Input:** root = [5,3,6,2,4,null,null,1], k = 3
**Output:** 3

```

**Constraints:**

	
- The number of nodes in the tree is `n`.
	
- `1 <= k <= n <= 10^(4)`
	
- `0 <= Node.val <= 10^(4)`

**Follow up:** If the BST is modified often (i.e., we can do insert and delete operations) and you need to find the kth smallest frequently, how would you optimize?

This is a common problem. An easy idea is to sort the elements in ascending order and then find the k-th element. The in-order traversal of a BST gives you the elements in order, so finding the k-th smallest is not hard.

Following this idea, you can write the code like this:

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        // use the in-order traversal property of BST
        traverse(root, k);
        return res;
    }

    // record the result
    int res = 0;
    // record the rank of the current element
    int rank = 0;
    void traverse(TreeNode root, int k) {
        if (root == null) {
            return;
        }
        traverse(root.left, k);
        // in-order code position
        rank++;
        if (k == rank) {
            // found the k-th smallest element
            res = root.val;
            return;
        }

        traverse(root.right, k);
    }
}
```

<visual slug='kth-smallest-element-in-a-bst'/>

That's it for this problem. But let me say more. This solution is not the most efficient. It only works well for this problem.

In the previous article [Efficiently Find the Median of a Data Stream](https://labuladong.online/en/algo/practice-in-action/find-median-from-data-stream/), we talked about this problem:

::: info Info

If you need to implement a method `select(int k)` in a binary search tree to get the element by rank, how would you design it?

:::

If you use the in-order traversal method we just mentioned, each time you want to find the k-th smallest element, you must traverse the whole tree. The worst-case time complexity is $O(N)$, where `N` is the number of nodes in the BST.

But the BST is powerful. For example, in balanced BSTs like Red-Black Trees, insert, delete, search, and update are all $O(\log N)$ time. But finding the k-th smallest element this way is $O(N)$, which is not efficient.

So, to find the k-th smallest element, the best algorithm should also have logarithmic time complexity. But this depends on how much information each node in the BST stores.

Why are BST operations so fast? For example, to search for a value, BST can find an element in logarithmic time because of its property: the left subtree is smaller, the right subtree is bigger. Each node can compare its value to the target and decide to go left or right. This avoids searching the whole tree and keeps complexity low.

Back to our problem: to find the k-th smallest element, or the element with rank k, we also want logarithmic time complexity. The key is: every node should know its own rank.

For example, if you want to find the element with rank k, and the current node knows its own rank is m, you can compare `m` and `k`:

1. If `m == k`, you found the k-th element. Return the current node.
2. If `k < m`, the k-th element is in the left subtree. Go search the left subtree for the k-th element.
3. If `k > m`, the k-th element is in the right subtree. Search the right subtree for the `(k - m - 1)`-th element.

With this, the time complexity becomes $O(\log N)$.

So, how can each node know its rank?

This is where we need to keep extra information in each node. **Each node should record the number of nodes in its subtree (including itself).**

So, the `TreeNode` should look like this:

```java
class TreeNode {
    int val;
    // total number of nodes in the tree rooted at this node
    int size;
    TreeNode left;
    TreeNode right;
}
```

With the `size` field and the BST property (left is smaller, right is bigger), each node can use `node.left` to calculate its rank. This makes the algorithm logarithmic time.

Of course, the `size` field must be updated correctly when nodes are added or removed. LeetCode's `TreeNode` does not have this field, so for this problem, you can only use the in-order traversal method. But the idea above is a common BST optimization. It's good to understand it.


## Convert BST to Greater Tree

LeetCode problems 538 and 1038 are actually the same question. You can solve them together. Here is the problem:

**LeetCode 538. Convert BST to Greater Tree** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given the `root` of a Binary Search Tree (BST), convert it to a Greater Tree such that every key of the original BST is changed to the original key plus the sum of all keys greater than the original key in BST.

As a reminder, a *binary search tree* is a tree that satisfies these constraints:

	
- The left subtree of a node contains only nodes with keys **less than** the node's key.
	
- The right subtree of a node contains only nodes with keys **greater than** the node's key.
	
- Both the left and right subtrees must also be binary search trees.

Example 1:**

![](https://assets.leetcode.com/uploads/2019/05/02/tree.png)

```

**Input:** root = [4,1,6,0,2,5,7,null,null,null,3,null,null,null,8]
**Output:** [30,36,21,36,35,26,15,null,null,null,33,null,null,null,8]

```

Example 2:**

```

**Input:** root = [0,null,1]
**Output:** [1,null,1]

```

**Constraints:**

	
- The number of nodes in the tree is in the range `[0, 10^(4)]`.
	
- `-10^(4) <= Node.val <= 10^(4)`
	
- All the values in the tree are **unique**.
	
- `root` is guaranteed to be a valid binary search tree.

**Note:** This question is the same as 1038: [https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/](https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/)

The problem is easy to understand. For example, look at node 5 in the picture. To convert to a greater tree, you need to add all nodes greater than 5, which are 6, 7, and 8, plus 5 itself. So, the value of this node in the greater tree should be 5+6+7+8=26.

We need to convert a BST to a greater tree. The function signature is as follows:

```java
TreeNode convertBST(TreeNode root)
```

According to the usual way of solving binary tree problems, we should think about what to do at each node. But for this problem, it is hard to come up with a good idea right away.

Each node in a BST has smaller values on the left and bigger values on the right. This seems useful. Since we want the sum of all values greater than or equal to the current node, can't we just calculate the sum of the right subtree for every node?

Actually, this is not enough. The right subtree does contain bigger values, but the parent node might also have a bigger value. We cannot be sure because we do not have a pointer to the parent node. So, the usual binary tree approach does not work here.

**Let's try another way and use the property of BST's in-order traversal**.

Earlier, we said that an in-order traversal of a BST prints the node values in increasing order. What if we want to print the values in decreasing order?

It is simple. We just change the recursion order: visit the right subtree first, then the left subtree:

```java
void traverse(TreeNode root) {
    if (root == null) return;
    // First, recursively traverse the right subtree
    traverse(root.right);
    // Inorder traversal code position
    print(root.val);
    // Then, recursively traverse the left subtree
    traverse(root.left);
}
```

**This code can print the BST node values in decreasing order. If we keep an external variable `sum` and set each BST node's value to `sum`, we can convert the BST to a greater tree.**

Let's look at the code:

```java
class Solution {
    public TreeNode convertBST(TreeNode root) {
        traverse(root);
        return root;
    }

    // record the cumulative sum
    int sum = 0;
    void traverse(TreeNode root) {
        if (root == null) {
            return;
        }
        traverse(root.right);
        // maintain the cumulative sum
        sum += root.val;
        // convert bst to cumulative tree
        root.val = sum;
        traverse(root.left);
    }
}
```

<visual slug='convert-bst-to-greater-tree'/>

That solves the problem. The key is still the in-order traversal of the BST. We just changed the order of recursion to visit the right subtree first, so we visit nodes from biggest to smallest, which matches the requirement for the greater tree.

To sum up, for BST problems, you should either use the BST property (left is smaller, right is bigger) to make your algorithm faster, or use in-order traversal to meet the problem's needs. That's the main idea.

That's all for this article. For more classic binary tree problems and practice with recursion, please see the [Exercise section](https://labuladong.online/en/algo/problem-set/bst1/) in the binary tree chapter.
