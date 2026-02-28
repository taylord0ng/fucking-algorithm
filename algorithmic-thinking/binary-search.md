::: info Prerequisite

Before reading this article, you should first learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)

:::

First, a small joke:

One day, A-Dong borrowed `N` books from the library. When he was leaving, the alarm went off. The security guard stopped him to check which book was not registered. A-Dong was about to scan each book through the gate one by one to find the problem book, but the guard looked at him with disdain: "You don't even know binary search?"

So the guard split the books into two piles. He let the first pile pass the gate. The alarm sounded, which meant the problem book was in this pile. Then he split this pile into two piles again and let the first pile pass the gate. The alarm sounded again, and he kept splitting into two piles...

After `logN` checks, the guard finally found the book that triggered the alarm. He looked proud and mocking. Then A-Dong walked away with all the remaining books.

From then on, the library lost `N - 1` books.

Binary search is not simple. Knuth (the one who invented the KMP algorithm) said about binary search: **the idea is simple, the devil is in the details**. Many people like to talk about the integer overflow bug, but the real traps in binary search are not that small detail. The real questions are: should you add one or subtract one from `mid`? Should the while condition be `<=` or `<`?

If you do not really understand these details, writing binary search will feel like magic. You will rely on luck to avoid bugs, and only the writer knows how unreliable it is.

This article will explore the three most common binary search cases: find a number, find the left boundary, and find the right boundary. These three cases use one unified framework: the while condition is always `<=`, and the boundary updates always use `mid ± 1`, which is easy to remember.

## Binary Search Framework

There are two main styles of binary search:

- both ends closed, `[left, right]`
- left closed, right open, `[left, right)`

