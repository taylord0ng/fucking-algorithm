Today, let's discuss a seemingly simple yet ingenious problem: finding the missing and duplicate elements. A similar problem was covered in a previous article [Common Bit Manipulations](https://labuladong.online/en/algo/frequency-interview/bitwise-operation/), but the techniques used in this case are different.

This is LeetCode problem 645 "[Set Mismatch](https://leetcode.com/problems/set-mismatch/)", and I'll describe the problem:

Given an array `nums` of length `N`, originally containing the `N` elements from `[1..N]` in no particular order. However, some errors have occurred: one element in `nums` appears twice, leading to another element missing. Write an algorithm to find the values of the duplicate and missing elements in `nums`.

```java
// Return two numbers, which are {dup, missing}
int[] findErrorNums(int[] nums);
```

For example, given the input: `nums = [1,2,2,4]`, the algorithm should return `[2,3]`.

This problem can be easily solved by first traversing the array and using a hash table to record the frequency of each number, then traversing `[1..N]` to find which element appears twice and which is missing.

However, this conventional solution requires a hash table, resulting in O(N) space complexity. Given the problem's conditions, one might think of using a more elegant method to solve it.

Traversing the array in O(N) time complexity is unavoidable, but we can try to reduce the space complexity. Is it possible to find the duplicate and missing elements with an O(1) space complexity?


<!-- hide -->

## Thought Process Analysis

The characteristic of this problem is that each element corresponds to a certain index in the array.

Let's modify the problem temporarily: **Let the elements in `nums` be `[0..N-1]`, so each element corresponds to an array index, which makes it easier to understand**.

If there are no duplicate or missing elements in `nums`, each element corresponds to a unique index value, right?

The current issue is that one element is duplicated, causing another element to be missing. What phenomenon does this create? **It results in two elements corresponding to the same index, and one index having no corresponding element**.

So, if I can find the index with duplicate correspondence, wouldn't that identify the duplicate element? And finding the index with no corresponding element identifies the missing element, right?

How can we determine, without extra space, how many elements correspond to a certain index? This is the clever part of the problem:

**By turning the element corresponding to each index into a negative number, we indicate that this index has been corresponded to once**, as shown in the GIF below:

![](../pictures/dupmissing/1.gif)

If a duplicate element `4` appears, the intuitive result is that the element corresponding to index `4` is already negative:

![](../pictures/dupmissing/2.jpg)

For the missing element `3`, the intuitive result is that the element corresponding to index `3` is positive:

![](../pictures/dupmissing/3.jpg)

This phenomenon can be translated into code:


```java
int[] findErrorNums(int[] nums) {
    int n = nums.length;
    int dup = -1;
    for (int i = 0; i < n; i++) {
        int index = Math.abs(nums[i]);
        // if nums[index] is less than 0, it indicates a duplicate visit
        if (nums[index] < 0)
            dup = Math.abs(nums[i]);
        else
            nums[index] *= -1;
    }

    int missing = -1;
    for (int i = 0; i < n; i++)
        // if nums[i] is greater than 0, it indicates it was not visited
        if (nums[i] > 0)
            missing = i;
    
    return new int[]{dup, missing};
}
```

This issue is essentially resolved. Don't forget that for convenience, we assumed elements were `[0..N-1]`, but the problem requires `[1..N]`. So, we can obtain the original answer by simply modifying two parts:

```java
class Solution {
    public int[] findErrorNums(int[] nums) {
        int n = nums.length;
        int dup = -1;
        for (int i = 0; i < n; i++) {
            // the current elements start from 1
            int index = Math.abs(nums[i]) - 1;
            if (nums[index] < 0)
                dup = Math.abs(nums[i]);
            else
                nums[index] *= -1;
        }

        int missing = -1;
        for (int i = 0; i < n; i++)
            if (nums[i] > 0)
                // convert index to element
                missing = i + 1;

        return new int[]{dup, missing};
    }
}
```

<visual slug='set-mismatch'/>

Actually, starting elements from 1 is logical and necessary, because if elements start from 0, the opposite of 0 is still itself. So, if the number 0 is repeated or missing, the algorithm cannot determine if 0 has been visited. Our previous assumption was just to simplify the problem and make it more understandable.

## Final Summary

For such array problems, **the key point is that elements and indices appear in pairs, and the common methods are sorting, XOR, and mapping**.

The mapping approach is what we analyzed earlier, mapping each index to an element, using positive and negative signs to record whether an element has been mapped.

The sorting method is also easy to understand. For this problem, if the elements are sorted in ascending order, and an index does not match the corresponding element, the repeated or missing elements can be found.

XOR operation is commonly used as well, due to its properties `a ^ a = 0, a ^ 0 = a`. By XORing both indices and elements, paired indices and elements can be eliminated, leaving the repeated or missing elements. You can refer to the previous article [Common Bit Operations](https://labuladong.online/en/algo/frequency-interview/bitwise-operation/) for more on this method.
