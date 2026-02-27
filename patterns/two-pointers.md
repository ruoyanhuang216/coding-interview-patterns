# Two Pointers

## What Is It?

The Two Pointers pattern processes linear data structures (arrays, strings, linked lists) by using two integer variables (pointers) to traverse the data simultaneously. The core philosophy is **search space reduction** — by leveraging the order or structure of the data, you intelligently move the pointers to skip unnecessary comparisons, reducing an O(N^2) or O(N^3) brute-force down to O(N) or O(N log N).

## When to Use It

- **The data is linear** — Array, String, or Linked List
- **Order matters** — The data is sorted, or you're looking for subarrays, palindromes, or specific sequential patterns
- **Pairs or boundaries are involved** — Finding two numbers that sum to a target, reversing elements, or tracking boundaries of a partition
- **O(1) auxiliary space is required** — In-place modifications

## Real-Life Analogies

| Analogy | Maps to |
|---------|---------|
| **Organizing a bookshelf** — Scan books one-by-one (read pointer); every time you find an unread book, place it at the next available slot on the left (write pointer) | Read/Write pattern |
| **Finding a meeting time** — You and a colleague both have sorted free-time lists. Whoever's slot ends earlier moves to their next slot | Two pointers on two lists |
| **Traffic flow** — A sensor counts cars in the fast lane for every one car in the slow lane | Fast/Slow spacing |

---

## Archetypes

### Group 1: Opposite Direction (Converging / Symmetry)

| # | Problem | Difficulty |
|---|---------|------------|
| 15 | 3Sum | Medium |
| 151 | Reverse Words in a String | Medium |
| 246 | Strobogrammatic Number | Easy |
| 344 | Reverse String | Easy |
| 977 | Squares of a Sorted Array | Easy |
| 2193 | Minimum Number of Moves to Make Palindrome | Hard |
| 2824 | Count Pairs Whose Sum is Less Than Target | Easy |

**Interview logistics**: Place `left` at index 0 and `right` at the end. Maintain a strict invariant — evaluate the condition and move pointers inward based on magnitude or a matching condition. The loop terminates when `left >= right`. If the data isn't sorted and magnitude matters (like 3Sum), sort first.

### Group 2: Same Direction (Read / Write / Partition)

| # | Problem | Difficulty |
|---|---------|------------|
| 27 | Remove Element | Easy |
| 75 | Sort Colors | Medium |
| 283 | Move Zeroes | Easy |
| 443 | String Compression | Medium |

**Interview logistics**: Both pointers start at 0. The fast (read) pointer scans unconditionally. The slow (write) pointer tracks the boundary of "processed" valid data. Only advance the write pointer when the read pointer finds an element that belongs in the processed section. For partitioning (like Sort Colors), use three pointers for three distinct regions.

### Group 3: Two Pointers on Two Iterables (Matching / Subsequence)

| # | Problem | Difficulty |
|---|---------|------------|
| 321 | Create Maximum Number | Hard |
| 408 | Valid Word Abbreviation | Easy |
| 1537 | Get the Maximum Score | Hard |
| 2486 | Append Characters to String to Make Subsequence | Medium |

**Interview logistics**: Place `pointer_a` on the first string/array and `pointer_b` on the second. Usually matching a subsequence or merging sorted data. Advance `pointer_a` to scan the main data, advance `pointer_b` *only* on a successful match. If `pointer_b` reaches the end, you've found the subsequence.

### Group 4: Linked List & Reset Tricks

| # | Problem | Difficulty |
|---|---------|------------|
| 19 | Remove Nth Node From End of List | Medium |
| 160 | Intersection of Two Linked Lists | Easy |
| 1650 | Lowest Common Ancestor of a Binary Tree III | Medium |

**Interview logistics**: You can't go backward in a singly linked list. To find the Nth node from the end, give the fast pointer an N-step head start, then move both together. For intersection of two lists with different lengths, when a pointer reaches the end of its list, reset it to the head of the *other* list to equalize traversal distances.

### Group 5: Boundary Expansion & Suffix Scanning (Greedy)

| # | Problem | Difficulty |
|---|---------|------------|
| 31 | Next Permutation | Medium |
| 189 | Rotate Array | Medium |
| 763 | Partition Labels | Medium |
| 1842 | Next Palindrome Using Same Digits | Hard |
| 2193 | Minimum Number of Moves to Make Palindrome | Hard |
| 2444 | Count Subarrays With Fixed Bounds | Hard |

