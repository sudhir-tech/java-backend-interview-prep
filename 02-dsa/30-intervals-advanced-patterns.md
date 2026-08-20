# DSA — Intervals / Advanced Problem Patterns

Interval problems are about ranges represented by:

```text
[start, end]
```

or:

```text
[start, end)
```

They appear frequently in coding interviews.

Core patterns:

- Sorting intervals
- Merge Intervals
- Insert Interval
- Non-overlapping Intervals
- Meeting Rooms
- Meeting Rooms II
- Interval intersections
- Minimum arrows
- Minimum platforms
- Sweep Line
- Difference Array
- Event scheduling
- Greedy interval selection
- Heap + intervals
- Coordinate compression
- Line sweep
- Overlap counting
- Range merging
- Advanced interval patterns

---

# 1. What Is an Interval?

An interval represents a continuous range.

Example:

```text
[2, 5]
```

means the range starts at:

```text
2
```

and ends at:

```text
5
```

In most interview problems, endpoints are integers.

Always check whether the interval is:

```text
closed: [l, r]
```

or:

```text
half-open: [l, r)
```

because endpoint handling can change the answer.

---

# 2. Interval Overlap

For two closed intervals:

```text
[a, b]
[c, d]
```

they overlap if:

```text
max(a, c) <= min(b, d)
```

Equivalently:

```text
a <= d
&&
c <= b
```

---

# 3. No Overlap

Two closed intervals do not overlap when:

```text
b < c
```

or:

```text
d < a
```

For half-open intervals, endpoint equality is handled differently.

Always confirm the problem's definition.

---

# 4. Merge Intervals

Given:

```text
[1,3]
[2,6]
[8,10]
[9,12]
```

merge overlapping intervals:

```text
[1,6]
[8,12]
```

The standard approach is:

```text
Sort by start
↓
Scan from left to right
↓
Merge overlapping intervals
```

---

# 5. Why Sort by Start?

After sorting by start:

```text
current.start
```

is never smaller than the previous interval's start.

Therefore, while scanning, we only need to track the current merged interval.

---

# 6. Merge Intervals — Java

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

Complexity:

```text
O(n log n)
```

---

# 7. Insert Interval

Given sorted non-overlapping intervals and a new interval:

```text
[2,5]
```

insert it while maintaining sorted non-overlapping intervals.

Three phases:

```text
1. Intervals completely before new interval
2. Intervals overlapping new interval
3. Intervals completely after new interval
```

---

# 8. Insert Interval — Java

```java
static int[][] insert(
        int[][] intervals,
        int[] newInterval) {

    List<int[]> result =
        new ArrayList<>();

    int i = 0;

    while (i < intervals.length
            && intervals[i][1]
                < newInterval[0]) {

        result.add(intervals[i]);
        i++;
    }

    int start = newInterval[0];
    int end = newInterval[1];

    while (i < intervals.length
            && intervals[i][0]
                <= end) {

        start =
            Math.min(
                start,
                intervals[i][0]
            );

        end =
            Math.max(
                end,
                intervals[i][1]
            );

        i++;
    }

    result.add(
        new int[]{
            start,
            end
        }
    );

    while (i < intervals.length) {

        result.add(intervals[i]);
        i++;
    }

    return result.toArray(
        new int[0][]
    );
}
```

Complexity:

```text
O(n)
```

after assuming the input intervals are already sorted.

---

# 9. Non-Overlapping Intervals

Remove the minimum number of intervals so the remaining intervals do not overlap.

Equivalent:

```text
Keep maximum number of
non-overlapping intervals.
```

Therefore use:

```text
earliest finish time
```

greedy strategy.

---

# 10. Non-Overlapping Intervals — Java

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

# 11. Activity Selection

This is the classic interval greedy problem.

Goal:

```text
Select maximum number of
non-overlapping intervals.
```

Greedy rule:

```text
Choose the interval that finishes earliest.
```

Why?

An earlier finishing interval leaves more room for future intervals.

---

# 12. Meeting Rooms

Determine whether a person can attend all meetings.

Sort by:

```text
start time
```

Then check adjacent intervals.

If:

```text
current.start < previous.end
```

there is an overlap.

---

# 13. Meeting Rooms — Java

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

# 14. Meeting Rooms II

Find the minimum number of rooms required.

