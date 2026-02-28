::: info Prerequisite Knowledge

Before reading this article, you should first study:

- [Core Framework of Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

Many people frequently ask about the knapsack problem. Actually, it's not difficult. Using the dynamic programming approach, it's just about states and choices—nothing special. Today, let's discuss the knapsack problem, focusing on the most common 0-1 knapsack problem. Problem description:

You have a knapsack with a capacity of `W`, and `N` items. Each item has a weight and a value. The weight of the ith item is `wt[i]`, and its value is `val[i]`. You can put items into the knapsack, but each item can only be used once. What is the maximum total value you can carry in the knapsack without exceeding its capacity?

![](../pictures/knapsack/1.png)

Here's a simple example. Input:

```py
N = 3, W = 4
wt = [2, 1, 3]
val = [4, 2, 3]
```

The algorithm returns 6. You can put the first two items into the knapsack, with a total weight of 3, which is less than `W`, and the maximum value you can get is 6.

That's the whole problem—a classic dynamic programming problem. The items cannot be divided; you either put an item in the knapsack or you don't. You can't split an item. This is where the name 0-1 knapsack comes from.

There is no clever sorting or shortcut to solve this problem. You have to try all possible combinations. Following the pattern from our [Dynamic Programming Detailed Explanation](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), just follow the process step by step.


## Standard DP Pattern

In almost every dynamic programming article, we repeat this pattern. All DP problems in past articles follow this pattern.

**Step 1: Make clear two things: "state" and "choice".**

First, the state. How do we describe the situation of the problem?  
If we are given some items and a capacity limit of a bag, we get a knapsack problem.  
**So we have two states: "capacity of the knapsack" and "items we can choose".**

Next, the choice. This is also easy. For each item, what can you do?  
**The choice is "put it into the knapsack" or "do not put it into the knapsack".**

Once you understand the states and choices, the DP problem is almost solved.  
For a bottom-up solution, the general code pattern is:

```python
for 状态1 in 状态1的所有取值：
    for 状态2 in 状态2的所有取值：
        for ...
            dp[状态1][状态2][...] = 择优(选择1，选择2...)
```

**Step 2: Define the `dp` array.**

Look at the states we found. There are two.  
So we need a 2D `dp` array.

::: important Definition of `dp` in the knapsack problem  

`dp[i][w]` is defined as: for the first `i` items, if the current knapsack capacity is `w`, then the maximum value we can get is `dp[i][w]`.

For example, if `dp[3][5] = 6`, it means: from the first 3 items, when the knapsack capacity is 5, the maximum value we can put into the bag is 6.

:::

Why define it like this? Because this helps us find the state transition.  
You can remember this as a pattern for knapsack-like problems.  
When you see similar problems, you can try this definition.

With this definition, the final answer we want is `dp[N][W]`.  
The base cases are `dp[0][..] = 0` and `dp[..][0] = 0`, because if there are no items or no capacity, the max value is 0.

Now refine the framework:

```python
int[][] dp[N+1][W+1]
dp[0][..] = 0
dp[..][0] = 0

for i in [1..N]:
    for w in [1..W]:
        dp[i][w] = max(
            把物品 i 装进背包,
            不把物品 i 装进背包
        )
return dp[N][W]
```

**Step 3: Use the "choices" to write the state transition.**

We now ask: how do we write

- "put item `i` into the knapsack"
- "do not put item `i` into the knapsack"

in code?

We go back to the definition of `dp` and see how these choices change the state.

Recall the definition:

`dp[i][w]` means: for the first `i` items (indexed from 1), when the capacity is `w`, the maximum value is `dp[i][w]`.

**If you do not put item `i` into the knapsack**,  
then clearly `dp[i][w]` should just be `dp[i-1][w]`.  
You just keep the previous result.

**If you do put item `i` into the knapsack**,  
then `dp[i][w]` should be `val[i-1] + dp[i-1][w - wt[i-1]]`.

Array indices start from 0, but our `i` starts from 1.  
So `val[i-1]` and `wt[i-1]` are the value and weight of item `i`.

If you choose to put item `i` into the bag:

- you get its value `val[i-1]`
- with the remaining capacity `w - wt[i-1]`
- you can only choose from the first `i-1` items

So the best you can do is `dp[i-1][w - wt[i-1]]`.  
Add them together: `val[i-1] + dp[i-1][w - wt[i-1]]`.

These are the two choices. Now we have the state transition equation and can refine the code:

```python
for i in [1..N]:
    for w in [1..W]:
        dp[i][w] = max(
            dp[i-1][w],
            dp[i-1][w - wt[i-1]] + val[i-1]
        )
return dp[N][W]
```

**Step 4: Translate the pseudocode into real code and handle edge cases.**

Here is Java code that implements the idea and handles the case where `w - wt[i-1]` might be negative (which would cause an index error):

```java
int knapsack(int W, int[] wt, int[] val) {
    int N = wt.length;
    // base case has been initialized
    int[][] dp = new int[N + 1][W + 1];
    for (int i = 1; i <= N; i++) {
        for (int w = 1; w <= W; w++) {
            if (w - wt[i - 1] < 0) {
                // in this case, we can only choose not to put it in the backpack
                dp[i][w] = dp[i - 1][w];
            } else {
                // choose the better option between putting it in the backpack or not
                dp[i][w] = Math.max(
                    dp[i - 1][w - wt[i-1]] + val[i-1], 
                    dp[i - 1][w]
                );
            }
        }
    }
    
    return dp[N][W];
}
```

<visual slug='mydata-knapsack'/>

## Space Optimization

We can see that `dp[i][..]` only depends on `dp[i-1][..]`.  
So we can do [space compression for DP](https://labuladong.online/en/algo/dynamic-programming/space-optimization/) to reduce space.

More concretely, we do not need a 2D array with `N` rows.  
We only need a 2D array with 2 rows: `dp[2][W+1]`.  
When we compute row `i`, we only use row `(i-1) % 2`.

```java
// Rolling array optimization
int[][] dp = new int[2][W+1];

for (int i = 1; i <= N; i++) {
    int curr = i % 2;
    int prev = (i + 1) % 2;
    for (int w = 1; w <= W; w++) {
        if (w - wt[i-1] < 0) {
            // In this case, we can only choose not to put it into the bag
            dp[curr][w] = dp[prev][w];
        } else {
            // Put it in or not, take the better one
            dp[curr][w] = Math.max(
                dp[prev][w],
                dp[prev][w - wt[i-1]] + val[i-1]
            );
        }
    }
}
return dp[N % 2][W];
```

More often, we compress it further into a 1D array, but then you must be careful with the loop order.  
If you are interested, you can click the link to learn more.

Now the knapsack problem is solved.  
Compared to other DP problems, this one is quite simple, because the state transition is very natural.  
Once you make the `dp` definition clear, the transition almost follows directly.
