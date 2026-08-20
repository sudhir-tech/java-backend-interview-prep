# DSA — Greedy Algorithms

Greedy algorithms solve optimization problems by making the best-looking local choice at each step.

The key idea is:

```text
Make a locally optimal choice
+
Never reconsider that choice
=
Potentially obtain a globally optimal solution
```

Greedy is powerful, but it must be justified. A locally optimal choice does **not** automatically produce a globally optimal answer.

Topics covered:

- Greedy fundamentals
- Greedy-choice property
- Exchange argument
- Activity Selection
- Interval Scheduling
- Fractional Knapsack
- Jump Game
- Gas Station
- Assign Cookies
- Meeting Rooms
- Minimum Platforms
- Merge Intervals
- Non-overlapping Intervals
- Job Sequencing
- Huffman Coding
- Minimum Coins
- Scheduling
- Two-pointer greedy
- Heap + greedy
- Sorting + greedy
- Greedy vs DP
- Interview patterns

---

# 1. What Is a Greedy Algorithm?

A greedy algorithm chooses the best available option at the current step.

Example:

```text
Choose the activity that finishes earliest.
```

Then:

```text
Choose the next compatible activity
that finishes earliest.
```

The algorithm does not try every possible combination.

---

# 2. Greedy vs Brute Force

Brute force:

```text
Try every possibility
↓
Choose the best
```

Greedy:

```text
Make the best local choice
↓
Continue
```

Greedy is usually much faster, but it only works when the problem has the required structure.

---

# 3. Greedy-Choice Property

A problem has the greedy-choice property when an optimal solution can be obtained by making an appropriate greedy choice first.

This is the key question:

```text
Can I safely make this choice
without losing the possibility of an optimal solution?
```

If yes, greedy may be appropriate.

---

# 4. Optimal Substructure

Greedy problems often also have optimal substructure.

After making a valid greedy choice:

```text
remaining problem
```

must still be solvable optimally.

This concept also appears in DP.

---

# 5. Greedy Proof

Do not simply say:

```text
"It looks optimal."
```

For interviews, explain why.

Two common proof techniques are:

```text
Exchange Argument
```

and:

```text
Stays-Ahead Argument
```

---

# 6. Exchange Argument

Suppose your greedy choice is:

```text
G
```

and an optimal solution uses:

```text
O
```

Show that `O` can be replaced with `G` without making the solution worse.

Then:

```text
There exists an optimal solution
that contains the greedy choice.
```

This proves the greedy choice is safe.

---

# 7. Activity Selection

Given activities:

```text
start[i]
finish[i]
```

choose the maximum number of non-overlapping activities.

Greedy rule:

```text
Always choose the activity
that finishes earliest.
```

---

# 8. Why Earliest Finish Works

Suppose:

```text
Activity A finishes earlier than Activity B.
```

Choosing A leaves at least as much remaining time as choosing B.

Therefore, replacing B with A cannot reduce the number of activities we can schedule afterward.

This is the classic exchange argument.

---

# 9. Activity Selection — Java

```java
static int activitySelection(
        int[][] activities) {

    Arrays.sort(
        activities,
        Comparator.comparingInt(
            a -> a[1]
        )
    );

    int count = 0;
    int lastFinish =
        Integer.MIN_VALUE;

    for (int[] activity :
            activities) {

        int start = activity[0];
        int finish = activity[1];

        if (start >= lastFinish) {

            count++;
            lastFinish = finish;
        }
    }

    return count;
}
```

Complexity:

```text
O(n log n)
```

because of sorting.

---

# 10. Interval Scheduling

The same pattern appears in interval scheduling.

Given:

```text
[start, end]
```

choose the maximum number of non-overlapping intervals.

Sort by:

```text
end time
```

Then greedily select compatible intervals.

---

# 11. Important Interval Sorting Rules

Different interval problems require different sorting criteria.

Common choices:

```text
Sort by start
Sort by end
Sort by duration
Sort by end descending
```

Do not automatically sort every interval problem the same way.

---

# 12. Non-overlapping Intervals

Given intervals, remove the minimum number so the remaining intervals do not overlap.

Equivalent:

```text
Keep the maximum number
of non-overlapping intervals.
```

Therefore:

```text
Sort by end time.
```

Then greedily keep compatible intervals.

Answer:

```text
n - maximumKept
```

---