**Interview logistics**: The hardest tier. Instead of simple convergence, you're manipulating boundaries. For string partitions (763), track the maximum last occurrence of characters to define the "cut". For permutations (31, 1842), scan right-to-left to find the first decrease (the pivot), swap it, and reverse the suffix.

---

## Problem Checklist

### Group 1: Opposite Direction (Converging / Symmetry)

---

#### LC 15 — 3Sum `Medium`

**The spiel**: "To avoid an O(N^3) brute-force approach, I will sort the array first. This allows me to fix one number and reduce the remaining problem to a standard Two-Sum on a sorted array. For each fixed element, I'll use converging left and right pointers to find pairs that sum to the negative of my fixed element. Crucially, I will skip duplicate values for both my fixed element and my moving pointers to avoid duplicate triplets."

**Complexity**: Time O(N^2) — sorting is O(N log N) plus N iterations of an O(N) two-pointer search. Space O(1) to O(N) depending on the sort.

```python
def threeSum(nums: list[int]) -> list[list[int]]:
    res = []
    nums.sort()  # Sorting unlocks the two-pointer logic

    for i in range(len(nums)):
        # Skip duplicate fixed numbers
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        l, r = i + 1, len(nums) - 1
        while l < r:
            total = nums[i] + nums[l] + nums[r]
            if total > 0:
                r -= 1
            elif total < 0:
                l += 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                r -= 1
                # Skip duplicate left values to prevent duplicate triplets
                while l < r and nums[l] == nums[l - 1]:
                    l += 1

    return res
```

---

#### LC 151 — Reverse Words in a String `Medium`

**The spiel**: "I will split the string by whitespace to extract the words, which naturally handles multiple spaces. Then I use two converging pointers on the word list to reverse their order in place. Finally I join them back with a single space. This achieves the reversal without manually tracking character boundaries."

**Complexity**: Time O(N), Space O(N) for the word list.

```python
def reverseWords(s: str) -> str:
    words = s.split()  # Split by any whitespace, removes extra spaces

    # Two-pointer swap to reverse the word list
    l, r = 0, len(words) - 1
    while l < r:
        words[l], words[r] = words[r], words[l]
        l += 1
        r -= 1

    return ' '.join(words)
```

---

#### LC 246 — Strobogrammatic Number `Easy`

**The spiel**: "A strobogrammatic number looks the same when rotated 180 degrees. I use two converging pointers from the ends. At each step, I check whether the pair of digits is one of the valid strobogrammatic pairs: (0,0), (1,1), (6,9), (8,8), (9,6). If any pair is invalid, I return false. When the pointers meet in the middle, the number is strobogrammatic."

**Complexity**: Time O(N), Space O(1).

```python
def isStrobogrammatic(num: str) -> bool:
    pairs = {'0': '0', '1': '1', '6': '9', '8': '8', '9': '6'}
    l, r = 0, len(num) - 1

    while l <= r:
        if num[l] not in pairs or pairs[num[l]] != num[r]:
            return False
        l += 1
        r -= 1

    return True
```

---

#### LC 344 — Reverse String `Easy`

**The spiel**: "This is the purest converging two-pointer problem. I place left at 0 and right at the end. I swap the characters at both pointers and move them inward until they meet. This reverses the array in place with no extra memory."

**Complexity**: Time O(N), Space O(1).

```python
def reverseString(s: list[str]) -> None:
    l, r = 0, len(s) - 1

    while l < r:
        s[l], s[r] = s[r], s[l]
        l += 1
        r -= 1
```

---

#### LC 977 — Squares of a Sorted Array `Easy`

**The spiel**: "Since the array is sorted but contains negative numbers, the largest squares will always be at the far edges (either the smallest negative or largest positive). I will initialize two pointers at the ends of the array. I'll compare the absolute values, square the larger one, and write it into a result array from right to left (filling the largest values first)."

**Complexity**: Time O(N), Space O(N) for the result.

```python
def sortedSquares(nums: list[int]) -> list[int]:
    n = len(nums)
    res = [0] * n
    l, r = 0, n - 1

    # Fill the result array from back to front
    for i in range(n - 1, -1, -1):
        if abs(nums[l]) > abs(nums[r]):
            res[i] = nums[l] ** 2
            l += 1
        else:
            res[i] = nums[r] ** 2
            r -= 1

    return res
```

