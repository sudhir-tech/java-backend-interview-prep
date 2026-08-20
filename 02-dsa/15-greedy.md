# DSA — Greedy Algorithms

A **Greedy Algorithm** makes the best available choice at each step, with the goal that these local choices lead to a globally optimal solution.

Greedy algorithms are important in interviews because they test:

- Sorting
- Decision making
- Intervals
- Scheduling
- Priority Queues
- Optimization
- Proof of correctness
- Exchange arguments
- Choosing the right ordering

The key question is:

> Can I make the best local choice without preventing an optimal global solution?

---

# 1. What is Greedy?

A greedy algorithm repeatedly chooses the option that looks best **right now**.

Example:

```text
Choose the activity that finishes earliest.
```

Then continue with the remaining activities.

The algorithm does not usually reconsider previous choices.

---

# 2. Greedy vs Brute Force

### Brute Force

Try many or all possibilities:

```text
Choice 1
Choice 2
Choice 3
...
```

### Greedy

Choose the best-looking option immediately:

```text
Best current choice
↓
Best current choice
↓
Best current choice
```

Greedy is usually much faster when its correctness can be proven.

---

# 3. Greedy vs Dynamic Programming

This distinction is important.

### Greedy

```text
Make one locally optimal choice
and never revisit it.
```

### Dynamic Programming

```text
Explore multiple states
and combine optimal subproblem results.
```

A greedy solution is not automatically correct just because each individual choice looks optimal.

---

# 4. When Should You Think Greedy?

Look for phrases such as:

```text
minimum number of...
maximum number of...
earliest...
latest...
best possible...
minimum cost...
maximum profit...
schedule...
intervals...
choose...
```

Also look for problems where:

```text
sorting + one pass
```

seems possible.

---

# 5. Common Greedy Patterns

Important patterns include:

```text
1. Sort by a useful property
2. Earliest finish time
3. Smallest/largest available value
4. Interval scheduling
5. Two-pointer greedy
6. Greedy + PriorityQueue
7. Greedy + sorting
8. Greedy + frequency/counting
9. Greedy + exchange argument
10. Greedy + invariant
```

---

# 6. Activity Selection

Given activities with:

```text
start time
end time
```

select the maximum number of non-overlapping activities.

Example:

```text
(1,2)
(2,3)
(3,4)
(1,4)
```

Choose:

```text
(1,2)
(2,3)
(3,4)
```

---

# 7. Activity Selection Strategy

Sort activities by:

```text
end time ascending
```

Then:

```text
Choose earliest-finishing activity
↓
Choose next compatible activity
↓
Repeat
```

Why earliest finish?

It leaves the largest possible amount of time for future activities.

---

# 8. Activity Selection — Java

```java
static int maxActivities(
        int[][] activities) {

    Arrays.sort(
        activities,
        Comparator.comparingInt(
            activity -> activity[1]
        )
    );

    int count = 0;
    int lastEnd = Integer.MIN_VALUE;

    for (int[] activity :
            activities) {

        int start = activity[0];
        int end = activity[1];

        if (start >= lastEnd) {

            count++;
            lastEnd = end;
        }
    }

    return count;
}
```

Complexity:

```text
Time: O(n log n)
Space: O(1) auxiliary
```

excluding sorting implementation details.

---

# 9. Interval Scheduling

This is one of the most important greedy problems.

Given intervals:

```text
[start, end]
```

choose the maximum number of non-overlapping intervals.

The standard strategy is:

```text
Sort by end time.
```

Then greedily select compatible intervals.

---

# 10. Why Sort by End Time?

Suppose:

```text
A ends at 4
B ends at 7
```

If both are currently available, choosing `A` leaves more time for future intervals.

This is the key greedy insight.

---

# 11. Meeting Rooms

A different interval problem asks:

> How many rooms are required to hold all meetings?

Example:

```text
[0,30]
[5,10]
[15,20]
```

Answer:

```text
2
```

This is not the same as activity selection.

---

# 12. Meeting Rooms II — Two Arrays

Separate:

```text
start times
end times
```

Sort both.

Then use two pointers.