**This article uses the “both ends closed” style** because it is easier to remember and unify. The left-closed-right-open style is explained in another article: [Binary Search: Left-Closed-Right-Open Style](https://labuladong.online/en/algo/essential-technique/binary-search-left-open/).

No matter which style you use, the code follows this general framework:

```java
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while(...) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...
        } else if (nums[mid] > target) {
            right = ...
        }
    }
    return ...;
}
```

The parts marked with `...` are where details can go wrong. When you read any binary search code, first look at these places. Later we will see how they change in different cases.

One trick to write correct binary search is: do not use plain `else`. Instead, write all branches as `else if`, so every case is clear. After you fully understand all the details, you can simplify the code yourself.

**Another point: when computing `mid`, you must avoid overflow.** In the code, `left + (right - left) / 2` is the same as `(left + right) / 2` in result, but it prevents overflow when `left` and `right` are very large.

## The Idea of Search Interval

To really understand binary search, you must understand the idea of the “search interval” — what `left` and `right` mean.

For example, if we set `right = nums.length - 1`, the index of the last element, then the search interval is a closed interval `[left, right]`. This is exactly the range we search each time.

If we set `right = nums.length`, which is last index + 1, then the search interval is left-closed-right-open `[left, right)`.

**The essence of binary search is to use a `while` loop to move `left` and `right`, shrinking the search interval again and again, until we lock onto the index of the target value.**

Different open/closed choices give different code. For a correct binary search, we must make sure:

- When the search interval is empty, the search must stop. Otherwise, the algorithm will run forever.
- During the search, we must not skip any element. Otherwise, the answer can be wrong.

Keep these two rules in mind. Next, we will use the **both-ends-closed** style to handle three cases: find a number, find the left boundary, and find the right boundary.

## Find a Number

This is the simplest case: search for a number. If it exists, return its index; otherwise, return -1.

```java
class Solution {
    // standard binary search framework, search for the index of the target element, return -1 if it does not exist
    public int search(int[] nums, int target) {
        int left = 0;
        // note
        int right = nums.length - 1;

        while(left <= right) {
            int mid = left + (right - left) / 2;
            if(nums[mid] == target) {
                return mid;   
            } else if (nums[mid] < target) {
                // note
                left = mid + 1;
            } else if (nums[mid] > target) {
                // note
                right = mid - 1;
            }
        }
        return -1;
    }
}
```

<visual slug='binary-search'/>

This code solves LeetCode 704 “[Binary Search](https://leetcode.com/problems/binary-search/)”. Let’s look at the details.

### Why is the while condition `<=` and not `<`?

For a closed search interval, `<=` will not miss elements.

The loop `while (left <= right)` stops when `left == right + 1`. The search interval is then `[right + 1, right]`. There is no index that is both `>= right + 1` and `<= right`, so the interval is empty. Stopping here is correct.

If we use `while (left < right)`, the loop stops when `left == right`. The interval is `[left, left]`, which still has **one element**, but the while loop has stopped. That element is skipped. If the target is that element, the algorithm will wrongly think the target does not exist.

This shows why binary search is hard to write perfectly: bugs are not always visible. Most test cases might pass, and only some special cases will fail.

You must really understand the search interval to write fully correct code.

### Why `left = mid + 1` and `right = mid - 1`?

Because `mid` has already been checked, so we must remove it from the search interval.

Our interval is closed: `[left, right]`. When we find that index `mid` is not the target, the next search interval must be either `[left, mid - 1]` or `[mid + 1, right]`.

### What limitation does this algorithm have?

For example, in a sorted array `nums = [1,2,2,2,3]`, with `target = 2`, this algorithm returns index 2. That is fine. But what if we want the **left boundary** (index 1), or the **right boundary** (index 3)? This algorithm cannot do that.

You might say: once we find one `target`, we can linearly scan left or right. That works, but then we lose the logarithmic time advantage of binary search.

The next algorithms will handle these two “boundary search” cases, which are very common in real problems.

## Find the Left Boundary

First, look at the code for finding the left boundary:

```java
int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    // the search range is [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            // the search range becomes [mid+1, right]
            left = mid + 1;
        } else if (nums[mid] > target) {
            // the search range becomes [left, mid-1]
            right = mid - 1;
        } else if (nums[mid] == target) {
            // shrink the right boundary
            right = mid - 1;
        }
    }
    return left;
}
```

<visual slug='binary-search-left-bound'/>

### Why does this algorithm find the left boundary?

The key is how we handle `nums[mid] == target`:

```java
if (nums[mid] == target) {
    // shrink the right boundary
    right = mid - 1;
}
```

We do not return as soon as we find `target`. Instead, we shrink the right boundary with `right = mid - 1` and keep searching in `[left, mid - 1]`. This keeps moving the interval to the left, so we can lock onto the leftmost position of `target`.

### What if `target` does not exist?

If `target` does not exist, `left_bound` returns the index of the **smallest element that is greater than `target`**.

You do not need to memorize this. Just see an example: `nums = [2,3,5,7], target = 4`. The return value is 2, because the element 5 is the smallest element greater than 4.

If you want to return -1 when `target` does not exist, you can check `nums[left]` at the end:

```java
// if index is out of bounds, target definitely does not exist
if (left < 0 || left >= nums.length) {
    return -1;
}
// if nums[left] is not target, then target does not exist
if (nums[left] != target) {
    return -1;
}
// nums[left] == target, so left is the target index
return left;
```

## Find the Right Boundary

Now look at the code for finding the right boundary:

```java
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // here change to shrink the left boundary
            left = mid + 1;
        }
    }
    // finally change to return right
    return right;
}
```

<visual slug='binary-search-right-bound'/>

### Why does this algorithm find the right boundary?

Again, the key is how we handle `nums[mid] == target`:

```java
if (nums[mid] == target) {
    // shrink the left boundary
    left = mid + 1;
}
```

We do not return as soon as we find `target`. Instead, we shrink the left boundary with `left = mid + 1` and keep searching in `[mid + 1, right]`. This keeps moving the interval to the right, so we can lock onto the rightmost position of `target`.

### What if `target` does not exist?

If `target` does not exist, `right_bound` returns the index of the **largest element that is less than `target`**.

For example, `nums = [2,3,5,7], target = 4`. The return value is 1, because the element 3 is the largest element smaller than 4.

If you want to return -1 when `target` does not exist, you can check `nums[right]` at the end:

```java
// if index is out of bounds, target does not exist
if (right < 0 || right >= nums.length) {
    return -1;
}
// if nums[right] is not target, then target does not exist
if (nums[right] != target) {
    return -1;
}
// nums[right] == target, so right is the target index
return right;
```

## Unified Template

All three cases use the same closed search interval `[left, right]`. The only difference is what we do when `nums[mid] == target`:

```java
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // return directly
            return mid;
        }
    }
    // return directly
    return -1;
}

int left_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // do not return, lock the left boundary
            right = mid - 1;
        }
    }
    // determine if target exists in nums
    if (left < 0 || left >= nums.length) {
        return -1;
    }
    // check if nums[left] is the target
    return nums[left] == target ? left : -1;
}

int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // do not return, lock the right boundary
            left = mid + 1;
        }
    }
    // since the while loop ends with right == left - 1 and we are now looking for the right boundary
    // it's easier to remember to use right instead of left - 1
    if (right < 0 || right >= nums.length) {
        return -1;
    }
    return nums[right] == target ? right : -1;
}
```

From this article, you learned:

1. When you read or write binary search code, avoid plain `else`. Use only `if` and `else if` so that all cases are clear.
2. Always think about the “search interval” and the stopping condition of the while loop. Stop when the search interval is empty.
3. Use the same closed-interval style for all three cases, and only change the `nums[mid] == target` branch and the return logic.

In the end, this binary search framework is the “technique” level. On the “idea” level, **the core of binary search thinking is: use the known information to shrink (halve) the search space as much as possible**, so brute-force search becomes much faster.

If you understand this article, you can write correct binary search code. The closed-interval style here is my recommended style: one unified framework for all three cases, easy to remember.

You may also see another style on the internet: left-closed-right-open `[left, right)`, using `while (left < right)` and `right = mid`. That style is also common. If you want to deeply understand it, see [Binary Search: Left-Closed-Right-Open Style](https://labuladong.online/en/algo/essential-technique/binary-search-left-open/).

In real problems you are rarely asked to just “write binary search code”. I will further explain how to use binary search thinking to improve brute-force solutions in [Applications of Binary Search](https://labuladong.online/en/algo/frequency-interview/binary-search-in-action/) and [More Binary Search Exercises](https://labuladong.online/en/algo/problem-set/binary-search/).
