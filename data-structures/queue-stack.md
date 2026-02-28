::: info Prerequisites

Before reading this article, you should first learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)
- [Linked List Basics](https://labuladong.online/en/algo/data-structure-basic/linkedlist-basic/)
- [Queue and Stack Basics](https://labuladong.online/en/algo/data-structure-basic/queue-stack-basic/)

:::

A queue is a first-in-first-out (FIFO) data structure. A stack is a last-in-first-out (LIFO) data structure. Here is a simple illustration:

![](../pictures/stack-queue/1.jpg)

Both of these data structures are usually implemented using arrays or linked lists. Their APIs make them different. For more details, see [Principles and Implementation of Queue/Stack](https://labuladong.online/en/algo/data-structure-basic/queue-stack-basic/).

Today, let's see how to use a stack to build a queue, and how to use a queue to build a stack.


### 1. Implementing a Queue Using Stacks

LeetCode Problem 232 "[Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)" asks us to implement the following API:

```java
class MyQueue {
    
    // add element to the end of the queue
    public void push(int x);
    
    // remove the element at the front of the queue and return it
    public int pop();
    
    // return the front element
    public int peek();
    
    // check if the queue is empty
    public boolean empty();
}
```

We can use two stacks, `s1` and `s2`, to build a queue. (The diagram below shows how the stacks are arranged for better understanding.)

![](../pictures/stack-queue/2.jpg)

When you call `push` to add an element to the queue, just push the element onto `s1`. For example, if you `push` three elements 1, 2, and 3, the stacks look like this:

![](../pictures/stack-queue/3.jpg)

Now, what if you want to use `peek` to see the front element of the queue? The front of the queue should be 1, but in `s1`, the number 1 is at the bottom. This is where `s2` comes in. When `s2` is empty, you can pop all elements from `s1` and push them into `s2`. **Now the elements in `s2` are in the correct queue order (first in, first out).**

![](../pictures/stack-queue/4.jpg)

When `s2` has elements, you can just use `pop` on `s2` to remove the oldest element. This is how you get the `pop` operation of a queue.

Here is the complete code:

```java
class MyQueue {
    private Stack<Integer> s1, s2;

    public MyQueue() {
        s1 = new Stack<>();
        s2 = new Stack<>();
    }

    
    // Push element x to the back of the queue.
    public void push(int x) {
        s1.push(x);
    }

    // Removes the element from in front of queue and returns that element.
    public int pop() {
        // call peek first to ensure s2 is not empty
        peek();
        return s2.pop();
    }

    // Get the front element.
    public int peek() {
        if (s2.isEmpty())
            // push elements from s1 to s2
            while (!s1.isEmpty())
                s2.push(s1.pop());
        return s2.peek();
    }

    // Returns whether the queue is empty.
    public boolean empty() {
        return s1.isEmpty() && s2.isEmpty();
    }
}
```

With this method, we use two stacks to make a queue. The key idea is to let the two stacks work together.

Now, what is the time complexity of these operations?

The `peek` operation is interesting. When you call `peek`, it may trigger a `while` loop, making the time complexity O(N). But in most cases, the `while` loop is not triggered, so the time complexity is O(1). The `pop` operation also calls `peek`, so its time complexity is the same as `peek`.

In this situation, we can say the **worst-case time complexity** is O(N), because the `while` loop **might** need to move all elements from `s1` to `s2`.

But the **amortized time complexity** is O(1). Each element will be moved at most once, so on average, every `peek` operation takes O(1) time per element.

For more on analyzing time complexity, see [Practical Analysis of Time and Space Complexity](https://labuladong.online/en/algo/essential-technique/complexity-analysis/).


### 2. Implementing a Stack Using a Queue

If using two stacks to make a queue is clever, then using a queue to make a stack is more simple and direct. You only need one queue as the base data structure.

LeetCode Problem 225 “[Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)” asks us to build these APIs:

```java
class MyStack {
    
    // add element to the top of the stack
    public void push(int x);
    
    // remove the top element of the stack and return it
    public int pop();
    
    // return the top element of the stack
    public int top();
    
    // check if the stack is empty
    public boolean empty();
}
```

Let's start with the `push` API. Just add the element to the queue and record the last element of the queue. The last element is like the top of the stack. If you want to use `top` to see the top element, you can return it directly:

```java
class MyStack {
    Queue<Integer> q = new LinkedList<>();
    int top_elem = 0;

    // add element to the top of the stack
    public void push(int x) {
        // x is the tail of the queue, which is the top of the stack
        q.offer(x);
        top_elem = x;
    }
    
    // return the top element of the stack
    public int top() {
        return top_elem;
    }

    public boolean empty() {
        return q.isEmpty();
    }
}
```

Our base data structure is a queue, which is first in, first out. Each time you `pop`, you can only take from the front. But a stack is last in, first out, which means the `pop` API needs to take from the end:

![](../pictures/stack-queue/5.jpg)

The solution is simple. Take all the elements from the front of the queue and add them to the end, except the last one. This way, the original last element moves to the front, so you can take it out:

![](../pictures/stack-queue/6.jpg)

```java
class MyStack {
    // To save space, the code above is omitted...

    // Delete the top element of the stack and return it
    public int pop() {
        int size = q.size();
        while (size > 1) {
            q.offer(q.poll());
            size--;
        }
        // The previous last element of the queue is now at the front
        return q.poll();
    }
}
```

There is one small problem. The last element of the queue is moved to the front and removed, but the `top_elem` variable is not updated. We need to make a small change:

```java
class MyStack {
    // To save space, the code given above is omitted...

    // remove the top element and return it
    public int pop() {
        int size = q.size();
        // leave the last 2 elements
        while (size > 2) {
            q.offer(q.poll());
            size--;
        }
        // record the new last element
        top_elem = q.peek();
        q.offer(q.poll());
        // remove the previous last element
        return q.poll();
    }
}
```

Now it works. Here is the complete code:

```java
class MyStack {
    Queue<Integer> q = new LinkedList<>();
    int top_elem = 0;

    // Push element x onto stack
    public void push(int x) {
        // x is the tail of the queue, which is the top of the stack
        q.offer(x);
        top_elem = x;
    }

    // Removes the element on top of the stack and returns that element
    public int top() {
        return top_elem;
    }

    
    // Removes the element on top of the stack
    public int pop() {
        int size = q.size();
        // leave the last 2 elements in the queue
        while (size > 2) {
            q.offer(q.poll());
            size--;
        }
        // record the new tail element
        top_elem = q.peek();
        q.offer(q.poll());
        // remove the previous tail element
        return q.poll();
    }

    // Returns whether the stack is empty
    public boolean empty() {
        return q.isEmpty();
    }
}
```

It is clear that when you use a queue to make a stack, the `pop` operation takes O(N) time, and the other operations are O(1).

In my opinion, using a queue to make a stack is not very interesting, but using two stacks to make a queue is a good thing to learn.

![](../pictures/stack-queue/4.jpg)

After moving elements from stack `s1` to `s2`, the elements in `s2` become first in, first out, just like a queue. This is a bit like “two negatives make a positive” and is not easy to think of at first.
