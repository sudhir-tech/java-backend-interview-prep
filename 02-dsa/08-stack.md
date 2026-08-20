# DSA — Stack

A **Stack** is a linear data structure that follows:

```text
LIFO
Last In, First Out
```

The last element added is the first element removed.

Stacks are extremely important in Java backend interviews and are commonly used for:

- Parentheses problems
- Expression evaluation
- Monotonic stack problems
- Next Greater Element
- Previous Smaller Element
- Histogram problems
- Undo/redo
- Backtracking
- DFS
- Function call management
- Parsing

---

# 1. Stack Basics

Example:

```text
push 10
push 20
push 30

Stack:

30 ← top
20
10
```

If we pop:

```text
30
```

comes out first.

---

# 2. Stack Operations

Typical operations:

```text
push    → add element
pop     → remove top element
peek    → view top element
isEmpty → check whether empty
```

Typical complexity:

```text
push:    O(1)
pop:     O(1)
peek:    O(1)
```

---

# 3. Stack in Java

For DSA problems, prefer:

```java
Deque<Integer> stack =
    new ArrayDeque<>();
```

Push:

```java
stack.push(10);
```

Peek:

```java
int top = stack.peek();
```

Pop:

```java
int value = stack.pop();
```

Check empty:

```java
stack.isEmpty();
```

---

# 4. Why Prefer Deque?

Java has the legacy:

```java
Stack<E>
```

class.

For modern Java code, prefer:

```java
Deque<E>
```

with:

```java
ArrayDeque<E>
```

because `Deque` provides stack behavior without the legacy `Stack` API.

---

# 5. Basic Stack Example

```java
Deque<Integer> stack =
    new ArrayDeque<>();

stack.push(10);
stack.push(20);
stack.push(30);

System.out.println(stack.peek());
// 30

System.out.println(stack.pop());
// 30

System.out.println(stack.pop());
// 20
```

---

# 6. Reverse a String Using Stack

```java
static String reverse(String s) {

    Deque<Character> stack =
        new ArrayDeque<>();

    for (char c : s.toCharArray()) {
        stack.push(c);
    }

    StringBuilder result =
        new StringBuilder();

    while (!stack.isEmpty()) {
        result.append(stack.pop());
    }

    return result.toString();
}
```

For simple string reversal, `StringBuilder.reverse()` is more direct, but the stack approach demonstrates the data structure.

---

# 7. Valid Parentheses

One of the most important stack interview problems.

Example:

```text
"()[]{}"
```

is valid.

Example:

```text
"([)]"
```

is invalid.

---

# 8. Valid Parentheses — Java

```java
static boolean isValid(
        String s) {

    Deque<Character> stack =
        new ArrayDeque<>();

    for (char c : s.toCharArray()) {

        if (c == '('
                || c == '['
                || c == '{') {

            stack.push(c);

        } else {

            if (stack.isEmpty()) {
                return false;
            }

            char top =
                stack.pop();

            if ((c == ')' && top != '(')
                    || (c == ']' && top != '[')
                    || (c == '}' && top != '{')) {

                return false;
            }
        }
    }

    return stack.isEmpty();
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 9. Why Stack Works for Parentheses

The most recently opened bracket must be closed first.

Example:

```text
( [ { } ] )
  ↑
```

This is exactly:

```text
LIFO
```

behavior.

---

# 10. Remove Adjacent Duplicates

Example:

```text
abbaca
```

Remove adjacent duplicates:

```text
ca
```

Use a stack:

```java
static String removeDuplicates(
        String s) {

    Deque<Character> stack =
        new ArrayDeque<>();

    for (char c : s.toCharArray()) {

        if (!stack.isEmpty()
                && stack.peek() == c) {

            stack.pop();

        } else {

            stack.push(c);
        }
    }

    StringBuilder result =
        new StringBuilder();

    while (!stack.isEmpty()) {
        result.append(stack.removeLast());
    }

    return result.toString();
}
```

---

# 11. Min Stack

Problem:

> Design a stack that supports retrieving the minimum element in O(1).

One approach uses two stacks:

```text
main stack
min stack
```

---

# 12. Min Stack — Java

```java
class MinStack {

    private final Deque<Integer> stack =
        new ArrayDeque<>();

    private final Deque<Integer> minStack =
        new ArrayDeque<>();

    public void push(int value) {

        stack.push(value);

        if (minStack.isEmpty()
                || value <= minStack.peek()) {

            minStack.push(value);
        }
    }

