::: info Prerequisite

Before reading this article, you should first learn:

- [Basics of Binary Tree Structure](https://labuladong.online/en/algo/data-structure-basic/binary-tree-basic/)
- [DFS/BFS Traversal of Binary Trees](https://labuladong.online/en/algo/data-structure-basic/binary-tree-traverse-basic/)

:::

In the previous article [Binary Search Tree Basics (Features)](https://labuladong.online/en/algo/data-structure/bst-part1/), we introduced the basic features of BST and used the "in-order traversal is sorted" property to solve some problems. In this article, we'll implement the basic operations of BST: checking if a BST is valid, inserting, deleting, and searching. Among these, "deleting" and "checking validity" are a bit more complex.

The basic operations of BST mainly rely on the "left smaller, right bigger" property. This allows us to do binary search-like operations in the tree, making it very efficient to find an element. For example, the following is a valid binary search tree:

![](../pictures/bst/0.png)

For BST problems, you will often see code logic like this:

```java
void BST(TreeNode root, int target) {
    if (root.val == target)
        // found the target, do something
    if (root.val < target) 
        BST(root.right, target);
    if (root.val > target)
        BST(root.left, target);
}
```

This code framework is similar to the usual binary tree traversal, just using the BST "left smaller, right bigger" feature. Next, let's see how the basic operations on BST are implemented.

## 1. Check if a BST is Valid

LeetCode Problem 98 "[Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)" asks you to check if a given BST is valid:

**LeetCode 98. Validate Binary Search Tree** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given the `root` of a binary tree, *determine if it is a valid binary search tree (BST)*.

A **valid BST** is defined as follows:

	
- The left subtree of a node contains only nodes with keys **less than** the node's key.
	
- The right subtree of a node contains only nodes with keys **greater than** the node's key.
	
- Both the left and right subtrees must also be binary search trees.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/12/01/tree1.jpg)

```

**Input:** root = [2,1,3]
**Output:** true

```

Example 2:**

![](https://assets.leetcode.com/uploads/2020/12/01/tree2.jpg)

```

**Input:** root = [5,1,4,null,null,3,6]
**Output:** false
**Explanation:** The root node's value is 5 but its right child's value is 4.

```

**Constraints:**

	
- The number of nodes in the tree is in the range `[1, 10^(4)]`.
	
- `-2^(31) <= Node.val <= 2^(31) - 1`

Be careful, there is a trap here. According to the BST property, every node should be compared with its left and right children to check if it is valid. It seems you might write code like this:

```java
boolean isValidBST(TreeNode root) {
    if (root == null) return true;
    // the left side of the root should be smaller
    if (root.left != null && root.left.val >= root.val)
        return false;
    // the right side of the root should be larger
    if (root.right != null && root.right.val <= root.val)
        return false;

    return isValidBST(root.left)
        && isValidBST(root.right);
}
```

But this algorithm is wrong. For BST, every node must be less than **all** nodes in its right subtree. The following tree is not a valid BST because there is a node `8` in the left subtree of node `7`, but our code would say it is valid:

```yaml BinaryTree
root:
  value: 7
  color: orange
  left:
    value: 4
    left:
      value: 1
    right:
      value: 8
      color: orange
  right:
    value: 9
    right:
      value: 10
```

**The reason for the mistake is that, for each node `root`, the code only checks its left and right child nodes. But, by the definition of BST, the whole left subtree of `root` must be less than `root.val`, and the whole right subtree must be greater than `root.val`.**

The problem is, for a node `root`, it can only directly check its children. How do we pass this constraint down to all nodes of the left and right subtree? Here is the correct code:

```java
class Solution {
    public boolean isValidBST(TreeNode root) {
        return isValidBST(root, null, null);
    }

    // nodes in the subtree rooted at root must satisfy max.val > root.val > min.val
    boolean isValidBST(TreeNode root, TreeNode min, TreeNode max) {
        // base case
        if (root == null) return true;
        // if root.val does not satisfy the constraints of max and min, it is not a valid BST
        if (min != null && root.val <= min.val) return false;
        if (max != null && root.val >= max.val) return false;
        // the maximum value in the left subtree is root.val, and the minimum value in the right subtree is root.val
        return isValidBST(root.left, min, root)
                && isValidBST(root.right, root, max);
    }
}
```

<visual slug='validate-binary-search-tree'/>

We use a helper function and add extra parameters to pass down these constraints to all child nodes. This is also a useful trick in binary tree algorithms.


## Search for an Element in a BST

LeetCode problem 700, "[Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/)," asks you to find a node with value `target` in a BST. The function signature is as follows:

```java
TreeNode searchBST(TreeNode root, int target);
```

If you are searching in a normal binary tree, you can write the code like this:

```java
TreeNode searchBST(TreeNode root, int target) {
    if (root == null) return null;
    if (root.val == target) return root;
    // if the current node is not found, recursively search the left and right subtrees
    TreeNode left = searchBST(root.left, target);
    TreeNode right = searchBST(root.right, target);

    return left != null ? left : right;
}
```

This code is correct, but it checks all nodes, which is brute-force and works for any binary tree. But how can we use the special property of BST, where the left side is smaller and the right side is larger?

It is simple. You do not need to search both sides. You can use a binary search idea: compare `target` with `root.val`. This way, you can ignore one side. Let's change the code using this idea:

```java
// define: search for the node with value target in the BST rooted at root, return the node
TreeNode searchBST(TreeNode root, int target) {
    if (root == null) {
        return null;
    }
    // search in the left subtree
    if (root.val > target) {
        return searchBST(root.left, target);
    }
    // search in the right subtree
    if (root.val < target) {
        return searchBST(root.right, target);
    }
    // the current node is the target value
    return root;
}
```

<visual slug='search-in-a-binary-search-tree'/>

## Insert a Number into a BST

When working with data structures, you usually travel through (find) and visit (change) the nodes. For this problem, to insert a number, you first find the right position, then insert it.

BSTs usually do not have duplicate values, so you do not need to insert a value that is already in the BST. **The code below assumes you will not insert a value that already exists in the BST.**

In the last problem, we summarized the way to travel through a BST, which is the "find" part. Now, just add the "change" part.

**If you need to change the tree, it is like building a binary tree; the function should return `TreeNode`, and you need to use the result from the recursive call.**

LeetCode problem 701, "[Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/)," is about this:

**LeetCode 701. Insert into a Binary Search Tree** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given the `root` node of a binary search tree (BST) and a `value` to insert into the tree. Return *the root node of the BST after the insertion*. It is **guaranteed** that the new value does not exist in the original BST.

**Notice** that there may exist multiple valid ways for the insertion, as long as the tree remains a BST after insertion. You can return **any of them**.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/05/insertbst.jpg)

```

**Input:** root = [4,2,7,1,3], val = 5
**Output:** [4,2,7,1,3,5]
**Explanation:** Another accepted tree is:
![](https://assets.leetcode.com/uploads/2020/10/05/bst.jpg)

```

Example 2:**

```

**Input:** root = [40,20,60,10,30,50,70], val = 25
**Output:** [40,20,60,10,30,50,70,null,null,25]

```

Example 3:**

```

**Input:** root = [4,2,7,1,3,null,null,null,null,null,null], val = 5
**Output:** [4,2,7,1,3,5]

```

**Constraints:**

	
- The number of nodes in the tree will be in the range `[0, 10^(4)]`.
	
- `-10^(8) <= Node.val <= 10^(8)`
	
- All the values `Node.val` are **unique**.
	
- `-10^(8) <= val <= 10^(8)`
	
- It's **guaranteed** that `val` does not exist in the original BST.

Let's look at the solution code. You can use the comments and visual panel to help you understand:

```java
class Solution {
    public TreeNode insertIntoBST(TreeNode root, int val) {
        // find the empty spot to insert the new node
        if (root == null) return new TreeNode(val);
        // if (root.val == val)
        // generally, in a bst, we don't insert an element that already exists
        if (root.val < val)
            root.right = insertIntoBST(root.right, val);
        if (root.val > val)
            root.left = insertIntoBST(root.left, val);
        return root;
    }
}
```

<visual slug='insert-into-a-binary-search-tree'/>


## 3. Delete a Node in a BST

LeetCode Problem 450: [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) asks you to delete a node with value `key` from a BST:

**LeetCode 450. Delete Node in a BST** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given a root node reference of a BST and a key, delete the node with the given key in the BST. Return *the **root node reference** (possibly updated) of the BST*.

Basically, the deletion can be divided into two stages:

	
- Search for a node to remove.
	
- If the node is found, delete the node.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/09/04/del_node_1.jpg)

```

**Input:** root = [5,3,6,2,4,null,7], key = 3
**Output:** [5,4,6,2,null,null,7]
**Explanation:** Given key to delete is 3. So we find the node with value 3 and delete it.
One valid answer is [5,4,6,2,null,null,7], shown in the above BST.
Please notice that another valid answer is [5,2,6,null,4,null,7] and it's also accepted.
![](https://assets.leetcode.com/uploads/2020/09/04/del_node_supp.jpg)

```

Example 2:**

```

**Input:** root = [5,3,6,2,4,null,7], key = 0
**Output:** [5,3,6,2,4,null,7]
**Explanation:** The tree does not contain a node with value = 0.

```

Example 3:**

```

**Input:** root = [], key = 0
**Output:** []

```

**Constraints:**

	
- The number of nodes in the tree is in the range `[0, 10^(4)]`.
	
- `-10^(5) <= Node.val <= 10^(5)`
	
- Each node has a **unique** value.
	
- `root` is a valid binary search tree.
	
- `-10^(5) <= key <= 10^(5)`

**Follow up:** Could you solve it with time complexity `O(height of tree)`?

This problem is a bit tricky. Like insertion, you need to "find" first, then "change." Let's write the basic structure first:

```java
TreeNode deleteNode(TreeNode root, int key) {
    if (root.val == key) {
        // found it, proceed to delete
    } else if (root.val > key) {
        // go to the left subtree
        root.left = deleteNode(root.left, key);
    } else if (root.val < key) {
        // go to the right subtree
        root.right = deleteNode(root.right, key);
    }
    return root;
}
```

After finding the target node, let's say it is node `A`, how do we delete it? This is the main challenge, because you can't break the properties of the BST. There are three possible cases, shown with pictures.

**Case 1**: `A` is a leaf node (both children are null). You can simply delete it.

![](../pictures/bst/bst_deletion_case_1.png)

```java
if (root.left == null && root.right == null)
    return null;
```

**Case 2**: `A` has only one non-empty child. Let this child take `A`'s place.

![](../pictures/bst/bst_deletion_case_2.png)

```java
// After excluding case 1
if (root.left == null) return root.right;
if (root.right == null) return root.left;
```

**Case 3**: `A` has two children. This is more complex. To keep the BST property, you must find either the largest node in the left subtree or the smallest node in the right subtree to replace `A`. We will explain the second way.

![](../pictures/bst/bst_deletion_case_3.png)

```java
if (root.left != null && root.right != null) {
    // Find the smallest node in the right subtree
    TreeNode minNode = getMin(root.right);
    // Replace root with minNode
    root.val = minNode.val;
    // Delete minNode
    root.right = deleteNode(root.right, minNode.val);
}
```

After explaining all three cases, fill them into the framework and simplify the code:

```java
class Solution {
    public TreeNode deleteNode(TreeNode root, int key) {
        if (root == null) return null;
        if (root.val == key) {
            // these two if statements correctly handle cases 1 and 2
            if (root.left == null) return root.right;
            if (root.right == null) return root.left;
            // handle case 3
            // get the smallest node in the right subtree
            TreeNode minNode = getMin(root.right);
            // delete the smallest node in the right subtree
            root.right = deleteNode(root.right, minNode.val);
            // replace the root node with the smallest node from the right subtree
            minNode.left = root.left;
            minNode.right = root.right;
            root = minNode;
        } else if (root.val > key) {
            root.left = deleteNode(root.left, key);
        } else if (root.val < key) {
            root.right = deleteNode(root.right, key);
        }
        return root;
    }

    TreeNode getMin(TreeNode node) {
        // the leftmost node in a BST is the smallest
        while (node.left != null) node = node.left;
        return node;
    }
}
```

<visual slug='delete-node-in-a-bst'/>

Now, the delete operation is finished. Note: In case 3, the code above replaces the `root` node with `minNode` by swapping their values, which is a bit simpler:

```java
// Handle case 3
// Get the smallest node in the right subtree
TreeNode minNode = getMin(root.right);
// Delete the smallest node in the right subtree
root.right = deleteNode(root.right, minNode.val);
// Replace root node with the smallest node in the right subtree
minNode.left = root.left;
minNode.right = root.right;
root = minNode;
```

Some readers may wonder why we need to swap the nodes with pointer operations. Why not just change the `val` field? It looks easier:

```java
// Handle case 3
// Get the smallest node in the right subtree
TreeNode minNode = getMin(root.right);
// Delete the smallest node in the right subtree
root.right = deleteNode(root.right, minNode.val);
// Replace root node with the smallest node in the right subtree
root.val = minNode.val;
```

For this algorithm problem, it is fine. But in real applications, we do not swap nodes by just changing the internal value. The data inside a BST node could be very complex, and the BST structure should not care about the actual data. So, we prefer using pointers to swap nodes.

Let's summarize a few key points from this article:

1. If the current node will affect its children, you can pass information by adding parameters to helper functions.
2. Master how to insert, delete, search, and update nodes in a BST.
3. When changing a data structure with recursion, always receive the return value and return the updated node.

That's all for this article. For more classic binary tree problems and recursion practice, see the [Recursion Practice for Binary Search Trees](https://labuladong.online/en/algo/problem-set/bst1/) in the binary tree chapter.
