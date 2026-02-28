::: info Prerequisites

Before reading this article, you need to learn:

- [Dynamic Programming Core Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

Many readers complain that the stock series problems on LeetCode have too many different solutions. If you encounter these problems in an interview, you probably won't think of those clever approaches. What should you do? **So this article won't cover those overly clever ideas. Instead, we'll take a solid, step-by-step approach, using just one universal method to solve all problems - adapting to any situation with a consistent strategy**.

This article references the approach from this [highly upvoted solution](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/discuss/108870/Most-consistent-ways-of-dealing-with-the-series-of-stock-problems), using state machine techniques to solve these problems, all of which can pass submission. Don't let this fancy term intimidate you - it's just a literary expression. In reality, it's just a DP table, and you'll understand it at a glance.

Let's randomly pick a problem and look at someone else's solution:

```java
int maxProfit(int[] prices) {
    if(prices.empty()) return 0;
    int s1 = -prices[0], s2 = INT_MIN, s3 = INT_MIN, s4 = INT_MIN;

    for(int i = 1; i < prices.size(); ++i) {            
        s1 = max(s1, -prices[i]);
        s2 = max(s2, s1 + prices[i]);
        s3 = max(s3, s2 - prices[i]);
        s4 = max(s4, s3 + prices[i]);
    }
    return max(0, s4);
}
```

Can you understand it? Can you solve it now? Impossible - you can't understand it, and that's normal. Even if you barely understand it, you still won't be able to solve the next problem. Why can others write such strange yet efficient solutions? Because there's a framework for this type of problem, but they won't tell you. Once they tell you, you'll learn it in five minutes, and this algorithm problem will no longer be mysterious - it becomes easy to crack.

This article will tell you this framework, then guide you through solving each problem quickly. We'll use state machine techniques to solve these problems, all of which can pass submission. Don't let this fancy term intimidate you - it's just a literary expression. In reality, it's just a DP table, and you'll understand it at a glance.

These 6 problems share common characteristics. We only need to focus on LeetCode problem 188 "[Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)" because this is the most generalized form. The other problems are simplifications of this form. Let's look at the problem:

**LeetCode 188. Best Time to Buy and Sell Stock IV** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day, and an integer `k`.

Find the maximum profit you can achieve. You may complete at most `k` transactions: i.e. you may buy at most `k` times and sell at most `k` times.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

Example 1:**

```

**Input:** k = 2, prices = [2,4,1]
**Output:** 2
**Explanation:** Buy on day 1 (price = 2) and sell on day 2 (price = 4), profit = 4-2 = 2.

```

Example 2:**

```

**Input:** k = 2, prices = [3,2,6,5,0,3]
**Output:** 7
**Explanation:** Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4. Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.

```

**Constraints:**

	
- `1 <= k <= 100`
	
- `1 <= prices.length <= 1000`
	
- `0 <= prices[i] <= 1000`

The first problem allows only one transaction, equivalent to `k = 1`; the second problem allows unlimited transactions, equivalent to `k = +infinity`; the third problem allows only 2 transactions, equivalent to `k = 2`; the remaining two problems also allow unlimited transactions, but add extra conditions like "cooldown period" and "transaction fee" - they're actually variants of the second problem and are easy to handle.

Now let's get down to business and start solving.


## Brute-Force Framework

First, let's think about the same question: how do we brute-force all possibilities?

As mentioned in [Dynamic Programming Core Pattern](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), dynamic programming is essentially about brute-forcing all "states" and then picking the optimal solution from all "choices".

For this problem, let's look at each day and see how many possible "states" there are, then find the "choices" for each "state". We need to enumerate all "states", and the purpose is to update states based on the corresponding "choices". This sounds abstract, but just remember the two words "state" and "choice", and it will become clear once we work through an example.

```python
for state1 in all_values_of_state1:
    for state2 in all_values_of_state2:
        for ...
            dp[state1][state2][...] = best(choice1, choice2...)
```

For this problem, **there are three "choices" each day**: buy, sell, or do nothing. We use `buy`, `sell`, `rest` to represent these three choices.

But the issue is, we can't freely choose any of these three options every day. `sell` must come after `buy`, and `buy` must come after `sell`. Also, `rest` should be split into two states: one is `rest` after `buy` (holding stock), and the other is `rest` after `sell` (not holding stock). Don't forget we also have a limit on the number of transactions `k`, which means `buy` can only happen when `k > 0`.

::: note Note

Note that I will frequently use the word "transaction" in this article. **We define one buy and one sell as one "transaction"**.

:::


Seems complex, right? Don't worry, our goal right now is just to brute-force. No matter how many states you have, we'll list them all out.

**This problem has three "states"**: the first is the day number, the second is the maximum number of transactions allowed, and the third is the current holding status (the `rest` state mentioned earlier; we can use 1 for holding and 0 for not holding). We can use a 3D array to store all combinations of these states:

```python
dp[i][k][0 or 1]
0 <= i <= n - 1, 1 <= k <= K
n is the number of days, K is the maximum number of transactions, 0 and 1 represent whether holding stock.
This problem has n × K × 2 states in total. Enumerate all of them and we're done.

for 0 <= i < n:
    for 1 <= k <= K:
        for s in {0, 1}:
            dp[i][k][s] = max(buy, sell, rest)
```

We can describe each state in plain language. For example, `dp[3][2][1]` means: today is day 3, I'm currently holding stock, and I've made at most 2 transactions so far. Another example, `dp[2][3][0]` means: today is day 2, I'm not holding any stock, and I've made at most 3 transactions so far. Easy to understand, right?

The final answer we want is `dp[n - 1][K][0]`, which means on the last day, with at most `K` transactions allowed, what's the maximum profit.

You might ask why not `dp[n - 1][K][1]`? Because `dp[n - 1][K][1]` means you're still holding stock on the last day, while `dp[n - 1][K][0]` means you've sold all your stock by the last day. Obviously, the latter gives more profit than the former.

Remember how to interpret "states". Whenever something feels confusing, translate it into plain language and it becomes easier to understand.


## State Transition Framework

Now that we've completed the brute-force enumeration of "states", let's think about what "choices" each state has and how to update the "state".

Looking only at the "holding state", we can draw a state transition diagram:

![](../pictures/stock/1.png)

This diagram clearly shows how each state (0 and 1) transitions. Based on this diagram, let's write the state transition equations:

```python
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
              max( choose rest today,   choose sell today    )
```

Explanation: Today I don't hold any stock. There are two possibilities, and I want the maximum profit from these two:

1. I didn't hold stock yesterday, and the maximum transaction limit up to yesterday was `k`. Then I choose `rest` today, so I still don't hold stock today, and the maximum transaction limit remains `k`.

2. I held stock yesterday, and the maximum transaction limit up to yesterday was `k`. But today I `sell`, so I no longer hold stock today, and the maximum transaction limit remains `k`.

```python
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
              max( choose rest today,    choose buy today     )
```

Explanation: Today I'm holding stock with a maximum transaction limit of `k`. For yesterday, there are two possibilities, and I want the maximum profit from these two:

1. I was holding stock yesterday, and the maximum transaction limit up to yesterday was `k`. Then I choose `rest` today, so I'm still holding stock today, and the maximum transaction limit remains `k`.

2. I wasn't holding stock yesterday, and the maximum transaction limit up to yesterday was `k - 1`. But today I choose `buy`, so now I'm holding stock today, and the maximum transaction limit is `k`.

::: note Note

Here's an important reminder: **always keep the definition of "state" in mind**. State `k` is not defined as "the number of completed transactions", but rather "the upper limit on maximum transactions". If you decide to make a transaction today and want to ensure the maximum transaction limit up to today is `k`, then yesterday's maximum transaction limit must be `k - 1`. For a concrete example, suppose you need at least $100 in your bank account today, and you know you can earn $10 today, then you need to make sure your bank account had at least $90 yesterday.

:::

This explanation should be clear. If you `buy`, you subtract `prices[i]` from the profit; if you `sell`, you add `prices[i]` to the profit. Today's maximum profit is the larger of these two possible choices.

Note the constraint on `k`: when choosing `buy`, it's like starting a new transaction, so for yesterday, the transaction limit `k` should decrease by 1.

::: note Note

Here's a correction: I used to think that decreasing `k` by 1 when `sell` and decreasing `k` by 1 when `buy` were equivalent. But a careful reader raised a question, and after deeper thought, I realized the former is indeed wrong. Since a transaction starts with `buy`, if the `buy` choice doesn't change the transaction count `k`, you'll get errors where the transaction count exceeds the limit.

:::

Now we've completed the most difficult part of dynamic programming: the state transition equations. **If you understand everything above, you can solve all these problems—just apply this framework**. But there's one more thing: defining the base case, the simplest situations.

```python
dp[-1][...][0] = 0
Explanation: Since i starts from 0, i = -1 means we haven't started yet, so the profit is naturally 0.

dp[-1][...][1] = -infinity
Explanation: Before we start, it's impossible to hold any stock.
Since our algorithm seeks a maximum value, we set the initial value to a minimum to facilitate finding the maximum.

dp[...][0][0] = 0
Explanation: Since k starts from 1, k = 0 means no transactions are allowed, so the profit is naturally 0.

dp[...][0][1] = -infinity
Explanation: When no transactions are allowed, it's impossible to hold any stock.
Since our algorithm seeks a maximum value, we set the initial value to a minimum to facilitate finding the maximum.
```

Let's summarize the state transition equations above:

```python
base case:
dp[-1][...][0] = dp[...][0][0] = 0
dp[-1][...][1] = dp[...][0][1] = -infinity

state transition equations:
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
```

You might ask: how do we represent an array index of -1 in code, and how do we represent negative infinity? These are implementation details with many solutions. **The most common technique is "index offset"**: we expand the `dp` array size from `n` to `n + 1`, letting `dp[0]` represent the base case (the original `dp[-1]`), and `dp[i]` represent the state of the original day `i - 1`. This way we avoid handling boundary cases in the loop. Now the complete framework is ready. Let's get specific.


## Solving the Problems

### 121. Best Time to Buy and Sell Stock

**For the first problem, let's look at LeetCode problem 121 "[Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)", which is equivalent to the case where `k = 1`**:

**LeetCode 121. Best Time to Buy and Sell Stock** <span class="inline-block w-2 h-2 rounded-full bg-green-500"></span>

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day.

You want to maximize your profit by choosing a **single day** to buy one stock and choosing a **different day in the future** to sell that stock.

Return *the maximum profit you can achieve from this transaction*. If you cannot achieve any profit, return `0`.

Example 1:**

```

**Input:** prices = [7,1,5,3,6,4]
**Output:** 5
**Explanation:** Buy on day 2 (price = 1) and sell on day 5 (price = 6), profit = 6-1 = 5.
Note that buying on day 2 and selling on day 1 is not allowed because you must buy before you sell.

```

Example 2:**

```

**Input:** prices = [7,6,4,3,1]
**Output:** 0
**Explanation:** In this case, no transactions are done and the max profit = 0.

```

**Constraints:**

	
- `1 <= prices.length <= 10^(5)`
	
- `0 <= prices[i] <= 10^(4)`

We can directly apply the state transition equation and simplify it based on the base case:

```python
dp[i][1][0] = max(dp[i-1][1][0], dp[i-1][1][1] + prices[i])
dp[i][1][1] = max(dp[i-1][1][1], dp[i-1][0][0] - prices[i]) 
            = max(dp[i-1][1][1], -prices[i])
Explanation: From the base case where k = 0, we have dp[i-1][0][0] = 0.

Now we notice that k is always 1 and never changes, meaning k no longer affects the state transition.
We can simplify further by removing all k:
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], -prices[i])
```

Here's the direct code implementation:


```java
int n = prices.length;
int[][] dp = new int[n][2];
for (int i = 0; i < n; i++) {
    dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i-1][1], -prices[i]);
}
return dp[n - 1][0];
```


Obviously, when `i = 0`, `dp[i-1]` is an invalid index because we haven't handled the base case. The correct approach is to use the "index offset" technique, where `dp[0]` represents the base case before we start, and `dp[i]` represents the state on day `i - 1`:


```java
int n = prices.length;
// dp[0] represents base case, dp[i] represents day i-1
int[][] dp = new int[n + 1][2];
// base case: dp[0][0] = 0, dp[0][1] = -infinity
dp[0][0] = 0;
dp[0][1] = Integer.MIN_VALUE;

for (int i = 1; i <= n; i++) {
    // prices[i-1] is the stock price on day i-1
    dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i-1]);
    dp[i][1] = Math.max(dp[i-1][1], -prices[i-1]);
}
return dp[n][0];
```


This way, the base case is initialized once outside the loop, and the loop body doesn't need any conditional checks, making the code cleaner. Notice in the state transition equation that each new state only depends on one adjacent state. So we can use the space compression technique from [Space Compression in Dynamic Programming](https://labuladong.online/en/algo/dynamic-programming/space-optimization/). Instead of using the entire `dp` array, we only need a variable to store the adjacent state, reducing space complexity to O(1):


```java
// Original version
int maxProfit_k_1(int[] prices) {
    int n = prices.length;
    // dp[0] represents base case, dp[i] represents day i-1
    int[][] dp = new int[n + 1][2];
    dp[0][0] = 0;
    dp[0][1] = Integer.MIN_VALUE;

    for (int i = 1; i <= n; i++) {
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i-1]);
        dp[i][1] = Math.max(dp[i-1][1], -prices[i-1]);
    }
    return dp[n][0];
}

// Space complexity optimized version
int maxProfit_k_1(int[] prices) {
    int n = prices.length;
    // base case: dp[-1][0] = 0, dp[-1][1] = -infinity
    int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
    for (int i = 0; i < n; i++) {
        // dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
        dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
        // dp[i][1] = max(dp[i-1][1], -prices[i])
        dp_i_1 = Math.max(dp_i_1, -prices[i]);
    }
    return dp_i_0;
}
```


<visual slug="best-time-to-buy-and-sell-stock" />

Both approaches yield the same result, but the space-compressed version is much more concise. In the following problems, you can compare how to optimize away the `dp` array space.


### 122. Best Time to Buy and Sell Stock II

**The second problem, let's look at LeetCode problem 122 "[Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)", which is the case where `k` is positive infinity**:

**LeetCode 122. Best Time to Buy and Sell Stock II** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day.

On each day, you may decide to buy and/or sell the stock. You can only hold **at most one** share of the stock at any time. However, you can buy it then immediately sell it on the **same day**.

Find and return *the **maximum** profit you can achieve*.

Example 1:**

```

**Input:** prices = [7,1,5,3,6,4]
**Output:** 7
**Explanation:** Buy on day 2 (price = 1) and sell on day 3 (price = 5), profit = 5-1 = 4.
Then buy on day 4 (price = 3) and sell on day 5 (price = 6), profit = 6-3 = 3.
Total profit is 4 + 3 = 7.

```

Example 2:**

```

**Input:** prices = [1,2,3,4,5]
**Output:** 4
**Explanation:** Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
Total profit is 4.

```

Example 3:**

```

**Input:** prices = [7,6,4,3,1]
**Output:** 0
**Explanation:** There is no way to make a positive profit, so we never buy the stock to achieve the maximum profit of 0.

```

**Constraints:**

	
- `1 <= prices.length <= 3 * 10^(4)`
	
- `0 <= prices[i] <= 10^(4)`

The problem specifically emphasizes that you can sell on the same day, but I think this condition is redundant. If you buy and sell on the same day, the profit is obviously 0, which is the same as not making any transaction, right? The key characteristic of this problem is that there's no limit on the total number of transactions `k`, which is equivalent to `k` being positive infinity.

If `k` is positive infinity, then we can consider `k` and `k - 1` to be the same. We can rewrite the framework like this:

```python
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
            = max(dp[i-1][k][1], dp[i-1][k][0] - prices[i])

We find that k in the array no longer changes, which means we don't need to track the state k anymore:
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i])
```

Translating directly into code:


```java
// Original version
int maxProfit_k_inf(int[] prices) {
    int n = prices.length;
    int[][] dp = new int[n + 1][2];
    dp[0][0] = 0;
    dp[0][1] = Integer.MIN_VALUE;

    for (int i = 1; i <= n; i++) {
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i-1]);
        dp[i][1] = Math.max(dp[i-1][1], dp[i-1][0] - prices[i-1]);
    }
    return dp[n][0];
}

// Space complexity optimized version
int maxProfit_k_inf(int[] prices) {
    int n = prices.length;
    int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
    for (int i = 0; i < n; i++) {
        int temp = dp_i_0;
        dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
        dp_i_1 = Math.max(dp_i_1, temp - prices[i]);
    }
    return dp_i_0;
}
```


<visual slug="best-time-to-buy-and-sell-stock-ii" />


### 309. Best Time to Buy and Sell Stock with Cooldown

**The third problem, LeetCode problem 309 "[Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)", where `k` is positive infinity but with a cooldown period**:

**LeetCode 309. Best Time to Buy and Sell Stock with Cooldown** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day.

Find the maximum profit you can achieve. You may complete as many transactions as you like (i.e., buy one and sell one share of the stock multiple times) with the following restrictions:

	
- After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day).

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

