Reversing a singly linked list with iteration is not hard, but the recursive solution is a bit tricky. If we add more difficulty and only reverse part of a linked list, can you solve it with both iteration and recursion? Going further, if you need to reverse the list in groups of k, how would you handle that?

This article will go from easy to hard and solve these linked list problems step by step. I will use both recursive and iterative methods, and use visual panels to help you understand. This will strengthen your recursive thinking and your skill in operating linked list pointers.

## Reverse the entire singly linked list

On LeetCode, the common structure of a singly linked list is like this:

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int x) { val = x; }
}
```

Reversing a singly linked list is a basic algorithm problem. LeetCode 206 “[Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)” is exactly this problem:

**LeetCode 206. Reverse Linked List** <span class="inline-block w-2 h-2 rounded-full bg-green-500"></span>

Given the `head` of a singly linked list, reverse the list, and return *the reversed list*.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)

```

**Input:** head = [1,2,3,4,5]
**Output:** [5,4,3,2,1]

```

Example 2:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)

```

**Input:** head = [1,2]
**Output:** [2,1]

```

Example 3:**

```

**Input:** head = []
**Output:** []

```

**Constraints:**

	
- The number of nodes in the list is the range `[0, 5000]`.
	
- `-5000 <= Node.val <= 5000`

**Follow up:** A linked list can be reversed either iteratively or recursively. Could you implement both?

Now let’s try several methods to solve this problem.

### Iterative solution

The standard way to solve this problem is the iterative solution. We operate several pointers to reverse the direction of each node’s `next` pointer. There is no big difficulty, the key is to handle pointer details correctly.

Here is the code. With the comments and the visual panel, it should be easy to understand:

```java
class Solution {
    // Reverse the linked list starting from head
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        // Due to the structure of a singly linked list, at least three pointers are needed to complete the iterative reversal
        // cur is the current node being traversed, pre is the predecessor node of cur, nxt is the successor node of cur
        ListNode pre, cur, nxt;
        pre = null; cur = head; nxt = head.next;
        while (cur != null) {
            // Reverse each node
            cur.next = pre;
            // Update pointer positions
            pre = cur;
            cur = nxt;
            if (nxt != null) {
                nxt = nxt.next;
            }
        }
        // Return the head node after reversal
        return pre;
    }
}
```

<visual slug="reverse-linked-list-iter" >

You can open the visual panel below, and click the line <code type="click">cur.next = pre</code> many times. Then you can clearly see the reversing process of the singly linked list:

</visual>

::: tip Small tips for pointer operations

The logic of the code above is not complicated, and there is more than one correct way to write it. But when working with pointers, there are some very basic and simple tips that can make your thinking clearer:

1. Once you see an operation like `nxt.next`, you should immediately think: first check whether `nxt` is null, otherwise you may get a null pointer exception.

2. Pay attention to the loop end condition. You must know where each pointer is when the loop ends. Then you can return the correct answer. If you feel it is complicated, draw the simplest case and run the algorithm by hand. For example, in this problem you can draw a list with only two nodes `1->2`. Then you can figure out exactly where each pointer is when the loop finishes.

:::


### Recursive Solution

The iterative solution above is a bit tedious because of pointer operations, but the idea is still clear. Now, what if you are asked to reverse a singly linked list using recursion? Do you have any ideas?

For beginners, it might be hard to think of a recursive way. That is normal. If you learn the way of thinking in binary tree algorithms later, you may be able to come up with this algorithm yourself.

A binary tree is actually an extension of a singly linked list—it's like a "binary linked list". So the recursive thinking for binary trees also works for linked lists.

**The key to reversing a linked list recursively is that this problem has a subproblem structure.**

For example, suppose you have a singly linked list `1->2->3->4` with `1` as the head. If you ignore the head node `1` and just look at the sublist `2->3->4`, it is still a singly linked list, right?

So, your `reverseList` function should be able to reverse any linked list given as input. Can you use this function to reverse the sublist `2->3->4` first, and then think about how to attach `1` to the end of the reversed list `4->3->2`? This way, you will finish reversing the whole list.

```java
reverseList(1->2->3->4) = reverseList(2->3->4) -> 1
```

**This is the idea of "breaking down the problem". Using the definition of the recursive function, we break the original problem into smaller, similar subproblems, and then combine their results to solve the original problem.**

There will be special chapters and exercises about this idea later, so we won’t go into more detail here.

Let's look at the code for recursively reversing a singly linked list:

```java
class Solution {
    // Definition: Input the head node of a singly linked list, reverse the list, and return the new head node
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode last = reverseList(head.next);
        /**<extend up -200>
        ![](../pictures/reverse-linked-list/3.jpg)
        */
        head.next.next = head;
        /**<extend up -200>
        ![](../pictures/reverse-linked-list/4.jpg)
        */
        head.next = null;
        /**<extend up -200>
        ![](../pictures/reverse-linked-list/5.jpg)
        */
        return last;
    }
}
```

This algorithm often shows the beauty and cleverness of recursion. Next, let's explain this code in detail. We will also provide a visual panel so you can explore the recursion yourself.

For recursive algorithms that "break down the problem", the most important thing is to be clear about the definition of the recursive function. Specifically, our `reverseList` function is defined as:

**Given a node `head`, reverse the list starting from `head` and return the new head of the reversed list.**

Once you understand the function definition, let's look at the problem. For example, we want to reverse this list:

![](../pictures/reverse-linked-list/1.jpg)

When you call `reverseList(head)`, recursion happens here:

```java
ListNode last = reverseList(head.next);
```

Don’t get lost in recursion (your brain can only hold so many stacks!). Instead, use the function definition to understand what this code does:

![](../pictures/reverse-linked-list/2.jpg)

After `reverseList(head.next)` finishes, the whole list becomes:

![](../pictures/reverse-linked-list/3.jpg)

And according to the function definition, the `reverseList` function returns the new head of the reversed list, which we store in the variable `last`.

Now, look at the next line:

```java
head.next.next = head;
```

![](../pictures/reverse-linked-list/4.jpg)

Next:

```java
head.next = null;
return last;
```

![](../pictures/reverse-linked-list/5.jpg)

Amazing! Now the whole linked list is reversed. Recursive code is simple and elegant, but there are two things to pay attention to:

1. The recursive function needs a base case, which is this line:

```java
if (head == null || head.next == null) {
    return head;
}
```

This means if the list is empty or only has one node, the reversed result is itself, so just return it.

2. After the list is reversed recursively, the new head is `last`, and the old `head` becomes the last node. Don't forget to set its `next` to null:

```java
head.next = null;
```

This way, the whole linked list is reversed. Isn’t it amazing? Here is a visual process of recursive reversal:

<visual slug="reverse-linked-list" />

::: note Do not get lost in recursion details

The visual panel can show all the steps of the recursion, but I do not suggest beginners focus too much on the details. First, use the way of thinking explained above to understand recursion, then use the visual panel to deepen your understanding.

:::

::: note Recursion is less efficient than iteration for linked lists

It is worth mentioning that recursion is not very efficient for linked lists.

The recursive and iterative solutions both have time complexity O(N), but the iterative one uses O(1) space, while the recursive one needs stack space, so its space complexity is O(N).

So, recursion is good for practicing thinking, but for efficiency, iteration is better.

:::

## Reverse the First N Nodes of a Linked List

Now let’s implement a function like this:

```java
// Reverse the first n nodes of the linked list (n <= length of the list)
ListNode reverseN(ListNode head, int n)
```

For example, for the linked list below, if you run `reverseN(head, 3)`:

![](../pictures/reverse-linked-list/6.jpg)


<!-- hide -->

### Iterative Solution

The iterative solution is easy to write. You can modify the `reverseList` function we wrote before:

```java
ListNode reverseN(ListNode head, int n) {
    if (head == null || head.next == null) {
        return head;
    }
    ListNode pre, cur, nxt;
    pre = null; cur = head; nxt = head.next;
    while (n > 0) {
        cur.next = pre;
        pre = cur;
        cur = nxt;
        if (nxt != null) {
            nxt = nxt.next;
        }
        n--;
    }
    // At this point, cur is the (n+1)th node, and head is the tail node after reversal
    head.next = cur;
    /**<extend up>
    ![](../pictures/reverse-linked-list/6.jpg)
    */
    // At this point, pre is the head node after reversal
    return pre;
}
```

<visual slug="reverse-n-iter" />

### Recursive Solution

The recursive idea is similar to reversing the whole linked list. You just need a small change:

```java
// Successor node
ListNode successor = null;