```java
static int minMeetingRooms(
        int[][] intervals) {

    int n = intervals.length;

    int[] starts = new int[n];
    int[] ends = new int[n];

    for (int i = 0;
         i < n;
         i++) {

        starts[i] =
            intervals[i][0];

        ends[i] =
            intervals[i][1];
    }

    Arrays.sort(starts);
    Arrays.sort(ends);

    int rooms = 0;
    int endIndex = 0;

    for (int start : starts) {

        if (start < ends[endIndex]) {

            rooms++;

        } else {

            endIndex++;
        }
    }

    return rooms;
}
```

Complexity:

```text
Time: O(n log n)
Space: O(n)
```

---

# 13. Meeting Rooms II — PriorityQueue

Another approach:

```text
Sort by start time
+
Min heap of end times
```

The heap root is the meeting that finishes earliest.

If:

```text
current start >= earliest end
```

reuse the room.

Otherwise:

```text
need another room
```

---

# 14. Merge Intervals

Given:

```text
[1,3]
[2,6]
[8,10]
[9,12]
```

Result:

```text
[1,6]
[8,12]
```

Strategy:

```text
Sort by start time
↓
Compare current interval with previous
↓
Merge if overlapping
```

---

# 15. Merge Intervals — Java

```java
static int[][] merge(
        int[][] intervals) {

    if (intervals.length <= 1) {
        return intervals;
    }

    Arrays.sort(
        intervals,
        Comparator.comparingInt(
            interval -> interval[0]
        )
    );

    List<int[]> result =
        new ArrayList<>();

    int start =
        intervals[0][0];

    int end =
        intervals[0][1];

    for (int i = 1;
         i < intervals.length;
         i++) {

        if (intervals[i][0] <= end) {

            end = Math.max(
                end,
                intervals[i][1]
            );

        } else {

            result.add(
                new int[]{start, end}
            );

            start =
                intervals[i][0];

            end =
                intervals[i][1];
        }
    }

    result.add(
        new int[]{start, end}
    );

    return result.toArray(
        new int[result.size()][2]
    );
}
```

---

# 16. Non-Overlapping Intervals

Given intervals, remove the minimum number so that the remaining intervals do not overlap.

Equivalent idea:

```text
Keep the maximum number of non-overlapping intervals.
```

Therefore:

```text
answer =
total intervals
-
maximum compatible intervals
```

Use:

```text
sort by end time
```

---

# 17. Insert Interval

Given sorted, non-overlapping intervals and a new interval:

```text
[2,5]
```

insert it while maintaining sorted non-overlapping intervals.

Typical approach:

```text
Add intervals ending before new interval
↓
Merge overlapping intervals
↓
Add remaining intervals
```

This is a common interview problem.

---

# 18. Jump Game

Given:

```text
nums[i] = maximum jump length
```

determine whether the last index is reachable.

Example:

```text
[2,3,1,1,4]
```

Answer:

```text
true
```

---

# 19. Jump Game — Greedy Idea

Maintain:

```text
farthest reachable index
```

For every index:

```java
farthest =
    Math.max(
        farthest,
        i + nums[i]
    );
```

If:

```text
i > farthest
```

we can no longer reach this index.

---

# 20. Jump Game — Java