Example 1:**

```

**Input:** prices = [1,2,3,0,2]
**Output:** 3
**Explanation:** transactions = [buy, sell, cooldown, buy, sell]

```

Example 2:**

```

**Input:** prices = [1]
**Output:** 0

```

**Constraints:**

	
- `1 <= prices.length <= 5000`
	
- `0 <= prices[i] <= 1000`

Similar to the previous problem, but after each `sell` you must wait one day before trading again. We just need to incorporate this feature into the state transition equation from the previous problem:

```python
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])
Explanation: When choosing to buy on day i, we transition from state i-2, not i-1.
```

Since the state transition requires `dp[i-2][0]`, we need to offset by 2 positions, letting both `dp[0]` and `dp[1]` represent the base case, and `dp[i]` represent the state of day `i - 2`:

```java
// Original version
int maxProfit_with_cool(int[] prices) {
    int n = prices.length;
    // dp[0], dp[1] represent base case, dp[i] represents day i-2
    int[][] dp = new int[n + 2][2];
    dp[0][0] = 0;
    dp[0][1] = Integer.MIN_VALUE;
    dp[1][0] = 0;
    dp[1][1] = Integer.MIN_VALUE;

    for (int i = 2; i <= n + 1; i++) {
        // prices[i-2] is the stock price on day i-2
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i-2]);
        dp[i][1] = Math.max(dp[i-1][1], dp[i-2][0] - prices[i-2]);
    }
    return dp[n + 1][0];
}

// Space complexity optimized version
int maxProfit_with_cool(int[] prices) {
    int n = prices.length;
    int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
    // represents dp[i-2][0]
    int dp_pre_0 = 0;
    for (int i = 0; i < n; i++) {
        int temp = dp_i_0;
        dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
        dp_i_1 = Math.max(dp_i_1, dp_pre_0 - prices[i]);
        dp_pre_0 = temp;
    }
    return dp_i_0;
}
```