This is equivalent to finding:

```text
maximum number of overlapping intervals.
```

Two major solutions:

```text
1. Min Heap
2. Sweep Line / Two Pointers
```

---

# 15. Meeting Rooms II — Min Heap

Sort meetings by start time.

Maintain:

```text
minimum ending meeting
```

in a min heap.

If the earliest ending meeting finishes before the current meeting starts:

```text
reuse the room.
```

Otherwise:

```text
need another room.
```

---

# 16. Meeting Rooms II — Java

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

    PriorityQueue<Integer> heap =
        new PriorityQueue<>();

    for (int[] interval :
            intervals) {

        int start = interval[0];
        int end = interval[1];

        if (!heap.isEmpty()
                && heap.peek() <= start) {

            heap.poll();
        }

        heap.offer(end);
    }

    return heap.size();
}
```

Complexity:

```text
O(n log n)
```

---

# 17. Meeting Rooms II — Sweep Line

Separate:

```text
start times
end times
```

Sort both arrays.

Use:

```text
start pointer
end pointer
```

If:

```text
start[startPointer]
<
end[endPointer]
```

a new room is needed.

Otherwise:

```text
an existing room becomes free.
```

---

# 18. Interval Intersection

Given:

```text
A = [[0,2],[5,10]]
B = [[1,5],[8,12]]
```

find intersections:

```text
[[1,2],[5,5],[8,10]]
```

Use two pointers.

For intervals:

```text
A[i]
B[j]
```

intersection:

```text
[max(startA,startB),
 min(endA,endB)]