```java
static boolean canJump(
        int[] nums) {

    int farthest = 0;

    for (int i = 0;
         i < nums.length;
         i++) {

        if (i > farthest) {
            return false;
        }

        farthest =
            Math.max(
                farthest,
                i + nums[i]
            );

        if (farthest
                >= nums.length - 1) {
            return true;
        }
    }

    return true;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 21. Jump Game II

Now find the minimum number of jumps required to reach the end.

Example:

```text
[2,3,1,1,4]
```

Answer:

```text
2
```

Path:

```text
0 → 1 → 4
```

---

# 22. Jump Game II — Greedy

Maintain:

```text
currentEnd
farthest
jumps
```

Process all positions reachable with the current number of jumps.

When reaching:

```text
currentEnd
```

we must make another jump.

---

# 23. Jump Game II — Java

```java
static int jump(
        int[] nums) {

    int jumps = 0;
    int currentEnd = 0;
    int farthest = 0;

    for (int i = 0;
         i < nums.length - 1;
         i++) {

        farthest =
            Math.max(
                farthest,
                i + nums[i]
            );

        if (i == currentEnd) {

            jumps++;
            currentEnd =
                farthest;
        }
    }

    return jumps;
}
```

Complexity:

```text
Time: O(n)
Space: O(1)
```

---

# 24. Gas Station

Given:

```text
gas[i]
cost[i]
```

find a starting station from which the complete circular route is possible.

Greedy insight:

If the total gas is at least the total cost, a solution exists.

If the current tank becomes negative after reaching station `i`, then the current starting point cannot work.

---

# 25. Gas Station — Java

```java
static int canCompleteCircuit(
        int[] gas,
        int[] cost) {

    int total = 0;
    int current = 0;
    int start = 0;

    for (int i = 0;
         i < gas.length;
         i++) {

        int gain =
            gas[i] - cost[i];

        total += gain;
        current += gain;

        if (current < 0) {

            start = i + 1;
            current = 0;
        }
    }

    return total >= 0
        ? start
        : -1;
}
```

Complexity:

```text
O(n)
```

---

# 26. Partition Labels

Given a string, partition it so that each character appears in at most one partition.

Example:

```text
ababcbacadefegdehijhklij
```

One valid partition size sequence:

```text
9, 7, 8
```

Greedy idea:

```text
Find last occurrence of every character.
```

While scanning a partition, extend its end to the farthest last occurrence of any character seen.

---

# 27. Partition Labels — Java

```java
static List<Integer>
partitionLabels(String s) {

    int[] last =
        new int[26];

    for (int i = 0;
         i < s.length();
         i++) {

        last[
            s.charAt(i) - 'a'
        ] = i;
    }

    List<Integer> result =
        new ArrayList<>();

    int start = 0;
    int end = 0;

    for (int i = 0;
         i < s.length();
         i++) {

        end = Math.max(
            end,
            last[
                s.charAt(i) - 'a'
            ]
        );

        if (i == end) {

            result.add(
                end - start + 1
            );

            start = i + 1;
        }
    }

    return result;
}
```

---

# 28. Assign Cookies

Each child has a greed factor.

Each cookie has a size.

Goal:

```text
maximize number of satisfied children
```

Sort both arrays.

Use two pointers.

Strategy:

```text
Give the smallest cookie
that can satisfy the least greedy child.
```

---

# 29. Assign Cookies — Java

```java
static int findContentChildren(
        int[] greed,
        int[] cookies) {

    Arrays.sort(greed);
    Arrays.sort(cookies);

    int child = 0;

    for (int cookie : cookies) {

        if (child < greed.length
                && cookie >= greed[child]) {

            child++;
        }
    }

    return child;
}
```

Complexity:

```text
O(n log n + m log m)
```

---

# 30. Lemonade Change

Customers pay with:

```text $5
$10
$20
```

A lemonade costs:

```text $5
```

Maintain counts of:

```text $5 bills
$10 bills
```

For `$20`, prefer giving:

```text $10 + $5
```

instead of:

```text $5 + $5 + $5
```

because `$5` bills are more flexible.

---

# 31. Lemonade Change — Java

```java
static boolean lemonadeChange(
        int[] bills) {

    int five = 0;
    int ten = 0;

    for (int bill : bills) {

        if (bill == 5) {

            five++;

        } else if (bill == 10) {

            if (five == 0) {
                return false;
            }

            five--;
            ten++;

        } else {

            if (ten > 0
                    && five > 0) {

                ten--;
                five--;

            } else if (five >= 3) {

                five -= 3;

            } else {

                return false;
            }
        }
    }

    return true;
}
```

---

# 32. Candy

Each child has a rating.

Rules:

```text
Every child gets at least one candy.
A child with a higher rating than an adjacent child
must receive more candy.
```

Greedy solution:

```text
Left → Right
```

then:

```text
Right → Left
```

---

# 33. Candy — Java

```java
static int candy(
        int[] ratings) {

    int n = ratings.length;

    if (n == 0) {
        return 0;
    }

    int[] candies =
        new int[n];

    Arrays.fill(
        candies,
        1
    );

    for (int i = 1;
         i < n;
         i++) {

        if (ratings[i]
                > ratings[i - 1]) {

            candies[i] =
                candies[i - 1] + 1;
        }
    }

    for (int i = n - 2;
         i >= 0;
         i--) {

        if (ratings[i]
                > ratings[i + 1]) {

            candies[i] =
                Math.max(
                    candies[i],
                    candies[i + 1] + 1
                );
        }
    }

    int total = 0;

    for (int value : candies) {
        total += value;
    }

    return total;
}
```

---

# 34. Two-Pointer Greedy

Some greedy problems combine:

```text
sorting
+
two pointers
```

Example:

```text
Assign Cookies
Boats to Save People
Container-related selection problems
```

Typical pattern:

```text
sort
↓
left/right pointers
↓
make locally optimal decision
```

---

# 35. Boats to Save People

Each boat can carry at most two people.

Given:

```text
weights
limit
```

minimize boats.

Sort weights.

Use:

```text
lightest
heaviest
```

If both fit:

```text
put both together
```

Otherwise:

```text
heaviest goes alone
```

---

# 36. Boats — Java

```java
static int numRescueBoats(
        int[] people,
        int limit) {

    Arrays.sort(people);

    int left = 0;
    int right = people.length - 1;
    int boats = 0;

    while (left <= right) {

        if (people[left]
                + people[right]
                <= limit) {

            left++;
        }

        right--;
        boats++;
    }

    return boats;
}
```

Complexity:

```text
O(n log n)
```

---

# 37. Minimum Number of Arrows

Given balloons represented by intervals:

```text
[start, end]
```

one arrow can burst all balloons whose intervals overlap at the arrow position.

Greedy:

```text
Sort by end coordinate.
Shoot at the current end.
```

If the next balloon starts after the arrow position:

```text
need another arrow
```

---

# 38. Minimum Arrows — Java

```java
static int findMinArrowShots(
        int[][] points) {

    if (points.length == 0) {
        return 0;
    }

    Arrays.sort(
        points,
        Comparator.comparingLong(
            point -> point[1]
        )
    );

    int arrows = 1;

    long arrowPosition =
        points[0][1];

    for (int i = 1;
         i < points.length;
         i++) {

        if (points[i][0]
                > arrowPosition) {

            arrows++;

            arrowPosition =
                points[i][1];
        }
    }

    return arrows;
}
```

---

# 39. Fractional Knapsack

Unlike 0/1 Knapsack, items can be divided.

Each item has:

```text
value
weight
```

Choose items based on:

```text
value / weight
```

highest ratio first.

---

# 40. Fractional Knapsack — Java

```java
record Item(
    int value,
    int weight
) {}