# 13. Non-overlapping Intervals — Java

```java
static int eraseOverlapIntervals(
        int[][] intervals) {

    Arrays.sort(
        intervals,
        Comparator.comparingInt(
            a -> a[1]
        )
    );

    int kept = 0;
    int lastEnd =
        Integer.MIN_VALUE;

    for (int[] interval :
            intervals) {

        if (interval[0] >= lastEnd) {

            kept++;
            lastEnd = interval[1];
        }
    }

    return intervals.length - kept;
}
```

---

# 14. Merge Intervals

Merge all overlapping intervals.

Typical approach:

```text
Sort by start time
↓
Scan from left to right
↓
Merge when current.start <= previous.end
```

This is a sorting + greedy pattern.

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
            a -> a[0]
        )
    );

    List<int[]> result =
        new ArrayList<>();

    int start = intervals[0][0];
    int end = intervals[0][1];

    for (int i = 1;
         i < intervals.length;
         i++) {

        int nextStart =
            intervals[i][0];

        int nextEnd =
            intervals[i][1];

        if (nextStart <= end) {

            end =
                Math.max(
                    end,
                    nextEnd
                );

        } else {

            result.add(
                new int[]{
                    start,
                    end
                }
            );

            start = nextStart;
            end = nextEnd;
        }
    }

    result.add(
        new int[]{
            start,
            end
        }
    );

    return result.toArray(
        new int[0][]
    );
}
```

---

# 16. Fractional Knapsack

In fractional knapsack, an item can be divided.

Example:

```text
Take 50% of an item.
```

Greedy rule:

```text
Choose the item with the highest
value / weight ratio.
```

Unlike 0/1 knapsack, fractional knapsack is a greedy problem.

---

# 17. Fractional Knapsack

For every item:

```text
ratio = value / weight
```

Sort descending by ratio.

Then:

```text
Take as much as possible
from the highest-ratio item.
```

Continue until capacity is full.

---

# 18. Fractional Knapsack — Java

```java
static double fractionalKnapsack(
        int[] weights,
        int[] values,
        int capacity) {

    int n = weights.length;

    Integer[] indices =
        new Integer[n];

    for (int i = 0;
         i < n;
         i++) {

        indices[i] = i;
    }

    Arrays.sort(
        indices,
        (a, b) -> Double.compare(
            (double) values[b]
                / weights[b],
            (double) values[a]
                / weights[a]
        )
    );

    double total = 0.0;

    for (int index : indices) {

        if (capacity == 0) {
            break;
        }

        int weight =
            weights[index];

        int value =
            values[index];

        if (weight <= capacity) {

            capacity -= weight;
            total += value;

        } else {

            total +=
                (double) value
                * capacity
                / weight;

            capacity = 0;
        }
    }

    return total;
}
```

Complexity:

```text
O(n log n)
```

---

# 19. 0/1 Knapsack vs Fractional Knapsack

### 0/1 Knapsack

Item:

```text
take once
or
do not take
```

Usually:

```text
Dynamic Programming
```

### Fractional Knapsack

Item:

```text
can be divided
```

Use:

```text
Greedy by value/weight ratio
```

This distinction is extremely important.

---

# 20. Assign Cookies

Given:

```text
children's greed factors
cookie sizes
```

A child is satisfied if:

```text
cookie >= greed
```

Greedy strategy:

```text
Sort both arrays.
Use the smallest cookie
that can satisfy the current least-greedy child.
```

---

# 21. Assign Cookies — Java

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

# 22. Two-Pointer Greedy

Many greedy problems combine:

```text
sorting
+
two pointers
```

Example:

```text
Assign Cookies
Boats to Save People
```

General pattern:

```text
Sort
↓
Maintain two candidates
↓
Make the safest local choice
```

---

# 23. Boats to Save People

Each boat can carry at most two people and has a weight limit.

Sort weights.

Use:

```text
lightest
heaviest
```

If:

```text
lightest + heaviest <= limit
```

put them together.

Otherwise:

```text
heaviest travels alone.
```

---

# 24. Boats — Java

```java
static int numRescueBoats(
        int[] people,
        int limit) {

    Arrays.sort(people);

    int left = 0;
    int right =
        people.length - 1;

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

---

# 25. Jump Game

Given an array where:

```text
nums[i]
```

is the maximum jump length from position `i`.

Determine whether the last index is reachable.

Greedy idea:

```text
Maintain the farthest reachable index.
```

---

# 26. Jump Game — Java

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
    }

    return true;
}
```

Complexity:

```text
O(n)
```

---

# 27. Jump Game II

Find the minimum number of jumps needed to reach the last index.

Greedy idea:

```text
Current jump range
+
Farthest next reach
```

When we reach the end of the current range:

```text
take another jump
```

---

# 28. Jump Game II — Java

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
            currentEnd = farthest;
        }
    }

    return jumps;
}
```