```

exists if:

```text
start <= end
```

---

# 19. Interval Intersection — Java

```java
static int[][] intervalIntersection(
        int[][] first,
        int[][] second) {

    List<int[]> result =
        new ArrayList<>();

    int i = 0;
    int j = 0;

    while (i < first.length
            && j < second.length) {

        int start =
            Math.max(
                first[i][0],
                second[j][0]
            );

        int end =
            Math.min(
                first[i][1],
                second[j][1]
            );

        if (start <= end) {

            result.add(
                new int[]{
                    start,
                    end
                }
            );
        }

        if (first[i][1]
                < second[j][1]) {

            i++;

        } else {

            j++;
        }
    }

    return result.toArray(
        new int[0][]
    );
}
```

Complexity:

```text
O(n + m)
```

after the input intervals are already sorted.

---

# 20. Why Move the Interval That Ends First?

Suppose:

```text
A ends before B.
```

Then A cannot intersect any future interval after its endpoint.

Therefore:

```text
move A
```

This is the same two-pointer principle used in many sorted problems.

---

# 21. Minimum Number of Arrows

Balloons are intervals:

```text
[xStart, xEnd]
```

One arrow can burst all balloons whose intervals contain the arrow position.

Goal:

```text
minimum arrows.
```

Greedy:

```text
Sort by end coordinate.
Shoot at the earliest possible end.
```

---

# 22. Minimum Arrows — Java

```java
static int findMinArrowShots(
        int[][] points) {

    if (points.length == 0) {
        return 0;
    }

    Arrays.sort(
        points,
        Comparator.comparingInt(
            a -> a[1]
        )
    );

    int arrows = 1;
    int position = points[0][1];

    for (int i = 1;
         i < points.length;
         i++) {

        if (points[i][0]
                > position) {

            arrows++;
            position =
                points[i][1];
        }
    }

    return arrows;
}
```

---

# 23. Minimum Platforms

Given train arrival and departure times, find the minimum platforms needed so no train waits.

This is:

```text
maximum overlap
```

of intervals.

Sort:

```text
arrivals
departures
```

Then use two pointers.

---

# 24. Minimum Platforms — Java

```java
static int minPlatforms(
        int[] arrival,
        int[] departure) {

    Arrays.sort(arrival);
    Arrays.sort(departure);

    int i = 0;
    int j = 0;

    int platforms = 0;
    int answer = 0;

    while (i < arrival.length
            && j < departure.length) {

        if (arrival[i]
                <= departure[j]) {

            platforms++;

            answer =
                Math.max(
                    answer,
                    platforms
                );

            i++;

        } else {

            platforms--;
            j++;
        }
    }

    return answer;
}
```

The equality rule depends on whether a train departing at the exact arrival time frees the platform. Always follow the problem's endpoint convention.

---

# 25. Sweep Line

Sweep Line processes intervals as events.

For:

```text
[start, end]
```

create:

```text
start → +1
end   → -1
```

Then process events in sorted coordinate order.

Track:

```text
current overlap
maximum overlap
```

---

# 26. Sweep Line Example

Intervals:

```text
[1,5]
[2,4]
[3,6]
```

Events:

```text
1 → +1
2 → +1
3 → +1
4 → -1
5 → -1
6 → -1
```

Maximum overlap:

```text
3
```

---

# 27. Sweep Line with Event Objects

```java
static int maxOverlap(
        int[][] intervals) {

    List<int[]> events =
        new ArrayList<>();

    for (int[] interval :
            intervals) {

        events.add(
            new int[]{
                interval[0], 1
            }
        );

        events.add(
            new int[]{
                interval[1], -1
            }
        );
    }

    events.sort(
        (a, b) -> {

            if (a[0] != b[0]) {
                return Integer.compare(
                    a[0],
                    b[0]
                );
            }

            return Integer.compare(
                a[1],
                b[1]
            );
        }
    );

    int current = 0;
    int answer = 0;

    for (int[] event : events) {

        current += event[1];

        answer =
            Math.max(
                answer,
                current
            );
    }

    return answer;
}
```

For closed intervals, equal start/end ordering may need to be adjusted according to the problem.

---

# 28. Sweep Line with Difference Array

If coordinates are small:

```text
diff[start] += 1
diff[end] -= 1
```

Then prefix-sum the difference array.

This is often simpler and faster than sorting events.

---

# 29. Difference Array vs Sweep Line

Use a difference array when:

```text
coordinate range is small.
```

Use sweep line when:

```text
coordinate range is huge
or sparse.
```

Example:

```text
coordinates up to 1,000,000,000
```

A direct array is usually inappropriate.

---

# 30. Coordinate Compression

Suppose intervals contain:

```text
10
1000000
1000000000
```

Instead of allocating a billion-sized array, compress coordinates.

Example:

```text
10 → 0
1000000 → 1
1000000000 → 2
```

Then process the compressed representation.

---

# 31. Interval Scheduling with Profit

Each interval has:

```text
start
end
profit
```

Goal:

```text
maximum total profit
```

This is not solved by simple earliest-finish greedy.

It is a classic:

```text
Weighted Interval Scheduling
```

problem.

Use:

```text
DP + Binary Search
```

---

# 32. Weighted Interval Scheduling

Sort intervals by:

```text
end time
```

For each interval `i`, find the latest interval `j` that finishes before `i` starts.

Then:

```text
dp[i]
=
max(
    dp[i - 1],
    profit[i] + dp[j]
)
```

Finding `j` uses binary search.

---

# 33. Weighted Interval Scheduling — Java

```java
static int maxProfit(
        int[][] jobs) {

    Arrays.sort(
        jobs,
        Comparator.comparingInt(
            a -> a[1]
        )
    );

    int n = jobs.length;

    int[] dp =
        new int[n];

    dp[0] = jobs[0][2];

    for (int i = 1;
         i < n;
         i++) {

        int include =
            jobs[i][2];

        int previous =
            findPreviousJob(
                jobs,
                i
            );

        if (previous != -1) {

            include +=
                dp[previous];
        }

        dp[i] =
            Math.max(
                dp[i - 1],
                include
            );
    }

    return dp[n - 1];
}

static int findPreviousJob(
        int[][] jobs,
        int index) {

    int left = 0;
    int right = index - 1;

    int answer = -1;

    while (left <= right) {

        int mid =
            left
            + (right - left) / 2;

        if (jobs[mid][1]
                <= jobs[index][0]) {

            answer = mid;
            left = mid + 1;

        } else {

            right = mid - 1;
        }
    }

    return answer;
}
```

Complexity:

```text
O(n log n)
```

---

# 34. Why Weighted Scheduling Needs DP

Suppose:

```text
Interval A → profit 10
Interval B → profit 20
Interval C → profit 25
```

The interval with the earliest end is not necessarily the highest-profit choice.

A local choice can prevent a combination of future intervals with larger total profit.

Therefore:

```text
Greedy is not sufficient.
```

Use:

```text
DP + Binary Search.
```

---

# 35. Calendar Conflict Detection

Many real-world scheduling problems reduce to:

```text
sort intervals
+
detect overlap.
```

Examples:

```text
Meeting scheduling
Room booking
Interview scheduling
Resource allocation
Calendar conflicts
```

---

# 36. Event Scheduling with Heap

Sometimes intervals arrive in sorted order but we need to choose among available events.

Use:

```text
Sort by start time
+
Min Heap / Max Heap
```

Example pattern:

```text
while events remain:

    add all available events to heap

    choose best available event

    update time