<visual slug="best-time-to-buy-and-sell-stock-with-cooldown" />


### 714. Best Time to Buy and Sell Stock with Transaction Fee

**The fourth problem, LeetCode problem 714 "[Best Time to Buy and Sell Stock with Transaction Fee](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)", is the case where `k` is positive infinity with transaction fees**:

**LeetCode 714. Best Time to Buy and Sell Stock with Transaction Fee** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day, and an integer `fee` representing a transaction fee.

Find the maximum profit you can achieve. You may complete as many transactions as you like, but you need to pay the transaction fee for each transaction.

**Note:**

	
- You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).
	
- The transaction fee is only charged once for each stock purchase and sale.

Example 1:**

```

**Input:** prices = [1,3,2,8,4,9], fee = 2
**Output:** 8
**Explanation:** The maximum profit can be achieved by:
- Buying at prices[0] = 1
- Selling at prices[3] = 8
- Buying at prices[4] = 4
- Selling at prices[5] = 9
The total profit is ((8 - 1) - 2) + ((9 - 4) - 2) = 8.

```

Example 2:**

```

**Input:** prices = [1,3,7,5,10,3], fee = 3
**Output:** 6

```

**Constraints:**

	
- `1 <= prices.length <= 5 * 10^(4)`
	
- `1 <= prices[i] < 5 * 10^(4)`
	