---

# 29. Gas Station

You have:

```text
gas[i]
cost[i]
```

Determine whether you can complete the circular route and find a valid starting station.

Key observations:

```text
If total gas < total cost:
    impossible.
```

Otherwise, there is a valid starting point.

---

# 30. Gas Station Greedy Insight

Maintain:

```text
currentTank
```

If:

```text
currentTank < 0
```

at station `i`, then the current starting point cannot work.

Therefore:

```text
start = i + 1
currentTank = 0
```

---

# 31. Gas Station — Java

```java
static int canCompleteCircuit(
        int[] gas,
        int[] cost) {

    int total = 0;
    int tank = 0;
    int start = 0;

    for (int i = 0;
         i < gas.length;
         i++) {

        int gain =
            gas[i] - cost[i];

        total += gain;
        tank += gain;

        if (tank < 0) {

            start = i + 1;
            tank = 0;
        }
    }

    return total >= 0
        ? start
        : -1;
}
```

---

# 32. Why Gas Station Greedy Works

If a start at `start` cannot reach station `i` because the tank becomes negative, then none of the stations between:

```text
start ... i
```

can be a valid start either.

Therefore we can safely skip all of them.

---

# 33. Meeting Rooms

Given meeting intervals, determine whether one person can attend all meetings.

Sort by start time.

If:

```text
current.start < previous.end
```

there is an overlap.

---

# 34. Meeting Rooms — Java

```java
static boolean canAttendMeetings(
        int[][] intervals) {

    Arrays.sort(
        intervals,
        Comparator.comparingInt(
            a -> a[0]
        )
    );

    for (int i = 1;
         i < intervals.length;
         i++) {

        if (intervals[i][0]
                < intervals[i - 1][1]) {

            return false;
        }
    }

    return true;
}
```

---

# 35. Meeting Rooms II

Find the minimum number of rooms required.

Two common approaches:

```text
Min Heap
```

or:

```text
Separate start/end arrays
+
Two pointers
```

---

# 36. Meeting Rooms II — Heap

Sort by start time.

Maintain a min heap containing:

```text
end times
```

For each meeting:

```text
If earliest ending room is free:
    reuse it
Else:
    allocate another room
```

---

# 37. Meeting Rooms II — Java

```java
static int minMeetingRooms(
        int[][] intervals) {

    if (intervals.length == 0) {
        return 0;
    }

    Arrays.sort(
        intervals,
        Comparator.comparingInt(
            a -> a[0]
        )
    );

    PriorityQueue<Integer> rooms =
        new PriorityQueue<>();

    for (int[] interval :
            intervals) {

        int start = interval[0];
        int end = interval[1];

        if (!rooms.isEmpty()
                && rooms.peek() <= start) {

            rooms.poll();
        }

        rooms.offer(end);
    }

    return rooms.size();
}
```

Complexity:

```text
O(n log n)
```

---

# 38. Minimum Platforms

Given train arrival and departure times, find the minimum platforms needed.

This is essentially:

```text
maximum number of overlapping intervals.
```

Sort:

```text
arrival times
departure times
```

Then use two pointers.

---

# 39. Job Sequencing with Deadlines

Each job has:

```text
deadline
profit
```

Each job takes one unit of time.

Goal:

```text
maximize total profit.
```

Greedy strategy:

```text
Sort jobs by profit descending.
Place each job in the latest available slot
before its deadline.
```

---

# 40. Job Sequencing Example

Jobs:

```text
A: deadline 2, profit 100
B: deadline 1, profit 50
C: deadline 2, profit 30
```

Take:

```text
A → slot 2
B → slot 1
```

Profit:

```text
150
```

Placing A as late as possible preserves earlier slots for other jobs.

---

# 41. Job Sequencing — Java

A simple implementation:

