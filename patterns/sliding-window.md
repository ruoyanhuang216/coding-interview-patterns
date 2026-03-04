# Sliding Window

## What Is It?

The Sliding Window pattern is a specialized optimization of the Two Pointers technique. Two pointers (`left` and `right`) traverse a linear data structure in the **same direction**, defining a "window" of contiguous elements. By incrementally updating the window's state as it expands and shrinks — rather than recomputing from scratch — you reduce O(N^2) or O(N^3) brute-force solutions down to O(N) linear time.

## When to Use It

- **"Contiguous" / "Subarray" / "Substring"** — The elements must be adjacent. If the problem asks for a "subsequence", sliding window is usually the wrong tool
- **Optimization over a range** — Find the *shortest*, *longest*, *maximum*, or *minimum* contiguous sequence meeting a condition
- **Monotonic window state** — Adding an element to the window moves the state in one direction (e.g., sum increases, count increases); removing moves it in the opposite direction. If this monotonicity doesn't hold (e.g., negative numbers in a sum window), basic sliding window breaks down

## Real-Life Analogies

| Analogy | Maps to |
|---------|---------|
| **Reading a ruler through a magnifying glass** — The glass is a fixed width; slide it along the ruler to inspect each segment | Fixed-size window |
| **Adjusting a camera crop** — Widen the frame to include more of the scene, then tighten it to remove unwanted edges | Dynamic expand/shrink |
| **Counting cars on a highway segment** — A sensor reports totals for each rolling 5-minute interval, updating by adding the newest minute and dropping the oldest | Rolling aggregate |

## Key Techniques

These three mechanics recur across nearly every sliding window variant.

**1. The Infinity Initialization**: When tracking a *minimum* (like shortest subarray length), initialize to `float('inf')`. The first valid window overwrites it cleanly. If it remains infinity after the loop, no valid window was found — return 0 or empty.

**2. The Subarray Counting Trick**: When counting *how many* valid subarrays exist (not just the longest/shortest): if the window `[left...right]` is valid and every shorter subarray ending at `right` is also valid, then the number of new valid subarrays contributed by this `right` position is exactly `right - left + 1`. Accumulate this each step.

**3. The "At Most K" Decomposition**: Standard sliding window can solve "at most K" but not "exactly K" directly — because when the window is valid, you don't know whether to shrink further. The fix: `Exact(K) = AtMost(K) - AtMost(K - 1)`. Write one helper that solves "at most K", then call it twice.

---

## Archetypes

### Group 1: Dynamic Expanding & Shrinking

| # | Problem | Difficulty |
|---|---------|------------|
| 3 | Longest Substring Without Repeating Characters | Medium |
| 76 | Minimum Window Substring | Hard |
| 209 | Minimum Size Subarray Sum | Medium |
| 340 | Longest Substring with At Most K Distinct Characters | Medium |
| 424 | Longest Repeating Character Replacement | Medium |
| 904 | Fruit Into Baskets | Medium |
| 1004 | Max Consecutive Ones III | Medium |

