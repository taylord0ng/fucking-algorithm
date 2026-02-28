::: info Prerequisites

Before reading this article, you should first learn:

- [Binary Tree Structure Basics](https://labuladong.online/en/algo/data-structure-basic/binary-tree-basic/)
- [Binary Tree DFS/BFS Traversal](https://labuladong.online/en/algo/data-structure-basic/binary-tree-traverse-basic/)

:::

::: important How to Read This Article

This article abstracts and summarizes many algorithms, so it contains numerous links to other articles.

**If you're reading this for the first time and encounter unfamiliar algorithms or concepts, skip them. Just get a general impression of the theories summarized here**. As you learn the algorithm techniques in later chapters, you'll gradually understand the essence of this article. When you revisit it later, you'll gain deeper insights.

:::

All articles on this site follow the framework proposed in [Framework Thinking for Learning Data Structures and Algorithms](https://labuladong.online/en/algo/essential-technique/algorithm-summary/), which emphasizes the importance of binary tree problems. That's why this article is placed in the essential reading section of the first chapter.

After years of solving problems, I've distilled the core principles of binary tree algorithms here. The terminology might not be textbook-perfect, and no textbook would include these experience-based insights, but no binary tree problem on any coding platform can escape the framework outlined in this article. If you find a problem that doesn't fit this framework, please let me know in the comments.

Let me summarize upfront. Binary tree problem-solving falls into two mental models:

**1. Can you get the answer by traversing the binary tree once?** If yes, use a `traverse` function with external variables. This is the "traversal" mindset.

**2. Can you define a recursive function that derives the answer to the original problem from the answers of subproblems (subtrees)?** If yes, write out this recursive function's definition and fully utilize its return value. This is the "decompose the problem" mindset.

Regardless of which mindset you use, you need to think:

**If you isolate a single binary tree node, what does it need to do? When should it do this (preorder/inorder/postorder position)?** Don't worry about the other nodes—the recursive function will execute the same operation on all nodes for you.

I'll use problems as examples in this article, but they're all very simple ones. Don't worry about not understanding them. I'll help you extract the common patterns from the simplest problems, elevate the thinking embedded in binary trees, and apply it to [Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), [Backtracking](https://labuladong.online/en/algo/essential-technique/backtrack-framework/), [Divide and Conquer](https://labuladong.online/en/algo/essential-technique/divide-and-conquer/), and [Graph Algorithms](https://labuladong.online/en/algo/data-structure-basic/graph-basic/). This is why I keep emphasizing framework thinking. I hope after you learn these advanced algorithms, you'll come back to reread this article—you'll understand it much more deeply.

First, let me once again stress the importance of binary trees as a data structure and its related algorithms.

## The Importance of Binary Trees

For example, consider two classic sorting algorithms: [Quick Sort](https://labuladong.online/en/algo/practice-in-action/quick-sort/) and [Merge Sort](https://labuladong.online/en/algo/practice-in-action/merge-sort/). What's your understanding of them?

**If you tell me that quick sort is essentially a preorder traversal of a binary tree, and merge sort is essentially a postorder traversal, then I know you're an algorithm expert**.

Why can quick sort and merge sort be related to binary trees? Let's briefly analyze their algorithmic ideas and code frameworks:

The logic of quick sort is: to sort `nums[lo..hi]`, first find a partition point `p`. By swapping elements, make all elements in `nums[lo..p-1]` less than or equal to `nums[p]`, and all elements in `nums[p+1..hi]` greater than `nums[p]`. Then recursively find new partition points in `nums[lo..p-1]` and `nums[p+1..hi]`. Eventually, the entire array is sorted.

The code framework for quick sort:

```java
void sort(int[] nums, int lo, int hi) {
    if (lo >= hi) {
        return;
    }
    // ****** pre-order position ******
    // partition the nums[lo..hi], put nums[p] in the right position
    // such that nums[lo..p-1] <= nums[p] < nums[p+1..hi]
    int p = partition(nums, lo, hi);

    // recursively partition the left and right subarrays
    sort(nums, lo, p - 1);
    sort(nums, p + 1, hi);
}
```

First construct the partition point, then go to the left and right subarrays to construct partition points. Isn't this just a preorder traversal of a binary tree?

Now let's talk about merge sort's logic. To sort `nums[lo..hi]`, first sort `nums[lo..mid]`, then sort `nums[mid+1..hi]`, and finally merge these two sorted subarrays. The entire array is then sorted.

The code framework for merge sort:

```java
// definition: sort nums[lo..hi]
void sort(int[] nums, int lo, int hi) {
    if (lo == hi) {
        return;
    }
    int mid = (lo + hi) / 2;
    // use the definition to sort nums[lo..mid]
    sort(nums, lo, mid);
    // use the definition to sort nums[mid+1..hi]
    sort(nums, mid + 1, hi);

    // ****** Post-order position ******
    // at this point, the two subarrays are already sorted
    // merge the two sorted arrays to make nums[lo..hi] sorted
    merge(nums, lo, mid, hi);
}
```

First sort the left and right subarrays, then merge them (similar to merging sorted linked lists). Isn't this the postorder traversal framework of a binary tree? Also, this is the legendary divide and conquer algorithm—nothing more than this.

If you can see through these sorting algorithms at a glance, do you still need to memorize them? No. You can derive them effortlessly from the binary tree traversal framework.

The point of all this is: binary tree algorithmic thinking has wide applications. You could even say that anything involving recursion can be abstracted as a binary tree problem.

Next, let's start from preorder, inorder, and postorder traversal of binary trees, so you can deeply understand the elegance of this data structure.


## Understand Preorder, Inorder, and Postorder Traversal

Let me give you a few questions. Think about them for 30 seconds:

1. What do you think preorder, inorder, and postorder traversals of a binary tree are? Are they just three lists in different orders?

2. Can you explain what is special about postorder traversal?

3. Why does an n-ary tree (a tree with more than two children per node) not have an inorder traversal?

If you cannot answer these, it means you only know the textbook understanding of these traversals. Don't worry, I will use simple comparisons to explain how I see these traversals.

First, let's review the binary tree recursive traversal framework mentioned in [Binary Tree DFS/BFS Traversal](https://labuladong.online/en/algo/data-structure-basic/binary-tree-traverse-basic/):

```java
// Binary tree traversal framework
void traverse(TreeNode root) {
    if (root == null) {
        return;
    }
    // pre-order position
    traverse(root.left);
    // in-order position
    traverse(root.right);
    // post-order position
}
```

Ignore the terms preorder, inorder, and postorder for now. Look at the `traverse` function. What is it really doing?

It is simply a function that visits every node in the binary tree. This is just like traversing an array or a linked list.

```java
// iterate through the array
void traverse(int[] arr) {
    for (int i = 0; i < arr.length; i++) {

    }
}

// recursively traverse the array
void traverse(int[] arr, int i) {
    if (i == arr.length) {
        return;
    }
    // pre-order position
    traverse(arr, i + 1);
    // post-order position
}

// iterate through the singly linked list
void traverse(ListNode head) {
    for (ListNode p = head; p != null; p = p.next) {

    }
}

// recursively traverse the singly linked list
void traverse(ListNode head) {
    if (head == null) {
        return;
    }
    // pre-order position
    traverse(head.next);
    // post-order position
}
```

You can traverse a singly linked list or an array by iteration or recursion. **A binary tree is just a type of linked structure**, but it cannot be easily rewritten as a for loop. So we usually use recursion to traverse a binary tree.

You may also have noticed that recursive traversals can have a "preorder position" and a "postorder position". Preorder is before the recursion, postorder is after the recursion.

**Preorder position is when you just enter a node. Postorder position is when you are about to leave a node.** If you put code in different positions, it will run at different times:

![](../pictures/binary-tree-summary/1.jpeg)

For example, if you want to **print the values of a singly linked list in reverse order**, how would you do it?

There are many ways, but if you understand recursion well, you can use the postorder position:

```java
// recursively traverse a singly linked list and print the elements in reverse order
void traverse(ListNode head) {
    if (head == null) {
        return;
    }
    traverse(head.next);
    // post-order position
    print(head.val);
}
```

With the picture above, you can see why this code prints the linked list in reverse order. It uses the call stack from recursion to print from the end to the start.

The same idea applies to binary trees, but there is also an "inorder position".

Textbooks often ask you to list the results of preorder, inorder, and postorder traversals. So, many people think these are just three lists in different orders.

But I want to say, **preorder, inorder, and postorder are actually three special moments when you process each node during tree traversal**, not just three lists.

- Preorder position: code runs when you just enter a binary tree node.
- Postorder position: code runs when you are about to leave a binary tree node.
- Inorder position: code runs when you have finished visiting the left subtree and are about to visit the right subtree of a node.

Notice that I always say "preorder/inorder/postorder position" instead of "traversal". This is to show the difference: you can put code at the preorder position to add elements to a list, and you will get the preorder traversal result. But you can also write more complex code to do other things.

Here is a picture of the three positions on a binary tree:

![](../pictures/binary-tree-summary/2.jpeg)

**You can see that every node has its own unique preorder, inorder, and postorder position.** So, preorder/inorder/postorder are three special times to process each node during traversal.

Now you can also understand why an n-ary tree does not have an inorder position. In a binary tree, each node switches from left to right subtree only once, so there is a unique inorder position. But in an n-ary tree, a node can have many children and switch between them many times, so there is no unique inorder position.

All of this is to help you build the right understanding of binary trees. You will see:

**All binary tree problems can be solved by writing smart logic at preorder, inorder, or postorder positions. You only need to think about what to do with each node. The traversal framework and recursion will handle the rest and apply your code to all nodes.**

You can also see in [Graph Algorithm Basics](https://labuladong.online/en/algo/data-structure-basic/graph-basic/) that the binary tree traversal framework can be extended to graphs, and many classic graph algorithms are built on traversal. But that's a topic for another article.


## Two Approaches to Problem Solving

As mentioned in [My Algorithm Learning Insights](https://labuladong.online/en/algo/essential-technique/algorithm-summary/):

**Recursive solutions for binary tree problems fall into two categories: the first is to traverse the tree once to get the answer, and the second is to calculate the answer by breaking down the problem. These two approaches correspond to [Backtracking Algorithm Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/) and [Dynamic Programming Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/) respectively.**

::: tip Tip

Let me explain my function naming convention: When solving binary tree problems with the traversal approach, the function signature is typically `void traverse(...)` with no return value, relying on external variables to compute the result. When using the problem decomposition approach, the function name depends on its specific purpose and usually has a return value that represents the result of subproblems.

Correspondingly, you'll notice that in [Backtracking Algorithm Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/), the function signature is typically `void backtrack(...)` without a return value, while in [Dynamic Programming Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), the function signature is a `dp` function with a return value. This reveals the deep connections between these two approaches and binary trees.

While there's no strict requirement for function naming, I recommend you follow this style. It better highlights the function's purpose and the problem-solving mindset, making it easier for you to understand and apply.

:::

I previously used the maximum depth of a binary tree as an example, focusing on comparing these two approaches with dynamic programming and backtracking algorithms. This article focuses on analyzing how these two approaches solve binary tree problems.

LeetCode problem 104 "[Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)" is about maximum depth. The maximum depth is the number of nodes along the longest path from the root node to the farthest leaf node. For example, given this binary tree, the algorithm should return 3:

![](../pictures/binary-tree-summary/tree.jpg)

How would you approach this problem? Simply traverse the tree once, use an external variable to record the depth of each node, and take the maximum value to get the maximum depth. **This is the traversal approach to computing the answer.**

Here's the solution code:

```java
// Traversal Idea
class Solution {

    // Record the depth of the traversed node
    int depth = 0;

    // Record the maximum depth
    int res = 0;

    public int maxDepth(TreeNode root) {
        traverse(root);
        return res;
    }

    // Traverse the binary tree
    void traverse(TreeNode root) {
        if (root == null) {
            return;
        }

        // Pre-order traversal position (entering a node) increases depth
        depth++;
        // Record the maximum depth during traversal
        if (root.left == null && root.right == null) {
            res = Math.max(res, depth);
        }
        traverse(root.left);
        traverse(root.right);

        // Post-order traversal position (leaving a node) decreases depth
        depth--;
    }
}
```

<visual slug='mydata-maxdepth1' title='Visualization of Traversal Approach' />

This solution should be straightforward, but why do we need to increment `depth` at the preorder position and decrement it at the postorder position?

As mentioned earlier, the preorder position is when you enter a node, and the postorder position is when you leave a node. The `depth` variable tracks the current node's depth in the recursion. Think of `traverse` as a pointer walking through the binary tree, so naturally you need to maintain it this way.

As for updating `res`, you can place it at preorder, inorder, or postorder positions, as long as it happens after entering the node and before leaving it (i.e., after `depth` increments and before it decrements).

Of course, you can easily see that the maximum depth of a binary tree can be derived from the maximum depths of its subtrees. **This is the problem decomposition approach to computing the answer.**

Here's the solution code:

```java
// Divide and Conquer Idea
class Solution {
    // Definition: Given a node, return the maximum depth of the binary tree rooted at that node
    public int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        // Use the definition to calculate the maximum depth of the left and right subtrees
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);

        // Derive the maximum depth of the original binary tree based on the maximum depth of the left and right subtrees
        // The maximum depth of the entire tree is the maximum of the left and right subtree depths,
        // plus one for the root node itself
        return 1 + Math.max(leftMax, rightMax);
    }
}
```

<visual slug='mydata-maxdepth2' title='Visualization of Problem Decomposition Approach' />

This solution isn't hard to understand once you clarify the recursive function's definition, but why is the main logic concentrated at the postorder position?

Because the key to this approach is that you can indeed derive the tree's depth from the maximum depths of its subtrees. So naturally, you first use the recursive function definition to calculate the maximum depths of the left and right subtrees, then derive the original tree's maximum depth. The main logic naturally goes at the postorder position.

If you understand the two approaches to the maximum depth problem, **let's revisit the most basic binary tree traversals: preorder, inorder, and postorder**. For example, LeetCode problem 144 "[Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)" asks you to compute the preorder traversal result.

The familiar solution uses the "traversal" approach, which I think needs little explanation:

```java
// Calculate the preorder traversal result using the traversal idea
class Solution {
    // store the result of the preorder traversal
    List<Integer> res = new LinkedList<>();

    public List<Integer> preorderTraversal(TreeNode root) {
        traverse(root);
        return res;
    }

    // binary tree traversal function
    void traverse(TreeNode root) {
        if (root == null) {
            return;
        }
        // preorder position
        res.add(root.val);
        traverse(root.left);
        traverse(root.right);
    }
}
```

But can you use the "problem decomposition" approach to compute the preorder traversal result?

In other words, without using helper functions like `traverse` or any external variables, can you solve it purely with the given `preorderTraverse` function recursively?

We know that preorder traversal has this characteristic: the root node's value comes first, followed by the left subtree's preorder traversal result, and finally the right subtree's preorder traversal result:

![](../pictures/binary-tree-summary/3.jpeg)

So this can be decomposed, right? **A binary tree's preorder traversal result = root node + left subtree's preorder traversal result + right subtree's preorder traversal result**.

Therefore, you can implement the preorder traversal algorithm like this:

```java
class Solution {
    // Definition: Given the root node of a binary tree, return the preorder traversal result of this tree
    List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new LinkedList<>();
        if (root == null) {
            return res;
        }
        // The result of preorder traversal, root.val is first
        res.add(root.val);
        // Using the function definition, followed by the preorder traversal result of the left subtree
        res.addAll(preorderTraversal(root.left));
        // Using the function definition, finally followed by the preorder traversal result of the right subtree
        res.addAll(preorderTraversal(root.right));
        /**<extend up -200>
        ![](../pictures/binary-tree-summary/3.jpeg)
        */
        return res;
    }
}
```

Inorder and postorder traversals are similar—just place `add(root.val)` at the corresponding inorder and postorder positions.

This solution is short and elegant, but why isn't it common?

One reason is that **the complexity of this algorithm is hard to control** and depends heavily on language features.

In Java, whether using ArrayList or LinkedList, the `addAll` method has O(N) complexity, so the overall worst-case time complexity reaches O(N^2), unless you implement your own O(1) `addAll` method. This is achievable with an underlying linked list implementation, since multiple linked lists can be connected with simple pointer operations.

Of course, the main reason is that textbooks never teach it this way...

The above examples are simple, but many binary tree problems can be approached and solved with both mindsets. This requires you to practice and think more on your own—don't just settle for one familiar solution approach.

In summary, here's the general thinking process when encountering a binary tree problem:

**1. Can you get the answer by traversing the tree once?** If so, use a `traverse` function with external variables to implement it.

**2. Can you define a recursive function that derives the original problem's answer from subproblem (subtree) answers?** If so, write out this recursive function's definition and fully utilize its return value.

**3. Regardless of which approach you use, you need to understand what each node in the binary tree needs to do and when (preorder, inorder, or postorder) to do it**.

The [Binary Tree Recursion Practice](https://labuladong.online/en/algo/intro/binary-tree-practice/) on this site lists over 100 binary tree problems, using the two approaches above to guide you step by step through practice, helping you fully master recursive thinking and more easily understand advanced algorithms.


## The Special Properties of Postorder Position

Before discussing postorder position, let's briefly talk about preorder and inorder.

Preorder position itself doesn't actually have any special properties. The reason you might notice that many problems seem to write code in preorder position is actually because we habitually write code that isn't sensitive to preorder, inorder, or postorder positions in the preorder position.

Inorder position is mainly used in BST scenarios. You can completely think of BST inorder traversal as traversing a sorted array.

::: important Key Point

**Observe carefully: the capabilities of code in preorder, inorder, and postorder positions increase progressively.**

Code in preorder position can only get data passed from the parent node through function parameters.

Code in inorder position can not only get parameter data, but also get data passed back from the left subtree through the function's return value.

Code in postorder position is the most powerful. It can not only get parameter data, but also simultaneously get data passed back from both left and right subtrees through function return values.

Therefore, in certain situations, moving code to postorder position is most efficient; some things can only be done by code in postorder position.

:::

Let's look at some concrete examples to feel the difference in their capabilities. Now I'll give you a binary tree and ask you two simple questions:

1. If you consider the root node as level 1, how do you print the level number each node is at?

2. How do you print how many nodes each node's left and right subtrees have?

For the first question, you can write code like this:

```java
// Binary tree traversal function
void traverse(TreeNode root, int level) {
    if (root == null) {
        return;
    }
    // Preorder position
    printf("Node %s is at level %d", root.val, level);
    traverse(root.left, level + 1);
    traverse(root.right, level + 1);
}

// Call it like this
traverse(root, 1);
```

For the second question, you can write code like this:

```java
// Definition: given a binary tree, return the total number of nodes in this tree
int count(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int leftCount = count(root.left);
    int rightCount = count(root.right);
    // Postorder position
    printf("Node %s has %d nodes in left subtree and %d nodes in right subtree",
            root, leftCount, rightCount);

    return leftCount + rightCount + 1;
}
```

::: info The fundamental difference between these two problems is

What level a node is at can be recorded on the fly as you traverse from the root node, and can be passed down through the recursive function's parameters. But how many nodes are in the entire subtree rooted at a node - you must finish traversing the subtree before you can count it clearly, then get the answer through the recursive function's return value.

Combined with these two simple problems, you can appreciate the characteristics of postorder position. Only postorder position can get subtree information through return values.

**In other words, once you discover that a problem is related to subtrees, you'll likely need to set a reasonable definition and return value for your function, and write code in postorder position.**

:::

Next, let's look at how postorder position plays a role in actual problems. Let's briefly discuss LeetCode problem 543 "[Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)", which asks you to calculate the longest diameter length of a binary tree.

The so-called "diameter" length of a binary tree is the path length between any two nodes. The longest "diameter" doesn't necessarily pass through the root node. For example, in the binary tree below:

![](../pictures/binary-tree-summary/tree1.png)

Its longest diameter is 3, namely the "diameter" formed by paths like `[4,2,1,3]`, `[4,2,1,9]`, or `[5,2,1,3]`.

The key to solving this problem is: **the "diameter" length of each binary tree is the sum of the maximum depths of a node's left and right subtrees**.

Now if you want me to find the longest "diameter" in the entire tree, the straightforward approach is to traverse every node in the entire tree, then calculate each node's "diameter" through the maximum depth of each node's left and right subtrees, and finally find the maximum of all "diameters".

We just implemented the maximum depth algorithm, so the above approach can be written as the following code:

```java
class Solution {
    // record the length of the maximum diameter
    int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        // calculate the diameter for each node and find the maximum diameter
        traverse(root);
        return maxDiameter;
    }

    // traverse the binary tree
    void traverse(TreeNode root) {
        if (root == null) {
            return;
        }
        // calculate the diameter for each node
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        int myDiameter = leftMax + rightMax;
        // update the global maximum diameter
        maxDiameter = Math.max(maxDiameter, myDiameter);
        
        traverse(root.left);
        traverse(root.right);
    }

    // calculate the maximum depth of the binary tree
    int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        return 1 + Math.max(leftMax, rightMax);
    }
}
```

This solution is correct, but the running time is very long. The reason is also obvious: when `traverse` visits each node, it also calls the recursive function `maxDepth`, and `maxDepth` has to traverse all nodes of the subtree, so the worst-case time complexity is O(N^2).

This is exactly the situation we just discussed. **Preorder position cannot get subtree information, so each node can only call the `maxDepth` function to calculate the subtree's depth**.

So how do we optimize? We should put the logic for calculating "diameter" in postorder position. To be precise, it should be placed in the postorder position of `maxDepth`, because the postorder position of `maxDepth` knows the maximum depths of the left and right subtrees.

So, slightly changing the code logic gives us a better solution:

```java
class Solution {
    // record the length of the maximum diameter
    int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        maxDepth(root);
        return maxDiameter;
    }

    int maxDepth(TreeNode root) {
        if (root == null) {
            return 0;
        }
        int leftMax = maxDepth(root.left);
        int rightMax = maxDepth(root.right);
        // post-order position, calculate the maximum diameter by the way
        int myDiameter = leftMax + rightMax;
        maxDiameter = Math.max(maxDiameter, myDiameter);

        return 1 + Math.max(leftMax, rightMax);
    }
}
```

<visual slug='mydata-diameter-of-binary-tree'/>

Now the time complexity is only O(N) from the `maxDepth` function.

At this point, let me echo what was said earlier: when you encounter subtree problems, the first thing to think about is setting a return value for your function, then working on the postorder position.

::: info Info

Exercise: Please think about whether problems that use postorder position employ a "traversal" approach or a "decomposition" approach?

:::

::: details Click to view answer

Problems that utilize postorder position generally use a "decomposition" approach. Because the current node receives and uses information returned from subtrees, this means you've decomposed the original problem into the current node + the subproblems of the left and right subtrees.

:::

Conversely, if you write a solution that involves recursion within recursion like the initial approach, you should probably reflect on whether it can be optimized through postorder traversal.

For more exercises utilizing postorder position, see [Hands-on Guide to Binary Tree Problems (Postorder Edition)](https://labuladong.online/en/algo/data-structure/binary-tree-part3/), [Hands-on Guide to Binary Search Tree Problems (Postorder Edition)](https://labuladong.online/en/algo/data-structure/bst-part4/), and [Solving Problems Using Postorder Position](https://labuladong.online/en/algo/problem-set/binary-tree-post-order-i/).


## Understanding DP/Backtracking/DFS Through the Lens of Trees

Earlier I said that dynamic programming and backtracking are just two different manifestations of binary tree thinking. If you've read this far, you probably agree. But some sharp readers keep asking: your way of thinking really opened my eyes, but it seems like you've never covered DFS?

I actually used DFS in [Solving All Island Problems in One Go](https://labuladong.online/en/algo/frequency-interview/island-dfs-summary/), but I never devoted a standalone article to it. **That's because DFS and backtracking are very similar—they only differ in one subtle detail**.

What's the difference exactly? It boils down to whether "making a choice" and "undoing a choice" happen inside or outside the for loop. DFS puts them outside; backtracking puts them inside.

Why the difference? Again, it all comes back to binary trees. In this section, I'll use one sentence each to explain the connections and differences among backtracking, DFS, and dynamic programming—and how they relate to binary tree algorithms:

::: important Key Takeaway

DP, DFS, and backtracking can all be viewed as extensions of binary tree problems. The difference is what they focus on:

- Dynamic programming uses a decomposition (divide-and-conquer) approach, focusing on entire "subtrees."
- Backtracking uses a traversal approach, focusing on the "edges" between nodes.
- DFS uses a traversal approach, focusing on individual "nodes."

:::

How to understand this? Three examples will make it crystal clear.

### Example 1: The Decomposition Mindset

**First example**: given a binary tree, write a `count` function using the decomposition approach to count the total number of nodes. The code is straightforward—we've already seen it:

```java
// Definition: input a binary tree, return the total number of nodes in this binary tree
int count(TreeNode root) {
    if (root == null) {
        return 0;
    }
    // The current node cares about the total number of nodes in each of its subtrees
    // Because the result of the subproblems can be used to derive the result of the original problem
    int leftCount = count(root.left);
    int rightCount = count(root.right);
    // In the postorder position, the total number of nodes in the left and right subtrees plus the current node itself is the total number of nodes in the entire tree
    return leftCount + rightCount + 1;
}
```

**See? This is the decomposition mindset of dynamic programming. It always focuses on structurally identical subproblems, which correspond to "subtrees" in a binary tree**.

Now look at a concrete DP problem. For instance, the Fibonacci example from [Dynamic Programming Framework Explained](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)—our focus is on the return values of each subtree:

```java
// f(n) calculates the nth Fibonacci number
int fib(int n) {
    // base case
    if (n == 0 || n == 1){
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}
```

![](../pictures/dynamic-programming/2.jpg)


### Example 2: The Backtracking Mindset

**Second example**: given a binary tree, write a `traverse` function using the traversal approach to print out the traversal process. Check out the code:

```java
void traverse(TreeNode root) {
    if (root == null) return;
    printf("enter node %s from node %s", root.left, root);
    traverse(root.left);
    printf("leave node %s, return to node %s", root.left, root);

    printf("enter node %s from node %s", root.right, root);
    traverse(root.right);
    printf("leave node %s, return to node %s", root.right, root);
}
```

Not hard to understand, right? Now let's level up from binary tree to N-ary tree—the code looks similar:


```java
// N-ary tree node
class Node {
    int val;
    Node[] children;
}

void traverse(Node root) {
    if (root == null) return;
    for (Node child : root.children) {
        printf("enter node %s from node %s", child, root);
        traverse(child);
        printf("leave node %s, return to node %s", child, root);
    }
}
```

This N-ary tree traversal framework naturally extends into the backtracking framework from [Backtracking Algorithm Framework Explained](https://labuladong.online/en/algo/essential-technique/backtrack-framework/):

```java
// backtracking framework
void backtrack(...) {
    // base case
    if (...) return;

    for (int i = 0; i < ...; i++) {
        // make a choice
        ...

        // enter the next level of the decision tree
        backtrack(...);

        // undo the choice
        ...
    }
}
```

**See? This is the traversal mindset of backtracking. It always focuses on the process of moving between nodes, which corresponds to "edges" in a binary tree**.

Now look at a concrete backtracking problem. For instance, the permutation problem from [Backtracking: 9 Variations of Permutations and Combinations](https://labuladong.online/en/algo/essential-technique/permutation-combination-subset-all-in-one/)—our focus is on individual tree edges:

```java
// core backtracking code
void backtrack(int[] nums) {
    // backtracking framework
    for (int i = 0; i < nums.length; i++) {
        // make a choice
        used[i] = true;
        track.addLast(nums[i]);

        // enter the next level of the backtracking tree
        backtrack(nums);

        // undo the choice
        track.removeLast();
        used[i] = false;
    }
}
```

![](../pictures/permutation/2.jpeg)

### Example 3: The DFS Mindset

**Third example**: given a binary tree, write a `traverse` function that increments every node's value by one. Simple enough:

```java
void traverse(TreeNode root) {
    if (root == null) return;
    // increment the value of each traversed node by one
    root.val++;
    traverse(root.left);
    traverse(root.right);
}
```

**See? This is the traversal mindset of DFS. It always focuses on individual nodes, which corresponds to processing each "node" in a binary tree**.

Now look at a concrete DFS problem. For instance, the first few problems from [Solving All Island Problems in One Go](https://labuladong.online/en/algo/frequency-interview/island-dfs-summary/)—our focus is on each cell (node) in the `grid` array. We need to process every visited cell, which is why I say we're using DFS to solve these problems:

```java
// core DFS logic
void dfs(int[][] grid, int i, int j) {
    int m = grid.length, n = grid[0].length;
    if (i < 0 || j < 0 || i >= m || j >= n) {
        return;
    }
    if (grid[i][j] == 0) {
        return;
    }
    // mark each visited cell as 0
    grid[i][j] = 0;
    dfs(grid, i + 1, j);
    dfs(grid, i, j + 1);
    dfs(grid, i - 1, j);
    dfs(grid, i, j - 1);
}
```

![](../pictures/island/5.jpg)

Take a moment to digest these three examples. See the pattern? Dynamic programming focuses on entire "subtrees," backtracking focuses on "edges" between nodes, and DFS focuses on individual "nodes."

With this foundation, it's easy to understand why "making a choice" and "undoing a choice" are placed differently in backtracking vs. DFS code. Look at these two snippets:

```java
// DFS algorithm puts the logic of "making a choice" and "undoing a choice" outside the for loop
void dfs(Node root) {
    if (root == null) return;
    // make a choice
    print("enter node %s", root);
    for (Node child : root.children) {
        dfs(child);
    }
    // undo a choice
    print("leave node %s", root);
}

// Backtracking algorithm puts the logic of "making a choice" and "undoing a choice" inside the for loop
void backtrack(Node root) {
    if (root == null) return;
    for (Node child : root.children) {
        // make a choice
        print("I'm on the branch from %s to %s", root, child);
        backtrack(child);
        // undo a choice
        print("I'll leave the branch from %s to %s", child, root);
    }
}
```

See the difference? Backtracking must put "make a choice" and "undo the choice" inside the for loop—otherwise, how would you capture both endpoints of an "edge"?

## Level-Order Traversal

Binary tree problems mainly serve to build your recursive thinking. Level-order traversal, on the other hand, is iterative and relatively simple. Let's quickly go over the framework:

```java
// Input the root node of a binary tree, perform level-order traversal on the binary tree
void levelTraverse(TreeNode root) {
    if (root == null) return;
    Queue<TreeNode> q = new LinkedList<>();
    q.offer(root);

    int depth = 1;
    // Traverse each level of the binary tree from top to bottom
    while (!q.isEmpty()) {
        int sz = q.size();
        // Traverse each node of the level from left to right
        for (int i = 0; i < sz; i++) {
            TreeNode cur = q.poll();

            // put the nodes of the next level into the queue
            if (cur.left != null) {
                q.offer(cur.left);
            }
            if (cur.right != null) {
                q.offer(cur.right);
            }
        }
        depth++;
    }
}
```

The while loop handles top-to-bottom traversal, while the for loop handles left-to-right traversal:

![](../pictures/dijkstra/1.jpeg)

The [BFS Algorithm Framework](https://labuladong.online/en/algo/essential-technique/bfs-framework/) is a direct extension of binary tree level-order traversal, commonly used for **shortest path** problems in unweighted graphs.

You can also modify this framework flexibly. When the problem doesn't require tracking levels (steps), you can drop the for loop. For example, [Dijkstra's Algorithm](https://labuladong.online/en/algo/data-structure/dijkstra/) explores how to extend BFS for shortest path problems in weighted graphs.

Worth mentioning: some binary tree problems that obviously call for level-order traversal can also be solved with recursive traversal. The techniques involved are trickier and really test your mastery of preorder, inorder, and postorder positions.

Alright, this article is long enough. We've thoroughly covered all the key patterns in binary tree problems by building everything around preorder, inorder, and postorder positions. How much of this you can actually apply comes down to practice and hands-on problem solving.

Finally, the [Binary Tree Recursion Practice](https://labuladong.online/en/algo/intro/binary-tree-practice/) section will walk you through applying the techniques from this article step by step.


## Answering Questions from the Comments

Let me talk a bit more about level order traversal (and the [BFS algorithm framework](https://labuladong.online/en/algo/essential-technique/bfs-framework/) that comes from it).

If you know enough about binary trees, you might think of many ways to get the level order result using recursion, like the example below:

```java
class Solution {
    List<List<Integer>> res = new ArrayList<>();

    public List<List<Integer>> levelTraverse(TreeNode root) {
        if (root == null) {
            return res;
        }
        // root is considered as level 0
        traverse(root, 0);
        return res;
    }

    void traverse(TreeNode root, int depth) {
        if (root == null) {
            return;
        }
        // pre-order position, check if nodes of depth level are already stored
        if (res.size() <= depth) {
            // first time entering depth level
            res.add(new LinkedList<>());
        }
        // pre-order position, add root node's value at depth level
        res.get(depth).add(root.val);
        traverse(root.left, depth + 1);
        traverse(root.right, depth + 1);
    }
}
```

This method does give you the level order result, but in essence, it is still a preorder traversal of the binary tree, or in other words, it uses DFS thinking, not true level order or BFS thinking. This is because the solution depends on preorder traversal's top-down, left-to-right order to get the right answer.

**To put it simply, this solution is more like a "column order traversal" from left to right, not a "level order traversal" from top to bottom.** So, for problems like finding the minimum distance, this solution works the same as DFS, and does not have the performance advantage of BFS.

Some readers also shared another recursive way to do level order traversal:

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();

    public List<List<Integer>> levelTraverse(TreeNode root) {
        if (root == null) {
            return res;
        }
        List<TreeNode> nodes = new LinkedList<>();
        nodes.add(root);
        traverse(nodes);
        return res;
    }

    void traverse(List<TreeNode> curLevelNodes) {
        // base case
        if (curLevelNodes.isEmpty()) {
            return;
        }
        // pre-order position, calculate the values of the current level and the node list of the next level
        List<Integer> nodeValues = new LinkedList<>();
        List<TreeNode> nextLevelNodes = new LinkedList<>();
        for (TreeNode node : curLevelNodes) {
            nodeValues.add(node.val);
            if (node.left != null) {
                nextLevelNodes.add(node.left);
            }
            if (node.right != null) {
                nextLevelNodes.add(node.right);
            }
        }
        // add results in pre-order position to get top-down level order traversal
        res.add(nodeValues);
        traverse(nextLevelNodes);
        // add results in post-order position to get bottom-up level order traversal
        // res.add(nodeValues);
    }
}
```

The `traverse` function here is a bit like a recursive function for traversing a singly linked list. It treats each level of the binary tree as a node in a linked list.

Compared to the previous recursive solution, this one does a top-down "level order traversal." It is closer to the core idea of BFS. You can use this as a recursive way to implement BFS and expand your thinking.