// Reverse n nodes starting from head, return the new head node
ListNode reverseN(ListNode head, int n) {
    if (n == 1) {
        // Record the (n+1)th node
        successor = head.next;
        return head;
    }
    // Starting from head.next, need to reverse the first n-1 nodes
    ListNode last = reverseN(head.next, n - 1);

    head.next.next = head;
    // Connect the reversed head node with the nodes after it
    head.next = successor;
    return last;
    /**<extend up -90>
    ![](../pictures/reverse-linked-list/7.jpg)
    */
}
```

Main differences:

1. The base case becomes `n == 1`. Reversing one element means it stays the same. **You also need to record the successor node**, which is the `n + 1` node.

2. Before, we set `head.next` to null because after reversing the whole list, the original `head` becomes the last node. Now, after recursion, `head` may not be the last node. So you need to record the successor (`n + 1` node) and connect `head` to it after reversing.

![](../pictures/reverse-linked-list/7.jpg)

<visual slug="list-reverse-n" />

## Reverse a Part of a Linked List

We can go further. Given a range of indices, reverse that part in the linked list and keep other parts unchanged.

LeetCode 92 "[Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)" is this problem:

**LeetCode 92. Reverse Linked List II** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given the `head` of a singly linked list and two integers `left` and `right` where `left <= right`, reverse the nodes of the list from position `left` to position `right`, and return *the reversed list*.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/02/19/rev2ex2.jpg)

```

