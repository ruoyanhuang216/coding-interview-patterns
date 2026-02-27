# Fast & Slow Pointers

## What Is It?

This pattern uses two pointers moving through an iterable data structure at **different speeds** — typically, the slow pointer moves one step at a time while the fast pointer moves two steps. The speed differential creates mathematical guarantees about where the pointers will meet, enabling cycle detection and midpoint finding without extra memory.

## When to Use It

- **Finding the middle** — You need the exact midpoint of a linked list in a single pass
- **Cycle detection** — You need to know if a linked list or sequence of transformations loops back on itself
- **O(1) space constraint** — You're forbidden from using a HashSet to track visited nodes
- **"Array as a graph" problems** — Values are restricted to a range, meaning they can act as indices pointing to other elements

## Real-Life Analogies

| Analogy | Maps to |
|---------|---------|
| **Runners on a track** — One runs at 5 mph, the other at 10 mph. On a straight line, the fast runner finishes and leaves. On a circular track, the fast runner eventually laps the slow one. | Cycle detection |
| **Folding a towel** — Pull one end twice as fast as the other. When the fast end reaches the slow end, the fold is exactly in the middle. | Finding the middle |

## The Math: Floyd's Cycle Detection Algorithm

Understanding *why* the pointers meet — not just *that* they do — is critical for staff-level interviews.

**Let:**
- `L` = distance from head to cycle start
- `C` = cycle circumference
- `x` = distance from cycle start to meeting point

**When slow and fast collide:**
1. Distance of slow: `L + x`
2. Distance of fast: `L + x + nC` (fast lapped the cycle `n` times)

Since fast moves at 2x speed:

```
L + x + nC = 2(L + x)
         nC = L + x
          L = nC - x
```

**The key insight**: `L = nC - x` means the distance from the **head to cycle start** equals the distance from the **meeting point to cycle start** (walking the remaining loop). So if you place one pointer at the head and one at the meeting point, advancing both at speed 1, they collide exactly at the cycle entrance.

---

## Archetypes

### Group 1: The "Find the Middle" Foundation

| # | Problem | Difficulty |
|---|---------|------------|
| 234 | Palindrome Linked List | Easy |
| 876 | Middle of the Linked List | Easy |
| 2130 | Maximum Twin Sum of a Linked List | Medium |
| 2674 | Split a Circular Linked List | Medium |

**Interview logistics**: Both pointers start at the head. `slow` moves 1 step, `fast` moves 2 steps. When `fast` reaches the end (`None`) or the last node, `slow` is sitting exactly on the middle node. In FAANG interviews, this is almost never the whole problem — it's Step 1. Step 2 usually involves reversing the second half starting from `slow`, and Step 3 involves comparing or merging the two halves (palindrome check, twin sum, etc.).

### Group 2: Explicit Cycle Detection

| # | Problem | Difficulty |
|---|---------|------------|
| 141 | Linked List Cycle | Easy |
| 142 | Linked List Cycle II | Medium |
| 202 | Happy Number | Easy |

**Interview logistics**: You're checking for loops. For linked lists, it's standard Floyd's. For math transformations like Happy Number (202), treat the mathematical operation `sum_of_squares(n)` as the `.next` pointer. If `slow == fast`, a cycle exists. If `fast` hits 1 or `None`, no cycle.

### Group 3: Disguised Cycles (Indices as Pointers)

| # | Problem | Difficulty |
|---|---------|------------|
| 287 | Find the Duplicate Number | Medium |
| 457 | Circular Array Loop | Medium |

**Interview logistics**: Hard/Medium arrays disguised as linked lists. The value at `array[i]` tells you the next index to jump to. Use Floyd's algorithm to find the cycle. In 287 (Duplicate Number), multiple nodes point to the same index, creating a cycle at the duplicate. In 457 (Circular Array), you must also enforce single-direction movement and reject self-loops.

---

## Problem Checklist

### Group 1: The "Find the Middle" Foundation

---

#### LC 876 — Middle of the Linked List `Easy`

**The spiel**: "To find the middle in a single pass without extra memory, I use a fast and slow pointer. The slow pointer advances one step at a time, while the fast pointer advances two. Because fast travels at twice the speed, by the time it reaches the end, slow will be situated exactly at the halfway mark."

**Complexity**: Time O(N), Space O(1).

```python
def middleNode(head):
    slow = fast = head

    # Fast checks its own position and the next to avoid null pointer errors
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow  # Slow is exactly at the middle
```

---

#### LC 234 — Palindrome Linked List `Easy`

**The spiel**: "I will break this into three steps to achieve O(1) space. First, I use fast/slow pointers to find the middle. Second, I reverse the second half in-place. Finally, I use two pointers — one at the head and one at the start of the reversed second half — and compare values to verify symmetry."

**Complexity**: Time O(N), Space O(1).

```python
def isPalindrome(head) -> bool:
    # 1. Find Middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # 2. Reverse Second Half
    prev = None
    curr = slow
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt

    # 3. Compare Halves (prev is now the head of the reversed half)
    left, right = head, prev
    while right:  # Right half might be shorter if odd length
        if left.val != right.val:
            return False
        left = left.next
        right = right.next

    return True
```

---

#### LC 2130 — Maximum Twin Sum of a Linked List `Medium`

**The spiel**: "The 'twins' mirror a palindrome structure — first pairs with last, second with second-to-last. To pair them up without O(N) extra space, I use fast/slow pointers to find the midpoint, reverse the second half, then iterate both halves simultaneously maintaining a running maximum of their summed values."

**Complexity**: Time O(N), Space O(1).

