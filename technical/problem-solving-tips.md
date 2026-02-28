First, let's address a question: Is it better to practice LeetCode problems directly on the website or on a local IDE?

If the format is like the Nowcoder coding assessments, where you handle input and output by yourself, it's definitely better to write in an IDE. However, for LeetCode's format, I personally prefer solving problems directly on the website for two reasons:

**1. Convenience**

LeetCode has some custom data structures, such as `TreeNode` and `ListNode`. In a local environment, you would need to copy these classes over. Also, you can't test effectively in an IDE; after writing your code, you would still need to paste it into the website to run tests, so you might as well write it directly on the website.

Algorithms aren't like engineering code; they are relatively small in scale, so the benefits of IDE auto-completion are negligible.

**2. Practicality**

During interviews, most interviewers expect you to solve algorithm problems directly on a website, preferably explaining your thought process as you code. If you are used to coding without IDE auto-completion and compiling in your mind during practice, you'll write code faster and more confidently during interviews.

When I interviewed with Kuaishou, an interviewer asked me to [implement the LRU algorithm](https://labuladong.online/en/algo/data-structure/lru-cache/). I implemented both the doubly linked list and the hash-linked list directly on the website, and everything worked perfectly on the first try, which surprised the interviewer.

During the fall recruitment, I was able to secure offers largely because my handwritten algorithm solutions exceeded interviewers' expectations. This was all thanks to the practice I had doing problems directly on the website.

Of course, if you really don't want to solve problems on the website, you can use my VSCode or JetBrains plugin for solving problems. These plugins are perfectly integrated with the content on my website.

Next, I'll introduce some practical "tricks" and debugging techniques to comprehensively improve your chances of passing coding assessments.


<!-- hide -->

## Avoiding Complex Solutions

As you may know, most coding test questions require you to handle input data yourself and then have your program print the output. The underlying principle of judging is to use the Linux redirection symbol `>` to write the output of your program to a file, and then compare your output to the correct answer.

In some cases, the complexity of the problem becomes unnecessary. We can simplify our approach. For example, suppose a question asks you to reverse a singly linked list given as a space-separated string of characters, emphasizing that you must convert the input to a singly linked list before reversing it.

What would you do? Would you define a `ListNode` class for the singly linked list node, write code to convert the input into a singly linked list, and then perform the tedious pointer operations to reverse the list?

Remember, our goal is to get the problem accepted (AC), not to learn algorithmic thinking. The judging system cannot accurately assess algorithm logic, only whether your output is correct. So a clever approach is to store the input in an array, reverse it with a few lines of code using the [two-pointer technique](https://labuladong.online/en/algo/essential-technique/array-two-pointers-summary/), and print the result.

I've seen many such problems. For instance, a problem may require you to reverse a singly linked list in groups and emphasize using recursion, like the algorithm in [Reverse Nodes in k-Group](https://labuladong.online/en/algo/data-structure/reverse-linked-list-recursion/). If you reverse using an array, it takes just two minutes to write the solution.

Similarly, in the problem discussed in [Flatten Nested List](https://labuladong.online/en/algo/data-structure/flatten-nested-list-iterator/), the approach is quite clever. But if you encounter it in a test with input like a string `[1,[4,[6]]]`, you can simply use a regular expression to extract the numbers, resulting in a flattened list.


## Choosing a Programming Language

From the perspective of solving algorithm problems, I personally recommend using Java as the programming language for coding interviews. This is because JetBrains' IntelliJ is exceptionally user-friendly. Compared to editors for other languages, it not only offers convenient shortcuts like `psvm` and `sout` (if you don't know these, it's time to hit the books), but it also helps catch many common mistakes, such as forgetting to increment variables in a `while` loop or mistakenly placing a `return` statement inside a loop due to oversight.

C++ is also acceptable, but I find it not as convenient as Java. From what I remember, C++ doesn't even have a `split` function for strings, which alone makes me reluctant to use C++.

Moreover, C++ has stricter time constraints. While other languages might have a time limit of 4000ms, C++ is limited to 2000ms, which seems unfair. No wonder when I see others writing algorithms in C++, they avoid using the standard library's `vector` container and instead opt for raw `int[]` arrays, which looks quite cumbersome to me.

As for Python, I don’t use it much for problem-solving because I’m not a fan of dynamic languages—they can be difficult to debug. However, it does provide many useful features. If you are well-versed in Python, you can sometimes find shortcuts. For instance, in our previous discussion on the [expression evaluation algorithm](https://labuladong.online/en/algo/data-structure/implement-calculator/), which is a difficult-level algorithm, you can directly obtain the answer using Python's built-in `exec` function.

This can certainly be advantageous in coding tests because, as mentioned earlier, the focus is on the result, and no one cares how you achieve it.


## Layered Code Structure

Layering code is considered a good practice as it can increase coding speed and reduce debugging difficulty.

Simply put, avoid writing all your code in the `main` function. The approach I always use is to have the `main` function handle input data, add a `solution` function to manage data processing and output the result, and then use another function like `backtrack` to handle the specific algorithm logic.

For example, if I am solving a problem using dynamic programming with memoization, the general structure of the code would look like this:


```java
public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        // mainly responsible for receiving data
        int N = scanner.nextInt();
        int[][] orders = new int[N][2];
        for (int i = 0; i < N; i++) {
            orders[i][0] = scanner.nextInt();
            orders[i][1] = scanner.nextInt();
        }
        // delegate solution to solve the problem
        solution(orders);
    }

    static void solution(int[][] orders) {
        // exclude some basic boundary cases
        if (orders.length == 0) {
            System.out.println("None");
            return;
        }
        // delegate the dp function to execute specific algorithm logic
        int res = dp(orders, 0);
        // responsible for outputting the result
        System.out.println(res);
    }

    // memoization
    static HashMap<String, Integer> memo = new HashMap<>();
    static int dp(int[][] orders, int start) {
        // specific algorithm logic
    }
}
```

Notice how clear it is when you structure the code this way: each function has its own primary task. If something goes wrong, it's easier to debug.

It's not about writing the code in a highly standardized way. You can skip constraints like `private`, and even using pinyin for variable names is fine. The key point is not to write all the code directly in the `main` function. It becomes chaotic, and while it might not cause errors immediately, debugging will be quite challenging if issues arise. Losing track of the problem is something to be avoided.

## How to Debug Algorithms

Errors in code are unavoidable. Sometimes, the entire idea might be wrong, or it could be a minor detail, such as swapping `i` and `j`. How do you troubleshoot such issues?

I believe that general algorithm issues should not be too difficult to identify. Visual inspection should reveal most problems, and if not, printing the values of key variables should eventually uncover the issue.

**The most challenging aspect is debugging recursive algorithms.**

Without certain experience, the recursive process of functions is hard to understand correctly. Therefore, let's focus on how to efficiently debug recursive algorithms.

Some readers might suggest copying the algorithm into an IDE and setting breakpoints to step through it. Isn't that sufficient?

This method is certainly viable, but as discussed in previous articles, recursive functions are best understood from a global perspective, rather than delving into specific details.

If you are not familiar enough with recursion and lack a global perspective, stepping through with breakpoints can easily lead to confusion.

**My suggestion is to print key values directly within the recursive function and use indentation to visually observe the execution of the recursive function.**

Indentation significantly enhances debugging efficiency. Besides the solution function, we define a new function `printIndent` and a global variable `count`:


```java
// Global variable to record the recursion depth of the recursive function
int count = 0;

// Input n, print n tab indents
void printIndent(int n) {
    for (int i = 0; i < n; i++) {
        printf("   ");
    }
}
```

Next, here comes the strategy:

**At the beginning of the recursive function, call `printIndent(count++)` and print key variables; then, before all `return` statements, call `printIndent(--count)` and print the return values**.

Let's consider a concrete example. In the previous article [Dynamic Programming in Fallout 4](https://labuladong.online/en/algo/dynamic-programming/freedom-trail/), a recursive `dp` function was implemented. Its basic structure is as follows:


```java
int dp(String ring, int i, String key, int j) {
    // base case
    if (j == key.length()) {
        return 0;
    }
    
    // state transition
    for (int k : charToIndex.get(key.charAt(j))) {
        int subProblem = dp(ring, k, key, j + 1);
    }
    
    return res;
}
```

After debugging, this recursive `dp` function has been transformed into the following:


```java
int count = 0;
void printIndent(int n) {
    for (int i = 0; i < n; i++) {
        System.out.print("   ");
    }
}

int dp(String ring, int i, String key, int j) {
    // printIndent(count++);
    // printf("i = %d, j = %d
", i, j);
    
    if (j == key.length()) {
        // printIndent(--count);
        // printf("return 0
");
        return 0;
    }
    

    for (int k : charToIndex.get(key.charAt(j))) {
        int subProblem = dp(ring, k, key, j + 1);
    }
    
    // printIndent(--count);
    // printf("return %d
", res);
    return res;
}
```

**Simply add some print statements at the beginning of the function and at every `return` statement.**

If you remove the comments and run a test case, the output would look like this:

![](../pictures/algo-debug-tech/1.jpg)

By comparing the corresponding indentation, you can know the key parameters `i, j` entered each time in the recursion, as well as the result returned from each recursive call.

**Most importantly, this provides a clear view of the recursive process. Have you noticed that it resembles a recursion tree?**

![](../pictures/algo-debug-tech/2.jpg)

In the previous article [Detailed Explanation of Dynamic Programming Patterns](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/), it was mentioned that understanding a recursive function is best achieved by drawing a recursion tree. With this print method, there's no need to draw the recursion tree yourself, and you can clearly see the return value of each recursion.

**This is a small but effective technique to greatly enhance the "joy" of solving problems, more efficient than setting breakpoints in an IDE.**

I have directly integrated this feature into the visualization panel, where the printed values in the recursive function automatically include indentation. For more details, please refer to [Visualization Panel Instructions](https://labuladong.online/en/algo/intro/visualize/).


## Exam Preparation Strategy

It's not worth getting stuck on a single algorithm problem right before the exam.

Instead, try to look at as many different types of problems as possible. Spend five minutes thinking about each one, and if you can't figure out the solution, directly check other people's answers. Just understanding the approach is enough; there's no need to write it out yourself, as it's often a waste of time.

During the written test, the biggest fear is having no ideas. By reviewing various problem types, you will at least feel more at ease. As long as you have an approach in mind, solving a problem in 20 to 30 minutes on average is not too difficult.

As previously mentioned, there is no problem that brute-force enumeration can't solve. You can directly use the [Backtracking Algorithm Framework](https://labuladong.online/en/algo/essential-technique/backtrack-framework/) to force a solution. If necessary, add a memoization technique, and it becomes the [Dynamic Programming Framework](https://labuladong.online/en/algo/essential-technique/dynamic-programming-framework/). Worst case, if you can't solve it, achieving 60% of the test cases with brute force is still acceptable.

I won't say much more about strategies. They sound simple and can be easy to grasp, but the challenge is that you need to recognize them to understand them. In this article, I've briefly introduced some techniques for algorithm exams. Take your time to savor them~