```java
static int maxProfit(
        int[][] jobs) {

    Arrays.sort(
        jobs,
        (a, b) ->
            Integer.compare(
                b[1],
                a[1]
            )
    );

    int maxDeadline = 0;

    for (int[] job : jobs) {

        maxDeadline =
            Math.max(
                maxDeadline,
                job[0]
            );
    }

    boolean[] used =
        new boolean[maxDeadline + 1];

    int profit = 0;

    for (int[] job : jobs) {

        int deadline = job[0];
        int value = job[1];

        for (int slot = deadline;
             slot >= 1;
             slot--) {

            if (!used[slot]) {

                used[slot] = true;
                profit += value;
                break;
            }
        }
    }

    return profit;
}
```

The simple implementation can be:

```text
O(n log n + nD)
```

where `D` is the maximum deadline.

---

# 42. Huffman Coding

Huffman coding constructs an optimal prefix code.

Given character frequencies:

```text
A: 5
B: 9
C: 12
D: 13
```

repeatedly combine the two smallest frequencies.

Data structure:

```text
Min Heap
```

---

# 43. Huffman Algorithm

```text
Put all frequencies into min heap
↓
Remove two smallest
↓
Combine them
↓
Insert combined frequency
↓
Repeat
```

The resulting tree produces prefix-free codes.

---

# 44. Huffman Complexity

With `n` frequencies:

```text
O(n log n)
```

because each heap operation costs:

```text
O(log n)
```

---

# 45. Minimum Coins

For some coin systems, greedy works:

```text
Choose the largest possible coin
```

Example:

```text
coins = [25, 10, 5, 1]
amount = 41
```

Greedy:

```text
25
10
5
1
```

Total:

```text
4 coins
```

---

# 46. Important Warning: Greedy Coin Change Is Not Always Correct

For arbitrary coin denominations, greedy can fail.

Example:

```text
coins = [1, 3, 4]
amount = 6
```

Greedy:

```text
4 + 1 + 1
= 3 coins
```

Optimal:

```text
3 + 3
= 2 coins
```

Therefore:

```text
Coin Change
```

is generally a DP problem unless the denomination system guarantees greedy correctness.

---

# 47. Scheduling Problems

Greedy is common in scheduling.

Typical goals:

```text
Maximum number of tasks
Minimum machines
Minimum lateness
Maximum profit
Minimum waiting time
```

The correct greedy rule depends on the exact objective.

---

# 48. Scheduling by Earliest Finish

For:

```text
maximum number of non-overlapping activities
```

use:

```text
earliest finish time.
```

This is one of the most important greedy rules.

---

# 49. Scheduling by Earliest Deadline

For minimizing maximum lateness in certain single-machine scheduling problems:

```text
sort by earliest deadline first.
```

This is another classic greedy pattern.

---

# 50. Sorting + Greedy

Many greedy algorithms start with:

```text
Sort the input
```

because sorting reveals the order in which safe choices should be made.

Common examples:

```text
Activity Selection
Fractional Knapsack
Assign Cookies
Boats
Meeting Rooms
Non-overlapping Intervals
Job Sequencing
```

---

# 51. Heap + Greedy

A heap is useful when the best available choice changes dynamically.

Examples:

```text
Meeting Rooms II
Huffman Coding
IPO
Task Scheduling
Maximum Performance
```

Pattern:

```text
Sort events
+
Maintain best candidates in heap
```

---

# 52. Greedy + Two Pointers

Examples:

```text
Assign Cookies
Boats to Save People
Minimum platforms
```

Typical structure:

```text
Sort
↓
left / right
↓
Make locally optimal pairing
```

---

# 53. Greedy + Prefix/Suffix

Some problems maintain a running best:

```text
prefix maximum
suffix minimum
```

Then make a greedy decision at each position.

Examples include:

```text
Partition Labels
Jump Game variants
Array partition problems
```

---

# 54. Partition Labels

Given a string, partition it so that each character appears in at most one partition.

Greedy idea:

```text
Find the last occurrence of each character.
```

While scanning:

```text
current partition must extend
to the furthest last occurrence
of any character seen.
```

---

# 55. Partition Labels — Java