```python
def pairSum(head) -> int:
    # 1. Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # 2. Reverse second half
    prev, curr = None, slow
    while curr:
        curr.next, prev, curr = prev, curr, curr.next

    # 3. Iterate and find max sum
    max_sum = 0
    left, right = head, prev
    while right:
        max_sum = max(max_sum, left.val + right.val)
        left, right = left.next, right.next

    return max_sum
```

---

#### LC 2674 — Split a Circular Linked List `Medium`

**The spiel**: "I use fast/slow pointers adapted for a circular list — instead of checking for `None`, I stop when `fast.next` or `fast.next.next` loops back to the head. When slow reaches the midpoint, I split by making `slow.next` the head of the second list. I then close both halves into their own circular loops: `slow` points back to the original head, and the original tail (tracked via `fast`) points back to the second head."

**Complexity**: Time O(N), Space O(1).

```python
def splitCircularLinkedList(head):
    slow = fast = head

    # Adapt the standard loop for circular lists
    while fast.next != head and fast.next.next != head:
        slow = slow.next
        fast = fast.next.next

    # If even number of nodes, advance fast one more step to reach the tail
    if fast.next.next == head:
        fast = fast.next

    # Now: slow = end of first half, fast = tail of original circular list
    head2 = slow.next
    slow.next = head    # Close first circular list
    fast.next = head2   # Close second circular list

    return [head, head2]
```

---

### Group 2: Explicit Cycle Detection

---

#### LC 141 — Linked List Cycle `Easy`

**The spiel**: "I will use Floyd's Cycle Detection algorithm. By initializing a slow pointer moving one step and a fast pointer moving two steps, I guarantee that if a cycle exists, the fast pointer will eventually lap the slow pointer and they will point to the same node. If the fast pointer reaches a null terminator, the list is linear."

**Complexity**: Time O(N), Space O(1).

```python
def hasCycle(head) -> bool:
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True  # Lapped!

    return False
```

---

#### LC 142 — Linked List Cycle II `Medium`

**The spiel**: "I detect the cycle using Floyd's algorithm. Once they meet, I rely on the mathematical proof that the distance from the head to the cycle's start equals the distance from the meeting point to the cycle's start. I leave one pointer at the meeting point, reset the other to the head, and move both at speed 1. They collide exactly at the start of the cycle."

**Complexity**: Time O(N), Space O(1).

```python
def detectCycle(head):
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            # Cycle found — apply the mathematical proof
            pointer1 = head
            pointer2 = slow
            while pointer1 != pointer2:
                pointer1 = pointer1.next
                pointer2 = pointer2.next
            return pointer1  # The start of the cycle

    return None
```

---

#### LC 202 — Happy Number `Easy`

**The spiel**: "This is cycle detection in disguise. Instead of traversing node pointers, the 'next node' is generated by the sum-of-squares operation. I use fast and slow pointers, applying the operation once for slow and twice for fast. If fast reaches 1, the number is happy. If they meet at any other number, we're trapped in a cycle."

**Complexity**: Time O(log N), Space O(1).

```python
def isHappy(n: int) -> bool:
    def get_next(number):
        return sum(int(digit) ** 2 for digit in str(number))

    slow = n
    fast = get_next(n)

    while fast != 1 and slow != fast:
        slow = get_next(slow)          # 1 step
        fast = get_next(get_next(fast)) # 2 steps

    return fast == 1
```

---

### Group 3: Disguised Cycles (Indices as Pointers)

---

#### LC 287 — Find the Duplicate Number `Medium`

**The spiel**: "Because the array contains N+1 integers strictly in the range [1, N], we can treat array values as 'next' pointers (index -> nums[index]). A duplicate means multiple indices point to the same target, creating a cycle. I use Floyd's algorithm to find the intersection point, then apply phase two to find the cycle entrance, which corresponds to the duplicate number."

**Complexity**: Time O(N), Space O(1).

```python
def findDuplicate(nums: list[int]) -> int:
    # Phase 1: Detect intersection point
    slow = fast = nums[0]
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]
        if slow == fast:
            break

    # Phase 2: Find the cycle entrance (the duplicate number)
    slow2 = nums[0]
    while slow != slow2:
        slow = nums[slow]
        slow2 = nums[slow2]

    return slow
```

---

#### LC 457 — Circular Array Loop `Medium`

**The spiel**: "This requires cycle detection where array values dictate relative jumps. For every unvisited index, I launch a fast/slow traversal. I enforce two strict invariants: movement direction must remain constant, and cycle length must be > 1 (no self-loops). To prevent O(N^2) worst-case, once a path is confirmed invalid, I execute a pruning step to mark all nodes on that path as 'dead ends' (0), ensuring O(N) amortized time."

**Complexity**: Time O(N), Space O(1).

```python
def circularArrayLoop(nums: list[int]) -> bool:
    n = len(nums)

    def next_step(i):
        return (i + nums[i]) % n

    for i in range(n):
        if nums[i] == 0:
            continue

        slow = fast = i
        is_forward = nums[i] > 0

        # Check direction invariant while advancing
        while (nums[fast] != 0 and (nums[fast] > 0) == is_forward and
               nums[next_step(fast)] != 0 and (nums[next_step(fast)] > 0) == is_forward):
            slow = next_step(slow)
            fast = next_step(next_step(fast))

            if slow == fast:
                if slow == next_step(slow):  # Self-loop check
                    break
                return True

        # Pruning: mark failed path as 0
        slow = i
        while nums[slow] != 0 and (nums[slow] > 0) == is_forward:
            nxt = next_step(slow)
            nums[slow] = 0
            slow = nxt

    return False
```
