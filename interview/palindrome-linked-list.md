::: info Prerequisites

Before reading this article, you need to learn:

- [Linked List Basics](https://labuladong.online/en/algo/data-structure-basic/linkedlist-basic/)
- [Two-Pointer Skills for Linked Lists](https://labuladong.online/en/algo/essential-technique/linked-list-skills-summary/)
- [Two-Pointer Skills for Arrays (Summary)](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/)

:::

The previous article [Two-Pointer Skills for Arrays (Summary)](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/) talked about problems on palindromes. Let’s quickly review.

The key idea to **find** a palindrome string is to expand from the center to both sides:

<!-- hide -->

```java
// Find the longest palindrome in s with s[left] and s[right] as the center
String palindrome(String s, int left, int right) {
    // Prevent index out of bounds
    while (left >= 0 && right < s.length()
            && s.charAt(left) == s.charAt(right)) {
        // Two pointers, expand to both sides
        left--;
        right++;
    }
    // Return the longest palindrome with s[left] and s[right] as the center
    return s.substring(left + 1, right);
}
```

A palindrome can have odd or even length.  
For odd length, there is one center. For even length, there are two centers.  
So the function above needs two inputs: `l` and `r`.

But to **check** whether a string is a palindrome is much easier. You don’t need to care about odd/even length. You just use the [two-pointer technique](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/) and move from both ends to the middle:

```java
boolean isPalindrome(String s) {
    // two pointers, one on the left and one on the right, move towards each other
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

This code is easy to understand. **A palindrome is symmetric, so reading it forward and backward is the same. This is the key for palindrome problems.**

Now let’s extend this simplest case to solve: how to check whether a **singly linked list** is a palindrome.

## 1. Check a palindrome singly linked list

Look at LeetCode 234: [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/):

**LeetCode 234. Palindrome Linked List** <span class="inline-block w-2 h-2 rounded-full bg-green-500"></span>

Given the `head` of a singly linked list, return `true`* if it is a **palindrome** or *`false`* otherwise*.

Example 1:**

![](https://assets.leetcode.com/uploads/2021/03/03/pal1linked-list.jpg)

```

**Input:** head = [1,2,2,1]
**Output:** true

```

Example 2:**

![](https://assets.leetcode.com/uploads/2021/03/03/pal2linked-list.jpg)

```

**Input:** head = [1,2]
**Output:** false

```

**Constraints:**

	
- The number of nodes in the list is in the range `[1, 10^(5)]`.
	
- `0 <= Node.val <= 9`

**Follow up:** Could you do it in `O(n)` time and `O(1)` space?

The function signature is:

```java
boolean isPalindrome(ListNode head);
```

The key problem is: a singly linked list cannot be traversed backward, so you cannot directly use the two-pointer method.

The simplest way is: reverse the original list into a new list, then compare the two lists. For how to reverse a list, see [Reverse Part of a Linked List Using Recursion](https://labuladong.online/en/algo/data-structure/reverse-linked-list-recursion/).

I said in [Framework Thinking for Learning Data Structures](https://labuladong.online/en/algo/essential-technique/algorithm-summary/): a linked list has a recursive structure, and a tree is just derived from it. So **a linked list also has “preorder” and “postorder” traversal. With the idea of postorder traversal in a binary tree, you can traverse a list in reverse order without explicitly reversing it**:

```java
// Binary tree traversal framework
void traverse(TreeNode root) {
    // Pre-order traversal code
    traverse(root.left);
    // In-order traversal code
    traverse(root.right);
    // Post-order traversal code
}

// Recursively traverse a single linked list
void traverse(ListNode head) {
    // Pre-order traversal code
    traverse(head.next);
    // Post-order traversal code
}
```

What does this mean?  
If you want to print `val` in normal order, write code in the preorder position.  
If you want reverse order, write code in the postorder position:

```java
// Print the element values in a singly linked list in reverse order
void traverse(ListNode head) {
    if (head == null) return;
    traverse(head.next);
    // Post-order traversal code
    print(head.val);
}
```

Now we can modify it a bit, and mimic the two-pointer method to check palindrome:

```java
class Solution {
    // pointer moving from left to right
    ListNode left;
    // pointer moving from right to left
    ListNode right;

    // record if the linked list is a palindrome
    boolean res = true;

    boolean isPalindrome(ListNode head) {
        left = head;
        traverse(head);
        return res;
    }

    void traverse(ListNode right) {
        if (right == null) {
            return;
        }

        // use recursion to reach the end of the linked list
        traverse(right.next);

        // in post-order traversal, the right pointer points to the end of the linked list
        // so we can compare it with the left pointer to check if it's a palindrome linked list
        if (left.val != right.val) {
            res = false;
        }
        left = left.next;
    }
}
```

What is the core idea here? **You are basically pushing nodes into a stack and popping them out, so the order becomes reversed**. We just use the recursion call stack.

<visual slug='is-palindrome' >

You can open the panel below. Click the line <code type="click">if (right === null)</code> many times. You will see pointer `right` reach the tail using the recursion stack.  
Then click the line <code type="click">left = left.next;</code> many times. You will see `left` move forward and `right` move backward. They move toward each other and finish the palindrome check.

</visual>

Of course, whether you build a reversed list or use postorder traversal, the time and space complexity are both O(N). Now let’s think: can we solve it without extra space?

## 2. Improve space complexity

A better idea is:

**1) Use fast and slow pointers (from [Two-Pointer Skills for Linked Lists](https://labuladong.online/en/algo/essential-technique/linked-list-skills-summary/)) to find the middle of the list**:

```java
ListNode slow, fast;
slow = fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
// the slow pointer now points to the middle of the linked list
```

![](../pictures/palindrome-list/1.jpg)

**2) If `fast` is not `null`, the list length is odd, and `slow` should move one more step**:

```java
if (fast != null)
    slow = slow.next;
```

![](../pictures/palindrome-list/2.jpg)

**3) Reverse the second half starting from `slow`. Then compare to check palindrome**:

```java
ListNode left = head;
ListNode right = reverse(slow);

while (right != null) {
    if (left.val != right.val)
        return false;
    left = left.next;
    right = right.next;
}
return true;
```

![](../pictures/palindrome-list/3.jpg)

Now combine these 3 parts, and you can solve the problem efficiently. The `reverse` function can be found in [Reverse a Singly Linked List](https://labuladong.online/en/algo/data-structure/reverse-linked-list-recursion/):

```java
class Solution {
    public boolean isPalindrome(ListNode head) {
        ListNode slow, fast;
        slow = fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        if (fast != null)
            slow = slow.next;

        ListNode left = head;
        ListNode right = reverse(slow);
        while (right != null) {
            if (left.val != right.val)
                return false;
            left = left.next;
            right = right.next;
        }

        return true;
    }

    ListNode reverse(ListNode head) {
        ListNode pre = null, cur = head;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
}
```

<visual slug='palindrome-linked-list'>

You can open the panel below. Click the line <code type="click">while (right != null)</code> many times. You will see `left` and `right` move toward each other and finish the palindrome check.

</visual>

Overall time complexity is O(N), space complexity is O(1), which is optimal.

Some readers may ask: this method is fast, but it changes the input list. Can we avoid this?

Yes. The key is to get the positions of pointers `p` and `q`:

![](../pictures/palindrome-list/4.jpg)

Then, before `return`, add this code to restore the list:

```java
p.next = reverse(q);
```

Due to space, I won’t write the full code here. You can try it yourself.

## 3. Summary

To find a palindrome string, expand from the center to both sides. To check a palindrome string, shrink from both ends to the center. For a singly linked list, you cannot traverse backward. You can build a new reversed list, use postorder traversal of the list, or use a stack to process it in reverse order.

For the palindrome linked list problem, because of the palindrome property, you don’t need to reverse the whole list. You only reverse part of it, and reduce space complexity to O(1).