```

---

# 37. Minimum Cost to Hire / Interval-Like Scheduling

Some advanced scheduling problems combine:

```text
sorting
+
heap
+
greedy
```

The exact greedy criterion depends on the objective.

Do not assume:

```text
earliest end
```

works for every scheduling problem.

---

# 38. Interval Partitioning

Interval partitioning asks:

```text
What is the minimum number of groups
needed so that overlapping intervals
are never placed in the same group?
```

Answer:

```text
maximum overlap.
```

Examples:

```text
Meeting Rooms II
Minimum Platforms
Conference rooms
Resource allocation
```

---

# 39. Interval Covering

Another common problem:

```text
Cover a target range using the minimum number
of intervals.
```

Greedy strategy:

```text
Among all intervals starting before
the current uncovered position,
choose the one extending farthest.
```

This is used in:

```text
Jump Game style coverage
Minimum interval covering
Video stitching
```

---

# 40. Video Stitching

Given clips:

```text
[start, end]
```

cover:

```text
[0, time]
```

with the minimum number of clips.

Greedy:

```text
At the current reachable point,
choose the clip that extends coverage
the farthest.
```

---

# 41. Video Stitching — Java

```java
static int videoStitching(
        int[][] clips,
        int time) {

    Arrays.sort(
        clips,
        Comparator.comparingInt(
            a -> a[0]
        )
    );

    int count = 0;
    int currentEnd = 0;
    int farthest = 0;
    int i = 0;

    while (currentEnd < time) {

        while (i < clips.length
                && clips[i][0]
                    <= currentEnd) {

            farthest =
                Math.max(
                    farthest,
                    clips[i][1]
                );

            i++;
        }

        if (farthest == currentEnd) {
            return -1;
        }

        currentEnd = farthest;
        count++;
    }

    return count;
}
```

Complexity:

```text
O(n log n)
```

---

# 42. Interval Covering Pattern

When the question says:

```text
minimum intervals to cover a range
```

think:

```text
sort by start
+
choose farthest reachable end
```

This is a greedy coverage pattern.

---

# 43. Car Pooling

Trips can be represented as:

```text
[start, end, passengers]
```

Use:

```text
difference array
```

or:

```text
sweep line.
```

At start:

```text
+passengers
```

At end:

```text
-passengers
```

Track maximum passengers.

---

# 44. Corporate Flight Bookings

Given bookings:

```text
[first flight, last flight, seats]
```

add seats to all flights in the range.

Difference array:

```text
diff[first] += seats
diff[last + 1] -= seats
```

Then prefix sum.

This is a direct range-update pattern.

---

# 45. Range Addition

Given:

```text
length
updates:
[start, end, value]
```

apply all updates.

Difference array:

```text
diff[start] += value
diff[end + 1] -= value
```

Then:

```text
running += diff[i]
```

---

# 46. Interval Intersection vs Merge

### Merge

Input:

```text
one collection of intervals
```

Goal:

```text
combine overlaps.
```

Usually:

```text
sort by start.
```

### Intersection

Input:

```text
two sorted interval lists.
```

Goal:

```text
find common portions.
```

Usually:

```text
two pointers.
```

---

# 47. Interval Pattern Table

| Problem | Main Technique |
|---|---|
| Merge Intervals | Sort by start |
| Insert Interval | Three-phase scan |
| Non-overlapping Intervals | Greedy by end |
| Activity Selection | Greedy by end |
| Meeting Rooms | Sort by start |
| Meeting Rooms II | Heap / Sweep Line |
| Interval Intersection | Two pointers |
| Minimum Arrows | Greedy by end |
| Minimum Platforms | Two pointers |
| Maximum Overlap | Sweep Line |
| Range Addition | Difference Array |
| Car Pooling | Difference Array / Sweep Line |
| Weighted Scheduling | DP + Binary Search |
| Video Stitching | Greedy coverage |
| Interval Covering | Greedy farthest reach |

---

# 48. Common Interval Sorting Rules

### Sort by start

Use for:

```text
Merge Intervals
Insert Interval
Meeting Rooms
Coverage
```

### Sort by end

Use for:

```text
Activity Selection
Non-overlapping Intervals
Minimum Arrows
Weighted Scheduling
```

### Sort by custom criterion

Use when:

```text
profit
duration
priority
resource usage
```

is the actual optimization criterion.

---

# 49. Endpoint Rules

Always ask:

```text
Can intervals touch?
```

For example:

```text
[1,2]
[2,3]
```

Do they overlap?

If touching is allowed:

```text
2 is compatible.
```

If touching counts as overlap:

```text
they conflict.
```

This affects conditions such as:

```java
start >= end
```

versus:

```java
start > end
```

---

# 50. Half-Open Intervals

Many scheduling systems use:

```text
[start, end)
```

This means:

```text
start included
end excluded
```

Then:

```text
[1,2)
[2,3)
```

do not overlap.

This convention makes back-to-back events easier to handle.

---

# 51. Interval Normalization

Some problems may provide:

```text
[start, end]
```

where:

```text
start > end
```

If the problem allows reversed endpoints, normalize:

```java
int start =
    Math.min(a, b);