- `0 <= fee < 5 * 10^(4)`

Each transaction requires paying a fee, so we just need to subtract the fee from the profit. The modified equations are:

```python
dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])
dp[i][1] = max(dp[i-1][1], dp[i-1][0] - prices[i] - fee)
Explanation: This is equivalent to increasing the buying price of the stock.
Subtracting in the first equation works too - it's equivalent to decreasing the selling price.
```

::: note Note

If you directly subtract `fee` in the first equation, some test cases will fail. The error is caused by integer overflow, not a logic problem. One solution is to change all `int` types in the code to `long` types to avoid integer overflow.

:::

Translating directly to code, note that when the state transition equation changes, the base case must also change accordingly:

```java
// Original version
int maxProfit_with_fee(int[] prices, int fee) {
    int n = prices.length;
    int[][] dp = new int[n + 1][2];
    dp[0][0] = 0;
    dp[0][1] = Integer.MIN_VALUE;

    for (int i = 1; i <= n; i++) {
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i-1]);
        dp[i][1] = Math.max(dp[i-1][1], dp[i-1][0] - prices[i-1] - fee);
    }
    return dp[n][0];
}

// Space complexity optimized version
int maxProfit_with_fee(int[] prices, int fee) {
    int n = prices.length;
    int dp_i_0 = 0, dp_i_1 = Integer.MIN_VALUE;
    for (int i = 0; i < n; i++) {
        int temp = dp_i_0;
        dp_i_0 = Math.max(dp_i_0, dp_i_1 + prices[i]);
        dp_i_1 = Math.max(dp_i_1, temp - prices[i] - fee);
    }
    return dp_i_0;
}
```