---

#### LC 2193 — Minimum Number of Moves to Make Palindrome `Hard`

**The spiel**: "I use two pointers at the ends of the string to greedily construct the palindrome. I check if `s[left] == s[right]`. If not, I scan inward from right to find the closest matching character to `s[left]`. Once found, I simulate bubble-swapping it to the right position. If a character has no match (it belongs in the middle of an odd-length palindrome), I swap it toward the center."

**Complexity**: Time O(N^2) (inner scan and bubble swaps), Space O(N) (string converted to list).

```python
def minMovesToMakePalindrome(s: str) -> int:
    s = list(s)
    res = 0
    left, right = 0, len(s) - 1

    while left < right:
        if s[left] != s[right]:
            # Scan inward from the right to find the match for s[left]
            k = right
            while k > left and s[k] != s[left]:
                k -= 1

            if k == left:  # No match found — it's the center character
                s[left], s[left + 1] = s[left + 1], s[left]
                res += 1
                continue  # Do not advance left/right pointers yet

            # Bubble swap the matched character to the 'right' boundary
            while k < right:
                s[k], s[k + 1] = s[k + 1], s[k]
                k += 1
                res += 1

        left += 1
        right -= 1

    return res
```

---

#### LC 2824 — Count Pairs Whose Sum is Less Than Target `Easy`

**The spiel**: "After sorting the array, I use two converging pointers. If `nums[left] + nums[right] < target`, then every pair `(left, left+1), (left, left+2), ... (left, right)` is also valid because the array is sorted, giving me `right - left` pairs at once. I increment left. Otherwise the sum is too large so I decrement right."

**Complexity**: Time O(N log N) for sorting + O(N) scan, Space O(1).

```python
def countPairs(nums: list[int], target: int) -> int:
    nums.sort()
    count = 0
    l, r = 0, len(nums) - 1

    while l < r:
        if nums[l] + nums[r] < target:
            count += r - l  # All pairs (l, l+1..r) are valid
            l += 1
        else:
            r -= 1

    return count
```

---

### Group 2: Same Direction (Read / Write / Partition)

---

#### LC 27 — Remove Element `Easy`

**The spiel**: "I use a read/write pointer approach. The write pointer tracks the boundary of valid (non-target) elements. The read pointer scans every element. When the read pointer finds a value that is not the target, I copy it to the write position and advance the write pointer. The final write position is the new length of the array."

**Complexity**: Time O(N), Space O(1).

```python
def removeElement(nums: list[int], val: int) -> int:
    write = 0  # Write pointer for valid elements

    for read in range(len(nums)):  # Read pointer scans all
        if nums[read] != val:
            nums[write] = nums[read]
            write += 1

    return write  # New length of the array
```

---

#### LC 75 — Sort Colors `Medium`

**The spiel**: "This is the Dutch National Flag problem. I'll maintain three regions using three pointers: `low` for 0s, `mid` as the current scanner, and `high` for 2s. If `mid` finds a 0, I swap it with `low` and move both forward. If it finds a 1, it's already in the correct middle region, so I just move `mid` forward. If it finds a 2, I swap it with `high` and decrement `high`, but I do *not* move `mid` because the swapped element needs to be evaluated."

**Complexity**: Time O(N), Space O(1).

```python
def sortColors(nums: list[int]) -> None:
    low, mid, high = 0, 0, len(nums) - 1

    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:  # nums[mid] == 2
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1
```

---

#### LC 283 — Move Zeroes `Easy`

**The spiel**: "I will use a Read/Write two-pointer approach to modify the array in place. The left pointer tracks the next position for a non-zero element. The right pointer scans the array. Whenever the right pointer finds a non-zero element, I swap it with the element at the left pointer and increment left. This maintains relative order and naturally pushes zeroes to the end."

**Complexity**: Time O(N), Space O(1).

```python
def moveZeroes(nums: list[int]) -> None:
    left = 0  # Write pointer

    for right in range(len(nums)):  # Read pointer
        if nums[right] != 0:
            # Swap non-zero to the front
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
```

---

#### LC 443 — String Compression `Medium`

**The spiel**: "I use a read pointer to scan consecutive groups of characters and a write pointer to overwrite the array in place. For each group, I write the character at the write position. If the group count is more than 1, I also write each digit of the count. The read pointer jumps ahead by the group size. The final write position is the new compressed length."

