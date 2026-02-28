::: info Prerequisite Knowledge

Before reading this article, you should first study:

- [Basics and Traversal of N-ary Trees](https://labuladong.online/en/algo/data-structure-basic/n-ary-tree-traverse-basic/)
- [Basics and General Implementation of Graphs](https://labuladong.online/en/algo/data-structure-basic/graph-basic/)

:::

The Union-Find algorithm is designed for the problem of "dynamic connectivity". It is tested very often, and you must master it.

First, let’s talk about what dynamic connectivity in a graph means.

## 1. Dynamic Connectivity

Simply put, dynamic connectivity can be seen as connecting nodes in a graph. For example, in the graph below, there are 10 nodes. They are not connected to each other, and are labeled from 0 to 9:

![](../pictures/unionfind/1.jpg)

Our Union-Find algorithm mainly needs to implement these two APIs:

```java
class UF {
    // connect p and q
    public void union(int p, int q);
    // determine if p and q are connected
    public boolean connected(int p, int q);
    // return the number of connected components in the graph
    public int count();
}
```

The "connected" relation here is an equivalence relation. It has these three properties:

1. Reflexive: node `p` is connected to itself `p`.

2. Symmetric: if node `p` is connected to `q`, then `q` is also connected to `p`.

3. Transitive: if node `p` is connected to `q`, and `q` is connected to `r`, then `p` is connected to `r`.

For example, in the graph above, any two different nodes from 0 to 9 are not connected. Calling `connected` on any pair returns false, and there are 10 connected components.

If we call `union(0, 1)`, then 0 and 1 become connected, and the number of connected components drops to 9.

Then we call `union(1, 2)`. Now 0, 1, 2 are all connected. Calling `connected(0, 2)` will return true, and the number of connected components becomes 8.

![](../pictures/unionfind/2.jpg)

Checking this kind of equivalence relation is very useful, like when a compiler checks different references to the same variable, or when social networks compute friend circles, and so on.

Now you should have a basic idea of what dynamic connectivity is. The key of the Union-Find algorithm is the efficiency of the `union` and `connected` functions. So, what model should we use to represent the connectivity of this graph? What data structure should we use to implement the code?


<!-- hide -->

## 2. Basic Idea

Just now I separated the “model” from the concrete “data structure”, and there is a reason for that. We use a forest (several trees) to represent dynamic connectivity in a graph, and we use an array to implement this forest.

How do we use a forest to represent connectivity? We set a pointer in each node to point to its parent. If the node is a root, the pointer points to itself. For example, for the graph with 10 nodes we showed before, at the beginning, no nodes are connected, it looks like this:

![](../pictures/unionfind/3.jpg)

```java
class UF {
    // record the number of connected components
    private int count;
    // the parent node of node x is parent[x]
    private int[] parent;

    // constructor, n is the total number of nodes in the graph
    public UF(int n) {
        // initially, all nodes are not connected
        this.count = n;
        // the parent pointer initially points to itself
        parent = new int[n];
        for (int i = 0; i < n; i++)
            parent[i] = i;
    }

    // other functions
}
```

**If two nodes are connected, we connect the root of one node to the root of the other node (either one is fine)**:

![](../pictures/unionfind/4.jpg)

```java
class UF {
    // For the sake of brevity, the previous given code is omitted...

    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ)
            return;
        // merge two trees into one
        parent[rootP] = rootQ;
        // parent[rootQ] = rootP works as well

        // two components become one
        count--;
    }

    // return the root node of a node x
    private int find(int x) {
        // the root node has parent[x] == x
        while (parent[x] != x)
            x = parent[x];
        return x;
    }

    // return the number of connected components currently
    public int count() { 
        return count;
    }
}
```

**In this way, if nodes `p` and `q` are connected, they must have the same root**:

![](../pictures/unionfind/5.jpg)

```java
class UF {
    // For brevity, the previous given code part is omitted...

    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }
}
```

So the basic Union-Find algorithm is done. It is quite amazing that we can use an array to simulate a forest, and solve this complex problem in such a clever way.

What is the time complexity of this algorithm? We see that the main APIs `connected` and `union` are both dominated by the complexity of the `find` function, so their complexity is the same as `find`.

The main job of `find` is to move from a node upward to the root of its tree. So its time complexity is the height of the tree. We may be used to thinking that tree height is `logN`, but that is not always true. Height `logN` only appears in balanced binary trees. A general tree can be very unbalanced, almost like a linked list. In the worst case, the height can be `N`.

![](../pictures/unionfind/6.jpg)

So for the above solution, the time complexity of `find`, `union`, and `connected` is all $O(N)$. This is not good. Graph problems like social networks usually have huge data sizes. The operations `union` and `connected` are called very frequently, and it is unacceptable if each call needs linear time.

**The key question is: how can we avoid unbalanced trees?** We only need a small trick.

## 3. Balance Optimization

We need to know in what case the tree becomes unbalanced. The key is the `union` process:

```java
class UF {
    // To save space, the previous code is omitted...

    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ)
            return;
        // merge two trees into one
        parent[rootP] = rootQ;
        // parent[rootQ] = rootP also works
        count--;
    }
}
```

At first, we simply connect the tree of `p` under the root of the tree of `q`. This can create a “big head, small body” unbalanced shape, like this:

![](../pictures/unionfind/7.jpg)

If this keeps happening, the tree will become more and more unbalanced. **What we really want is to attach the smaller tree under the larger tree. This will keep the tree more balanced and avoid the big-head-small-body shape.** To do this, we use an extra `size` array, which records how many nodes each tree has. We can think of this as the “weight” of the tree:

```java
class UF {
    private int count;
    private int[] parent;
    // add an array to record the "weight" of the tree
    private int[] size;

    public UF(int n) {
        this.count = n;
        parent = new int[n];
        // initially each tree has only one node
        // the weight should be initialized to 1
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }
    // other functions
}
```

For example, `size[3] = 5` means the tree whose root is node `3` has `5` nodes in total. With this, we can change the `union` method:

```java
class UF {
    private int count;
    private int[] parent;
    private int[] size;

    public UF(int n) {
        this.count = n;
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }

    // Connect node p and node q
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ)
            return;
        
        // Attach the smaller tree under the larger tree for balance
        if (size[rootP] > size[rootQ]) {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        } else {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        count--;
    }

    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }

    private int find(int x) {
        while (parent[x] != x)
            x = parent[x];
        return x;
    }

    public int count() { 
        return count;
    }

    // Return the total number of nodes in the connected component where node x is located
    public int size(int x) {
        int root = find(x);
        return size[root];
    }
}
```

By comparing the weights of the two trees, we can keep the trees relatively balanced. The height of the tree will be about `logN`. This greatly improves performance.

Also, because we record the weight of every tree, we can add a `size(x)` method to quickly get the number of nodes in the connected component that contains `x`.

Now the time complexity of `find`, `union`, `connected`, and `size` all drops to $O(logN)$. Even if the data size reaches hundreds of millions, the time used is still very small.


## 4. Path Compression

This optimization is simple to write, but the idea is very clever.

**We do not care about the exact shape of each tree, we only care about the root.**

Because no matter how the tree looks, the root of every node in the tree is the same. So can we further reduce the height of each tree, and keep the tree height as a constant?

![](../pictures/unionfind/8.jpg)

In this way, the parent of every node is the root of the tree. The `find` function can find the root of a node in O(1) time. As a result, the time complexity of both `connected` and `union` also becomes O(1).

To do this, we mainly change the logic of the `find` function. It is very simple, but you may see two different ways to write it.

The first way is to add one line in `find`:

```java
class UF {
    // For brevity, the previous code is omitted...

    private int find(int x) {
        while (parent[x] != x) {
            // this line of code performs path compression
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }
}
```

This operation looks strange. But if you look at the GIF (this tree is extreme for clarity), you will understand:

![](../pictures/unionfind/9.gif)

In words, in every while loop, some child nodes move up. So each time you call `find` and walk toward the root, you also shorten the tree height.

The second way to do path compression is:

```java
class UF {
    // In order to save space, the previous given code part is omitted...
    
    // The second find method with path compression
    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }
}
```

I used to think this recursive version and the first iterative version did the same thing. But I was careless. A reader pointed out that this recursive version does path compression more efficiently than the first one.

This recursive process is a bit hard to understand. You can draw the recursion by hand. I translated what this function does into an iterative form, to help you understand its path compression idea:

```java
// This iterative code is for your understanding of what the recursive code does
public int find(int x) {
    // First, find the root node
    int root = x;
    while (parent[root] != root) {
        root = parent[root];
    }
    // Then connect all nodes from x to the root node directly below the root node
    int old_parent = parent[x];
    while (x != root) {
        parent[x] = root;
        x = old_parent;
        old_parent = parent[old_parent];
    }
    return root;
}
```

The effect of this path compression is:

![](../pictures/unionfind/10.jpeg)

Compared with the first kind of path compression, this method is clearly more thorough. It flattens a whole branch in one go, no surprise here. Even if in some extreme case we get a tall tree, one path compression call can greatly reduce its height. From the view of [amortized analysis](https://labuladong.online/en/algo/essential-technique/complexity-analysis/), the average time complexity of all operations is still O(1). So for efficiency, I suggest you use this version of path compression.

**Also, if you use path compression, then the `size` array for balance is not needed.** So the Union-Find algorithm you see in most cases will look like this:

```java
class UF {
    // the number of connected components
    private int count;
    // store the parent of each node
    private int[] parent;

    // n is the number of nodes in the graph
    public UF(int n) {
        this.count = n;
        parent = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }
    
    // connect node p and node q
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        
        if (rootP == rootQ)
            return;
        
        parent[rootQ] = rootP;
        // merge two connected components into one
        count--;
    }

    // determine if node p and node q are connected
    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }

    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // return the number of connected components in the graph
    public int count() {
        return count;
    }
}
```

We can analyze the complexity of Union-Find like this: the constructor that builds the data structure needs O(N) time and space. `union` (connect two nodes), `connected` (check if two nodes are connected), and `count` (count the number of connected components) all take O(1) time.

But for some problems, even if we use path compression, we may still need the `size` array. It can help us count the number of nodes in each connected component.

For example, you need to implement a method `size(x)` that returns the number of nodes in the connected component that contains node `x`. Then you must use the `size` array. Otherwise you would need to traverse the whole tree:

```java
class UF {
    // the number of connected components
    private int count;
    // store the parent of each node
    private int[] parent;
    // record the "weight" (number of nodes) of each tree
    private int[] size;

    // n is the number of nodes in the graph
    public UF(int n) {
        this.count = n;
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }
    
    // connect node p and node q
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        
        if (rootP == rootQ)
            return;
        
        // attach the smaller tree to the larger one, more balanced
        if (size[rootP] > size[rootQ]) {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        } else {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        // merge two connected components into one
        count--;
    }

    // determine if node p and node q are connected
    public boolean connected(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        return rootP == rootQ;
    }

    // using path compression
    public int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    // return the number of connected components in the graph
    public int count() {
        return count;
    }

    // return the total number of nodes in the connected component containing node x
    public int size(int x) {
        int root = find(x);
        return size[root];
    }
}
```

Note that in our `size(x)` method, we first get the root of `x`. The root node stores the size of the whole component. That is the number of nodes in this connected component.

Now you should have mastered the core logic of the Union-Find algorithm. Let’s summarize our optimization steps:

1. Use the `parent` array to record the parent of each node, which is like a pointer to its parent. So the `parent` array actually stores a forest (several multi-way trees).

2. Use the `size` array to record the weight of each tree. The goal is to keep the tree balanced after `union`, so that each API has O(logN) time complexity, and does not degrade to a linked list which would slow down operations.

3. Do path compression in the `find` function to keep the height of any tree as a constant, so each API has O(1) time complexity. After using path compression, you can skip the `size` array balance optimization.

::: tip Tip

In most written tests, you are allowed to use your own IDE to code. So you can write this `UF` class in your favorite language in advance, and just copy it when needed. Its code is a bit long, no need to write it from scratch on the spot.

:::