**Input:** head = [1,2,3,4,5], left = 2, right = 4
**Output:** [1,4,3,2,5]

```

Example 2:**

```

**Input:** head = [5], left = 1, right = 1
**Output:** [5]

```

**Constraints:**

	
- The number of nodes in the list is `n`.
	
- `1 <= n <= 500`
	
- `-500 <= Node.val <= 500`
	
- `1 <= left <= right <= n`

**Follow up:** Could you do it in one pass?

The function gets index range `[m, n]` (1-based). You only reverse the elements in this range. The function signature:

```java
ListNode reverseBetween(ListNode head, int m, int n)
```

### Iterative Solution

The iterative idea is simple. First, find the `m - 1` node, then use the `reverseN` function we wrote before:

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        if (m == 1) {
            return reverseN(head, n);
        }
        // find the predecessor of the m-th node
        ListNode pre = head;
        for (int i = 1; i < m - 1; i++) {
            pre = pre.next;
        }
        // start reversing from the m-th node
        pre.next = reverseN(pre.next, n - m + 1);
        return head;
    }

    ListNode reverseN(ListNode head, int n) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode pre, cur, nxt;
        pre = null; cur = head; nxt = head.next;
        while (n > 0) {
            cur.next = pre;
            pre = cur;
            cur = nxt;
            if (nxt != null) {
                nxt = nxt.next;
            }
            n--;
        }
        // at this point, cur is the (n+1)-th node, head is the tail node after reversal
        head.next = cur;
        /**<extend up>
        ![](../pictures/reverse-linked-list/6.jpg)
        */
        // at this point, pre is the head node after reversal
        return pre;
    }
}
```

<visual slug="reverse-linked-list-ii-iter" />

### Recursive Solution

For the recursive solution, also find the `m - 1` node and use the `reverseN` function.

The key is: how to find the `m - 1` node with recursion?

If we treat `head` as index 1, we want to reverse from the `m`th node. If we treat `head.next` as index 1, then we should reverse from the `m - 1`th node. For `head.next.next`, it’s from the `m - 2`th node, and so on...

