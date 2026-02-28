For small numbers, you can use the operators provided by your programming language to do calculations. But if the numbers are very large, the data type might overflow. One solution is to input the numbers as strings, then simulate the multiplication process we learned in elementary school, and use strings to represent the result.

Let's look at LeetCode problem 43, "[Multiply Strings](https://leetcode.com/problems/multiply-strings/)":

**LeetCode 43. Multiply Strings** <span class="inline-block w-2 h-2 rounded-full bg-yellow-500"></span>

Given two non-negative integers `num1` and `num2` represented as strings, return the product of `num1` and `num2`, also represented as a string.

**Note:** You must not use any built-in BigInteger library or convert the inputs to integer directly.

Example 1:**

```
**Input:** num1 = "2", num2 = "3"
**Output:** "6"

```

Example 2:**

```
**Input:** num1 = "123", num2 = "456"
**Output:** "56088"

```

**Constraints:**

	
- `1 <= num1.length, num2.length <= 200`
	
- `num1` and `num2` consist of digits only.
	
- Both `num1` and `num2` do not contain any leading zero, except the number `0` itself.

Note that `num1` and `num2` can be very long, so you can't just convert them to integers and multiply. The only way is to simulate the manual multiplication process.

For example, if you want to calculate `123 × 45` by hand, you would do it like this:

![](../pictures/string-multiply/1.jpg)

First, calculate `123 × 5`, then `123 × 4`, and finally add them with the correct offset. This is very easy for elementary school students. But can you **turn this calculation process into a set of algorithm steps** that a computer can follow?

In this simple process, there is multiplication with carry, addition with offset, and addition with carry. There are also some small problems. For example, multiplying two two-digit numbers may get a three- or four-digit result. How can you handle this in a standard way? This is the magic of algorithms. If you don't think like a computer, even simple problems can't be automated.

First, even this manual process is a bit "advanced." We can break it down even more. The process of `123 × 5` and `123 × 4` can be split into smaller steps, and then add them at the end:

![](../pictures/string-multiply/2.jpg)

Right now, `123` is not a large number. But if it is very big, you can't directly calculate the product. We can use an array to collect the result as we add up the products:

![](../pictures/string-multiply/3.jpg)

The whole process works like this: **use two pointers `i` and `j` to walk through `num1` and `num2`, calculate the products, and add the products to the correct position in `res`**, as shown in this GIF:

![](../pictures/string-multiply/4.gif)

Now there is a key question: how do you add the product to the correct position in `res`? Or, how do you find the right index in `res` using `i` and `j`?

If you look closely, you will see that **the product of `num1[i]` and `num2[j]` should go to `res[i+j]` and `res[i+j+1]`**.

![](../pictures/string-multiply/6.jpg)

Once you understand this, you can write code to simulate this process:

```java
class Solution {
    public String multiply(String num1, String num2) {
        int m = num1.length(), n = num2.length();
        // the result can be at most m + n digits
        int[] res = new int[m + n];
        // start multiplying from the least significant digit
        for (int i = m - 1; i >= 0; i--) {
            for (int j = n - 1; j >= 0; j--) {
                int mul = (num1.charAt(i) - '0') * (num2.charAt(j) - '0');
                // the product is at the corresponding index in res
                int p1 = i + j, p2 = i + j + 1;
                // add to res
                int sum = mul + res[p2];
                res[p2] = sum % 10;
                res[p1] += sum / 10;
            }
        }
        // the leading zeros that might exist in the result (unused digits)
        int i = 0;
        while (i < res.length && res[i] == 0)
            i++;
        // convert the calculation result to a string
        StringBuilder str = new StringBuilder();
        for (; i < res.length; i++)
            str.append(res[i]);
        
        return str.length() == 0 ? "0" : str.toString();
    }
}
```

Now, the string multiplication algorithm is done.

**To sum up**, the ways we usually think about calculation are actually hard for computers. The arithmetic processes we are used to are not complicated, but it's not easy to turn them into code. Algorithms need to simplify the process. Here, we get the result by adding as we calculate.

There is a saying: "Don't fall into fixed ways of thinking. Don't be too mechanical. Be creative." But I think being mechanical is not always bad. It can make things faster and reduce mistakes. Algorithms are a set of mechanical thinking steps. Only by being mechanical can computers help us solve hard problems!

Maybe algorithm is just a way to **find fixed ways of thinking**. I hope this article helps you.