    public void pop() {

        int value = stack.pop();

        if (value == minStack.peek()) {
            minStack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return minStack.peek();
    }
}
```

### Complexity

```text
push:    O(1)
pop:     O(1)
top:     O(1)
getMin:  O(1)
```

---

# 13. Stack Using Two Queues

A common interview design question.

The goal is to implement:

```text
push
pop
top
empty
```

using queues.

There are multiple approaches.

One approach makes `push` expensive:

```text
push → O(n)
pop  → O(1)
```

Another makes `pop` expensive:

```text
push → O(1)
pop  → O(n)
```

The key is understanding how to simulate LIFO using FIFO structures.

---

# 14. Queue Using Two Stacks

This is the reverse problem.

Use:

```text
input stack
output stack
```

When output is empty:

```text
Move everything from input → output
```

Then:

```text
oldest element
```

is on top of the output stack.

This gives amortized:

```text
O(1)
```

queue operations.

---

# 15. Next Greater Element

One of the most important **Monotonic Stack** problems.

For every element, find the first greater element to its right.

Example:

```text
[2, 1, 2, 4, 3]
```

Answer:

```text
[4, 2, 4, -1, -1]
```

---

# 16. Brute Force Next Greater Element

For every element:

```text
scan everything to the right
```

Complexity:

```text
O(n²)
```

A monotonic stack can reduce this to:

```text
O(n)
```

---

# 17. Next Greater Element — Java

```java
static int[] nextGreaterElement(
        int[] nums) {

    int n = nums.length;

    int[] result =
        new int[n];

    Arrays.fill(
        result,
        -1
    );

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = n - 1;
         i >= 0;
         i--) {

        while (!stack.isEmpty()
                && stack.peek()
                    <= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {
            result[i] =
                stack.peek();
        }

        stack.push(nums[i]);
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 18. Why Monotonic Stack Is O(n)

Although there is a `while` loop inside the `for` loop, every element is:

```text
pushed once
popped at most once
```

Therefore total stack operations are:

```text
O(n)
```

This is a very common interview explanation.

---

# 19. Monotonic Stack

A monotonic stack maintains elements in a specific order.

### Monotonic increasing stack

Elements are maintained in increasing order.

Useful for:

```text
Next smaller
Previous smaller
Histogram
```

### Monotonic decreasing stack

Elements are maintained in decreasing order.

Useful for:

```text
Next greater
Previous greater
Sliding maximum-related problems
```

---

# 20. Next Greater Element — Index Version

Sometimes values are duplicated, so storing indexes is safer.

```java
static int[] nextGreater(
        int[] nums) {

    int[] result =
        new int[nums.length];

    Arrays.fill(
        result,
        -1
    );

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = nums.length - 1;
         i >= 0;
         i--) {

        while (!stack.isEmpty()
                && nums[stack.peek()]
                    <= nums[i]) {

            stack.pop();
        }

        if (!stack.isEmpty()) {
            result[i] =
                nums[stack.peek()];
        }

        stack.push(i);
    }

    return result;
}
```

---

# 21. Daily Temperatures

Given:

```text
[73,74,75,71,69,72,76,73]
```

Find how many days until a warmer temperature.

Answer:

```text
[1,1,4,2,1,1,0,0]
```

Use a monotonic decreasing stack of indexes.

```java
static int[] dailyTemperatures(
        int[] temperatures) {

    int[] result =
        new int[temperatures.length];

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int i = 0;
         i < temperatures.length;
         i++) {

        while (!stack.isEmpty()
                && temperatures[i]
                    > temperatures[stack.peek()]) {

            int previous =
                stack.pop();

            result[previous] =
                i - previous;
        }

        stack.push(i);
    }

    return result;
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 22. Next Greater Element II

If the array is circular:

```text
[1,2,1]
```

the next greater of the last `1` can be `2`.

A common trick is to iterate:

```text
2 * n - 1
```

positions and use:

```java
int index = i % n;
```

Example:

```java
for (int i = 2 * n - 1;
     i >= 0;
     i--) {

    int index = i % n;

    while (!stack.isEmpty()
            && nums[stack.peek()]
                <= nums[index]) {

        stack.pop();
    }

    if (i < n
            && !stack.isEmpty()) {

        result[index] =
            nums[stack.peek()];
    }

    stack.push(index);
}
```

---

# 23. Previous Greater Element

Instead of scanning right:

```text
scan left → right
```

Maintain a monotonic stack.

For each element:

```text
remove smaller/equal values
top = previous greater
push current
```

This is the mirror image of Next Greater Element.

---

# 24. Next Smaller Element

Maintain an increasing stack.

For each value:

```text
while top >= current:
    pop

top = next smaller
push current
```

---

# 25. Previous Smaller Element

Same idea, but scan from:

```text
left → right
```

and maintain an increasing stack.

---

# 26. Largest Rectangle in Histogram

A classic hard stack problem.

Example:

```text
[2,1,5,6,2,3]
```

Maximum rectangle:

```text
10
```

The key idea is to find:

```text
previous smaller
next smaller
```

for each bar.

A monotonic increasing stack provides both boundaries efficiently.

---

# 27. Largest Rectangle — Java

```java
static int largestRectangleArea(
        int[] heights) {

    Deque<Integer> stack =
        new ArrayDeque<>();

    int maximum = 0;

    for (int i = 0;
         i <= heights.length;
         i++) {

        int current =
            i == heights.length
                ? 0
                : heights[i];

        while (!stack.isEmpty()
                && heights[stack.peek()]
                    > current) {

            int height =
                heights[stack.pop()];

            int left =
                stack.isEmpty()
                    ? -1
                    : stack.peek();

            int width =
                i - left - 1;

            maximum = Math.max(
                maximum,
                height * width
            );
        }

        stack.push(i);
    }

    return maximum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 28. Why Histogram Uses a Stack

When a smaller bar appears, it tells us:

```text
The previous taller bars cannot extend further right.
```

So we finalize their rectangles.

The stack stores bars whose right boundary is not yet known.

---

# 29. Maximal Rectangle

Given a binary matrix, find the largest rectangle containing only `1`s.

Transform each row into histogram heights.

For every row:

```text
Update heights
↓
Largest Rectangle in Histogram
```

Overall complexity can be:

```text
O(rows × cols)
```

using an O(cols) histogram solution per row.

---

# 30. Asteroid Collision

Example:

```text
[5,10,-5]
```

Result:

```text
[5,10]
```

Use a stack because the current asteroid may collide with the most recently surviving asteroid.

```java
static int[] asteroidCollision(
        int[] asteroids) {

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (int asteroid : asteroids) {

        boolean destroyed = false;

        while (!stack.isEmpty()
                && asteroid < 0
                && stack.peek() > 0) {

            int top = stack.peek();

            if (top < -asteroid) {
                stack.pop();
                continue;
            }

            if (top == -asteroid) {
                stack.pop();
            }

            destroyed = true;
            break;
        }

        if (!destroyed) {
            stack.push(asteroid);
        }
    }

    int[] result =
        new int[stack.size()];

    for (int i = result.length - 1;
         i >= 0;
         i--) {

        result[i] =
            stack.pop();
    }

    return result;
}
```

---

# 31. Decode String

Example:

```text
3[a2[c]]
```

Result:

```text
accaccacc
```

Nested structure naturally maps to stack behavior.

You can use stacks for:

```text
counts
strings
```

Typical approach:

```text
Read number
↓
Push current state
↓
Read '['
↓
Build nested string
↓
Read ']'
↓
Pop previous state
↓
Repeat
```

---

# 32. Evaluate Reverse Polish Notation

Example:

```text
["2","1","+","3","*"]
```

Evaluation:

```text
(2 + 1) * 3
=
9
```

Use a stack of numbers.

```java
static int evalRPN(
        String[] tokens) {

    Deque<Integer> stack =
        new ArrayDeque<>();

    for (String token : tokens) {

        if (token.equals("+")
                || token.equals("-")
                || token.equals("*")
                || token.equals("/")) {

            int b = stack.pop();
            int a = stack.pop();

            int result;

            switch (token) {

                case "+":
                    result = a + b;
                    break;

                case "-":
                    result = a - b;
                    break;

                case "*":
                    result = a * b;
                    break;

                default:
                    result = a / b;
            }

            stack.push(result);

        } else {

            stack.push(
                Integer.parseInt(token)
            );
        }
    }

    return stack.pop();
}
```

---

# 33. Infix, Prefix and Postfix

### Infix

Operator is between operands:

```text
A + B
```

### Prefix

Operator comes before operands:

```text
+ A B
```

### Postfix

Operator comes after operands:

```text
A B +
```

Reverse Polish Notation is postfix notation.

Stacks are commonly used for expression conversion/evaluation.

---

# 34. Evaluate Infix Expression

For more complex expressions, use:

```text
Operand stack
Operator stack
```

Consider:

```text
2 + 3 * 4
```

Operator precedence means:

```text
3 * 4
```

must happen before:

```text
2 + ...
```

A stack-based parser can handle this.

---

# 35. Simplify Unix Path

Example:

```text
/home//foo/../bar/
```

Result:

```text
/home/bar
```

Use a stack/deque of directory names.

Rules:

```text
"."  → ignore
".." → pop previous directory
name → push
```

---

# 36. Backspace String Compare

A stack can process:

```text
#
```

as backspace.

However, an O(1)-space two-pointer approach is also possible.

This is a good example of choosing between:

```text
Stack
```

and:

```text
Two Pointers
```

depending on space requirements.

---

# 37. Remove K Digits

Problem:

> Remove `k` digits to create the smallest possible number.

Example:

```text
num = "1432219"
k = 3
```

Answer:

```text
"1219"
```

Use a monotonic increasing stack.

If:

```text
top > current
```

and removals remain:

```text
pop
```

This greedily removes larger digits before smaller digits.

---

# 38. Remove K Digits — Core Idea

```java
Deque<Character> stack =
    new ArrayDeque<>();

for (char digit : num.toCharArray()) {

    while (!stack.isEmpty()
            && k > 0
            && stack.peek() > digit) {

        stack.pop();
        k--;
    }

    stack.push(digit);
}
```

After processing all digits, if:

```text
k > 0
```

remove from the end/top as necessary.

Then remove leading zeroes.

---

# 39. Stock Span

For each stock price, find how many consecutive previous days had prices less than or equal to today's price.

Example:

```text
[100,80,60,70,60,75,85]
```

Answer:

```text
[1,1,1,2,1,4,6]
```

Use a monotonic decreasing stack of indexes/prices.

---

# 40. Stock Span Pattern

For each price:

```text
while stack top <= current:
    pop

span =
    current index - previous greater index

push current
```

This is another previous-greater-element problem.

---

# 41. Online Stock Span

If prices arrive one at a time, maintain the monotonic stack as state.

This is useful for understanding how stack-based algorithms can process streaming input.

---

# 42. Frequency Stack

Design a stack that pops the most frequent element.

If multiple elements have the same frequency:

```text
return the most recently pushed
```

A common design uses:

```text
frequency map
+
map from frequency → stack
+
maximum frequency
```

This combines:

```text
HashMap + Stack
```

---

# 43. Browser History

Browser navigation can be modeled with stacks:

```text
back stack
forward stack
```

When visiting a new page:

```text
push current to back
clear forward
```

Back:

```text
move current → forward
pop back → current
```

Forward:

```text
move current → back
pop forward → current
```

---

# 44. Undo / Redo

A common design uses:

```text
undo stack
redo stack
```

When a new operation happens:

```text
push to undo
clear redo
```

Undo:

```text
move undo → redo
```

Redo:

```text
move redo → undo
```

---

# 45. DFS Using Stack

Depth First Search can be implemented iteratively using a stack.

```java
Deque<Integer> stack =
    new ArrayDeque<>();

stack.push(start);

while (!stack.isEmpty()) {

    int node =
        stack.pop();

    // Process node.

    for (int next : graph[node]) {
        stack.push(next);
    }
}
```

---

# 46. Stack and Recursion

The call stack is conceptually a stack.

For:

```java
factorial(5)
```

calls build up:

```text
factorial(5)
factorial(4)
factorial(3)
factorial(2)
factorial(1)
```

Then return in reverse order.

Understanding this helps explain:

```text
recursion
stack overflow
DFS
backtracking
```

---

# 47. Stack Overflow

Too much recursion can exhaust the call stack.

For example:

```java
void recurse() {
    recurse();
}
```

eventually causes:

```text
StackOverflowError
```

For very deep traversal, an explicit stack can sometimes avoid call-stack limitations.

---

# 48. Stack Memory vs Heap Memory

In Java:

```text
Stack
→ method frames
→ local variables/references
→ call state

Heap
→ objects
→ arrays
→ dynamically allocated data
```

Do not confuse the DSA Stack data structure with the JVM call stack.

---

# 49. Monotonic Stack Recognition

Think **Monotonic Stack** when you see:

```text
Next greater
Next smaller
Previous greater
Previous smaller
Nearest greater
Nearest smaller
Daily temperatures
Stock span
Histogram
```

The key phrase is:

```text
nearest element satisfying a comparison
```

---

# 50. Monotonic Stack Template — Next Greater

```java
Deque<Integer> stack =
    new ArrayDeque<>();

for (int i = n - 1;
     i >= 0;
     i--) {

    while (!stack.isEmpty()
            && nums[stack.peek()]
                <= nums[i]) {

        stack.pop();
    }

    // stack.peek() is next greater

    stack.push(i);
}
```

---

# 51. Monotonic Stack Template — Next Smaller

```java
Deque<Integer> stack =
    new ArrayDeque<>();

for (int i = n - 1;
     i >= 0;
     i--) {

    while (!stack.isEmpty()
            && nums[stack.peek()]
                >= nums[i]) {

        stack.pop();
    }

    // stack.peek() is next smaller

    stack.push(i);
}
```

---

# 52. Choosing Increasing vs Decreasing Stack

### Need next greater?

Use a stack that removes:

```text
smaller/equal
```

### Need next smaller?

Use a stack that removes:

```text
greater/equal
```

A practical way to remember:

```text
Looking for greater:
pop values that are too small.

Looking for smaller:
pop values that are too large.
```

---

# 53. Stack with Index vs Value

### Store values

Good when:

```text
Only the value matters.
```

### Store indexes

Better when you need:

```text
Distance
Width
Original position
Window boundaries
Duplicate-safe comparisons
```

Examples:

```text
Daily Temperatures → indexes
Histogram → indexes
Stock Span → indexes or compressed pairs
```

---

# 54. Common Stack Mistakes

### Mistake 1 — Calling `pop()` on an empty stack

Check:

```java
stack.isEmpty()
```

before popping when emptiness is possible.

---

### Mistake 2 — Confusing `peek()` and `pop()`

```text
peek → read top
pop  → remove top
```

---

### Mistake 3 — Wrong monotonic direction

For Next Greater:

```text
pop smaller/equal
```

For Next Smaller:

```text
pop greater/equal
```

---

### Mistake 4 — Storing values when indexes are needed

Histogram and temperature problems often require distances, so indexes are necessary.

---

### Mistake 5 — Thinking nested loops always mean O(n²)

In a monotonic stack, each element is usually:

```text
pushed once
popped once
```

so total work is:

```text
O(n)
```

---

### Mistake 6 — Forgetting a sentinel

Histogram problems often process an extra:

```text
height = 0
```

at the end to flush remaining bars.

---

# 55. Edge Cases

Test:

```text
Empty input
One element
All elements increasing
All elements decreasing
All elements equal
Duplicate values
No greater element
No smaller element
All brackets valid
Invalid closing bracket
Nested brackets
Deeply nested expressions
k = 0
k >= n
```

---

# 56. Interview Questions — Easy

1. Implement a stack.
2. Reverse a string using a stack.
3. Valid Parentheses.
4. Remove adjacent duplicates.
5. Evaluate postfix expression.
6. Implement Min Stack.
7. Implement Queue using Stacks.
8. Implement Stack using Queues.
9. Simplify Unix Path.
10. Backspace String Compare.

---

# 57. Interview Questions — Medium

11. Next Greater Element.
12. Next Smaller Element.
13. Previous Greater Element.
14. Previous Smaller Element.
15. Daily Temperatures.
16. Stock Span.
17. Asteroid Collision.
18. Remove K Digits.
19. Decode String.
20. Evaluate Reverse Polish Notation.
21. Online Stock Span.
22. Next Greater Element II.
23. Frequency Stack.

---

# 58. Interview Questions — Advanced

24. Largest Rectangle in Histogram.
25. Maximal Rectangle.
26. Trapping Rain Water using stack.
27. Basic Calculator.
28. Basic Calculator II.
29. Expression conversion.
30. Longest Valid Parentheses.
31. Minimum Remove to Make Valid Parentheses.
32. Design a frequency stack.
33. Design undo/redo using stacks.
34. Advanced monotonic stack problems.

---

# 59. Longest Valid Parentheses

Example:

```text
")()())"
```

Answer:

```text
4
```

One stack-based approach stores indexes.

Initialize:

```java
stack.push(-1);
```

When encountering:

```text
'('
```

push its index.

When encountering:

```text
')'
```

pop.

If the stack becomes empty:

```text
push current index as new boundary
```

Otherwise:

```text
current index - stack.peek()
```

is the valid length.

---

# 60. Longest Valid Parentheses — Java

```java
static int longestValidParentheses(
        String s) {

    Deque<Integer> stack =
        new ArrayDeque<>();

    stack.push(-1);

    int maximum = 0;

    for (int i = 0;
         i < s.length();
         i++) {

        if (s.charAt(i) == '(') {

            stack.push(i);

        } else {

            stack.pop();

            if (stack.isEmpty()) {

                stack.push(i);

            } else {

                maximum = Math.max(
                    maximum,
                    i - stack.peek()
                );
            }
        }
    }

    return maximum;
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 61. Minimum Remove to Make Valid Parentheses

Use a stack of indexes for unmatched opening brackets.

Approach:

```text
1. Push indexes of '('.
2. For ')' check whether a matching '(' exists.
3. Mark unmatched characters.
4. Build the result.
```

This demonstrates that stacks can store:

```text
indexes
```

rather than just values.

---

# 62. Stack + HashMap

Some problems combine both.

Example:

```text
Frequency Stack
```

requires:

```text
HashMap<value, frequency>
```

and:

```text
HashMap<frequency, Stack>
```

This is a good example of combining data structures to achieve O(1)-style operations.

---

# 63. Stack + Heap

Some advanced scheduling/order problems may combine:

```text
Stack
+
PriorityQueue
```

The right structure depends on whether the problem needs:

```text
LIFO
```

or:

```text
highest/lowest priority
```

Do not use a heap when the required ordering is simply LIFO.

---

# 64. Stack Problem-Solving Checklist

Before coding:

```text
□ Is this naturally LIFO?
□ Do I need matching/nesting?
□ Do I need the nearest greater/smaller value?
□ Is a monotonic stack appropriate?
□ Should I store indexes or values?
□ Which direction should I scan?
□ What values should be popped?
□ Does every element get pushed once?
□ Does every element get popped at most once?
□ Can the stack be empty?
□ Do I need a sentinel?
□ Is there a duplicate-handling issue?
```

---

# 65. Complexity Summary

| Problem | Technique | Time | Space |
|---|---|---:|---:|
| Valid Parentheses | Stack | O(n) | O(n) |
| Min Stack | Two Stacks | O(1) each | O(n) |
| Next Greater | Monotonic Stack | O(n) | O(n) |
| Daily Temperatures | Monotonic Stack | O(n) | O(n) |
| Stock Span | Monotonic Stack | O(n) total | O(n) |
| Next Smaller | Monotonic Stack | O(n) | O(n) |
| Histogram | Monotonic Stack | O(n) | O(n) |
| Maximal Rectangle | Histogram + Stack | O(rc) | O(c) |
| Asteroid Collision | Stack | O(n) | O(n) |
| Decode String | Stack | O(n) relative to processed output | depends |
| RPN Evaluation | Stack | O(n) | O(n) |
| Remove K Digits | Monotonic Stack | O(n) | O(n) |
| Longest Valid Parentheses | Stack | O(n) | O(n) |

---

# 66. Quick Revision

```text
Stack
│
├── Basic LIFO
│   ├── Push
│   ├── Pop
│   └── Peek
│
├── Matching
│   ├── Valid Parentheses
│   ├── Longest Valid Parentheses
│   └── Expression Parsing
│
├── Monotonic Stack
│   ├── Next Greater
│   ├── Next Smaller
│   ├── Previous Greater
│   ├── Previous Smaller
│   ├── Daily Temperatures
│   └── Stock Span
│
├── Histogram
│   ├── Largest Rectangle
│   └── Maximal Rectangle
│
├── Simulation
│   ├── Asteroid Collision
│   ├── Decode String
│   └── Remove K Digits
│
├── Design
│   ├── Min Stack
│   ├── Queue Using Stacks
│   ├── Stack Using Queues
│   └── Frequency Stack
│
└── Other
    ├── DFS
    ├── Undo / Redo
    └── Browser History
```

---

## Interview Rule

> **When a problem asks for the nearest greater/smaller element, immediately consider a monotonic stack. The key insight is that each element is usually pushed once and popped at most once, giving an O(n) solution even when the code contains a nested `while` loop.**