**Complexity**: Time O(N), Space O(1).

```python
def compress(chars: list[str]) -> int:
    write = 0
    read = 0

    while read < len(chars):
        char = chars[read]
        count = 0

        # Count consecutive occurrences of the current character
        while read < len(chars) and chars[read] == char:
            read += 1
            count += 1

        # Write the character
        chars[write] = char
        write += 1

        # Write the count digits if count > 1
        if count > 1:
            for digit in str(count):
                chars[write] = digit
                write += 1

    return write  # New length of the compressed array
```

---

### Group 3: Two Pointers on Two Iterables (Matching / Subsequence)

---

#### LC 321 — Create Maximum Number `Hard`

**The spiel**: "I split the k digits between the two arrays: i from nums1 and k-i from nums2. For each split, I extract the maximum subsequence of the required length from each array using a monotonic stack (greedy drop). Then I merge the two subsequences using a two-pointer greedy comparison — at each step I pick from whichever subsequence is lexicographically larger. I take the maximum result across all valid splits."

**Complexity**: Time O(k * (m + n + k)) across all splits, Space O(k).

```python
def maxNumber(nums1: list[int], nums2: list[int], k: int) -> list[int]:
    def max_subsequence(nums, length):
        """Extract the largest subsequence of given length using monotonic stack."""
        stack = []
        drop = len(nums) - length  # Number of elements we can afford to drop
        for num in nums:
            while drop and stack and stack[-1] < num:
                stack.pop()
                drop -= 1
            stack.append(num)
        return stack[:length]

    def merge(a, b):
        """Merge two subsequences greedily, always picking the larger leader."""
        res = []
        while a or b:
            # Compare remaining sequences lexicographically
            if a > b:
                res.append(a[0])
                a = a[1:]
            else:
                res.append(b[0])
                b = b[1:]
        return res

    best = []
    for i in range(k + 1):
        j = k - i
        if i > len(nums1) or j > len(nums2):
            continue
        sub1 = max_subsequence(nums1, i)
        sub2 = max_subsequence(nums2, j)
        merged = merge(sub1, sub2)
        best = max(best, merged)

    return best
```

---

#### LC 408 — Valid Word Abbreviation `Easy`

**The spiel**: "I use two pointers: one on the word and one on the abbreviation. If the abbreviation character is a letter, I compare it directly with the word character and advance both. If it's a digit, I parse the full number (rejecting leading zeros) and advance the word pointer by that amount. At the end, both pointers must have reached the end of their respective strings for the abbreviation to be valid."

**Complexity**: Time O(N), Space O(1).

```python
def validWordAbbreviation(word: str, abbr: str) -> bool:
    i, j = 0, 0  # i for word, j for abbr

    while i < len(word) and j < len(abbr):
        if abbr[j].isdigit():
            if abbr[j] == '0':  # Leading zeros are not allowed
                return False
            num = 0
            while j < len(abbr) and abbr[j].isdigit():
                num = num * 10 + int(abbr[j])
                j += 1
            i += num  # Skip that many characters in word
        else:
            if word[i] != abbr[j]:
                return False
            i += 1
            j += 1

    return i == len(word) and j == len(abbr)
```

---

#### LC 1537 — Get the Maximum Score `Hard`

**The spiel**: "Both arrays are sorted and may share common values. I use two pointers, one per array, accumulating separate running sums. When both pointers land on the same value, I take the maximum of the two sums (choosing the better path), add the common value, and continue. At the end, I add the remaining elements from each array and take the final maximum. This is like choosing the optimal lane to drive in, switching lanes only at intersections."

**Complexity**: Time O(M + N), Space O(1).

```python
def maxSum(nums1: list[int], nums2: list[int]) -> int:
    MOD = 10**9 + 7
    i, j = 0, 0
    sum1, sum2 = 0, 0

    while i < len(nums1) and j < len(nums2):
        if nums1[i] < nums2[j]:
            sum1 += nums1[i]
            i += 1
        elif nums1[i] > nums2[j]:
            sum2 += nums2[j]
            j += 1
        else:  # Common value — choose the better path
            sum1 = sum2 = max(sum1, sum2) + nums1[i]
            i += 1
            j += 1

    # Drain remaining elements from each array
    while i < len(nums1):
        sum1 += nums1[i]
        i += 1
    while j < len(nums2):
        sum2 += nums2[j]
        j += 1

    return max(sum1, sum2) % MOD
```

