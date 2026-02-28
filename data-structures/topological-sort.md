::: info Prerequisites

Before reading this article, you need to learn:

- [DFS/BFS Traversal of Graph Structures](https://labuladong.online/en/algo/data-structure-basic/graph-traverse-basic/)
- [Cycle Detection in Directed Graphs](https://labuladong.online/en/algo/data-structure/cycle-detection/)

:::

::: important One-Sentence Summary

The reverse postorder of a graph's DFS traversal, or the BFS traversal order using an in-degree array, gives you the topological sort result.

:::

Topological sort is another classic algorithm in graph theory. Before performing a topological sort, you must first ensure the graph has no cycles. For cycle detection, refer to [Cycle Detection in Directed Graphs](https://labuladong.online/en/algo/data-structure/cycle-detection/).

**This article uses specific algorithm problems to implement topological sort using both DFS and BFS approaches**.

The BFS solution is somewhat cleaner in terms of code, but the DFS solution helps you better understand the essence of recursive data structure traversal. So in this article, I'll cover the DFS approach first, then the BFS approach.

## Topological Sort (DFS Version)

Take a look at LeetCode problem 210, "[Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/)":

**LeetCode 210. Course Schedule II** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`. You are given an array `prerequisites` where `prerequisites[i] = [ai, bi]` indicates that you **must** take course `bi` first if you want to take course `ai`.

	
- For example, the pair `[0, 1]`, indicates that to take course `0` you have to first take course `1`.

Return *the ordering of courses you should take to finish all courses*. If there are many valid answers, return **any** of them. If it is impossible to finish all courses, return **an empty array**.

Example 1:**

```

**Input:** numCourses = 2, prerequisites = [[1,0]]
**Output:** [0,1]
**Explanation:** There are a total of 2 courses to take. To take course 1 you should have finished course 0. So the correct course order is [0,1].

```

Example 2:**

```

**Input:** numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
**Output:** [0,2,1,3]
**Explanation:** There are a total of 4 courses to take. To take course 3 you should have finished both courses 1 and 2. Both courses 1 and 2 should be taken after you finished course 0.
So one correct course order is [0,1,2,3]. Another correct ordering is [0,2,1,3].

```

Example 3:**

```

**Input:** numCourses = 1, prerequisites = []
**Output:** [0]

```

**Constraints:**

	
- `1 <= numCourses <= 2000`
	
- `0 <= prerequisites.length <= numCourses * (numCourses - 1)`
	
- `prerequisites[i].length == 2`
	
- `0 <= ai, bi < numCourses`
	
- `ai != bi`
	
- All the pairs `[ai, bi]` are **distinct**.

This problem is an advanced version of the one in [Cycle Detection](https://labuladong.online/en/algo/data-structure/cycle-detection/). Instead of just determining whether you can complete all courses, it asks you to return a valid course order that ensures all prerequisites are completed before starting each course.

The function signature is:

```java
int[] findOrder(int numCourses, int[][] prerequisites);
```

Let me first explain the term Topological Sorting. The definitions you find online tend to be very mathematical, so let me just use this diagram from Baidu Baike to give you an intuitive feel:

![](../pictures/topological-sort/top.jpg)

**Intuitively, topological sorting means "flattening" a graph such that all arrows point in the same direction** — for example, all arrows point to the right in the diagram above.

Clearly, if a directed graph contains a cycle, topological sorting is impossible because you can't make all arrows point in the same direction. Conversely, if a graph is a DAG (Directed Acyclic Graph), topological sorting is always possible.

But what does this problem have to do with topological sorting?

**It's not hard to see: if you treat courses as nodes and dependencies between courses as directed edges, the topological sort of this graph gives you a valid course order**.

First, let's check whether the input course dependencies contain a cycle. If there's a cycle, topological sorting is impossible, so we can reuse the logic from [Cycle Detection](https://labuladong.online/en/algo/data-structure/cycle-detection/):

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    if (!canFinish(numCourses, prerequisites)) {
        // not possible to finish all courses
        return new int[]{};
    }
    // ...
}
```

Now here's the key question: how do you actually perform topological sorting? Do you need some fancy technique?

**It's actually very simple: reverse the postorder traversal result of the graph, and that's your topological sort**.

::: note Is Reversal Necessary?

Some readers have mentioned that the topological sort algorithms they've seen online use the postorder traversal result directly without reversal. Why is that?

You can indeed find such solutions. The reason is that they define edges differently when building the graph. In my graph, the arrow direction represents "depended upon" — for example, node `1` points to `2`, meaning node `1` is depended upon by node `2`, i.e., you must finish `1` before doing `2`. This feels more intuitive.

If you reverse this and define directed edges as "depends on," then all edges in the graph are flipped, and you don't need to reverse the postorder result. Specifically, just change `graph[from].add(to);` to `graph[to].add(from);` in my solution, and no reversal is needed.

:::

Let's look at the solution code directly. It builds on the cycle detection code by adding logic to record the postorder traversal result:

```java
class Solution {
    // record the postorder traversal result
    List<Integer> postorder = new ArrayList<>();
    // record if a cycle exists
    boolean hasCycle = false;
    boolean[] visited, onPath;

    // main function
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        List<Integer>[] graph = buildGraph(numCourses, prerequisites);
        visited = new boolean[numCourses];
        onPath = new boolean[numCourses];
        // traverse the graph
        for (int i = 0; i < numCourses; i++) {
            traverse(graph, i);
        }
        // cannot perform topological sort if there is a cycle
        if (hasCycle) {
            return new int[]{};
        }
        // reverse postorder traversal result is the topological sort result
        Collections.reverse(postorder);
        int[] res = new int[numCourses];
        for (int i = 0; i < numCourses; i++) {
            res[i] = postorder.get(i);
        }
        return res;
    }

    // graph traversal function
    void traverse(List<Integer>[] graph, int s) {
        if (onPath[s]) {
            // found a cycle
            hasCycle = true;
        }
        if (visited[s] || hasCycle) {
            return;
        }
        // pre-order traversal position
        onPath[s] = true;
        visited[s] = true;
        for (int t : graph[s]) {
            traverse(graph, t);
        }
        // post-order traversal position
        postorder.add(s);
        onPath[s] = false;
    }

    // build graph function
    List<Integer>[] buildGraph(int numCourses, int[][] prerequisites) {
        // code as seen earlier
    }
}
```

<visual slug='course-schedule-ii'>

Nodes with `visited` set to true are shown in green, and nodes with `onPath` set to true are shown in orange.

You can open the visualization panel and click <code type="click">if (onPath[s])</code> multiple times to watch the DFS traversal of the graph.

</visual>

The code may look long, but the logic should be clear. As long as the graph has no cycle, we call `traverse` to perform DFS on the graph, record the postorder traversal result, and then reverse it to get the final answer.

**So why is the reverse of the postorder traversal the topological sort?**

Instead of a mathematical proof, let me explain with an intuitive example using binary trees. Here's the binary tree traversal framework we've discussed many times:

```java
void traverse(TreeNode root) {
    // pre-order traversal position
    traverse(root.left)
    // in-order traversal position
    traverse(root.right)
    // post-order traversal position
}
```

When does postorder traversal execute? Only after both the left and right subtrees have been fully traversed. In other words, the child nodes are added to the result list before the root node.

**This property of postorder traversal is crucial. Topological sort is based on postorder traversal because a task can only begin after all the tasks it depends on have been completed**.

Think of a binary tree as a directed graph where edges point from parent nodes to child nodes, like this:

![](../pictures/topological-sort/2.jpeg)

In the standard postorder traversal result, the root node appears last. Simply reverse the traversal result, and you get the topological sort:

![](../pictures/topological-sort/3.jpeg)

I know some readers will ask: what's the relationship between the reversed postorder result and the preorder traversal result?

For binary trees, they might look related, but in reality they have no relationship whatsoever. Do not assume that reversing the postorder result gives you the preorder result.

The key difference was already explained in [Binary Tree Thinking (Master Plan)](https://labuladong.online/en/algo/essential-technique/binary-tree-summary/): postorder code executes only after both subtrees are fully traversed. Only postorder traversal can capture the "dependency" relationship — no other traversal order can do this.

## Topological Sort (BFS Version)

Make sure you understand the BFS algorithm from [Cycle Detection](https://labuladong.online/en/algo/data-structure/cycle-detection/) that uses an `indegree` array to determine whether a directed graph contains a cycle.

**If you can understand the BFS version of the cycle detection algorithm, the BFS version of topological sort is easy to grasp, because the node traversal order is the topological sort result**.

For example, in the cycle detection example, the value in each node below represents its enqueue order:

![](../pictures/topological-sort/13.jpeg)

Clearly, this order is a valid topological sort result.

So we just need to slightly modify the BFS cycle detection algorithm to record the node traversal order, and we get the topological sort result:

```java
class Solution {

    public int[] findOrder(int numCourses, int[][] prerequisites) {
        // Build the graph, same as in cycle detection algorithm
        List<Integer>[] graph = buildGraph(numCourses, prerequisites);
        // Calculate in-degrees, same as in cycle detection algorithm
        int[] indegree = new int[numCourses];
        for (int[] edge : prerequisites) {
            int from = edge[1], to = edge[0];
            indegree[to]++;
        }

        // Initialize the queue with nodes having in-degree of 0, same as in cycle detection algorithm
        Queue<Integer> q = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) {
                q.offer(i);
                /**<extend up -200>
                ![](../pictures/topological-sort/6.jpeg)
                */
            }
        }

        // Record the topological sort result
        int[] res = new int[numCourses];
        // Record the order of traversed nodes (index)
        int count = 0;
        // Start executing BFS algorithm
        while (!q.isEmpty()) {
            int cur = q.poll();
            // The order of nodes being dequeued is the topological sort result
            res[count] = cur;
            count++;
            for (int next : graph[cur]) {
                /**<extend up -200>
                ![](../pictures/topological-sort/7.jpeg)
                */
                indegree[next]--;
                if (indegree[next] == 0) {
                    q.offer(next);
                }
            }
        }

        if (count != numCourses) {
            // A cycle exists, topological sort is not possible
            return new int[]{};
        }
        
        return res;
    }

    // Function to build the graph
    List<Integer>[] buildGraph(int n, int[][] edges) {
        // See above
    }
}
```

In principle, [graph traversal](https://labuladong.online/en/algo/data-structure-basic/graph-basic/) requires a `visited` array to prevent revisiting nodes. Here, the BFS algorithm effectively uses the `indegree` array to serve the same purpose as a `visited` array — only nodes with an in-degree of 0 can enter the queue, which prevents infinite loops.