<visual slug="best-time-to-buy-and-sell-stock-with-transaction-fee" />


### 123. Best Time to Buy and Sell Stock III

**The fifth problem, LeetCode problem 123 "[Best Time to Buy and Sell Stock III](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/)", is the case where `k = 2`**:

**LeetCode 123. Best Time to Buy and Sell Stock III** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

You are given an array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day.

Find the maximum profit you can achieve. You may complete **at most two transactions**.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

Example 1:**

```

**Input:** prices = [3,3,5,0,0,3,1,4]
**Output:** 6
**Explanation:** Buy on day 4 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.
Then buy on day 7 (price = 1) and sell on day 8 (price = 4), profit = 4-1 = 3.
```

Example 2:**

```

**Input:** prices = [1,2,3,4,5]
**Output:** 4
**Explanation:** Buy on day 1 (price = 1) and sell on day 5 (price = 5), profit = 5-1 = 4.
Note that you cannot buy on day 1, buy on day 2 and sell them later, as you are engaging multiple transactions at the same time. You must sell before buying again.

```

Example 3:**

```

**Input:** prices = [7,6,4,3,1]
**Output:** 0
**Explanation:** In this case, no transaction is done, i.e. max profit = 0.

```

**Constraints:**

	
- `1 <= prices.length <= 10^(5)`
	
