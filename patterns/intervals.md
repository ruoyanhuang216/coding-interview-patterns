# Intervals

## What Is It?

The Intervals pattern reasons about problems involving ranges of values — time spans, numeric ranges, or geometric spans. Each interval is defined by `[start, end]`. Most problems reduce to sorting the intervals by one endpoint, then scanning left-to-right while maintaining some state (a merge boundary, a heap of active ends, a running count).

## When to Use It

- **Input is a list of `[start, end]` pairs** — meetings, time ranges, balloon positions
- **You need to merge, split, or count overlapping ranges**
- **Scheduling / resource allocation** — "how many rooms", "can we fit all events"
- **A greedy selection over ranges** — maximize non-overlapping, minimize removals

## Real-Life Analogies

| Analogy | Maps to |
|---------|---------|
| **Combining overlapping calendar events** — If two meetings overlap, merge them into a single block on your calendar | Merge intervals |
| **Booking conference rooms** — Each meeting needs a room; if two overlap, you need two rooms. Track the peak concurrency. | Sweep line / scheduling |
| **Shooting arrows at balloons on a wall** — Shoot at the rightmost point of the first balloon to pop as many overlapping ones as possible | Greedy (sort by end) |

## Key Techniques

**1. Sort by START (95% of problems)**: Ensures `next.start >= current.start`, letting you process intervals left-to-right in a single pass. Use for merging, inserting, finding intersections, and tracking concurrency.

**2. Sort by END (greedy selection)**: When you need to *maximize* non-overlapping intervals or *minimize* removals, pick the event that finishes earliest — this leaves maximum room for future events.

**3. Sweep Line Decomposition**: Break `[start, end]` intervals into `+1` and `-1` point events. Sort all events chronologically, then scan left-to-right with a running total. Detects peak concurrency without comparing intervals pairwise.

**4. Heap vs Deque**: Use a **min-heap** when you need the *earliest ending* active interval (scheduling, k-way merge). Use a **deque** when elements leave in the same order they arrived (sliding window). The heap costs O(log N) per operation but handles arbitrary removal order; the deque is O(1) but only works FIFO.

---

## Archetypes

### Group 1: Merge, Insert & Scan (Sort by Start)

| # | Problem | Difficulty |
|---|---------|------------|
| 56 | Merge Intervals | Medium |
| 57 | Insert Interval | Medium |
| 228 | Summary Ranges | Easy |
| 986 | Interval List Intersections | Medium |
| 1288 | Remove Covered Intervals | Medium |

