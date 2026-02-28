::: info Prerequisite Knowledge

Before reading this article, you should first learn:

- [Graph Basics and General Implementation](https://labuladong.online/en/algo/data-structure-basic/graph-basic/)
- [Overview of Shortest Path Algorithms in Graphs](https://labuladong.online/en/algo/data-structure-basic/graph-shortest-path/)
- [DFS/BFS Traversal in Graph Structures](https://labuladong.online/en/algo/data-structure-basic/graph-traverse-basic/)

:::

::: important Summary in One Sentence

Dijkstra's algorithm is used to find the shortest path from one node to all others in a graph. **It is basically the standard BFS algorithm plus a greedy idea.**

If the graph has negative weight edges, the greedy idea will not work, so **Dijkstra's algorithm can only handle graphs without negative weight edges**.

There are only two differences between Dijkstra's algorithm and standard BFS:

1. Standard BFS uses a regular queue. Dijkstra uses a priority queue, so nodes closer to the start are processed first (this is the greedy idea).
   
2. Standard BFS uses a `visited` array to record visited nodes to avoid getting stuck in a loop. Dijkstra uses a `distTo` array which also avoids loops and saves the shortest path from the start to every node.

:::

In [Overview of Shortest Path Algorithms in Graphs](https://labuladong.online/en/algo/data-structure-basic/graph-shortest-path/), we gave a brief introduction to shortest path problems, the difference between single-source and multi-source shortest paths, the problem with negative weight edges, and some classic shortest path algorithms including Dijkstra. In this article, we will cover:

1. How to build Dijkstra's algorithm based on the standard [Graph BFS Traversal Algorithm](https://labuladong.online/en/algo/data-structure-basic/graph-traverse-basic/).

2. Detailed explanation of how Dijkstra's algorithm works and why it is correct.

3. The shortest path between two points is a special case of the single-source shortest path. We can slightly change Dijkstra's algorithm to make it faster.

After you understand the logic and code, the next article will show how to change standard Dijkstra code to solve harder shortest path problems with more conditions.

We will use the graph interfaces defined in [Graph Basics and General Implementation](https://labuladong.online/en/algo/data-structure-basic/graph-basic/). Please read that part first to know how graphs are stored and defined.

Let's start.

## Dijkstra Algorithm Code

Let's start with the code. We only need to make a few changes to the standard BFS algorithm to get Dijkstra, so it's simple. We will first see what changes are needed, then explain why they work.

In [Graph BFS Traversal Algorithm](https://labuladong.online/en/algo/data-structure-basic/graph-traverse-basic/), we showed three ways to write BFS. The third one is best for Dijkstra, because we need to track the shortest path from the start to each visited node.

Here is the standard graph BFS traversal algorithm:

```java
// BFS traversal of a graph, starting from node s and recording the number of steps (the number of edges from the start node s to the current node)
// Each node maintains its own State class to record the number of steps from s
class State {
    // Current node ID
    int node;
    // Number of steps from the start node s to the current node
    int step;

    public State(int node, int step) {
        this.node = node;
        this.step = step;
    }
}


void bfs(Graph graph, int s) {
    boolean[] visited = new boolean[graph.size()];
    Queue<State> q = new LinkedList<>();

    q.offer(new State(s, 0));
    visited[s] = true;

    while (!q.isEmpty()) {
        State state = q.poll();
        int cur = state.node;
        int step = state.step;
        System.out.println("visit " + cur + " with step " + step);
        for (Edge e : graph.neighbors(cur)) {
            if (visited[e.to]) { // [!code highlight:5]
                continue;
            }
            q.offer(new State(e.to, step + 1));
            visited[e.to] = true;
        }
    }
}
```

In this algorithm, we use `State.step` to record the least number of steps (edges) from the start to the current node, and use a `visited` array so each node is visited only once (enters and leaves the queue only once), to avoid loops.

In a weighted graph, the shortest path problem wants to find the smallest total weight from the start to each node. Because edges can have different weights, the least step count used above does not help and cannot give the path with the smallest total weight.

Here is the code for Dijkstra's algorithm. The differences are highlighted below:


<!-- hide -->

```java
class State {
    // Current node ID
    int node;
    // Minimum path weight from the start node s to the current node
    int distFromStart;

    public State(int node, int distFromStart) {
        this.node = node;
        this.distFromStart = distFromStart;
    }
}

// Input the weighted graph (without negative weight edges) graph and the starting node src
// Return the minimum path weight from the starting node src to other nodes
int[] dijkstra(Graph graph, int src) {
    // Record the minimum path weight from the starting node src to other nodes
    // distTo[i] represents the minimum path weight from the starting node src to node i
    int[] distTo = new int[graph.size()];
    // Initialize all values to positive infinity, representing not calculated
    Arrays.fill(distTo, Integer.MAX_VALUE);

    // Priority queue, nodes with smaller distFromStart are in front
    Queue<State> pq = new PriorityQueue<>((a, b) -> {
        return a.distFromStart - b.distFromStart;
    });

    // Start BFS from the starting node src
    pq.offer(new State(src, 0));
    distTo[src] = 0;

    while (!pq.isEmpty()) {
        State state = pq.poll();
        int curNode = state.node;
        int curDistFromStart = state.distFromStart;

        if (distTo[curNode] < curDistFromStart) { // [!code highlight:5]
            // In Dijkstra's algorithm, the queue may contain duplicate nodes state
            // So it is necessary to check when the element leaves the queue to remove the worse duplicate nodes
            continue;
        }

        for (Edge e : graph.neighbors(curNode)) {
            int nextNode = e.to;
            int nextDistFromStart = curDistFromStart + e.weight;

            if (distTo[nextNode] <= nextDistFromStart) {
                continue;
            }

            // [!code highlight:6]
            // Add nextNode node to the priority queue
            pq.offer(new State(nextNode, nextDistFromStart));
            // Record the minimum path weight from the starting node to the nextNode node
            distTo[nextNode] = nextDistFromStart;
        }
    }

    return distTo;
}
```

## Key Differences Between Dijkstra and BFS

The logic of Dijkstra’s algorithm is almost the same as the standard BFS algorithm. The main changes are:

1. Add a `distFromStart` field to the `State` class. This records the total path weight from the starting node to the current node. Use a [priority queue](https://labuladong.online/en/algo/data-structure-basic/binary-heap-basic/) instead of a normal queue, so the node with the smallest `distFromStart` is processed first.

2. Use a `distTo` array instead of a `visited` array. In standard BFS, a node enters the queue only if `visited[node] == false`. In Dijkstra’s algorithm, a node state enters the queue only if it can make `distTo[node]` smaller.

3. When removing an element from the queue, prune if `distTo[curNode] < curDistFromStart`.

**First, let’s talk about why we use a priority queue.**

It is mainly to make the algorithm faster. The speed of the algorithm depends on the total number of nodes that enter the queue—the fewer nodes, the faster the search.

With Dijkstra, only nodes with `state.distFromStart` less than `distTo[state.node]` are allowed into the queue:

```java
if (distTo[nextNode] <= nextDistFromStart) {
    continue;
}
// Only allow nodes with nextDistFromStart < distTo[nextNode] into the queue
pq.offer(new State(nextNode, nextDistFromStart));
distTo[nextNode] = nextDistFromStart;
```

Using a priority queue means nodes with the smallest `state.distFromStart` are removed first, which usually have the smallest `distTo[state.node]`, helping avoid many non-optimal node states and making the search more efficient.

**Second, why do we use the `distTo` array instead of the `visited` array?**

In standard BFS, the path weight is equal to the number of edges because every edge has a weight of 1. So, the first time you reach a node, you have found the shortest path.

But in a weighted graph, the path weight and the number of edges may be different. A node reached early may have a high path weight, and a node reached later may have a lower path weight. So, we need a `distTo` array to record the shortest known path weight to each node. If we find a shorter path, we update the value of `distTo[node]`.

**Third, the most important difference: why do we prune when popping nodes from the queue?**

```java
// When removing a node from the queue, prune
int curNode = state.node;
int curDistFromStart = state.distFromStart;
if (distTo[curNode] < curDistFromStart) {
    continue;
}
```

The main reason is that Dijkstra’s queue can contain multiple states for the same node.

Here’s an example. Suppose node `1` is the start. On the first step, add `State(4, 2)` and `State(3, 7)` to the priority queue (for paths `1->4` and `1->3`). Update `distTo[4] = 2` and `distTo[3] = 7`:

![](../pictures/dijkstra/d1.jpg)

Then `State(4, 2)` is removed, finding the shortest path `1->4`. The priority queue still has `State(3, 7)`.

Next, add the neighbors of node `4` to the queue. If there is an edge `4->3`, as shown below:

![](../pictures/dijkstra/d5.jpg)

Now, `State(3, 3)` will go into the queue and update `distTo[3] = 3`:

Notice there are now two ways to reach node `3`—paths `1->4->3` and `1->3`, represented as `State(3, 3)` and `State(3, 7)`.

`State(3, 3)` comes out first, giving the shortest path `1->4->3` and pushes its neighbors into the queue.

When `State(3, 7)` is removed, since `distTo[3] < 7`, we skip it and stop searching that path.

**So we must prune when removing nodes from the queue. Otherwise, the algorithm might search the same node many times and get stuck.**

This example also gives us two key observations:

- **When a node enters the queue, even though `distTo[node]` is updated, this value may not be the final shortest path. There could be a shorter path later.**
- **When a node comes out of the queue, `state.distFromStart` may not be the shortest path, but `distTo[state.node]` is guaranteed to be the shortest path.**


## Complexity Analysis

Suppose the input graph has $V$ nodes and $E$ edges. Let's use the [time and space complexity analysis method](https://labuladong.online/en/algo/essential-technique/complexity-analysis/) to analyze the time and space complexity of Dijkstra's algorithm.

The space complexity of Dijkstra's algorithm depends on the number of states stored in the `distTo` array and the priority queue `pq`.

To analyze the time complexity, you need to understand that although the code seems to have nested for/while loops, the real focus should be on the enqueue and dequeue operations of `pq`. The time complexity depends on how many states are stored in `pq` and how many operations you do.

So, the key question is: how many node states can be stored in `pq` at most? We discussed before that there may be duplicate node states in the queue, so it's not just $O(V)$ states.

If you think carefully about the example above, node `3` appears twice in the queue. This is because its [in-degree](https://labuladong.online/en/algo/data-structure-basic/graph-basic/) is 2, which means there are 2 edges pointing to node `3`:

![](../pictures/dijkstra/d5.jpg)

In other words, the number of elements in `pq` is not proportional to the number of nodes $V$, but to the number of edges $E$. So, in the worst case, `pq` may hold $O(E)$ states.

In summary, `pq` can store up to $O(E)$ node states. Each enqueue and dequeue operation takes $O(\log{E})$ time. So, the **overall time complexity is $O(E\log{E})$, and the space complexity is $O(V + E)$**.

::: note Note

If you check other sources, you may see that Dijkstra's algorithm has a time complexity of $O(E\log{V})$ and space complexity of $O(V)$.

That's because, ideally, `pq` only holds up to $V$ nodes, and there are $O(E)$ operations on `pq`. So, the overall time complexity is $O(E\log{V})$ and space complexity is $O(V)$.

However, there are many ways to implement Dijkstra's algorithm. Different programming languages or different data structure APIs may change the complexity a bit.

For example, the Dijkstra algorithm in this article uses a PriorityQueue which is implemented by a binary heap. It only provides enqueue and dequeue APIs, but does not support updating elements in the queue directly. So duplicate node states can appear, and up to $E$ elements may be in the queue.

The above complexity analysis is for your reference.

:::

## Optimization for Point-to-Point Shortest Path

The Dijkstra algorithm code above computes the shortest path from the start node `src` to all other nodes.

In some problems, you only need to find the shortest path from `src` to a specific node `dst`. The code above can still solve this problem, but with a small change, the algorithm can be a bit faster.

As mentioned above, when a node State is dequeued from `pq`, `distTo[state.node]` is its final shortest path length. So, you can add a check when dequeuing: if `state.node == dst`, then exit early:

```java
// Calculate shortest path weight sum from src to dst
int dijkstra(Graph graph, int src, int dst) {
    while (!pq.isEmpty()) {
        State state = pq.poll();
        int curNode = state.node;
        int curDistFromStart = state.distFromStart;

        if (distTo[curNode] < curDistFromStart) {
            continue;
        }
        // Check when dequeuing; return when reaching destination
        if (curNode == dst) {
            return curDistFromStart;
        }
        ...

    }
    return -1;
}
```

This optimization can improve the real running speed of the algorithm, but from a theoretical point of view, it doesn't change the complexity. The time complexity is still $O(E\log{E})$, and the space complexity is still $O(V + E)$.

## Summary

The Dijkstra algorithm in this article has time complexity $O(E\log{E})$ and space complexity $O(V + E)$.

The core idea of Dijkstra is BFS + greedy. With the help of a priority queue, the first node dequeued has the smallest distance from the start node.

The greedy idea works because: when you add an edge to the path, the total weight should increase.

If the graph has any negative weight edges, this condition fails and the greedy idea no longer works. So Dijkstra's algorithm also does not work for graphs with negative edges.

In the next article, [Dijkstra Algorithm Extensions](https://labuladong.online/en/algo/data-structure/dijkstra-follow-up/), I will introduce Dijkstra's algorithm with more constraints. In the [Dijkstra Practice Problems](https://labuladong.online/en/algo/problem-set/dijkstra/), you will practice using Dijkstra to solve some interesting problems.