int end =
    Math.max(a, b);
```

Do not do this unless the problem definition permits reversed endpoints.

---

# 52. Advanced Pattern: Maximum Overlap

To find maximum overlap:

```text
1. Convert starts to +1.
2. Convert ends to -1.
3. Sort events.
4. Sweep.
5. Track maximum.
```

Complexity:

```text
O(n log n)
```

because of sorting.

---

# 53. Advanced Pattern: Minimum Resources

Questions like:

```text
minimum rooms
minimum platforms
minimum servers
minimum machines
```

often mean:

```text
maximum simultaneous overlap.
```

Think:

```text
Heap
or
Sweep Line
```

---

# 54. Advanced Pattern: Maximum Resources

Questions like:

```text
maximum simultaneous users
maximum concurrent jobs
maximum active sessions
```

also map naturally to:

```text
Sweep Line
```

or:

```text
Difference Array
```

---

# 55. Advanced Pattern: Coverage

Questions like:

```text
minimum intervals to cover [0, T]
```

usually use:

```text
farthest reachable endpoint.
```

At each step:

```text
consider all intervals
that start before current coverage
```

and choose the one that reaches farthest.

---

# 56. Advanced Pattern: Weighted Intervals

Questions like:

```text
maximum profit
maximum value
best collection of non-overlapping jobs
```

should trigger:

```text
Weighted Interval Scheduling
```

Usually:

```text
Sort by end
+
Binary Search previous compatible interval
+
DP
```

---

# 57. Advanced Pattern: Dynamic Interval Updates

If intervals are continuously added or removed and queries ask:

```text
maximum overlap
range sum
range minimum
range maximum
```

a static difference array may not be enough.

Consider:

```text
Segment Tree
Fenwick Tree
Interval Tree
Ordered Map
```

depending on the exact operations.

---

# 58. When Not to Use Simple Sorting

Sorting alone may not be enough when:

```text
intervals have weights
intervals arrive dynamically
updates happen online
queries are interleaved with updates
```

Then consider:

```text
DP
Heap
Segment Tree
Fenwick Tree
Sweep Line
Ordered Map
```

---

# 59. Common Interval Mistakes

### Mistake 1

Sorting by start when the problem requires earliest finish.

### Mistake 2

Using greedy for weighted scheduling.

### Mistake 3

Ignoring endpoint conventions.

### Mistake 4

Forgetting to merge nested intervals.

Example:

```text
[1,10]
[2,3]
```

The merged interval remains:

```text
[1,10]
```

### Mistake 5

Using an array for enormous coordinates.

Use:

```text
Sweep Line
+
Coordinate Compression
```

instead.

---

# 60. Edge Cases

Test:

```text
Empty intervals
One interval
Nested intervals
Identical intervals
Touching intervals
Completely disjoint intervals
One interval containing all others
Negative coordinates
Very large coordinates
Very large endpoint values
```

---

# 61. Complexity Summary

| Problem | Typical Complexity |
|---|---:|
| Merge Intervals | O(n log n) |
| Insert Interval | O(n) |
| Activity Selection | O(n log n) |
| Non-overlapping Intervals | O(n log n) |
| Meeting Rooms | O(n log n) |
| Meeting Rooms II | O(n log n) |
| Interval Intersection | O(n + m) |
| Minimum Arrows | O(n log n) |
| Minimum Platforms | O(n log n) |
| Sweep Line | O(n log n) |
| Weighted Scheduling | O(n log n) |
| Video Stitching | O(n log n) |
| Difference Array | O(n + q) |
| Coordinate Compression | O(n log n) |

---

# 62. Interview Questions — Easy

1. Merge Intervals.
2. Meeting Rooms.
3. Insert Interval.
4. Interval Intersection.
5. Minimum Platforms.

---

# 63. Interview Questions — Medium

6. Non-overlapping Intervals.
7. Meeting Rooms II.
8. Minimum Number of Arrows.
9. Car Pooling.
10. Range Addition.
11. Video Stitching.
12. Corporate Flight Bookings.

---

# 64. Interview Questions — Advanced

13. Weighted Interval Scheduling.
14. Maximum overlap using Sweep Line.
15. Coordinate Compression.
16. Dynamic interval queries.
17. Interval covering.
18. Minimum resources with dynamic events.
19. Segment Tree interval problems.

---

# 65. Interval Decision Tree

```text
Given intervals
      |
      +--------------------------+
      |                          |
 One collection             Two collections
      |                          |
      ↓                          ↓