- `0 <= prices[i] <= 10^(5)`

The case `k = 2` is slightly different from the previous problems, because those cases didn't really depend on `k`: either `k` was infinity and the state transition had nothing to do with `k`; or `k = 1`, which was close to the base case `k = 0`, and ultimately had no impact.

In this problem where `k = 2` and the upcoming case where `k` is any positive integer, the handling of `k` becomes important. Let's write the code and analyze the reasons as we go.

```python
The original state transition equation, with no simplification possible
dp[i][k][0] = max(dp[i-1][k][0], dp[i-1][k][1] + prices[i])
dp[i][k][1] = max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i])
```

Following the previous code, we might naturally write code like this (incorrect):


```java
int k = 2;
int[][][] dp = new int[n][k + 1][2];
for (int i = 0; i < n; i++) {
    if (i - 1 == -1) {
        // handle the base case
        dp[i][k][0] = 0;
        dp[i][k][1] = -prices[i];
        continue;
    }
    dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]);
    dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i]);
}
return dp[n - 1][k][0];
```


Why is this wrong? Didn't I write it according to the state transition equation?

Remember the "brute-force framework" we summarized earlier? It says we must enumerate all states. Actually, our previous solutions were enumerating all states, but in those problems `k` was simplified away.

For example, in the first problem, the code framework when `k = 1`:


```java
int n = prices.length;
int[][] dp = new int[n][2];
for (int i = 0; i < n; i++) {
    dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i]);
    dp[i][1] = Math.max(dp[i-1][1], -prices[i]);
}
return dp[n - 1][0];
```

But when `k = 2`, since the effect of `k` is not eliminated, we must enumerate `k`:

```java
// Original version
int maxProfit_k_2(int[] prices) {
    int max_k = 2, n = prices.length;
    int[][][] dp = new int[n + 1][max_k + 1][2];
    // base case
    for (int k = 0; k <= max_k; k++) {
        dp[0][k][0] = 0;
        dp[0][k][1] = Integer.MIN_VALUE;
    }

    for (int i = 1; i <= n; i++) {
        for (int k = max_k; k >= 1; k--) {
            dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i-1]);
            dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i-1]);
        }
    }
    // enumerated n × max_k × 2 states, correct
    return dp[n][max_k][0];
}

// Space complexity optimized version
int maxProfit_k_2(int[] prices) {
    // base case
    int dp_i10 = 0, dp_i11 = Integer.MIN_VALUE;
    int dp_i20 = 0, dp_i21 = Integer.MIN_VALUE;
    for (int price : prices) {
        dp_i20 = Math.max(dp_i20, dp_i21 + price);
        dp_i21 = Math.max(dp_i21, dp_i10 - price);
        dp_i10 = Math.max(dp_i10, dp_i11 + price);
        dp_i11 = Math.max(dp_i11, -price);
    }
    return dp_i20;
}
```