static double fractionalKnapsack(
        Item[] items,
        int capacity) {

    Arrays.sort(
        items,
        Comparator.comparingDouble(
            item ->
                (double) item.value()
                / item.weight()
        ).reversed()
    );

    double total = 0.0;

    for (Item item : items) {

        if (capacity == 0) {
            break;
        }

        int amount =
            Math.min(
                capacity,
                item.weight()
            );

        total +=
            (double) amount
            * item.value()
            / item.weight();

        capacity -= amount;
    }

    return total;
}
```

The greedy strategy works for fractional knapsack because items can be split.

---

# 41. 0/1 Knapsack Is Different

In 0/1 Knapsack:

```text
Take item
or
Do not take item
```

You cannot take a fraction.

A simple value/weight greedy strategy is not generally correct.

Typical solution:

```text
Dynamic Programming
```

This is an important interview distinction.

---

# 42. Job Sequencing with Deadlines

Each job has:

```text
deadline
profit
```

Each job takes:

```text
1 unit of time
```

Goal:

```text
maximize total profit
```

Greedy idea:

```text
Sort jobs by profit descending.
```

Schedule each job in the latest available slot before its deadline.

Why latest slot?

It preserves earlier slots for jobs with tighter deadlines.

---

# 43. Huffman Coding

Huffman coding repeatedly combines the two least frequent symbols.

Use:

```text
Min Heap
```

Pattern:

```text
insert all frequencies
↓
remove two smallest
↓
combine
↓
insert combined value
↓
repeat
```

This is a classic:

```text
Greedy + Heap
```

problem.

---

# 44. Minimum Cost to Connect Sticks

Always combine the two smallest sticks.

Use:

```java
PriorityQueue<Integer>
```

This is the same greedy principle used by Huffman coding.

Complexity:

```text
O(n log n)
```

---

# 45. Task Scheduler

Given tasks:

```text
A A A B B B
```

and a cooldown:

```text
n
```

schedule tasks while respecting the cooldown.

A common greedy approach uses:

```text
frequency counting
+
max heap
+
queue for cooldown
```

The most frequent task determines the schedule structure.

---

# 46. Reorganize String

Rearrange characters so that no two adjacent characters are equal.

Greedy approach:

```text
Always choose the most frequent character
that is different from the previously placed character.
```

A max heap is commonly used.

If at some point no valid character remains:

```text
no solution
```

---

# 47. Remove K Digits

Given a numeric string, remove `k` digits to create the smallest possible number.

Use a:

```text
monotonic increasing stack
```

Greedy rule:

If:

```text
previous digit > current digit
```

and removals remain:

```text
remove previous digit
```

This creates a smaller number as early as possible.

---

# 48. Monotonic Stack as Greedy

Some problems that look like stack problems are also greedy.

Example:

```text
Remove K Digits
```

The local choice:

```text
remove a larger previous digit
when a smaller current digit appears
```

is what makes the final number smaller.

---

# 49. Gas Station Greedy Insight

A useful proof idea:

If traveling from:

```text
start
```

to:

```text
i
```

makes the tank negative, then none of the stations between the old start and `i` can be a valid starting point.

Therefore:

```text
start = i + 1
```

This lets us solve the problem in one pass.

---

# 50. Greedy Proof Techniques

Knowing how to explain **why** greedy works is important.

Common proof techniques:

```text
Exchange Argument
Stays Ahead
Greedy Choice Property
Optimal Substructure
Invariant
```

---

# 51. Exchange Argument

Suppose the greedy algorithm chooses:

```text
A
```

while an optimal solution chooses:

```text
B
```

If we can replace `B` with `A` without making the solution worse, then there exists an optimal solution that begins with the greedy choice.

Repeat this argument.

This is one of the most common ways to prove greedy correctness.

---

# 52. Greedy Choice Property

A problem has the greedy-choice property if:

```text
A locally optimal choice
can be part of a globally optimal solution.
```

Example:

```text
Activity selection
```

Choosing the earliest finishing activity can always be part of some optimal solution.

---

# 53. Optimal Substructure

After making the greedy choice, the remaining problem should still have the structure of the original problem.

Example:

```text
Choose one activity.
```

Then:

```text
choose maximum compatible activities
from the remaining interval.
```

---

# 54. Invariant

An invariant is something that remains true throughout the algorithm.

Example in Jump Game:

```text
farthest
```

always represents the farthest reachable position among processed indices.

Maintaining this invariant gives a clean correctness argument.

---

# 55. Greedy Algorithm Checklist

Before using greedy, ask:

```text
1. What is the local choice?
2. Why is it safe?
3. Can I prove it?
4. Does the choice leave a smaller version of the same problem?
5. Can sorting expose the correct order?
6. Is there a counterexample?
```

If you cannot justify the greedy choice, consider:

```text
DP
Backtracking
Binary Search
Graph algorithms
```

instead.

---

# 56. Common Greedy Mistakes

### Mistake 1 — Assuming greedy is always optimal

It is not.

---

### Mistake 2 — Choosing the largest immediate value

The largest value now may block a better future combination.

---

### Mistake 3 — Ignoring sorting

Many greedy problems become obvious only after sorting.

---

### Mistake 4 — Not proving the choice

In interviews, explain:

```text
Why is this choice safe?
```

---

### Mistake 5 — Confusing greedy with DP

If current decisions affect many future possibilities, DP may be required.

---

### Mistake 6 — Using greedy for 0/1 Knapsack

Value/weight ratio does not generally solve 0/1 Knapsack.

---

# 57. Edge Cases

Always test:

```text
Empty input
One element
Already optimal input
All values equal
Duplicate intervals
Nested intervals
Intervals with same end
Negative values
Large values
No possible solution
Exactly enough capacity
k = 0
k = n
```

---

# 58. Interview Questions — Easy

1. Assign Cookies.
2. Lemonade Change.
3. Can Place Flowers.
4. Best Time to Buy and Sell Stock II.
5. Jump Game.
6. Maximum Units on a Truck.
7. Minimum Number of Arrows.
8. Merge Intervals.
9. Meeting Rooms.
10. Partition Labels.

---

# 59. Interview Questions — Medium

11. Jump Game II.
12. Gas Station.
13. Candy.
14. Non-overlapping Intervals.
15. Meeting Rooms II.
16. Boats to Save People.
17. Task Scheduler.
18. Reorganize String.
19. Remove K Digits.
20. Fractional Knapsack.
21. Job Sequencing.
22. Minimum Cost to Connect Sticks.
23. Huffman Coding.

---

# 60. Interview Questions — Advanced

24. Advanced interval scheduling.
25. Weighted interval scheduling.
26. Multi-resource scheduling.
27. Greedy + PriorityQueue optimization.
28. Advanced task scheduling.
29. Greedy graph algorithms.
30. Minimum spanning tree.
31. Kruskal's Algorithm.
32. Prim's Algorithm.
33. Greedy proofs using exchange arguments.
34. Problems where greedy must be distinguished from DP.

---

# 61. Greedy in Graph Algorithms

Greedy ideas appear in:

```text
Kruskal's Algorithm
Prim's Algorithm
Dijkstra's Algorithm
```

These algorithms repeatedly make a locally best choice while maintaining correctness conditions.

For example:

### Kruskal

Choose:

```text
smallest available edge
```

that does not create a cycle.

### Prim

Choose:

```text
minimum-weight edge
```

that expands the current tree.

### Dijkstra

Choose:

```text
unvisited node with smallest tentative distance.
```

---

# 62. Complexity Summary

| Problem | Greedy Technique | Time | Space |
|---|---|---:|---:|
| Activity Selection | Sort by end | O(n log n) | O(1) auxiliary |
| Merge Intervals | Sort by start | O(n log n) | O(n) |
| Meeting Rooms II | Sort + pointers | O(n log n) | O(n) |
| Jump Game | Farthest reach | O(n) | O(1) |
| Jump Game II | Range expansion | O(n) | O(1) |
| Gas Station | Running balance | O(n) | O(1) |
| Assign Cookies | Sort + two pointers | O(n log n + m log m) | O(1) auxiliary |
| Candy | Two passes | O(n) | O(n) |
| Boats | Sort + two pointers | O(n log n) | O(1) auxiliary |
| Fractional Knapsack | Sort by ratio | O(n log n) | O(1) auxiliary |
| Minimum Arrows | Sort by end | O(n log n) | O(1) auxiliary |
| Huffman Coding | Min Heap | O(n log n) | O(n) |
| Connect Sticks | Min Heap | O(n log n) | O(n) |
| Task Scheduler | Frequency + heap | Typically O(n log n) | O(n) |

---

# 63. Quick Revision

```text
Greedy
│
├── Core Idea
│   └── Best local choice
│
├── Sorting
│   ├── By end time
│   ├── By start time
│   ├── By ratio
│   └── By profit
│
├── Intervals
│   ├── Activity Selection
│   ├── Merge Intervals
│   ├── Meeting Rooms
│   ├── Non-overlapping Intervals
│   └── Minimum Arrows
│
├── Arrays
│   ├── Jump Game
│   ├── Gas Station
│   ├── Candy
│   └── Assign Cookies
│
├── Two Pointers
│   └── Boats to Save People
│
├── Heap + Greedy
│   ├── Huffman Coding
│   ├── Connect Sticks
│   ├── Task Scheduler
│   └── Reorganize String
│
└── Proof
    ├── Greedy Choice
    ├── Exchange Argument
    ├── Invariant
    └── Optimal Substructure
```

---

## Interview Rule

> **Don't just say “I'll use greedy.” Explain the local choice and why it is safe. A strong greedy answer usually looks like: sort by the right property → make the best available choice → maintain an invariant → prove that the choice cannot hurt the optimal solution.**
