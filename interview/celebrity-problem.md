::: info Prerequisite

Before reading this article, you should first learn:

- [Graph Basics and Common Implementations](https://labuladong.online/en/algo/data-structure-basic/graph-basic/)

:::

Today, let's talk about the classic "celebrity problem":

You are given the social relationships of `n` people (you know whether any two people know each other), and you need to find the "celebrity" among them.

A "celebrity" must meet two conditions:

1. Everyone else knows the celebrity.

2. The celebrity knows no one else.

This is a graph algorithm problem. Social relationships can be modeled as a graph.

If each person is a node, and "knows" is a directed edge from one node to another, then the celebrity is a special node in this graph:

![](../pictures/celebrity/1.jpeg)

**This node has no outgoing edges to others, but every other node has an edge pointing to it.**

In technical terms, the celebrity node has out-degree 0 and in-degree `n - 1`.

How do we represent the social relationships of these `n` people?

In the previous article [Graph Algorithms Basics](https://labuladong.online/en/algo/data-structure-basic/graph-basic/), we said that there are two main ways to store a graph: adjacency list and adjacency matrix. Adjacency lists save space; adjacency matrices make it fast to check if two nodes are connected.

For the celebrity problem, we often need to check if two people know each other (i.e., if two nodes are connected), so we can use an adjacency matrix to represent the relationships.

So, the algorithm problem can be described like this:

You are given an `n x n` matrix `graph` representing a graph with `n` nodes. Each person is a node, numbered from `0` to `n - 1`.

If `graph[i][j] == 1`, person `i` knows person `j`. If `graph[i][j] == 0`, person `i` does not know person `j`.

Given this graph, can you find out if there is a "celebrity" among the `n` people?

If there is, return the celebrity's number. If not, return -1.

The function signature is as follows:

```java
int findCelebrity(int[][] graph);
```

For example, suppose the input matrix looks like this:

![](../pictures/celebrity/2.jpeg)

Then the algorithm should return 2.

LeetCode Problem 277 "[Find the Celebrity](https://leetcode.com/problems/find-the-celebrity/)" is this classic problem. But instead of giving you the adjacency matrix directly, it only tells you the total number of people `n`, and provides an API `knows` to check if one person knows another:

```java
// can be called directly, returns whether i knows j
boolean knows(int i, int j);

// please implement: return the number of the "celebrity"
int findCelebrity(int n) {
    // todo
}
```

Basically, the `knows` API is just a way to access the adjacency matrix. To keep things simple, let's follow the LeetCode way to discuss this classic problem.

## Brute-force Solution

We can easily write a simple brute-force algorithm:

```java
class Solution extends Relation {
    public int findCelebrity(int n) {
        for (int cand = 0; cand < n; cand++) {
            int other;
            for (other = 0; other < n; other++) {
                if (cand == other) continue;
                // ensure that everyone else knows cand, and cand does not know anyone else
                // otherwise, cand cannot be the celebrity
                if (knows(cand, other) || !knows(other, cand)) {
                    break;
                }
            }
            if (other == n) {
                // found the celebrity
                return cand;
            }
        }
        // no one fits the characteristics of a celebrity
        return -1;
    }
}
```

`cand` stands for candidate. The brute-force idea is to check every person as a candidate and see if they meet the "celebrity" conditions.

As mentioned above, the `knows` function just accesses the adjacency matrix, and each call takes O(1) time. So the brute-force solution has a worst-case time complexity of O(N^2).

Is there a smarter way to optimize this? Actually, yes. Think about it: where do we spend the most time?

For every candidate `cand`, we use an inner for loop to check if `cand` really is a celebrity.

This inner loop seems silly. While checking if someone "is a celebrity" needs a loop, checking if someone "is not a celebrity" can be done more quickly.

**Since the definition of "celebrity" guarantees there can be at most one, we can use elimination: first rule out people who are obviously not celebrities. This avoids nested loops and reduces the time complexity.**


## Optimized Solution

Let me repeat the definition of a "celebrity":

1. Everyone else knows the celebrity.
2. The celebrity does not know anyone else.

This is interesting because it means there can be at most one celebrity in the group.

It is easy to understand. If there are two celebrities at the same time, the two rules would be in conflict.

**In other words, if you look at the relationship between any two candidates, you can always rule out one person as not being the celebrity.**

We cannot tell if the other person is the celebrity by just looking at this pair, but that's not important. The key is we can always remove one who cannot be the celebrity, so we reduce our search.

This is the main idea behind the optimization and can be hard to understand. Let's see why checking any two candidates lets us rule out one.

Think about it. What are the possible relationships between two people?

There are only four cases: you know me but I don’t know you, I know you but you don’t know me, we both know each other, or we both don’t know each other.

If we think of people as nodes, a red arrow means “does not know,” and a green arrow means “knows.” The relationships look like these four cases:

![](../pictures/celebrity/3.jpeg)

Let’s say the two people are called `cand` and `other`. Let’s look at each case and see who we can rule out.

Case 1: `cand` knows `other`, so `cand` can’t be the celebrity. The celebrity should not know anyone.

Case 2: `other` knows `cand`, so `other` can’t be the celebrity.

Case 3: They both know each other. Neither is the celebrity, so we can rule out either one.

Case 4: Neither knows the other. Again, neither is the celebrity, so we can rule out either one. The celebrity should be known by everyone.

So, by checking the relationship between any two candidates, we can always rule out at least one. We can use code like this to decide:

```java
if (knows(cand, other) || !knows(other, cand)) {
    // cand cannot be a celebrity
} else {
    // other cannot be a celebrity
}
```

If you understand this, the optimized solution is easy.

**We keep picking two candidates, compare them, and remove one, until there is only one left. Then we use a for loop to check if this person is really the celebrity.**

Here is the full code for this idea:

```java
class Solution extends Relation {
    public int findCelebrity(int n) {
        if (n == 1) return 0;
        // put all candidates into the queue
        LinkedList<Integer> q = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            q.addLast(i);
        }
        // keep eliminating until only one candidate is left
        while (q.size() >= 2) {
            // each time take out two candidates and eliminate one
            int cand = q.removeFirst();
            int other = q.removeFirst();
            if (knows(cand, other) || !knows(other, cand)) {
                // cand cannot be the celebrity, eliminate and let other rejoin the queue
                q.addFirst(other);
            } else {
                // other cannot be the celebrity, eliminate and let cand rejoin the queue
                q.addFirst(cand);
            }
        }

        // now there is only one candidate left, determine if he is really the celebrity
        int cand = q.removeFirst();
        for (int other = 0; other < n; other++) {
            if (other == cand) {
                continue;
            }
            // ensure that everyone else knows cand, and cand does not know anyone else
            if (!knows(other, cand) || knows(cand, other)) {
                return -1;
            }
        }
        // cand is the celebrity
        return cand;
    }
}
```

This algorithm avoids nested for loops. The time complexity is O(N). But we use a queue to store candidates, so space complexity is O(N).

::: note Note

`LinkedList` is just used as a container for the candidates. Each time, we pick two people to compare and remove one. It does not matter which two you pick, or the order they are added to the queue. We use `addFirst` for convenience, but you can also use `addLast`. The result is the same.

:::

Can we do better and also reduce the space complexity?

## Final Solution

If you understand the optimized solution above, you can solve the problem without extra space. Here is the code:

```java
class Solution extends Relation {
    public int findCelebrity(int n) {
        int cand = 0;
        for (int other = 1; other < n; other++) {
            if (!knows(other, cand) || knows(cand, other)) {
                // cand cannot be a celebrity, eliminate
                // assume other is a celebrity
                cand = other;
            } else {
                // other cannot be a celebrity, eliminate
                // do nothing, continue to assume cand is a celebrity
            }
        }

        // now cand is the last one remaining after elimination, but it's not guaranteed to be a celebrity
        for (int other = 0; other < n; other++) {
            if (cand == other) continue;
            // need to ensure everyone else knows cand, and cand does not know any other person
            if (!knows(other, cand) || knows(cand, other)) {
                return -1;
            }
        }

        return cand;
    }
}
```

Before, we used a `LinkedList` as a queue to store the candidates. In this improved solution, we use the change between `other` and `cand` to simulate the queue process, so we don’t need extra storage.

Now, our solution for the celebrity problem has time complexity O(N) and space complexity O(1). This is the best solution.