**Interview logistics**: Sort by start time. Scan left-to-right maintaining one or two tracking variables (typically `current_end` or a result list's last entry). For merging, compare `current.start` against the previous `end` to decide merge vs append. For intersections between two sorted lists, use two pointers and advance whichever interval ends first.

### Group 2: Sweep Line & Scheduling

| # | Problem | Difficulty |
|---|---------|------------|
| 252 | Meeting Rooms | Easy |
| 253 | Meeting Rooms II | Medium |
| 1094 | Car Pooling | Medium |

**Interview logistics**: For simple overlap detection (252), sort by start and check adjacent pairs. For counting peak concurrency (253), use a min-heap of end times — pop ended meetings before pushing new ones; heap size = rooms needed. For weighted events (1094), decompose into `(time, +delta)` and `(time, -delta)` events, sort, and sweep a running total.

### Group 3: Greedy Selection (Sort by End)

| # | Problem | Difficulty |
|---|---------|------------|
| 435 | Non-overlapping Intervals | Medium |
| 452 | Minimum Number of Arrows to Burst Balloons | Medium |

**Interview logistics**: Sort by **end time**. Greedily keep each interval that doesn't overlap with the last kept one (i.e., `current.start >= last_kept_end`). If it overlaps, skip/remove it. This maximizes the count of non-overlapping intervals because locking in the earliest finish leaves the most room.

### Group 4: K-Way Merge with Heap

| # | Problem | Difficulty |
|---|---------|------------|
| 759 | Employee Free Time | Hard |

**Interview logistics**: Multiple sorted interval lists. Push the first interval from each list into a min-heap (keyed by start time). Pop the earliest, check for gaps against `max_end_so_far`, then push the next interval from that same list. This processes N total intervals in O(N log K) where K is the number of lists.

---

## Problem Checklist

### Group 1: Merge, Insert & Scan (Sort by Start)

---

#### LC 56 — Merge Intervals `Medium`

**The spiel**: "I sort the intervals by start time, then scan left-to-right. I maintain a result list and compare each interval against the last merged interval. If the current interval's start is beyond the last merged end, there's a gap — I append it as a new interval. If they overlap, I extend the last merged interval's end using `max()`. This handles nested and partially overlapping intervals uniformly."

**Complexity**: Time O(N log N), Space O(N).

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    intervals.sort(key=lambda x: x[0])

    merged = [intervals[0]]

    for i in range(1, len(intervals)):
        # Gap — no overlap with the last merged interval
        if intervals[i][0] > merged[-1][1]:
            merged.append(intervals[i])
        # Overlap — extend the end of the last merged interval
        else:
            merged[-1][1] = max(merged[-1][1], intervals[i][1])

    return merged
```

---

#### LC 57 — Insert Interval `Medium`

**The spiel**: "The input is already sorted and non-overlapping, so I don't need to sort. I use a three-phase linear sweep. Phase 1: append all intervals that end strictly before the new interval starts (no overlap, strictly left). Phase 2: merge all overlapping intervals into the new interval by expanding its start and end with `min()`/`max()`. Phase 3: append all remaining intervals (strictly right). This is O(N) since the input is pre-sorted."

**Complexity**: Time O(N), Space O(N).

```python
def insert(intervals: list[list[int]], newInterval: list[int]) -> list[list[int]]:
    merged = []
    i, n = 0, len(intervals)

    # Phase 1: Strictly left — no overlap
    while i < n and intervals[i][1] < newInterval[0]:
        merged.append(intervals[i])
        i += 1

    # Phase 2: Merge zone — swallow all overlapping intervals
    while i < n and intervals[i][0] <= newInterval[1]:
        newInterval[0] = min(newInterval[0], intervals[i][0])
        newInterval[1] = max(newInterval[1], intervals[i][1])
        i += 1
    merged.append(newInterval)

    # Phase 3: Strictly right — no overlap
    while i < n:
        merged.append(intervals[i])
        i += 1

    return merged
```

---

#### LC 228 — Summary Ranges `Easy`

**The spiel**: "I scan left-to-right, tracking the start of each consecutive run. I keep advancing while the next number is exactly one more than the current. When the run breaks, I format the range as either a single number or `start->end` and move to the next run."

**Complexity**: Time O(N), Space O(1) excluding output.

```python
def summaryRanges(nums: list[int]) -> list[str]:
    res = []
    i = 0

    while i < len(nums):
        start = nums[i]

        # Extend the run while consecutive
        while i + 1 < len(nums) and nums[i + 1] == nums[i] + 1:
            i += 1

        if nums[i] == start:
            res.append(str(start))
        else:
            res.append(f"{start}->{nums[i]}")
        i += 1

    return res
```

---

#### LC 986 — Interval List Intersections `Medium`

**The spiel**: "Two sorted, non-overlapping lists. I use two pointers, one per list. For each pair, the intersection start is `max(start_i, start_j)` and the intersection end is `min(end_i, end_j)`. If start <= end, the intersection is valid and I append it. Then I advance the pointer whose interval ends earlier — that interval can't intersect with anything else."

**Step-by-Step Example:**
`L1: [[0, 4], [7, 12]]`, `L2: [[1, 5], [8, 10]]`
1. Compare `[0, 4]` and `[1, 5]`. Max start is `1`, Min end is `4`. `1 <= 4`, append `[1, 4]`. Because `4 < 5`, `L1` finishes first. Move `L1` pointer `i += 1`.
2. Compare `[7, 12]` and `[1, 5]`. Max start is `7`, Min end is `5`. `7 > 5` (Invalid). Because `5 < 12`, `L2` finishes first. Move `L2` pointer `j += 1`.
3. Compare `[7, 12]` and `[8, 10]`. Max start is `8`, Min end is `10`. `8 <= 10`, append `[8, 10]`. Move `L2` pointer `j += 1`. Loop ends.

**Complexity**: Time O(M + N), Space O(M + N) for the result.

```python
def intervalIntersection(firstList: list[list[int]], secondList: list[list[int]]) -> list[list[int]]:
    intersections = []
    i, j = 0, 0

    while i < len(firstList) and j < len(secondList):
        start_max = max(firstList[i][0], secondList[j][0])
        end_min = min(firstList[i][1], secondList[j][1])

        # Valid intersection exists
        if start_max <= end_min:
            intersections.append([start_max, end_min])

        # Advance the pointer whose interval finishes earlier
        if firstList[i][1] < secondList[j][1]:
            i += 1
        else:
            j += 1

    return intersections
```

---

#### LC 1288 — Remove Covered Intervals `Medium`

**The spiel**: "An interval `[a, b]` is covered by `[c, d]` if `c <= a` and `b <= d`. I sort by start ascending, then by end *descending* — so for the same start, the longest interval comes first. Then I scan left-to-right tracking `max_end`. If the current interval's end exceeds `max_end`, it's not covered by anything before it, so I count it and update `max_end`. Otherwise it's covered and I skip it."

**Complexity**: Time O(N log N), Space O(1).

```python
def removeCoveredIntervals(intervals: list[list[int]]) -> int:
    # Sort by start ascending; for ties, sort by end descending
    intervals.sort(key=lambda x: (x[0], -x[1]))

    count = 0
    max_end = 0

    for _, end in intervals:
        if end > max_end:
            count += 1     # Not covered — this interval extends further
            max_end = end
        # else: covered by a previous interval (skip)

    return count
```

---

### Group 2: Sweep Line & Scheduling

---

#### LC 252 — Meeting Rooms `Easy`

**The spiel**: "I sort the meetings by start time. Then I check each adjacent pair — if any meeting starts before the previous one ends, there's a conflict and the person can't attend all meetings. If I get through the entire list without conflicts, return true."

**Complexity**: Time O(N log N), Space O(1).

```python
def canAttendMeetings(intervals: list[list[int]]) -> bool:
    intervals.sort(key=lambda x: x[0])

    for i in range(1, len(intervals)):
        # Current meeting starts before the previous one ends
        if intervals[i][0] < intervals[i - 1][1]:
            return False

    return True
```

---

#### LC 253 — Meeting Rooms II `Medium`

**The spiel**: "I need the peak number of concurrent meetings. I sort by start time, then use a min-heap tracking the end times of active meetings. For each new meeting, if the earliest-ending active meeting has already finished (heap top <= current start), I pop it — that room is freed. Then I push the current meeting's end time. The heap size at any point equals the number of rooms in use; I track the maximum."

**Complexity**: Time O(N log N), Space O(N).

```python
import heapq

def minMeetingRooms(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[0])

    free_rooms = []  # Min-heap of end times for active meetings

    for start, end in intervals:
        # A room is freed if the earliest-ending meeting is done
        if free_rooms and free_rooms[0] <= start:
            heapq.heappop(free_rooms)

        heapq.heappush(free_rooms, end)

    return len(free_rooms)
```

**Mathematical proof (sweep line equivalence)**: At any instant `t`, the number of rooms in use equals the number of meetings that have started by `t` minus the number that have ended by `t`:

> `active_meetings(t) = started_by(t) − ended_by(t)`

If we sort all start times and all end times into two separate arrays, we can sweep them with two pointers `i` and `j`. When `starts[i] < ends[j]`, a new meeting begins before the earliest active meeting ends — we need an additional room. When `starts[i] >= ends[j]`, a meeting has ended — a room is freed and we advance `j`. The peak value of `i − j` over the sweep equals the maximum concurrency.

```python
def minMeetingRooms_sweep(intervals: list[list[int]]) -> int:
    starts = sorted(s for s, _ in intervals)
    ends = sorted(e for _, e in intervals)

    rooms = 0
    max_rooms = 0
    j = 0  # Pointer into ends[]

    for i in range(len(starts)):
        if starts[i] < ends[j]:
            # A meeting starts before the earliest end — need another room
            rooms += 1
        else:
            # A meeting ended — reuse that room
            j += 1

        max_rooms = max(max_rooms, rooms)

    return max_rooms
```

---

#### LC 1094 — Car Pooling `Medium`

**The spiel**: "I decompose each trip into two point events: passengers board at the start location (+delta) and exit at the end location (-delta). I sort all events by location. At ties, drop-offs (negative) naturally sort before pickups (positive), which correctly frees capacity before loading. Then I sweep left-to-right with a running total — if it ever exceeds capacity, return false."

**Complexity**: Time O(N log N), Space O(N).

```python
def carPooling(trips: list[list[int]], capacity: int) -> bool:
    events = []

    for num_passengers, start, end in trips:
        events.append((start, num_passengers))   # Pickup
        events.append((end, -num_passengers))     # Drop-off

    # Sort by location; at ties, drop-offs (-) process before pickups (+)
    events.sort()

    current_passengers = 0
    for _, passenger_change in events:
        current_passengers += passenger_change
        if current_passengers > capacity:
            return False

    return True
```

---

### Group 3: Greedy Selection (Sort by End)

---

#### LC 435 — Non-overlapping Intervals `Medium`

**The spiel**: "I sort by end time to apply the greedy 'earliest finish' strategy. I keep the first interval and track its end. For each subsequent interval: if it starts before the current end, it overlaps — I remove it (increment count). If it doesn't overlap, I keep it and update the current end. Sorting by end ensures the intervals we keep leave maximum room for future intervals."

**Step-by-Step Example:** `[[1, 2], [2, 3], [3, 4], [1, 3]]`
1. **Sort by End Time:** `[[1, 2], [2, 3], [1, 3], [3, 4]]`
2. **Keep `[1, 2]`:** `current_end = 2`.
3. **Check `[2, 3]`:** Start `2 >= current_end 2`. No overlap. Keep it. `current_end = 3`.
4. **Check `[1, 3]`:** Start `1 < current_end 3`. Overlap! Because we sorted by end time, the ones we kept are mathematically better. Drop this one. `removals += 1`.
5. **Check `[3, 4]`:** Start `3 >= current_end 3`. No overlap. Keep it. `current_end = 4`.
*Result: 1 removal.*

**Complexity**: Time O(N log N), Space O(1).

```python
def eraseOverlapIntervals(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda x: x[1])  # Sort by END time

    removal_count = 0
    current_end = intervals[0][1]

    for i in range(1, len(intervals)):
        if intervals[i][0] < current_end:
            # Overlap — remove this interval (the kept one ends earlier)
            removal_count += 1
        else:
            # No overlap — keep this interval
            current_end = intervals[i][1]

    return removal_count
```

---

#### LC 452 — Minimum Number of Arrows to Burst Balloons `Medium`

**The spiel**: "Same greedy logic as Non-overlapping Intervals. I sort balloons by end position. I shoot my first arrow at the end of the first balloon — this maximizes collateral by reaching as far right as possible within that balloon. For each subsequent balloon: if its start is beyond my last arrow position, it wasn't hit, so I need a new arrow (shot at its end). Otherwise my existing arrow already covers it."

**Complexity**: Time O(N log N), Space O(1).

```python
def findMinArrowShots(points: list[list[int]]) -> int:
    points.sort(key=lambda x: x[1])  # Sort by END position

    arrows = 1
    current_arrow = points[0][1]  # Shoot at the end of the first balloon

    for i in range(1, len(points)):
        # This balloon starts after our last arrow — need a new one
        if points[i][0] > current_arrow:
            arrows += 1
            current_arrow = points[i][1]

    return arrows
```

---

### Group 4: K-Way Merge with Heap

---

#### LC 759 — Employee Free Time `Hard`

**The spiel**: "I have K employees, each with a sorted schedule. Instead of flattening all intervals and sorting in O(N log N), I use a min-heap holding one interval per employee — always the earliest unprocessed one. I track `max_end_so_far` across all processed intervals. When I pop an interval whose start exceeds `max_end_so_far`, I've found a gap — that's free time for everyone. I then push the next interval from that employee's schedule. This runs in O(N log K)."

**Complexity**: Time O(N log K) where N is total intervals and K is number of employees. Space O(K).

```python
import heapq

def employeeFreeTime(schedule: list[list[list[int]]]) -> list[list[int]]:
    min_heap = []

    # Push the first interval of each employee: (start_time, emp_idx, interval_idx)
    for emp_idx, emp_schedule in enumerate(schedule):
        heapq.heappush(min_heap, (emp_schedule[0][0], emp_idx, 0))

    free_time = []
    _, first_emp, first_idx = min_heap[0]
    max_end_so_far = schedule[first_emp][first_idx][1]

    while min_heap:
        _, emp_idx, interval_idx = heapq.heappop(min_heap)
        current_interval = schedule[emp_idx][interval_idx]

        # Gap detected — everyone is free during this window
        if current_interval[0] > max_end_so_far:
            free_time.append([max_end_so_far, current_interval[0]])

        # Update the furthest right we've reached
        max_end_so_far = max(max_end_so_far, current_interval[1])

        # Push this employee's next interval
        if interval_idx + 1 < len(schedule[emp_idx]):
            next_start = schedule[emp_idx][interval_idx + 1][0]
            heapq.heappush(min_heap, (next_start, emp_idx, interval_idx + 1))

    return free_time
```