---

#### LC 2486 — Append Characters to String to Make Subsequence `Medium`

**The spiel**: "To find the minimum characters to append, I need to find the longest matching prefix of string `t` that exists as a subsequence in string `s`. I use two pointers: one iterating through `s` and one through `t`. I greedily advance the `s` pointer, and only advance the `t` pointer when characters match. The remaining characters in `t` when `s` is exhausted is my answer."

**Complexity**: Time O(N), Space O(1).

```python
def appendCharacters(s: str, t: str) -> int:
    i, j = 0, 0  # i for s, j for t

    while i < len(s) and j < len(t):
        if s[i] == t[j]:
            j += 1  # Match found, advance target pointer
        i += 1  # Always advance source pointer

    # The remaining characters in t need to be appended
    return len(t) - j
```

---

### Group 4: Linked List & Reset Tricks

---

#### LC 19 — Remove Nth Node From End of List `Medium`

**The spiel**: "To solve this in one pass, I'll use a fast and slow pointer with a fixed spacing. I introduce a dummy node to handle edge cases where the head itself needs to be removed. I advance the fast pointer N+1 steps ahead. Then, I move both together until fast hits the end. At this point, slow is positioned exactly before the node to delete."

**Complexity**: Time O(N), Space O(1).

```python
def removeNthFromEnd(head, n: int):
    # Dummy node handles the case where head is removed
    dummy = ListNode(0, head)
    slow = fast = dummy

    # Give fast pointer an n+1 step head start
    for _ in range(n + 1):
        fast = fast.next

    # Move both until fast reaches the end
    while fast:
        slow = slow.next
        fast = fast.next

    # Skip the nth node
    slow.next = slow.next.next

    return dummy.next
```

---

#### LC 160 — Intersection of Two Linked Lists `Easy`

**The spiel**: "To find the intersection without extra space, I need to account for the difference in lengths. I use two pointers starting at the respective heads. When a pointer reaches the end of its list, I redirect it to the head of the *other* list. Both pointers will travel the exact same total distance (Length A + Length B). They are mathematically guaranteed to collide at the intersection node on their second pass, or at `None` if no intersection exists."

**Complexity**: Time O(N+M), Space O(1).

```python
def getIntersectionNode(headA, headB):
    if not headA or not headB:
        return None

    ptrA, ptrB = headA, headB

    # If they intersect, they will meet. If not, both hit None simultaneously.
    while ptrA != ptrB:
        ptrA = ptrA.next if ptrA else headB
        ptrB = ptrB.next if ptrB else headA

    return ptrA
```

---

#### LC 1650 — Lowest Common Ancestor of a Binary Tree III `Medium`

**The spiel**: "Each node has a parent pointer, so the path to root forms a singly linked list. Finding the LCA is identical to finding the intersection of two linked lists. I use two pointers starting at `p` and `q`. When a pointer reaches the root (null parent), I redirect it to the *other* starting node. They will travel equal total distances and collide at the lowest common ancestor."

**Complexity**: Time O(H) where H is the tree height, Space O(1).

```python
def lowestCommonAncestor(p, q):
    ptrA, ptrB = p, q

    # Same reset trick as intersection of two linked lists
    while ptrA != ptrB:
        ptrA = ptrA.parent if ptrA else q
        ptrB = ptrB.parent if ptrB else p

    return ptrA
```

---

### Group 5: Boundary Expansion & Suffix Scanning (Greedy)

---

#### LC 31 — Next Permutation `Medium`

**The spiel**: "To find the next lexicographical permutation, I find the first point from the right where ascending order breaks — the 'pivot'. Then I scan from right to left again to find the smallest number strictly greater than the pivot. I swap them. Finally, because the suffix after the pivot is in descending order, I reverse it to make it ascending, giving the smallest possible next permutation."

**Complexity**: Time O(N), Space O(1).

```python
def nextPermutation(nums: list[int]) -> None:
    i = len(nums) - 2

    # 1. Find the pivot (first decrease from right to left)
    while i >= 0 and nums[i] >= nums[i + 1]:
        i -= 1

    if i >= 0:
        # 2. Find the rightmost successor to swap with
        j = len(nums) - 1
        while nums[j] <= nums[i]:
            j -= 1
        nums[i], nums[j] = nums[j], nums[i]

    # 3. Reverse the suffix to get the smallest lexicographical order
    left, right = i + 1, len(nums) - 1
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1
```