<visual slug="best-time-to-buy-and-sell-stock-iii" />


::: note Note

**Some readers might be confused here: the base case for `k` is 0, so logically shouldn't we enumerate state `k` starting from `k = 1, k++`? And if you actually traverse `k` from small to large like this, submitting it will also be accepted**.

:::

This question is valid, because in our previous article [Dynamic Programming Q&A](https://labuladong.online/en/algo/dynamic-programming/faq-summary/), we introduced how to determine the traversal order of the `dp` array, which is mainly based on the base case, starting from the base case and gradually approaching the result.

But why can I also get accepted by traversing `k` from large to small? Notice that `dp[i][k][..]` doesn't depend on `dp[i][k - 1][..]`, but depends on `dp[i - 1][k - 1][..]`, and `dp[i - 1][..][..]` has already been computed. So whether you use `k = max_k, k--` or `k = 1, k++`, you can get the correct answer.

Then why do I use the `k = max_k, k--` approach? Because it aligns with the semantics:

When you buy stocks, what's the initial "state"? It should start from day 0, with no trades made yet, so the maximum transaction limit `k` should be `max_k`. As the "state" progresses, you make trades, so the transaction limit `k` should decrease. Thinking this way, the `k = max_k, k--` approach is more realistic.

Of course, since `k` has a small range here, we can also skip the for loop and directly list out all cases for k = 1 and 2. The space-optimized version of the above code does exactly this. With the state transition equation and clearly named variables as guidance, you should easily understand it. Actually, we could be mysterious and replace those four variables with `a, b, c, d`. Then when others see your code, they'll be amazed and look at you with great respect.


### 188. Best Time to Buy and Sell Stock IV

The sixth problem is LeetCode problem 188 "[Best Time to Buy and Sell Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)", where `k` can be any number given by the problem:

**LeetCode 188. Best Time to Buy and Sell Stock IV** <span class="inline-block w-2 h-2 rounded-full bg-red-500"></span>

You are given an integer array `prices` where `prices[i]` is the price of a given stock on the `i^(th)` day, and an integer `k`.

Find the maximum profit you can achieve. You may complete at most `k` transactions: i.e. you may buy at most `k` times and sell at most `k` times.

**Note:** You may not engage in multiple transactions simultaneously (i.e., you must sell the stock before you buy again).

Example 1:**

```

**Input:** k = 2, prices = [2,4,1]
**Output:** 2
**Explanation:** Buy on day 1 (price = 2) and sell on day 2 (price = 4), profit = 4-2 = 2.

```

Example 2:**

```

**Input:** k = 2, prices = [3,2,6,5,0,3]
**Output:** 7
**Explanation:** Buy on day 2 (price = 2) and sell on day 3 (price = 6), profit = 6-2 = 4. Then buy on day 5 (price = 0) and sell on day 6 (price = 3), profit = 3-0 = 3.

```

**Constraints:**

	
- `1 <= k <= 100`
	
- `1 <= prices.length <= 1000`
	
- `0 <= prices[i] <= 1000`

With the foundation from the previous problem where `k = 2`, this problem should be no different from the first solution of that problem. You just need to replace `k = 2` with the input `k` from the problem.

But if you try it, you'll get a memory limit exceeded error. This is because the input `k` can be very large, making the `dp` array too big. So let's think about it: what's the maximum meaningful value for the number of transactions `k`?

One transaction consists of buying and selling, which requires at least two days. So the effective limit `k` should not exceed `n/2`. If it does, the constraint has no effect, which is equivalent to the case where `k` has no limit - and we've already solved that case before.

So we can directly reuse our previous code:


```java
int maxProfit_k_any(int max_k, int[] prices) {
    int n = prices.length;
    if (n <= 0) {
        return 0;
    }
    if (max_k > n / 2) {
        // reuse maxProfit_k_inf function
        return maxProfit_k_inf(prices);
    }

    int[][][] dp = new int[n + 1][max_k + 1][2];
    // base case
    for (int k = 0; k <= max_k; k++) {
        dp[0][k][0] = 0;
        dp[0][k][1] = Integer.MIN_VALUE;
    }

    for (int i = 1; i <= n; i++) {
        for (int k = max_k; k >= 1; k--) {
            dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i-1]);
            dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i-1][k-1][0] - prices[i-1]);
        }
    }
    return dp[n][max_k][0];
}
```

<visual slug="best-time-to-buy-and-sell-stock-iv" />


With this, all 6 problems are solved using a single state transition equation.


## All Methods Return to One

If you've made it this far, give yourself a round of applause. Understanding such complex dynamic programming problems for the first time must have consumed quite a few brain cells. But it's worth it—the stock trading series is among the more difficult dynamic programming problems. If you can figure out all these problems, what else could possibly stand in your way?

**Now you've passed 80 of the 81 trials. Let me challenge you one more time—please implement the following function**:

```java
int maxProfit_all_in_one(int max_k, int[] prices, int cooldown, int fee);
```

Given a stock price array `prices`, you can make at most `max_k` transactions. Each transaction costs an additional `fee` as transaction fee, and after each transaction you must wait `cooldown` days before making the next transaction. Calculate and return the maximum profit you can achieve.

Scary, right? If you gave someone this problem directly, they'd probably pass out on the spot. But having worked through it step by step, you should easily see that this problem is just a combination of the cases we've discussed before.

So we just need to mix together the code we implemented earlier, **adding the `cooldown` and `fee` constraints to both the base case and the state transition equation**.

Since state transition needs `dp[i-cooldown-1]`, we need to offset by `cooldown + 1` positions:


```java
// Consider the limit on the number of transactions, cooldown period, and transaction fees simultaneously
int maxProfit_all_in_one(int max_k, int[] prices, int cooldown, int fee) {
    int n = prices.length;
    if (n <= 0) {
        return 0;
    }
    if (max_k > n / 2) {
        // The case where the number of transactions k is unlimited
        return maxProfit_k_inf(prices, cooldown, fee);
    }

    // offset is cooldown + 1, ensuring access to dp[i - cooldown - 1]
    int offset = cooldown + 1;
    int[][][] dp = new int[n + offset][max_k + 1][2];
    // base case: first offset rows are all base cases
    for (int i = 0; i < offset; i++) {
        for (int k = 0; k <= max_k; k++) {
            dp[i][k][0] = 0;
            dp[i][k][1] = Integer.MIN_VALUE;
        }
    }

    for (int i = offset; i < n + offset; i++) {
        for (int k = max_k; k >= 1; k--) {
            // prices[i - offset] is the stock price on day i-offset
            dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i - offset]);
            // Consider both cooldown and fee simultaneously
            dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i - cooldown - 1][k-1][0] - prices[i - offset] - fee);
        }
    }
    return dp[n + offset - 1][max_k][0];
}

// k is unlimited, including transaction fees and cooldown period
int maxProfit_k_inf(int[] prices, int cooldown, int fee) {
    int n = prices.length;
    int offset = cooldown + 1;
    int[][] dp = new int[n + offset][2];
    // base case: first offset rows are all base cases
    for (int i = 0; i < offset; i++) {
        dp[i][0] = 0;
        dp[i][1] = Integer.MIN_VALUE;
    }

    for (int i = offset; i < n + offset; i++) {
        dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i - offset]);
        // Consider both cooldown and fee simultaneously
        dp[i][1] = Math.max(dp[i-1][1], dp[i - cooldown - 1][0] - prices[i - offset] - fee);
    }
    return dp[n + offset - 1][0];
}
```


You can use this `maxProfit_all_in_one` function to solve all 6 problems we discussed earlier. Since we can't optimize the `dp` array, the execution efficiency isn't optimal, but the correctness is guaranteed.

Let me wrap up. This article showed you how to solve complex problems through state transition, crushing 6 stock trading problems with a single state transition equation. Looking back now, it's not that scary, right?

The key is to enumerate all possible "states", then think about how to brute-force update these "states". We typically use a multi-dimensional `dp` array to store these states, starting from the base case and moving forward until we reach the final state—which is the answer we want. Thinking about this process, do you now understand what "dynamic programming" really means?

For the stock trading problem specifically, we identified three states and used a three-dimensional array. It's still just brute-force + update, but we can make it sound fancy—this is called "3D DP". Sounds impressive, doesn't it?
