::: info Prerequisites

Before reading this article, you should first learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)
- [Linked List Basics](https://labuladong.online/en/algo/data-structure-basic/linkedlist-basic/)
- [Queue/Stack Basics](https://labuladong.online/en/algo/data-structure-basic/queue-stack-basic/)

:::

A stack is a very simple data structure. It follows a first-in-last-out order, which fits some problems well, such as the function call stack. A monotonic stack is just a stack, but with some clever logic. After each new element is pushed, the elements in the stack stay ordered (monotonically increasing or decreasing).

Sounds a bit like a heap? No. A monotonic stack is not used as widely. It is used for a specific type of problem, such as “next greater element”, “previous smaller element”, and so on. This article explains the monotonic stack template to solve “next greater element” problems, and also talks about how to handle “circular arrays”. Other variants and classic problems will be in the next article: [Monotonic Stack Variants and Classic Problems](https://labuladong.online/en/algo/problem-set/monotonic-stack/).

## Monotonic Stack Template

Here is a problem: given an array `nums`, return a result array of the same length. For each index, store the next greater element. If there is no greater element, store -1. The function signature is:

```java
int[] calculateGreaterElement(int[] nums);
```

For example, if the input is `nums = [2,1,2,4,3]`, you should return `[4,2,4,-1,-1]`. The first 2 has a next greater element 4; after 1, the next greater element is 2; the second 2 also has 4 as its next greater element; 4 has no greater element after it, so we put -1; 3 also has no greater element after it, so we put -1.

The brute-force solution is easy: for each element, scan to the right and find the first greater element. But the time complexity is $O(n^2)$.

We can think about the problem like this: imagine the array elements as people standing in a line, and the value is the person’s height. They stand in a row facing you. How do you find the next greater element of the element “2”? Easy: if you can see “2”, then the first person you see behind “2” is its next greater element. Any person shorter than “2” will be blocked by “2”, so the first one who shows up must be taller and is the answer.

![](../pictures/monotonic-stack/1.jpeg)

This picture is easy to understand. With this in mind, look at the code:

```java
int[] calculateGreaterElement(int[] nums) {
    int n = nums.length;
    // array to store the answers
    int[] res = new int[n];
    Stack<Integer> s = new Stack<>(); 
    // push elements into the stack from the end
    for (int i = n - 1; i >= 0; i--) {
        // compare the heights
        while (!s.isEmpty() && s.peek() <= nums[i]) {
            // shorter ones step aside, they're blocked anyway...
            s.pop();
        }
        // the greater element behind nums[i]
        res[i] = s.isEmpty() ? -1 : s.peek();
        s.push(nums[i]);
    }
    return res;
}
```

This is the standard monotonic stack template. The for loop scans from right to left, because we use a stack. Pushing from right to left is like popping from left to right. The while loop removes elements between two “taller” elements, because those in the middle are useless. If a taller element stands in front of them, they can never be the next greater element for any future element.

The time complexity is not very obvious. Seeing a for loop with a nested while loop, you might think it is $O(n^2)$, but it is actually $O(n)$.

To analyze it, look at the whole process: there are `n` elements. Each element is pushed onto the stack once, and at most popped once. There is no extra work. So the total work is proportional to `n`, which is $O(n)$.

## Variants

The code for a monotonic stack is quite simple. Now let’s look at some concrete problems.

### 496. Next Greater Element I

First, an easy variant: LeetCode 496, “[Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/)”:

**LeetCode 496. Next Greater Element I** <span class="inline-block w-2 h-2 rounded-full bg-green-500"></span>

The **next greater element** of some element `x` in an array is the **first greater** element that is **to the right** of `x` in the same array.

You are given two **distinct 0-indexed** integer arrays `nums1` and `nums2`, where `nums1` is a subset of `nums2`.

For each `0 <= i < nums1.length`, find the index `j` such that `nums1[i] == nums2[j]` and determine the **next greater element** of `nums2[j]` in `nums2`. If there is no next greater element, then the answer for this query is `-1`.

Return *an array *`ans`* of length *`nums1.length`* such that *`ans[i]`* is the **next greater element** as described above.*

Example 1:**

```

**Input:** nums1 = [4,1,2], nums2 = [1,3,4,2]
**Output:** [-1,3,-1]
**Explanation:** The next greater element for each value of nums1 is as follows:
- 4 is underlined in nums2 = [1,3,4,2]. There is no next greater element, so the answer is -1.
- 1 is underlined in nums2 = [1,3,4,2]. The next greater element is 3.
- 2 is underlined in nums2 = [1,3,4,2]. There is no next greater element, so the answer is -1.

```

Example 2:**

```

**Input:** nums1 = [2,4], nums2 = [1,2,3,4]
**Output:** [3,-1]
**Explanation:** The next greater element for each value of nums1 is as follows:
- 2 is underlined in nums2 = [1,2,3,4]. The next greater element is 3.
- 4 is underlined in nums2 = [1,2,3,4]. There is no next greater element, so the answer is -1.

```

**Constraints:**

	
- `1 <= nums1.length <= nums2.length <= 1000`
	
- `0 <= nums1[i], nums2[i] <= 10^(4)`
	
- All integers in `nums1` and `nums2` are **unique**.
	
- All the integers of `nums1` also appear in `nums2`.

**Follow up:** Could you find an `O(nums1.length + nums2.length)` solution?

You are given two arrays `nums1` and `nums2`. For each element in `nums1`, find its next greater element in `nums2`. The function signature is:

```java
int[] nextGreaterElement(int[] nums1, int[] nums2);
```

We can solve it by slightly changing our previous code. Since `nums1` is a subset of `nums2`, we can first compute the next greater element for every element in `nums2`, and store it in a map. Then, for each element in `nums1`, we just look it up in the map:

```java
class Solution {
    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        // record the next greater element for each element in nums2
        int[] greater = nextGreaterElement(nums2);
        // convert to a map: element x -> next greater element of x
        HashMap<Integer, Integer> greaterMap = new HashMap<>();
        for (int i = 0; i < nums2.length; i++) {
            greaterMap.put(nums2[i], greater[i]);
        }
        // nums1 is a subset of nums2, so we can get the result based on greaterMap
        int[] res = new int[nums1.length];
        for (int i = 0; i < nums1.length; i++) {
            res[i] = greaterMap.get(nums1[i]);
        }
        return res;
    }

    // calculate the next greater element for each element in nums
    int[] nextGreaterElement(int[] nums) {
        int n = nums.length;
        // array to store the answers
        int[] res = new int[n];
        Stack<Integer> s = new Stack<>();
        // push elements onto the stack in reverse order
        for (int i = n - 1; i >= 0; i--) {
            // determine the height (size) of elements
            while (!s.isEmpty() && s.peek() <= nums[i]) {
                // remove the shorter elements as they are blocked anyway
                s.pop();
            }
            // the next greater element after nums[i]
            res[i] = s.isEmpty() ? -1 : s.peek();
            s.push(nums[i]);
        }
        return res;
    }
}
```

<visual slug='next-greater-element-i'/>

### 739. Daily Temperatures

Now look at LeetCode 739, “[Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)”:

You are given an array `temperatures` that records the temperature of each day. Return an array of the same length that, for each day, tells you how many days you have to wait until a warmer temperature. If there is no future day with a warmer temperature, put 0. The function signature is:

```java
int[] dailyTemperatures(int[] temperatures);
```

For example, if `temperatures = [73,74,75,71,69,76]`, the answer is `[1,1,3,2,1,0]`. For day 1 with 73°F, on day 2 it is 74°F, which is warmer. So for day 1, you need to wait 1 day; and so on.

This is also a “next greater element” problem. But now we do not return the value of the next greater element, we return the distance between the current index and the index of the next greater element.

We use the same idea, apply the monotonic stack template, and make small changes:

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        int n = temperatures.length;
        int[] res = new int[n];
        // Store element indices here, not the elements themselves
        Stack<Integer> s = new Stack<>(); 
        // Monotonic stack template
        for (int i = n - 1; i >= 0; i--) {
            while (!s.isEmpty() && temperatures[s.peek()] <= temperatures[i]) {
                s.pop();
            }
            // Get the distance between indices
            res[i] = s.isEmpty() ? 0 : (s.peek() - i); 
            // Push the index onto the stack, not the element
            s.push(i); 
        }
        return res;
    }
}
```

We are done with monotonic stacks. Next we will talk about another key point: how to handle “circular arrays”.


## How to Handle Circular Arrays

We still want to find the next greater element. But now the array is circular. How do we handle it? LeetCode 503 “[Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/)” is this problem: given a *circular array*, compute the next greater element for every element.

For example, input `[2,1,2,4,3]`, you should return `[4,2,4,-1,4]`. Because with the circular property, **the last element 3 can loop around and find 4 as its next greater element**.

If you have read the [Circular Array Techniques](https://labuladong.online/en/algo/data-structure-basic/cycle-array/) in the basics section, you should be familiar with this idea: we usually use the `%` operator (modulo) to simulate the circular behavior:

```java
int[] arr = {1,2,3,4,5};
int n = arr.length, index = 0;
while (true) {
    // Rotate in a circular array
    print(arr[index % n]);
    index++;
}
```

We still need to use the monotonic stack template to solve this problem. The hard part is: for input `[2,1,2,4,3]`, how can the last element 3 find 4 as its next greater element?

**A common trick for this kind of problem is to double the length of the array**:

![](../pictures/monotonic-stack/2.jpeg)

This way, element 3 can find element 4 as its next greater element, and other elements can also be computed correctly.

With this idea, the simplest way is to really build a new array of double length, then apply the template. But **we do not need to build a new array. We can use circular array tricks to *simulate* the effect of doubling the array**. Let’s go straight to the code:

```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] res = new int[n];
        Stack<Integer> s = new Stack<>();
        // Double the array length to simulate a circular array
        for (int i = 2 * n - 1; i >= 0; i--) {
            // Index i needs to be modulo, the rest is the same as the template
            while (!s.isEmpty() && s.peek() <= nums[i % n]) {
                s.pop();
            }
            res[i % n] = s.isEmpty() ? -1 : s.peek();
            s.push(nums[i % n]);
        }
        return res;
    }
}
```

<visual slug='next-greater-element-ii'/>

This nicely solves the circular array problem, with time complexity $O(N)$.

To end, think about some questions. The monotonic stack template we used is the `nextGreaterElement` function, which finds the next greater element for each position. But what if the problem asks for the *previous* greater element, or the previous greater or equal element? How should you change the template?

Also, in real problems, you are rarely asked to directly compute the next (or previous) greater (or smaller) element. How can you turn a problem into one that can be solved using a monotonic stack?

I will compare several variants of the monotonic stack and give classic practice problems in [Monotonic Stack Variants and Exercises](https://labuladong.online/en/algo/problem-set/monotonic-stack/). For more data structure design problems, see [Classic Data Structure Design Exercises](https://labuladong.online/en/algo/problem-set/ds-design/).