---

#### LC 189 — Rotate Array `Medium`

**The spiel**: "Rotating an array by k positions is equivalent to three reversals. First, I reverse the entire array. Then I reverse the first k elements, and finally reverse the remaining elements. Each reversal uses a simple two-pointer swap from the ends inward. I take `k % n` to handle cases where k exceeds the array length."

**Complexity**: Time O(N), Space O(1).

```python
def rotate(nums: list[int], k: int) -> None:
    n = len(nums)
    k %= n  # Handle k > n

    def reverse(l, r):
        """Two-pointer reversal in place."""
        while l < r:
            nums[l], nums[r] = nums[r], nums[l]
            l += 1
            r -= 1

    reverse(0, n - 1)  # Reverse entire array
    reverse(0, k - 1)  # Reverse first k elements
    reverse(k, n - 1)  # Reverse remaining elements
```

---

#### LC 763 — Partition Labels `Medium`

**The spiel**: "The invariant is that all occurrences of a character must appear in the same partition. First, I iterate through the string to record the last index of every character. Then, I iterate again with a two-pointer window. I update the window's end boundary to the maximum last occurrence of characters I'm looking at. When my current index `i` reaches `end`, the window is self-contained — I record the partition length and start a new window."

**Complexity**: Time O(N), Space O(1) (hash map bounded by 26 letters).

```python
def partitionLabels(s: str) -> list[int]:
    # Record the last occurrence of each character
    last_idx = {char: i for i, char in enumerate(s)}

    res = []
    size, end = 0, 0

    for i, char in enumerate(s):
        size += 1
        # Expand the boundary to encompass the current character's last appearance
        end = max(end, last_idx[char])

        # When we reach the boundary, cut the partition
        if i == end:
            res.append(size)
            size = 0

    return res
```

---

#### LC 1842 — Next Palindrome Using Same Digits `Hard`

**The spiel**: "Since the input is already a palindrome, I only need to find the next permutation of the first half, then mirror it. I extract the first half, apply the standard next-permutation algorithm (find pivot, swap with successor, reverse suffix). If no next permutation exists, the answer is empty. Otherwise I reconstruct the palindrome by mirroring the new first half, preserving the middle character for odd-length strings."

**Complexity**: Time O(N), Space O(N).

```python
def nextPalindrome(num: str) -> str:
    n = len(num)
    half = list(num[:n // 2])

    # Apply next permutation to the first half
    # 1. Find the pivot (rightmost descent)
    i = len(half) - 2
    while i >= 0 and half[i] >= half[i + 1]:
        i -= 1

    if i < 0:
        return ""  # No next permutation possible

    # 2. Find the rightmost element greater than pivot
    j = len(half) - 1
    while half[j] <= half[i]:
        j -= 1
    half[i], half[j] = half[j], half[i]

    # 3. Reverse suffix after pivot
    half[i + 1:] = reversed(half[i + 1:])

    # Reconstruct palindrome: first half + (middle if odd) + reversed first half
    first = ''.join(half)
    if n % 2:
        return first + num[n // 2] + first[::-1]
    else:
        return first + first[::-1]
```

---

#### LC 2444 — Count Subarrays With Fixed Bounds `Hard`

**The spiel**: "I scan left to right, tracking three positions: the last index where I saw `minK`, the last index where I saw `maxK`, and the last index of an element outside `[minK, maxK]` (a 'bad' element). For each index, the number of valid subarrays ending there is `max(0, min(lastMin, lastMax) - lastBad)`. This works because a valid subarray must include both bounds and cannot include any bad element."

**Complexity**: Time O(N), Space O(1).

```python
def countSubarrays(nums: list[int], minK: int, maxK: int) -> int:
    res = 0
    last_min = last_max = -1  # Last positions of minK and maxK
    last_bad = -1             # Last position of element outside [minK, maxK]

    for i, num in enumerate(nums):
        if num < minK or num > maxK:
            last_bad = i  # This element breaks any subarray containing it
        if num == minK:
            last_min = i
        if num == maxK:
            last_max = i

        # Valid subarrays end at i and start anywhere from (last_bad, min(last_min, last_max)]
        res += max(0, min(last_min, last_max) - last_bad)

    return res
```