```java
static List<Integer> partitionLabels(
        String s) {

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

    int end = 0;
    int start = 0;

    for (int i = 0;
         i < s.length();
         i++) {

        end =
            Math.max(
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

Complexity:

```text
O(n)
```

---

# 56. Candy

Each child has a rating.

Rules:

```text
Each child gets at least one candy.
A child with a higher rating than a neighbor
must get more candies.
```

Greedy solution:

```text
Left → right
+
Right → left
```

---

# 57. Candy — Java

```java
static int candy(
        int[] ratings) {

    int n = ratings.length;

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

    for (int candy : candies) {
        total += candy;
    }

    return total;
}
```

---

# 58. Minimum Number of Arrows

Given balloon intervals, one arrow can burst all balloons intersecting at the arrow's position.

Greedy rule:

```text
Sort by end coordinate.
Shoot at the earliest possible end.
```

This is closely related to interval scheduling.

---

# 59. Maximum Units on a Truck

Given box types:

```text
number of boxes
units per box
```

maximize units.

Greedy:

```text
Take boxes with highest units per box first.
```

Sort descending by:

```text
units per box.
```

---

# 60. Lemonade Change

Customers pay with:

```text
$5
$10
$20
```

Each lemonade costs:

```text
$5
```

Greedy principle:

```text
Always preserve $5 bills
```

For a `$20`, prefer giving:

```text
$10 + $5
```

over:

```text
$5 + $5 + $5
```

because $5 bills are more flexible.

---

# 61. Lemonade Change — Java

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

# 62. Task Scheduler

Given tasks with cooldown constraints, the goal may be to minimize total execution time.

A common greedy insight:

```text
Schedule the most frequent tasks carefully.
```

A max heap is often used to choose the task with the highest remaining frequency.

This is a:

```text
Greedy + Heap
```

pattern.

---

# 63. IPO / Maximum Capital

A common pattern:

```text
Projects have:
capital requirement
profit
```

At each step:

```text
Add all affordable projects to a max heap by profit.
Choose the most profitable affordable project.
```

This combines:

```text
sorting
+
priority queue
+
greedy
```

---

# 64. Greedy with Priority Queue

General pattern:

```text
Sort candidates by when they become available.
↓
Push available candidates into heap.
↓
Take the best candidate.
↓
Update state.
↓
Repeat.
```

This pattern is extremely useful in interviews.

---

# 65. Greedy vs DP

A common interview question:

```text
Why greedy instead of DP?
```

Good answer:

```text
The problem has a safe greedy choice.
Once that choice is made, an optimal solution can
still be constructed from the remaining problem.
Therefore we do not need to keep multiple future
possibilities as DP would.
```

---

# 66. Greedy Failure Example

Consider:

```text
coins = [1, 3, 4]
amount = 6
```

Greedy:

```text
4 + 1 + 1
```

Optimal:

```text
3 + 3
```

This proves that a seemingly natural greedy rule may fail.

---

# 67. How to Prove Greedy

When explaining a greedy solution:

```text
1. State the greedy choice.
2. Explain why it is safe.
3. Show an exchange argument if possible.
4. Explain the remaining subproblem.
5. Analyze complexity.
```

---

# 68. Greedy Exchange Argument Template

You can say:

```text
Consider an optimal solution O.

If O already contains the greedy choice,
we are done.

Otherwise, replace the choice in O with the greedy choice.

Because the greedy choice is at least as good
for the relevant criterion, this replacement
does not make the solution worse.

Therefore, there exists an optimal solution
containing the greedy choice.