Sort by start              Two pointers
      |
      +----------------------+
      |                      |
Merge/scan              Optimization
      |                      |
      ↓                      ↓
Intervals              Sort by end?
                           |
                      +----+----+
                      |         |
                     Yes        No
                      |         |
                  Greedy       DP/Heap
```

---

# 66. Advanced Interval Decision Tree

```text
What is being asked?
        |
        +-------------------------------+
        |               |               |
     Merge          Maximum overlap   Maximum profit
        |               |               |
 Sort by start     Sweep Line/Heap   Weighted DP
                                        |
                                        ↓
                              Binary Search previous
                                  compatible job
```

---

# 67. Quick Revision

```text
Intervals
│
├── Sorting
│   ├── By Start
│   └── By End
│
├── Greedy
│   ├── Activity Selection
│   ├── Non-overlapping
│   ├── Minimum Arrows
│   └── Coverage
│
├── Heap
│   └── Meeting Rooms II
│
├── Two Pointers
│   └── Interval Intersection
│
├── Sweep Line
│   └── Maximum Overlap
│
├── Difference Array
│   └── Range Updates
│
├── DP + Binary Search
│   └── Weighted Scheduling
│
└── Advanced
    ├── Coordinate Compression
    ├── Dynamic Intervals
    └── Segment Tree
```

---

# 68. Most Important Interval Rules

```text
Merge intervals
→ Sort by START

Maximum non-overlapping intervals
→ Sort by END

Minimum arrows
→ Sort by END

Meeting rooms
→ Sort by START

Minimum resources
→ Maximum overlap

Two interval lists
→ Two pointers

Range updates
→ Difference Array

Sparse coordinates
→ Sweep Line

Weighted intervals
→ DP + Binary Search

Minimum coverage
→ Greedy farthest reach
```

---

# 69. Interview Explanation Template

For a merge problem:

```text
I will sort the intervals by start time.

Then I maintain the current merged interval.

If the next interval overlaps with the current interval,
I extend the end.

Otherwise, I store the current interval and start a new one.

Sorting takes O(n log n), and the scan is O(n),
so the overall complexity is O(n log n).
```

For maximum overlap:

```text
I convert each interval into start and end events.

A start increases the active count and an end decreases it.

After sorting the events, I sweep from left to right
and keep track of the maximum active count.
```

For weighted scheduling:

```text
I sort jobs by end time.

For each job, I binary search for the latest compatible
previous job.

Then DP decides whether to skip the current job or
take it together with the best compatible solution.
```

---

# 70. Final Interview Rule

> **For intervals, first identify what the problem is really asking: merge ranges, select non-overlapping ranges, count overlaps, cover a target range, or maximize weighted profit. That decision usually tells you whether to use sorting, greedy, heap, sweep line, or DP.**
