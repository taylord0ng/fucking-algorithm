::: info Prerequisite Knowledge

Before reading this article, you should first learn:

- [Array Basics](https://labuladong.online/en/algo/data-structure-basic/array-basic/)
- [Prefix Sum Technique](https://labuladong.online/en/algo/data-structure/prefix-sum/)

:::

The [Prefix Sum Technique](https://labuladong.online/en/algo/data-structure/prefix-sum/) is mainly used when the original array does not change, and you need to quickly find the sum of any interval. The key code is below:

```java
class PrefixSum {
    // prefix sum array
    private int[] preSum;

    // input an array to construct the prefix sum
    public PrefixSum(int[] nums) {
        // preSum[0] = 0, convenient for calculating cumulative sum
        preSum = new int[nums.length + 1];
        // calculate the cumulative sum of nums
        for (int i = 1; i < preSum.length; i++) {
            preSum[i] = preSum[i - 1] + nums[i - 1];
        }
    }
    
    // query the cumulative sum of the closed interval [left, right]
    public int sumRange(int left, int right) {
        return preSum[right + 1] - preSum[left];
    }
}
```

![](../pictures/difference/1.jpeg)

`preSum[i]` means the sum of all elements from `nums[0]` to `nums[i-1]`. If you want to find the sum from `nums[i]` to `nums[j]`, just calculate `preSum[j+1] - preSum[i]`. You don't need to loop through the whole interval.

In this article, we talk about another technique similar to prefix sum: the **Difference Array**. The main use of the difference array is when you need to frequently increase or decrease the values in a range of the original array.

For example, suppose you have an array `nums`, and you need to:

- add 1 to all elements from `nums[2]` to `nums[6]`
- subtract 3 from all elements from `nums[3]` to `nums[9]`
- add 2 to all elements from `nums[0]` to `nums[4]`
- and so on...

After many such operations, what is the final value of the `nums` array?

The usual way is simple. If you want to add `val` to all elements from `nums[i]` to `nums[j]`, just use a for loop and add `val` to each element. But this takes $O(N)$ time for each operation. If there are many operations, this will be slow.

Here is where the difference array helps. Just like the prefix sum uses a `preSum` array, we can build a `diff` array for `nums`. **`diff[i]` is the difference between `nums[i]` and `nums[i-1]`**:

```java
int[] diff = new int[nums.length];
// construct the difference array
diff[0] = nums[0];
for (int i = 1; i < nums.length; i++) {
    diff[i] = nums[i] - nums[i - 1];
}
```

![](../pictures/difference/2.jpeg)

Using this `diff` array, you can get back the original `nums` array. The code is like this:

```java
int[] res = new int[diff.length];
// construct the result array based on the difference array
res[0] = diff[0];
for (int i = 1; i < diff.length; i++) {
    res[i] = res[i - 1] + diff[i];
}
```

**With the difference array `diff`, you can quickly increase or decrease a range of elements.** If you want to add 3 to all elements from `nums[i]` to `nums[j]`, just do `diff[i] += 3` and `diff[j+1] -= 3`:

![](../pictures/difference/3.jpeg)

**The idea is simple. When you set `diff[i] += 3`, it means you add 3 to all elements from `nums[i]` and after. Then `diff[j+1] -= 3` means you subtract 3 from all elements from `nums[j+1]` and after. In total, only the elements from `nums[i]` to `nums[j]` get increased by 3.**

You only need O(1) time to change the `diff` array, which is like updating an entire interval in the original `nums` array. Do all your changes to `diff`, then build the final `nums` from `diff`.

Now, let's make the difference array into a class with `increment` and `result` methods:

```java
// Difference Array Utility Class
class Difference {
    // difference array
    private int[] diff;
    
    // input an initial array, range operations will be performed on this array
    public Difference(int[] nums) {
        diff = new int[nums.length];
        // construct the difference array based on the initial array
        diff[0] = nums[0];
        for (int i = 1; i < nums.length; i++) {
            diff[i] = nums[i] - nums[i - 1];
        }
    }

    // increment the closed interval [i, j] by val (can be negative)
    public void increment(int i, int j, int val) {
        diff[i] += val;
        if (j + 1 < diff.length) {
            diff[j + 1] -= val;
        }
    }

    // return the result array
    public int[] result() {
        int[] res = new int[diff.length];
        // construct the result array based on the difference array
        res[0] = diff[0];
        for (int i = 1; i < diff.length; i++) {
            res[i] = res[i - 1] + diff[i];
        }
        return res;
    }
}
```

Notice the `if` statement in the `increment` method:

```java
void increment(int i, int j, int val) {
    diff[i] += val;
    if (j + 1 < diff.length) {
        diff[j + 1] -= val;
    }
}
```

When `j+1 >= diff.length`, it means you are changing all elements from `nums[i]` to the end, so you don't need to subtract `val` from `diff` anymore.

<visual slug="diff-array-example" >

You can open the panel below. Click the line <code type="click">diff[i] = nums[i] - nums[i - 1]</code> several times to see how the `diff` array is built. Then click <code type="click">df.increment</code> a few times to see how the `diff` array changes:

</visual>


## Practical Examples

LeetCode problem 370 "[Range Addition](https://leetcode.com/problems/range-addition/)" directly tests the difference array technique. You're given an array `nums` of length `n` initialized to all zeros, asked to perform increment/decrement operations on ranges, and finally return the resulting `nums` array.

Just copy over the `Difference` class we implemented and you're done:

```java
class Solution {
    public int[] getModifiedArray(int length, int[][] updates) {
        // nums initialized to all 0s
        int[] nums = new int[length];
        // construct difference method
        Difference df = new Difference(nums);
        for (int[] update : updates) {
            int i = update[0];
            int j = update[1];
            int val = update[2];
            df.increment(i, j, val);
        }
        return df.result();
    }

    class Difference {
        // difference array
        private int[] diff;

        public Difference(int[] nums) {
            diff = new int[nums.length];
            // construct difference array
            diff[0] = nums[0];
            for (int i = 1; i < nums.length; i++) {
                diff[i] = nums[i] - nums[i - 1];
            }
        }

        // increment closed interval [i, j] by val (can be negative)
        public void increment(int i, int j, int val) {
            diff[i] += val;
            if (j + 1 < diff.length) {
                diff[j + 1] -= val;
            }
        }

        public int[] result() {
            int[] res = new int[diff.length];
            // construct result array based on difference array
            res[0] = diff[0];
            for (int i = 1; i < diff.length; i++) {
                res[i] = res[i - 1] + diff[i];
            }
            return res;
        }
    }

}
```

Of course, in real problems you'll need to recognize and abstract the pattern—they won't be this obvious about using difference arrays. Let's look at LeetCode problem 1109 "[Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/)":

**LeetCode 1109. Corporate Flight Bookings** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

There are `n` flights that are labeled from `1` to `n`.

You are given an array of flight bookings `bookings`, where `bookings[i] = [firsti, lasti, seatsi]` represents a booking for flights `firsti` through `lasti` (**inclusive**) with `seatsi` seats reserved for **each flight** in the range.

Return *an array *`answer`* of length *`n`*, where *`answer[i]`* is the total number of seats reserved for flight *`i`.

Example 1:**

```

**Input:** bookings = [[1,2,10],[2,3,20],[2,5,25]], n = 5
**Output:** [10,55,45,25,25]
**Explanation:**
Flight labels:        1   2   3   4   5
Booking 1 reserved:  10  10
Booking 2 reserved:      20  20
Booking 3 reserved:      25  25  25  25
Total seats:         10  55  45  25  25
Hence, answer = [10,55,45,25,25]

```

Example 2:**

```

**Input:** bookings = [[1,2,10],[2,2,15]], n = 2
**Output:** [10,25]
**Explanation:**
Flight labels:        1   2
Booking 1 reserved:  10  10
Booking 2 reserved:      15
Total seats:         10  25
Hence, answer = [10,25]

```

**Constraints:**

	
- `1 <= n <= 2 * 10^(4)`
	
- `1 <= bookings.length <= 2 * 10^(4)`
	
- `bookings[i].length == 3`
	
- `1 <= firsti <= lasti <= n`
	
- `1 <= seatsi <= 10^(4)`

The function signature is:

```java
int[] corpFlightBookings(int[][] bookings, int n)
```

This problem wraps things in confusing language, but it's really just a difference array problem. Let me translate it for you:

You're given an array `nums` of length `n` where all elements are 0. You're also given `bookings`, which contains several triplets `(i, j, k)`. Each triplet means you need to add `k` to all elements in the closed interval `[i-1, j-1]` of `nums`. Return the final `nums` array.

::: note Note

Since the problem counts `n` starting from 1, but array indices start from 0, the triplet `(i, j, k)` corresponds to the array interval `[i-1, j-1]`.

:::

Now it's clear—this is a standard difference array problem! We can directly reuse the class we wrote:

```java
class Solution {
    public int[] corpFlightBookings(int[][] bookings, int n) {
        // initialize nums as all 0
        int[] nums = new int[n];
        // construct the difference array
        Difference df = new Difference(nums);

        for (int[] booking : bookings) {
            // note that converting to array index needs to subtract one
            int i = booking[0] - 1;
            int j = booking[1] - 1;
            int val = booking[2];
            // increment the range nums[i..j] by val
            df.increment(i, j, val);
        }
        // return the final result array
        return df.result();
    }

    class Difference {
        // difference array
        private int[] diff;

        public Difference(int[] nums) {
            diff = new int[nums.length];
            // construct the difference array
            diff[0] = nums[0];
            for (int i = 1; i < nums.length; i++) {
                diff[i] = nums[i] - nums[i - 1];
            }
        }

        // increment the closed interval [i, j] by val (can be negative)
        public void increment(int i, int j, int val) {
            diff[i] += val;
            if (j + 1 < diff.length) {
                diff[j + 1] -= val;
            }
        }

        public int[] result() {
            int[] res = new int[diff.length];
            // construct the result array based on the difference array
            res[0] = diff[0];
            for (int i = 1; i < diff.length; i++) {
                res[i] = res[i - 1] + diff[i];
            }
            return res;
        }
    }

}
```

Problem solved.

Here's another similar problem—LeetCode problem 1094 "[Car Pooling](https://leetcode.com/problems/car-pooling/)":

**LeetCode 1094. Car Pooling** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

There is a car with `capacity` empty seats. The vehicle only drives east (i.e., it cannot turn around and drive west).

You are given the integer `capacity` and an array `trips` where `trips[i] = [numPassengersi, fromi, toi]` indicates that the `i^(th)` trip has `numPassengersi` passengers and the locations to pick them up and drop them off are `fromi` and `toi` respectively. The locations are given as the number of kilometers due east from the car's initial location.

Return `true`* if it is possible to pick up and drop off all passengers for all the given trips, or *`false`* otherwise*.

Example 1:**

```

**Input:** trips = [[2,1,5],[3,3,7]], capacity = 4
**Output:** false

```

Example 2:**

```

**Input:** trips = [[2,1,5],[3,3,7]], capacity = 5
**Output:** true

```

**Constraints:**

	
- `1 <= trips.length <= 1000`
	
- `trips[i].length == 3`
	
- `1 <= numPassengersi <= 100`
	
- `0 <= fromi < toi <= 1000`
	
- `1 <= capacity <= 10^(5)`

The function signature is:

```java
boolean carPooling(int[][] trips, int capacity);
```

For example, given:

```
trips = [[2,1,5],[3,3,7]], capacity = 4
```

This trip can't be completed in one go because `trips[1]` can only pick up 2 more passengers at most, otherwise the car would be overloaded.

You've probably already connected this to difference arrays: **each `trips[i]` represents a range operation—passengers boarding and alighting correspond to incrementing and decrementing ranges. If every element in the result array is less than `capacity`, all passengers can be transported without overloading.**

But what should the length of the difference array (number of stops) be? The problem doesn't tell us directly, but gives us the data range:

```java
0 <= trips[i][1] < trips[i][2] <= 1000
```

Stop numbers range from 0 to at most 1000, meaning there are at most 1001 stops. So we can set our difference array length to 1001, which covers all possible stop numbers:

```java
class Solution {
    public boolean carPooling(int[][] trips, int capacity) {
        // at most there are 1000 stops
        int[] nums = new int[1001];
        // construct the difference array method
        Difference df = new Difference(nums);

        for (int[] trip : trips) {
            // number of passengers
            int val = trip[0];
            // passengers get on at stop trip[1]
            int i = trip[1];
            // passengers get off at stop trip[2],
            // meaning the interval passengers are on the car is [trip[1], trip[2] - 1]
            int j = trip[2] - 1;
            // perform interval operation
            df.increment(i, j, val);
        }

        int[] res = df.result();

        // the car should never exceed its capacity
        for (int i = 0; i < res.length; i++) {
            if (capacity < res[i]) {
                return false;
            }
        }
        return true;
    }

    // difference array utility class
    class Difference {
        // difference array
        private int[] diff;

        // input an initial array, interval operations will be performed on this array
        public Difference(int[] nums) {
            diff = new int[nums.length];
            // construct the difference array based on the initial array
            diff[0] = nums[0];
            for (int i = 1; i < nums.length; i++) {
                diff[i] = nums[i] - nums[i - 1];
            }
        }

        // increment the closed interval [i, j] by val (can be negative)
        public void increment(int i, int j, int val) {
            diff[i] += val;
            if (j + 1 < diff.length) {
                diff[j + 1] -= val;
            }
        }

        // return the result array
        public int[] result() {
            int[] res = new int[diff.length];
            // construct the result array based on the difference array
            res[0] = diff[0];
            for (int i = 1; i < diff.length; i++) {
                res[i] = res[i - 1] + diff[i];
            }
            return res;
        }
    }

}
```

And that solves this problem too.

Both difference arrays and prefix sum arrays are common and clever techniques. They suit different scenarios, and once you understand them, they're straightforward—but tricky if you don't.

## Further Reading

First question: to use the difference array technique, you need to create a `diff` array with the same length as your interval. What if you have a huge interval like `[0, 10^9]`? Do you really need to create an array of length `10^9` just to start doing range operations?

Second question: prefix sums enable fast range queries, while difference arrays enable fast range updates. Can we combine them to get both fast range updates AND fast range queries?

These are common questions when dealing with range problems. The ultimate answer is the [Segment Tree](https://labuladong.online/en/algo/data-structure-basic/segment-tree-basic/) data structure, which can perform both range updates and range queries on intervals of any length in $O(\log N)$ time.