We can safely make that choice and continue.
```

---

# 69. Greedy Interview Checklist

Before using greedy, ask:

```text
□ What is the local choice?
□ Why is it safe?
□ Can I exchange it into an optimal solution?
□ Does the remaining problem have the same structure?
□ Is sorting required?
□ Is a heap required?
□ Can I prove correctness?
```

---

# 70. Common Greedy Mistakes

### Mistake 1 — Assuming greedy always works

It does not.

### Mistake 2 — Choosing the largest value

The largest immediate value may hurt future choices.

### Mistake 3 — Choosing the smallest item

Same problem.

### Mistake 4 — Ignoring sorting

Many greedy solutions require sorting first.

### Mistake 5 — Using greedy for arbitrary Coin Change

Greedy is not universally correct.

### Mistake 6 — Not proving correctness

Interviewers often care about why the greedy choice is safe.

---

# 71. Edge Cases

Test:

```text
Empty input
One element
Already sorted
Reverse sorted
Duplicate values
Equal intervals
Nested intervals
Impossible solution
All elements compatible
No elements compatible
Large values
Integer overflow
```

---

# 72. Algorithm Selection

Use greedy when:

```text
A safe local choice can be proven.
```

Use DP when:

```text
Multiple choices affect future states
and subproblems overlap.
```

Use binary search when:

```text
The answer space is monotonic.
```

Use heap when:

```text
The best available candidate changes dynamically.
```

Use two pointers when:

```text
The sorted structure allows a directional scan.
```

---

# 73. Greedy Pattern Table

| Problem Pattern | Greedy Strategy |
|---|---|
| Activity Selection | Earliest finish |
| Non-overlapping Intervals | Earliest finish |
| Fractional Knapsack | Highest value/weight |
| Assign Cookies | Smallest sufficient cookie |
| Boats | Pair lightest with heaviest when possible |
| Jump Game | Farthest reachable |
| Gas Station | Reset after negative balance |
| Meeting Rooms | Earliest ending room |
| Job Sequencing | Highest profit + latest slot |
| Huffman | Combine two smallest |
| Partition Labels | Furthest required endpoint |
| Candy | Two directional passes |
| Lemonade Change | Preserve flexible bills |

---

# 74. Complexity Summary

| Algorithm | Time | Space |
|---|---:|---:|
| Activity Selection | O(n log n) | O(1) / O(n) |
| Fractional Knapsack | O(n log n) | O(n) |
| Assign Cookies | O(n log n + m log m) | O(1) extra |
| Jump Game | O(n) | O(1) |
| Jump Game II | O(n) | O(1) |
| Gas Station | O(n) | O(1) |
| Merge Intervals | O(n log n) | O(n) |
| Non-overlapping Intervals | O(n log n) | O(1) extra |
| Meeting Rooms | O(n log n) | O(1) extra |
| Meeting Rooms II | O(n log n) | O(n) |
| Job Sequencing | O(n log n + nD) | O(D) |
| Huffman Coding | O(n log n) | O(n) |
| Partition Labels | O(n) | O(1) |
| Candy | O(n) | O(n) |

---

# 75. Quick Revision

```text
Greedy Algorithms
│
├── Sorting + Greedy
│   ├── Activity Selection
│   ├── Fractional Knapsack
│   ├── Interval Scheduling
│   └── Job Sequencing
│
├── Two Pointer + Greedy
│   ├── Assign Cookies
│   ├── Boats
│   └── Minimum Platforms
│
├── Heap + Greedy
│   ├── Meeting Rooms II
│   ├── Huffman Coding
│   └── Task Scheduling
│
├── Linear Greedy
│   ├── Jump Game
│   ├── Gas Station
│   ├── Candy
│   └── Lemonade Change
│
└── Proof Techniques
    ├── Exchange Argument
    └── Stays-Ahead Argument
```

---

# 76. Most Important Greedy Rules

```text
Maximum non-overlapping intervals
→ earliest finish time

Fractional knapsack
→ highest value / weight

Jump Game
→ farthest reachable

Boats
→ pair lightest + heaviest if possible

Gas Station
→ reset start after negative balance

Job Sequencing
→ highest profit, latest available slot

Huffman
→ combine two smallest frequencies

Partition Labels
→ extend to furthest required endpoint
```

---

# 77. Greedy Mental Model

Think:

```text
What decision can I make now
that cannot hurt an optimal solution?
```

Then:

```text
Make the decision
↓
Remove the solved part
↓
Repeat
```

---

# 78. Greedy vs DP Decision Tree

```text
Optimization problem
        |
        ↓
Can a local choice be proven safe?
        |
   +----+----+
   |         |
  Yes        No
   |         |
Greedy      Check for
            overlapping
            subproblems
               |
               ↓
              DP
```

---

# 79. Interview Explanation Template

For a greedy problem, explain it like this:

```text
I will first sort the input based on [criterion].

The greedy choice is [choice].

This choice is safe because [reason].

After making the choice, the remaining problem
has the same structure.

Therefore we can repeat the process.

Sorting takes O(n log n), and the scan takes O(n),
so the overall complexity is O(n log n).
```

This is concise and interview-friendly.

---

# 80. Final Interview Rule

> **Greedy is not "pick what looks best." Greedy is "pick what can be proven safe now."**

If you cannot justify the local choice, consider:

```text
DP
Graph algorithms
Binary search
Backtracking
```

instead.