This is using recursion to move forward. The code can be written like this:

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int m, int n) {
        // base case
        if (m == 1) {
            return reverseN(head, n);
        }
        // advance to the starting point of reversal to trigger the base case
        head.next = reverseBetween(head.next, m - 1, n - 1);
        return head;
    }

    // successor node
    ListNode successor = null;

    // reverse n nodes starting from head, return the new head node
    ListNode reverseN(ListNode head, int n) {
        if (n == 1) {
            // record the (n+1)th node
            successor = head.next;
            return head;
        }
        ListNode last = reverseN(head.next, n - 1);

        head.next.next = head;
        head.next = successor;
        return last;
    }
}
```

<visual slug='reverse-linked-list-ii'/>


## Reverse Nodes in k-Group

This problem often appears in interviews, and its difficulty on LeetCode is Hard. Let's look at the question:

**LeetCode 25. Reverse Nodes in k-Group** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

Given the `head` of a linked list, reverse the nodes of the list `k` at a time, and return *the modified list*.

`k` is a positive integer and is less than or equal to the length of the linked list. If the number of nodes is not a multiple of `k` then left-out nodes, in the end, should remain as it is.

You may not alter the values in the list's nodes, only nodes themselves may be changed.

Example 1:**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex1.jpg)

```

**Input:** head = [1,2,3,4,5], k = 2
**Output:** [2,1,4,3,5]

```

Example 2:**

![](https://assets.leetcode.com/uploads/2020/10/03/reverse_ex2.jpg)

```

**Input:** head = [1,2,3,4,5], k = 3
**Output:** [3,2,1,4,5]

```

**Constraints:**

	
- The number of nodes in the list is `n`.
	
- `1 <= k <= n <= 5000`
	
- `0 <= Node.val <= 1000`

**Follow-up:** Can you solve the problem in `O(1)` extra memory space?

With the previous explanations, is this problem really that hard? Actually, if you use the idea of "breaking down the problem" and reuse the `reverseN` function from before, it becomes much easier.

### Idea

If you think carefully, you will find that **this problem has a recursive nature**.

For example, let's call `reverseKGroup(head, 2)` on a linked list. This means we reverse the list in groups of 2:

![](../pictures/kgroup/1.jpg)

If we manage to reverse the first 2 nodes, what about the rest of the nodes? The rest is also a linked list, but smaller in size. This is a smaller subproblem with the same structure.

We can move the `head` pointer to the start of the next part of the list, and then call `reverseKGroup(head, 2)` recursively:

![](../pictures/kgroup/2.jpg)

Once we find the recursive structure, the general algorithm is clear:

**1. First, reverse the first `k` nodes starting from `head`.** Here, we can reuse the `reverseN` function we implemented before.

![](../pictures/kgroup/3.jpg)

**2. Use the (`k+1`)th node as `head`, and make a recursive call to `reverseKGroup`.**

![](../pictures/kgroup/4.jpg)

**3. Connect the results from the above two steps.**

![](../pictures/kgroup/5.jpg)

### Code

With the step-by-step explanation above, the code can be written directly. Here I use the iterative version of the `reverseN` function. You can also use the recursive one if you like:

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null) return null;
        // interval [a, b) contains k elements to be reversed
        ListNode a, b;
        a = b = head;
        for (int i = 0; i < k; i++) {
            // if there are less than k elements, no need to reverse
            if (b == null) return head;
            b = b.next;
        }
        // reverse the first k elements
        ListNode newHead = reverseN(a, k);
        // at this point, b points to the head node of the next group to be reversed
        // recursively reverse the subsequent linked list and connect them
        a.next = reverseKGroup(b, k);
        /**<extend up -90>
        ![](../pictures/kgroup/6.jpg)
        */
        return newHead;
    }

    // function to reverse the first N nodes implemented above
    ListNode reverseN(ListNode head, int n) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode pre, cur, nxt;
        pre = null; cur = head; nxt = head.next;
        while (n > 0) {
            cur.next = pre;
            pre = cur;
            cur = nxt;
            if (nxt != null) {
                nxt = nxt.next;
            }
            n--;
        }
        head.next = cur;
        /**<extend up>
        ![](../pictures/reverse-linked-list/6.jpg)
        */
        return pre;
    }
}
```

Very quickly, the problem is solved.

<visual slug='reverse-nodes-in-k-group'/>

## Summary

The idea of recursion is a bit harder to understand than iteration. The trick is: do not get lost in recursion, but use clear definitions to write your algorithm.

When you face a difficult problem, try to break it into smaller parts. Modify simple solutions to solve harder problems.