**Interview logistics**: The `right` pointer expands the window in an outer `for` loop. An inner `while` loop shrinks from the `left` once the window becomes invalid (or once a target condition is met and you're minimizing). Track the window state with a hash map, counter, or running sum. Update your answer after each shrink (for minimization) or after each expansion (for maximization).

### Group 2: Fixed-Size Windows & Hashes

| # | Problem | Difficulty |
|---|---------|------------|
| 30 | Substring with Concatenation of All Words | Hard |
| 438 | Find All Anagrams in a String | Medium |
| 567 | Permutation in String | Medium |
| 643 | Maximum Average Subarray I | Easy |
| 1456 | Maximum Number of Vowels in a Substring of Given Size | Medium |

**Interview logistics**: The window size `k` is fixed. As `right` advances past index `k-1`, simultaneously remove the element at `right - k` (the element sliding out). For anagram/permutation problems, maintain a frequency counter and compare with the target counter — use a `matches` variable or direct Counter comparison for O(1) per-step checks.

### Group 3: Combinatorics (Counting Subarrays)

| # | Problem | Difficulty |
|---|---------|------------|
| 713 | Subarray Product Less Than K | Medium |
| 930 | Binary Subarrays With Sum | Medium |
| 992 | Subarrays with K Different Integers | Hard |
| 1248 | Count Number of Nice Subarrays | Medium |

**Interview logistics**: Instead of tracking max/min length, you accumulate `count += right - left + 1` at each step. For "exactly K" problems, use the `AtMost(K) - AtMost(K - 1)` decomposition — write one generic helper and call it twice.

### Group 4: Monotonic Deque (Sliding Window Max/Min)

| # | Problem | Difficulty |
|---|---------|------------|
| 239 | Sliding Window Maximum | Hard |
| 1438 | Longest Continuous Subarray With Absolute Diff <= Limit | Medium |

**Interview logistics**: A plain sliding window would need to rescan the entire window for the max/min each step, costing O(N*K). A monotonic deque maintains candidates in sorted order, giving O(1) access to the current window's extreme value. The deque stores *indices* (not values) so you can evict elements that have slid out of the window.

---

## Problem Checklist

### Group 1: Dynamic Expanding & Shrinking

---

#### LC 3 — Longest Substring Without Repeating Characters `Medium`

**The spiel**: "I use a hash set to track characters in the current window. As my right pointer expands, if it hits a duplicate already in the set, I shrink from the left — removing characters one by one until the duplicate is evicted. After each expansion I record the maximum window length. Every character enters and leaves the set at most once, so this is O(N)."

**Complexity**: Time O(N), Space O(min(N, M)) where M is the character set size.

```python
def lengthOfLongestSubstring(s: str) -> int:
    char_set = set()
    left = 0
    max_length = 0

    for right in range(len(s)):
        # Shrink until the duplicate is removed
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1

        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)

    return max_length
```

---

#### LC 76 — Minimum Window Substring `Hard`

**The spiel**: "I maintain a frequency map of what characters I still need from `t` and a `missing` counter for how many total characters remain unmatched. As right expands, I decrement needs. Once `missing` hits 0 — meaning the window contains all of `t` — I shrink from the left to find the smallest valid window, updating my answer each time. When I remove a character that is actually needed, `missing` goes back up and I resume expanding."

**Complexity**: Time O(N + M), Space O(M) where M is the character set of `t`.

```python
from collections import Counter

def minWindow(s: str, t: str) -> str:
    need = Counter(t)
    missing = len(t)       # Total characters still needed
    left = 0
    start, end = 0, float('inf')

    for right in range(len(s)):
        if need[s[right]] > 0:
            missing -= 1   # This character was actually needed
        need[s[right]] -= 1

        # All characters found — shrink to find minimum window
        while missing == 0:
            if right - left < end - start:
                start, end = left, right
            need[s[left]] += 1
            if need[s[left]] > 0:
                missing += 1  # We lost a needed character
            left += 1

    return "" if end == float('inf') else s[start:end + 1]
```

---

#### LC 209 — Minimum Size Subarray Sum `Medium`

**The spiel**: "Since all values are positive, the running sum is monotonically increasing as I expand — which is exactly the property sliding window requires. I expand right to grow the sum. Once the sum meets or exceeds the target, I shrink from the left to minimize the window length, recording the shortest valid window found. The positivity guarantee means shrinking always decreases the sum."

**Complexity**: Time O(N), Space O(1).

```python
def minSubArrayLen(target: int, nums: list[int]) -> int:
    left = 0
    current_sum = 0
    min_len = float('inf')

    for right in range(len(nums)):
        current_sum += nums[right]

        # Shrink while the window meets the target
        while current_sum >= target:
            min_len = min(min_len, right - left + 1)
            current_sum -= nums[left]
            left += 1

    return min_len if min_len != float('inf') else 0
```

---

#### LC 340 — Longest Substring with At Most K Distinct Characters `Medium`

**The spiel**: "I maintain a hash map counting the frequency of each character in the window. As right expands, I add the new character. If the number of distinct characters exceeds K, I shrink from the left — decrementing counts and deleting keys that hit zero — until we're back to at most K distinct. I track the maximum window length throughout."

**Complexity**: Time O(N), Space O(K).

```python
def lengthOfLongestSubstringKDistinct(s: str, k: int) -> int:
    count = {}
    left = 0
    max_len = 0

    for right in range(len(s)):
        count[s[right]] = count.get(s[right], 0) + 1

        # Shrink until we have at most k distinct characters
        while len(count) > k:
            count[s[left]] -= 1
            if count[s[left]] == 0:
                del count[s[left]]
            left += 1

        max_len = max(max_len, right - left + 1)

    return max_len
```

---

#### LC 424 — Longest Repeating Character Replacement `Medium`

**The spiel**: "The window is valid when the number of characters that need replacement — `window_size - max_frequency` — is at most K. I track character frequencies and the historical max frequency seen. As right expands, I update the frequency. If the window becomes invalid (replacements needed > K), I shrink from the left. The key insight is that `max_freq` never needs to decrease — a window can only grow the answer when max_freq increases, so a stale max_freq simply keeps the window from growing without affecting correctness."

**Complexity**: Time O(N), Space O(1) (at most 26 keys).

```python
def characterReplacement(s: str, k: int) -> int:
    count = {}
    left = 0
    max_freq = 0
    max_len = 0

    for right in range(len(s)):
        count[s[right]] = count.get(s[right], 0) + 1
        max_freq = max(max_freq, count[s[right]])

        # Window invalid: more than k characters need replacement
        while (right - left + 1) - max_freq > k:
            count[s[left]] -= 1
            left += 1

        max_len = max(max_len, right - left + 1)

    return max_len
```

---

#### LC 904 — Fruit Into Baskets `Medium`

**The spiel**: "This is 'longest subarray with at most 2 distinct values' in disguise. I use a hash map to count fruit types in the window. As right expands, I add the new fruit. If the window has more than 2 types, I shrink from the left until it's valid again. I track the maximum window length — that's the maximum number of fruits I can collect."

**Complexity**: Time O(N), Space O(1) (at most 3 keys in the map).

```python
def totalFruit(fruits: list[int]) -> int:
    count = {}
    left = 0
    max_len = 0

    for right in range(len(fruits)):
        count[fruits[right]] = count.get(fruits[right], 0) + 1

        # More than 2 types — shrink
        while len(count) > 2:
            count[fruits[left]] -= 1
            if count[fruits[left]] == 0:
                del count[fruits[left]]
            left += 1

        max_len = max(max_len, right - left + 1)

    return max_len
```

---

#### LC 1004 — Max Consecutive Ones III `Medium`

**The spiel**: "I can flip at most K zeros, so this becomes 'find the longest window containing at most K zeros.' I expand right, counting zeros as I go. When the zero count exceeds K, I shrink from the left until a zero drops out of the window. The maximum window length is my answer."

**Complexity**: Time O(N), Space O(1).

```python
def longestOnes(nums: list[int], k: int) -> int:
    left = 0
    zeros = 0
    max_len = 0

    for right in range(len(nums)):
        if nums[right] == 0:
            zeros += 1

        # Too many zeros — shrink until one drops out
        while zeros > k:
            if nums[left] == 0:
                zeros -= 1
            left += 1

        max_len = max(max_len, right - left + 1)

    return max_len
```

---

### Group 2: Fixed-Size Windows & Hashes

---

#### LC 30 — Substring with Concatenation of All Words `Hard`

**The spiel**: "All words have equal length, so I can treat the string as a sequence of word-sized chunks. I run a sliding window for each starting offset (0 to word_len - 1) to cover all alignments. Within each pass, I maintain a frequency counter of words in the window. I expand by one word at a time; if the word isn't in the target or its count exceeds what's needed, I shrink from the left. When the window contains exactly all target words, I record the starting index."

**Complexity**: Time O(N * word_len), Space O(num_words * word_len).

```python
from collections import Counter

def findSubstring(s: str, words: list[str]) -> list[int]:
    if not s or not words:
        return []

    word_len = len(words[0])
    num_words = len(words)
    word_count = Counter(words)
    res = []

    # Try each starting offset within one word length
    for offset in range(word_len):
        left = offset
        window = Counter()
        count = 0  # Number of valid words in window

        for right in range(offset, len(s) - word_len + 1, word_len):
            word = s[right:right + word_len]

            if word in word_count:
                window[word] += 1
                count += 1

                # Shrink if this word appears too many times
                while window[word] > word_count[word]:
                    left_word = s[left:left + word_len]
                    window[left_word] -= 1
                    count -= 1
                    left += word_len

                if count == num_words:
                    res.append(left)
            else:
                # Invalid word — reset the window entirely
                window.clear()
                count = 0
                left = right + word_len

    return res
```

---

#### LC 438 — Find All Anagrams in a String `Medium`

**The spiel**: "I use a fixed-size window of length `len(p)` sliding over `s`. I maintain a frequency counter of the current window and compare it with the target counter of `p`. When the window counter matches the target, the current window start is an anagram position. As right advances, I add the new character and remove the character that slides out."

**Complexity**: Time O(N), Space O(1) (bounded by 26 letters).

```python
from collections import Counter

def findAnagrams(s: str, p: str) -> list[int]:
    if len(p) > len(s):
        return []

    p_count = Counter(p)
    window = Counter()
    k = len(p)
    res = []

    for right in range(len(s)):
        window[s[right]] += 1

        # Slide out the leftmost character once window exceeds size k
        if right >= k:
            ch = s[right - k]
            window[ch] -= 1
            if window[ch] == 0:
                del window[ch]

        # Compare window with target
        if window == p_count:
            res.append(right - k + 1)

    return res
```

---

#### LC 567 — Permutation in String `Medium`

**The spiel**: "This is the boolean version of 'Find All Anagrams.' I slide a window of size `len(s1)` over `s2`, maintaining a frequency counter. The moment the window's counter matches `s1`'s counter, a permutation exists and I return true. If no match is found after the full sweep, return false."

**Complexity**: Time O(N), Space O(1).

```python
from collections import Counter

def checkInclusion(s1: str, s2: str) -> bool:
    if len(s1) > len(s2):
        return False

    need = Counter(s1)
    window = Counter()
    k = len(s1)

    for right in range(len(s2)):
        window[s2[right]] += 1

        if right >= k:
            ch = s2[right - k]
            window[ch] -= 1
            if window[ch] == 0:
                del window[ch]

        if window == need:
            return True

    return False
```

---

#### LC 643 — Maximum Average Subarray I `Easy`

**The spiel**: "I compute the sum of the first K elements as the initial window. Then I slide right: add the new element entering the window, subtract the element leaving. Track the maximum sum, then divide by K at the end for the average."

**Complexity**: Time O(N), Space O(1).

```python
def findMaxAverage(nums: list[int], k: int) -> float:
    window_sum = sum(nums[:k])
    max_sum = window_sum

    for right in range(k, len(nums)):
        window_sum += nums[right] - nums[right - k]
        max_sum = max(max_sum, window_sum)

    return max_sum / k
```

---

#### LC 1456 — Maximum Number of Vowels in a Substring of Given Size `Medium`

**The spiel**: "Fixed-size window of size K. I count vowels in the initial window. As I slide, I add 1 if the incoming character is a vowel and subtract 1 if the outgoing character is a vowel. Track the maximum vowel count seen."

**Complexity**: Time O(N), Space O(1).

```python
def maxVowels(s: str, k: int) -> int:
    vowels = set('aeiou')
    count = sum(1 for c in s[:k] if c in vowels)
    max_count = count

    for right in range(k, len(s)):
        count += (s[right] in vowels) - (s[right - k] in vowels)
        max_count = max(max_count, count)

    return max_count
```

---

### Group 3: Combinatorics (Counting Subarrays)

---

#### LC 713 — Subarray Product Less Than K `Medium`

**The spiel**: "Since all values are positive, the product grows monotonically as the window expands. I expand right, multiplying into the running product. When the product reaches or exceeds K, I shrink from the left by dividing. At each step, every subarray ending at `right` and starting anywhere in `[left, right]` is valid, contributing `right - left + 1` to the count."

**Complexity**: Time O(N), Space O(1).

```python
def numSubarrayProductLessThanK(nums: list[int], k: int) -> int:
    if k <= 1:
        return 0

    product = 1
    left = 0
    count = 0

    for right in range(len(nums)):
        product *= nums[right]

        while product >= k:
            product //= nums[left]
            left += 1

        # All subarrays ending at right with start in [left, right]
        count += right - left + 1

    return count
```

---

#### LC 930 — Binary Subarrays With Sum `Medium`

**The spiel**: "This asks for subarrays with *exactly* `goal` sum. Since the array contains zeros, a direct sliding window can't tell when to stop shrinking. I use the At Most K decomposition: write a helper that counts subarrays with sum at most K, then `exactly(goal) = atMost(goal) - atMost(goal - 1)`. The helper is a standard sliding window that shrinks when the sum exceeds K."

**Complexity**: Time O(N), Space O(1).

```python
def numSubarraysWithSum(nums: list[int], goal: int) -> int:
    def at_most(k):
        if k < 0:
            return 0
        left = 0
        current = 0
        count = 0
        for right in range(len(nums)):
            current += nums[right]
            while current > k:
                current -= nums[left]
                left += 1
            count += right - left + 1
        return count

    return at_most(goal) - at_most(goal - 1)
```

---

#### LC 992 — Subarrays with K Different Integers `Hard`

**The spiel**: "Finding subarrays with *exactly* K distinct integers is hard because adding an element might not change the distinct count. I decompose it: write a helper for 'at most K distinct' using a frequency map that shrinks when distinct count exceeds K. Then `exactly(K) = atMost(K) - atMost(K - 1)`. Each helper call is O(N), so total is O(N)."

**Complexity**: Time O(N), Space O(K).

```python
def subarraysWithKDistinct(nums: list[int], k: int) -> int:
    def at_most(k):
        count = {}
        left = 0
        result = 0
        for right in range(len(nums)):
            count[nums[right]] = count.get(nums[right], 0) + 1
            while len(count) > k:
                count[nums[left]] -= 1
                if count[nums[left]] == 0:
                    del count[nums[left]]
                left += 1
            result += right - left + 1
        return result

    return at_most(k) - at_most(k - 1)
```

---

#### LC 1248 — Count Number of Nice Subarrays `Medium`

**The spiel**: "A 'nice' subarray has exactly K odd numbers. I treat odd numbers as 1 and even as 0, reducing this to 'subarrays with exactly K sum' — the same pattern as Binary Subarrays With Sum. I use `atMost(K) - atMost(K - 1)` where the helper counts windows with at most K odd numbers."

**Complexity**: Time O(N), Space O(1).

```python
def numberOfSubarrays(nums: list[int], k: int) -> int:
    def at_most(k):
        left = 0
        odds = 0
        count = 0
        for right in range(len(nums)):
            odds += nums[right] % 2
            while odds > k:
                odds -= nums[left] % 2
                left += 1
            count += right - left + 1
        return count

    return at_most(k) - at_most(k - 1)
```

---

### Group 4: Monotonic Deque (Sliding Window Max/Min)

---

#### LC 239 — Sliding Window Maximum `Hard`

**The spiel**: "A brute-force scan for the max in each window costs O(N*K). Instead, I maintain a monotonic decreasing deque of *indices*. As right advances, I pop all indices from the back whose values are smaller than the incoming element — they can never be the max while the new element is in the window. I also pop from the front if the front index has slid out of the window. The front of the deque is always the current window's maximum."

**Complexity**: Time O(N) — each element enters and leaves the deque at most once, Space O(K).

```python
from collections import deque

def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    dq = deque()  # Stores indices; values are monotonically decreasing
    res = []

    for right in range(len(nums)):
        # Remove smaller elements from back — they are now useless
        while dq and nums[dq[-1]] <= nums[right]:
            dq.pop()
        dq.append(right)

        # Remove front if it has slid out of the window
        if dq[0] <= right - k:
            dq.popleft()

        # Window is fully formed once right >= k - 1
        if right >= k - 1:
            res.append(nums[dq[0]])

    return res
```

---

#### LC 1438 — Longest Continuous Subarray With Absolute Diff <= Limit `Medium`

**The spiel**: "I need the max and min of the current window at all times. I maintain two monotonic deques: one decreasing (front = max) and one increasing (front = min). As right expands, I update both deques. If `max - min > limit`, I shrink from the left, popping from either deque if their front index falls out of the window. The longest valid window is my answer."

**Complexity**: Time O(N), Space O(N).

```python
from collections import deque

def longestSubarray(nums: list[int], limit: int) -> int:
    max_dq = deque()  # Decreasing — front is current max
    min_dq = deque()  # Increasing — front is current min
    left = 0
    max_len = 0

    for right in range(len(nums)):
        # Maintain monotonic decreasing deque for max
        while max_dq and nums[max_dq[-1]] <= nums[right]:
            max_dq.pop()
        # Maintain monotonic increasing deque for min
        while min_dq and nums[min_dq[-1]] >= nums[right]:
            min_dq.pop()
        max_dq.append(right)
        min_dq.append(right)

        # Shrink if the window violates the limit
        while nums[max_dq[0]] - nums[min_dq[0]] > limit:
            left += 1
            if max_dq[0] < left:
                max_dq.popleft()
            if min_dq[0] < left:
                min_dq.popleft()

        max_len = max(max_len, right - left + 1)

    return max_len
```
