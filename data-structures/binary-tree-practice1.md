::: info Prerequisites

Before reading this article, you should first study:

- [Basics of Binary Tree Structure](https://labuladong.online/en/algo/data-structure-basic/binary-tree-basic/)
- [Binary Tree DFS/BFS Traversal](https://labuladong.online/en/algo/data-structure-basic/binary-tree-traverse-basic/)
- [Binary Tree Essentials (Guiding Principles)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/)

:::

This article continues from [Binary Tree Essentials (Guiding Principles)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/). Let’s first review the key points summarized in the previous article:

::: note Note

There are two main thinking patterns for solving binary tree problems:

**1. Can you get the answer by traversing the binary tree once?**  
If yes, use a `traverse` function with external variables—this is called the "traversal" approach.

**2. Can you define a recursive function that derives the answer to the original problem using answers from its subproblems (subtrees)?**  
If yes, write out this recursive function and make full use of its return value—this is called the "problem decomposition" approach.

No matter which approach you use, you should always think:

**If you isolate a single tree node, what does it need to do? At what point (preorder/inorder/postorder) should it do it?**  
You don’t need to worry about other nodes, as the recursive function will perform the same operation on all of them.

:::

This article uses a few simple examples to help you practice these key principles, and to understand the differences and connections between the "traversal" and "problem decomposition" approaches.


<!-- hide -->

## Problem 1: Invert Binary Tree

Let's start with an easy problem. Look at LeetCode 226: [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/). The input is the root node `root` of a binary tree. You need to flip the tree so that it becomes its mirror image. For example, the input tree is:

```yaml BinaryTree
root:
  value: 4
  left:
    value: 2
    left:
      value: 1
    right:
      value: 3
  right:
    value: 7
    left:
      value: 6
    right:
      value: 9
```

After the algorithm inverts the tree, it should look like this:

```yaml BinaryTree
root:
  value: 4
  left:
    value: 7
    left:
      value: 9
    right:
      value: 6
  right:
    value: 2
    left:
      value: 3
    right:
      value: 1
```

It is easy to see that if you swap the left and right children of every node in the tree, the whole tree will be inverted.

Now, let's recall our general approach for binary tree problems:

**1. Can we solve this problem with a traversal approach?**

Yes. We can write a `traverse` function to visit every node, and at each node, swap its left and right children.

What should we do at each node? Just swap its left and right children.

When should we do this? We can do it in pre-order, in-order, or post-order. Any of them work.

So, the code can be written like this:

```java
class Solution {
    // main function
    public TreeNode invertTree(TreeNode root) {
        // traverse the binary tree, swap the child nodes of each node
        traverse(root);
        return root;
    }

    // binary tree traversal function
    void traverse(TreeNode root) {
        if (root == null) {
            return;
        }

        // *** pre-order position ***
        // what each node needs to do is to swap its left and right children
        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;

        // traversal framework, go traverse the nodes of the left and right subtrees
        traverse(root.left);
        traverse(root.right);
    }
}
```

<visual slug='mydata-invert-tree'/>

You can move the swap code from pre-order to post-order and it will still work. But moving it to in-order requires some changes. This should be easy to see, so I won't explain more.

So, we have solved the problem using traversal. But let's also think about another way.

**2. Can we solve this problem with a divide-and-conquer approach?**

Let's define what the `invertTree` function does:

```java
// Definition: Invert the binary tree rooted at 'root', return the root node of the inverted tree
TreeNode invertTree(TreeNode root);
```

Now, for a given node `x`, what can we do with `invertTree(x)`?

We can call `invertTree(x.left)` to invert the left subtree, and `invertTree(x.right)` to invert the right subtree. Then, we swap the left and right children of `x`. This completes the inversion of the subtree rooted at `x`.

Here is the code:

```java
class Solution {
    // Definition: Invert the binary tree rooted at 'root', return the root node of the inverted tree
    public TreeNode invertTree(TreeNode root) {
        if (root == null) {
            return null;
        }
        // Utilize the function definition to invert left and right subtrees first
        TreeNode left = invertTree(root.left);
        TreeNode right = invertTree(root.right);

        // Then swap the left and right child nodes
        root.left = right;
        root.right = left;

        // Consistent with the definition logic: the binary tree rooted at 'root' has been inverted, return root
        return root;
    }
}
```

<visual slug='mydata-invert-tree2'/>

In this divide-and-conquer way, the key is to give the recursive function a clear definition, and then write the code based on this definition. If your logic is self-consistent, your algorithm should be correct.

That's all for this problem. Both traversal and divide-and-conquer work. Let's look at the next problem.


## Problem 2: Populating Next Right Pointers in Each Node

This is LeetCode problem 116: [Populating Next Right Pointers in Each Node](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/). Let's look at the problem:

**LeetCode 116. Populating Next Right Pointers in Each Node** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given a **perfect binary tree** where all leaves are on the same level, and every parent has two children. The binary tree has the following definition:

```

struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}

```

Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to `NULL`.

Initially, all next pointers are set to `NULL`.

Example 1:**

![](https://assets.leetcode.com/uploads/2019/02/14/116_sample.png)

```

**Input:** root = [1,2,3,4,5,6,7]
**Output:** [1,#,2,3,#,4,5,6,7,#]
**Explanation: **Given the above perfect binary tree (Figure A), your function should populate each next pointer to point to its next right node, just like in Figure B. The serialized output is in level order as connected by the next pointers, with '#' signifying the end of each level.

```

Example 2:**

```

**Input:** root = []
**Output:** []

```

**Constraints:**

	
- The number of nodes in the tree is in the range `[0, 2^(12) - 1]`.
	
- `-1000 <= Node.val <= 1000`

**Follow-up:**

	
- You may only use constant extra space.
	
- The recursive approach is fine. You may assume implicit stack space does not count as extra space for this problem.

```java
// Function signature
Node connect(Node root);
```

The goal is to connect all nodes at the same level in a binary tree using the `next` pointer:

![](../pictures/binary-tree-i/1.png)

The problem also says the input is a "perfect binary tree," which means the tree is shaped like an equilateral triangle. Except for the rightmost node at each level, every node has a neighbor to its right, and the `next` pointer should point to that neighbor. The rightmost node's `next` pointer should point to `null`.

How do we solve this problem? Let's recall the general ideas for solving binary tree problems:

**1. Can we solve this problem using a "traversal" approach?**

Of course, we can.

Each node just needs to set its `next` pointer to the node on its right.

You might want to write code similar to the previous problem, like this:

```java
// Binary tree traversal function
void traverse(Node root) {
    if (root == null || root.left == null) {
        return;
    }
    // Point the next pointer of the left child to the right child
    root.left.next = root.right;

    traverse(root.left);
    traverse(root.right);
}
```

But this code has a big problem. It only connects two nodes with the same parent. Look at this picture again:

![](../pictures/binary-tree-i/1.png)

Node 5 and node 6 don't have the same parent. According to this code, they won't be connected, which is not what we want. So where is the problem?

**The traditional `traverse` function visits every node in the binary tree, but now we want to visit the "gap" between two neighboring nodes.**

So, let's try to see the tree differently. Think of each box in the picture as a node:

![](../pictures/binary-tree-i/3.png)

**Now, the binary tree becomes like a ternary tree. Each node in the ternary tree represents two neighboring nodes from the binary tree.**

We just need to write a `traverse` function for this ternary tree. Each "ternary tree node" connects its two binary tree nodes:

```java
class Solution {
    // Main function
    public Node connect(Node root) {
        if (root == null) return null;
        // Traverse the "ternary tree" and connect adjacent nodes
        traverse(root.left, root.right);
        return root;
    }

    // Ternary tree traversal framework
    void traverse(Node node1, Node node2) {
        if (node1 == null || node2 == null) {
            return;
        }
        // *** Pre-order position ***
        // Connect the two input nodes
        node1.next = node2;
        
        // Connect two child nodes with the same parent
        traverse(node1.left, node1.right);
        traverse(node2.left, node2.right);
        // Connect two child nodes across parent nodes
        traverse(node1.right, node2.left);
    }
}
```

With this, the `traverse` function will visit all the "gaps" between neighboring binary tree nodes and connect them correctly. This solves the problem perfectly.

**2. Can we solve this problem by "breaking it into subproblems"?**

Hmm, there doesn't seem to be a good way to do this. So, we can't use the "decompose subproblems" idea for this problem.


## Problem 3: Flatten Binary Tree to Linked List

This is LeetCode problem 114 "[Flatten Binary Tree to Linked List](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/)". Let's look at the problem:

**LeetCode 114. Flatten Binary Tree to Linked List** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given the `root` of a binary tree, flatten the tree into a "linked list":

	
- The "linked list" should use the same `TreeNode` class where the `right` child pointer points to the next node in the list and the `left` child pointer is always `null`.
	
- The "linked list" should be in the same order as a [**pre-order**** traversal**](https://en.wikipedia.org/wiki/Tree_traversal#Pre-order,_NLR) of the binary tree.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/01/14/flaten.jpg)

```

**Input:** root = [1,2,5,3,4,null,6]
**Output:** [1,null,2,null,3,null,4,null,5,null,6]

```

Example 2:**

```

**Input:** root = []
**Output:** []

```

Example 3:**

```

**Input:** root = [0]
**Output:** [0]

```

**Constraints:**

	
- The number of nodes in the tree is in the range `[0, 2000]`.
	
- `-100 <= Node.val <= 100`

**Follow up:** Can you flatten the tree in-place (with `O(1)` extra space)?

```java
// The function signature is as follows
void flatten(TreeNode root);
```

**1. Can we solve this using the "traversal" approach?**

At first glance, it seems possible: perform a preorder traversal of the entire tree, building a "linked list" as you traverse:

```java
// Virtual head node, dummy.right is the result
TreeNode dummy = new TreeNode(-1);
// Pointer used to build the linked list
TreeNode p = dummy;

void traverse(TreeNode root) {
    if (root == null) {
        return;
    }
    // Pre-order position
    p.right = new TreeNode(root.val);
    p = p.right;

    traverse(root.left);
    traverse(root.right);
}
```

But notice the signature of the `flatten` function—its return type is `void`. This means the problem wants us to flatten the binary tree into a linked list in place.

This makes it impossible to solve the problem with simple binary tree traversal.

**2. Can we solve this using the "decompose the problem" approach?**

Let's try to define the `flatten` function:

```java
// Definition: Input node root, then the binary tree rooted at root will be flattened into a linked list
void flatten(TreeNode root);
```

With this function definition, how do we flatten a tree into a linked list as required?

For a node `x`, you can follow this process:

1. First use `flatten(x.left)` and `flatten(x.right)` to flatten `x`'s left and right subtrees.

2. Set the entire left subtree as the right subtree, then attach the original right subtree to the end of the current right subtree.

![](../pictures/binary-tree-i/2.jpeg)

This way, the entire binary tree rooted at `x` is flattened, which is exactly what the `flatten(x)` definition requires.

Here's the code implementation:

```java
class Solution {
    // Definition: flatten the tree with root as the root node into a linked list
    public void flatten(TreeNode root) {
        // base case
        if (root == null) return;
        
        // Utilize the definition to flatten the left and right subtrees
        flatten(root.left);
        flatten(root.right);

        // *** Post-order traversal position ***
        // 1. Left and right subtrees have been flattened into linked lists
        TreeNode left = root.left;
        TreeNode right = root.right;
        
        // 2. Make the left subtree the new right subtree
        root.left = null;
        root.right = left;

        // 3. Attach the original right subtree to the end of the current right subtree
        TreeNode p = root;
        while (p.right != null) {
            p = p.right;
        }
        p.right = right;
    }
}
```

<visual slug='flatten-binary-tree-to-linked-list' />

See, this is the magic of recursion. Can you explain exactly how the `flatten` function flattens the left and right subtrees?

It's not easy to articulate, but as long as you know the definition of `flatten` and use that definition to let each node do what it's supposed to do, the `flatten` function will work according to its definition.

With that, this problem is solved. The recursive approach in our previous article [Reverse Nodes in k-Group](https://labuladong.online/en/algo/data-structure/reverse-linked-list-recursion/) shares some similarities with this problem.

Finally, let's revisit the binary tree problem-solving framework to bring things full circle.

There are two approaches for solving binary tree problems:

**1. Can you get the answer by traversing the binary tree once?** If yes, use a `traverse` function with external variables. This is the "traversal" approach.

**2. Can you define a recursive function that derives the answer to the original problem from the answers of subproblems (subtrees)?** If yes, write out the definition of this recursive function and fully utilize its return value. This is the "decompose the problem" approach.

Regardless of which approach you use, you need to think about:

**If you isolate a single binary tree node, what does it need to do? When (pre/in/post-order position) does it need to do it?** Don't worry about the other nodes—the recursive function will execute the same operation on all nodes.

I hope you can internalize this and apply it to all binary tree problems.

That's all for this article. For more classic binary tree exercises and recursion training, see [Recursion Practice](https://labuladong.online/en/algo/intro/binary-tree-practice/) in the binary tree chapter.
