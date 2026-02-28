::: info Prerequisite Knowledge

Before reading this article, you should first learn:

- [Core Framework of Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/)

:::

In the previous article [A Few Brain Teasers](https://labuladong.online/en/algo/frequency-interview/one-line-solutions/), an interesting "Stone Game" was discussed, where, due to the constraints of the problem, the first player is guaranteed to win. However, brain teasers are ultimately just brain teasers; real algorithm problems cannot be solved through trickery. Therefore, this article uses the Stone Game to discuss how to solve problems of the type "assuming both players are smart, who will win in the end" using dynamic programming algorithms.

The approach to game theory problems is generally similar. The explanation below references [this YouTube video](https://www.youtube.com/watch?v=WxpIHvsu1RI), where the core idea is to use tuples to store the results of both players' games on the basis of 2D dynamic programming. Once you master this technique, when someone asks you about two pirates dividing gems or two people taking coins, you can simply say: "I’m too lazy to think, I'll just write an algorithm to calculate it."

We modify LeetCode problem 877, "[Stone Game](https://leetcode.com/problems/stone-game/)", to be more general:

You and your friend have a row of stone piles represented by an array `piles`, where `piles[i]` represents the number of stones in the `i-th` pile. You take turns picking stones, one pile at a time, but can only take the leftmost or rightmost pile. Whoever has more stones after all are taken wins.

The number of piles and the total number of stones can be any positive integer, thus breaking the first-player-win scenario. For example, with three piles `piles = [1, 100, 3]`, the first player, whether taking 1 or 3, will allow the second player to take the decisive 100, resulting in a win for the second player.

**Assuming both players are very smart**, write a `stoneGame` function that returns the difference between the final scores (total stones) of the first and second players. For example, in the case above, the first player can get 4 points, and the second player can get 100 points, so your algorithm should return -96:

```java
int stoneGame(int[] nums);
```

This generalization becomes a rather challenging dynamic programming problem. LeetCode problem 486, "[Predict the Winner](https://leetcode.com/problems/predict-the-winner/)", is a similar problem:

**LeetCode 486. Predict the Winner** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

You are given an integer array `nums`. Two players are playing a game with this array: player 1 and player 2.

Player 1 and player 2 take turns, with player 1 starting first. Both players start the game with a score of `0`. At each turn, the player takes one of the numbers from either end of the array (i.e., `nums[0]` or `nums[nums.length - 1]`) which reduces the size of the array by `1`. The player adds the chosen number to their score. The game ends when there are no more elements in the array.

Return `true` if Player 1 can win the game. If the scores of both players are equal, then player 1 is still the winner, and you should also return `true`. You may assume that both players are playing optimally.

Example 1:**

```

**Input:** nums = [1,5,2]
**Output:** false
**Explanation:** Initially, player 1 can choose between 1 and 2. 
If he chooses 2 (or 1), then player 2 can choose from 1 (or 2) and 5. If player 2 chooses 5, then player 1 will be left with 1 (or 2). 
So, final score of player 1 is 1 + 2 = 3, and player 2 is 5. 
Hence, player 1 will never be the winner and you need to return false.

```

Example 2:**

```

**Input:** nums = [1,5,233,7]
**Output:** true
**Explanation:** Player 1 first chooses 1. Then player 2 has to choose between 5 and 7. No matter which number player 2 choose, player 1 can choose 233.
Finally, player 1 has more score (234) than player 2 (12), so you need to return True representing player1 can win.

```

**Constraints:**

	
- `1 <= nums.length <= 20`
	
- `0 <= nums[i] <= 10^(7)`

The function signature is as follows:

```java
boolean predictTheWinner(int[] nums);
```

If you have a `stoneGame` function that calculates the score difference between the first and second players, the solution to this problem is straightforward:

```java
boolean predictTheWinner(int[] nums) {
    // If the first player's score is greater than or equal to the second player's, they can win
    return stoneGame(nums) >= 0;
}
```


How should the `stoneGame` function be written? The challenge in game theory problems lies in the fact that two players take turns making choices, and both are extremely clever. How can we represent this process in programming? It's not difficult; just follow the routine emphasized multiple times in the [Core Framework of Dynamic Programming](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/). First, clarify the meaning of the `dp` array, and then as long as you identify the "state" and "choice," everything will fall into place.

## 1. Define the Meaning of the `dp` Array

Defining the meaning of the `dp` array is quite technical. The same problem can have multiple definitions, leading to different state transition equations. However, as long as the logic is correct, the same answer can be obtained in the end.

I recommend not pursuing overly intricate solution ideas. It's better to be stable and adopt a solution that is most interpretable and easiest to generalize. This article provides a general design framework for game theory problems.

Before introducing the meaning of the `dp` array, let's take a look at the final form of the `dp` array:

![](../pictures/stone-game/1.png)

In the following explanation, assume that a tuple is a class containing `first` and `second` attributes, and for brevity, these two attributes are abbreviated as `fir` and `sec`. For example, with the data in the above image, we say `dp[1][3].fir = 11`, `dp[0][1].sec = 2`.

Let's answer some questions readers might have:

How to represent the tuple stored in this two-dimensional dp table in programming? How to optimize since half of this dp table is unused? It's simple: don't worry about it for now. First, understand the problem-solving approach, and discuss optimization later.

**Explanation of the Meaning of the `dp` Array:**

`dp[i][j].fir = x` means that for the stone piles `piles[i...j]`, the highest score the first player can achieve is `x`.

`dp[i][j].sec = y` means that for the stone piles `piles[i...j]`, the highest score the second player can achieve is `y`.

To understand with an example, assume `piles = [2, 8, 3, 5]`, with indexing starting from 0:

`dp[0][1].fir = 8` means that facing stone piles `[2, 8]`, the first player can obtain at most 8 points; `dp[1][3].sec = 5` means that facing stone piles `[8, 3, 5]`, the second player can obtain at most 5 points.

The answer we seek is the difference in final scores between the first and second players, according to this definition it is `dp[0][n-1].fir - dp[0][n-1].sec`, which is the difference between the optimal scores of the first and second players when facing the entire `piles`.


## 2. State Transition Equation

Writing the state transition equation is straightforward. First, identify all the "states" and the "choices" available in each state, then choose optimally.

According to the definition of the `dp` array, **there are evidently three states: starting index `i`, ending index `j`, and whose turn it is.**

```python
dp[i][j][fir or sec]
where:
0 <= i < piles.length
i <= j < piles.length
```

For each state in this problem, there are **two choices: choose the leftmost pile of stones, or choose the rightmost pile of stones.** We can enumerate all states as follows:

```python
n = piles.length
for 0 <= i < n:
    for j <= i < n:
        for who in {fir, sec}:
            dp[i][j][who] = max(left, right)
```

The pseudocode above outlines a general framework for dynamic programming. The challenge in this problem is that both players are smart and alternate in making choices, meaning the first player's choice affects the second player. How do we express this?

Based on our definition of the `dp` array, we can easily address this challenge and **write the state transition equation**:

```python
dp[i][j].fir = max(piles[i] + dp[i+1][j].sec, piles[j] + dp[i][j-1].sec)
dp[i][j].fir = max(     choosing the leftmost pile     ,     choosing the rightmost pile      )
# Explanation: As the first player, when facing piles[i...j], I have two choices:

# Either I choose the leftmost pile piles[i], changing the situation to piles[i+1...j],
# then it's the opponent's turn, and I become the second player. My optimal score as the second player is dp[i+1][j].sec

# Or I choose the rightmost pile piles[j], changing the situation to piles[i...j-1],
# then it's the opponent's turn, and I become the second player. My optimal score as the second player is dp[i][j-1].sec

if the first player chooses left:
    dp[i][j].sec = dp[i+1][j].fir
if the first player chooses right:
    dp[i][j].sec = dp[i][j-1].fir
# Explanation: As the second player, I have to wait for the first player to choose, with two scenarios:

# If the first player chooses the leftmost pile, leaving me with piles[i+1...j],
# now it's my turn, and I become the first player. My optimal score is dp[i+1][j].fir

# If the first player chooses the rightmost pile, leaving me with piles[i...j-1],
# now it's my turn, and I become the first player. My optimal score is dp[i][j-1].fir
```

Based on the definition of the dp array, we can also identify the **base case**, which is the simplest scenario:

```python
dp[i][j].fir = piles[i]
dp[i][j].sec = 0
where 0 <= i == j < n
# Explanation: i and j being equal means there's only one pile of stones piles[i] in front.
# Obviously, the first player's score is piles[i]
# The second player has no stones to take, so the score is 0
```

![](../pictures/stone-game/2.png)

One thing to note here is that we find the base case is diagonal, and when calculating `dp[i][j]`, we need `dp[i+1][j]` and `dp[i][j-1]`:

![](../pictures/stone-game/3.png)

According to the principle of determining the traversal direction of the `dp` array from the previous article [Dynamic Programming Q&A](https://labuladong.online/en/algo/dynamic-programming/faq-summary/), the algorithm should traverse the `dp` array backwards:


```java
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        dp[i][j] = ...
    }
}
```

![](../pictures/stone-game/4.png)

## 3. Code Implementation

How to implement the `fir` and `sec` tuples? You can use Python's built-in tuple type, or use C++'s `pair` container, or employ a three-dimensional array `dp[n][n][2]`, where the last dimension acts as a tuple. Alternatively, we can write a custom `Pair` class:

```java
class Pair {
    int fir, sec;
    Pair(int fir, int sec) {
        this.fir = fir;
        this.sec = sec;
    }
}
```

Then, directly translate our state transition equation into code, remembering to iterate the array in reverse:

```java
// Return the score difference between the first and second player at the end of the game
int stoneGame(int[] piles) {
    int n = piles.length;
    // Initialize the dp array
    Pair[][] dp = new Pair[n][n];
    for (int i = 0; i < n; i++) 
        for (int j = i; j < n; j++)
            dp[i][j] = new Pair(0, 0);
    // Fill in the base case
    for (int i = 0; i < n; i++) {
        dp[i][i].fir = piles[i];
        dp[i][i].sec = 0;
    }

    // Traverse the array in reverse order
    for (int i = n - 2; i >= 0; i--) {
        for (int j = i + 1; j < n; j++) {
            // The first player chooses the score from the leftmost or rightmost
            int left = piles[i] + dp[i+1][j].sec;
            int right = piles[j] + dp[i][j-1].sec;
            // Apply the state transition equation
            // The first player will definitely choose the larger result, and the second player's choice will change accordingly
            if (left > right) {
                dp[i][j].fir = left;
                dp[i][j].sec = dp[i+1][j].fir;
            } else {
                dp[i][j].fir = right;
                dp[i][j].sec = dp[i][j-1].fir;
            }
        }
    }
    Pair res = dp[0][n-1];
    return res.fir - res.sec;
}
```

Dynamic programming solutions can be quite confusing without the guidance of state transition equations. However, with the detailed explanation provided earlier, readers should clearly understand the essence of this code segment.

Moreover, note that calculating `dp[i][j]` only depends on elements to its left and below, indicating room for optimization by transforming it into a one-dimensional `dp`. Imagine flattening the two-dimensional plane, projecting it onto one dimension. However, one-dimensional `dp` is complex and less interpretable, so you need not spend time understanding it.

## 4. Conclusion

This article presents a dynamic programming solution for solving game problems. The premise of game problems generally involves two intelligent individuals. The typical method for describing such games in programming is using a two-dimensional `dp` array, where tuples in the array represent each participant's optimal decisions.

The reason for such design is that after the first player makes a choice, they become the second player, and after the second player completes their choice, they become the first player. **This role reversal allows us to reuse previous results, a hallmark of dynamic programming**.

Readers at this point should understand the algorithmic approach to solving game problems. When learning algorithms, focus on the template framework rather than seemingly sophisticated ideas, and don't expect to write an optimal solution immediately. Don't hesitate to use more space, avoid premature optimization, and don't fear multidimensional arrays. The `dp` array is for storing information to avoid redundant calculations. Use it freely until you are satisfied.
