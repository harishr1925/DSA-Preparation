# Two Pointers: Theory + Code Cheat Sheet

**Core idea:** use two indices instead of a nested loop, so O(n²) becomes **O(n) time and O(1) space**.
The three styles differ in **where the pointers start** and **how they move**.

**Quick decision flow (use this on any new problem):**

1. Comparing the two ends, or searching pairs in sorted data → **Opposite ends**
2. Filtering or rearranging in place, or a linked list cycle/middle → **Same direction (fast and slow)**
3. Only 3 categories, or a pivot partition in a single pass → **Dutch flag**

---

## 1. Opposite Ends (left and right)

**How it works:** one pointer at the start, one at the end. They move toward each other, and each step compares both values to decide which pointer moves.

**Real-life picture:** a menu sorted from cheapest to priciest, and you want two dishes that total exactly ₹500.
- Total too low → upgrade the cheap dish (left moves right).
- Total too high → downgrade the pricey dish (right moves left).
- You never re-check a pair you have already ruled out.

**Spot it by:**
- Sorted array (or you may sort it) plus "find a pair or triplet with a target sum"
- "Palindrome", "reverse", "mirror" (compare the two ends)
- "Maximize area or distance between two positions"
- Brute force would be a nested loop over every pair (i, j)

**Memory hook:** *"Squeeze from both sides."*

**Pointer rule:** the move is decided by comparing a running value (sum, height) with a target or with the other pointer. Loop while `left < right`.

```java
// Sorted array: is there a pair that adds up to target?
boolean hasPairWithSum(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left < right) {              // stop when they meet: one element can't pair with itself
        int sum = arr[left] + arr[right];
        if (sum == target) return true;
        else if (sum < target) left++;  // need a bigger sum
        else right--;                   // need a smaller sum
    }
    return false;
}

// No target needed: compare mirror positions
boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
```

**Solved so far:** Two Sum II, Container With Most Water, 3Sum (outer loop plus opposite ends inside).

---

## 2. Same Direction (Fast and Slow)

This style has **two flavours**.

### 2A. Reader and writer

**How it works:** slow = where the next kept value is written, fast = the element currently being scanned.

**Real-life picture:** a teacher reads the attendance register top to bottom (fast) and rewrites only the unique names at the top of the *same page* (slow).

**Spot it by:** "in-place", "O(1) extra space", "remove, move or keep elements that satisfy a condition".

**Memory hook:** *"One scouts ahead, one builds behind."*

```java
// LeetCode #27 Remove Element: keep everything that is not val, in-place
int removeElement(int[] nums, int val) {
    int write = 0;                                      // slow: where the next kept value goes
    for (int read = 0; read < nums.length; read++) {    // fast: scans every element
        if (nums[read] != val) {
            nums[write] = nums[read];
            write++;
        }
    }
    return write;                                       // how many values were kept
}
```

**Solved so far:** Remove Duplicates (where `count` was the write index).

### 2B. Tortoise and hare (linked list)

**How it works:** fast moves 2 steps, slow moves 1 step.

**Real-life picture:** two runners on a track, one twice as fast.
- **Circular track:** the fast runner laps the slow one and they meet, which means there is a cycle.
- **Straight road:** the fast runner simply reaches the end.
- When the fast runner finishes the road, the slow runner is at the halfway mark, which is the middle.

**Spot it by:** linked list plus "cycle", "middle node" or "nth node from the end".

**Memory hook:** *"Different speeds reveal the shape."*

```java
// Skeleton: the problem-specific check goes in the marked spot
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {   // both checks so fast.next.next can't throw a NullPointerException
    slow = slow.next;          // 1 step
    fast = fast.next.next;     // 2 steps
    // <-- problem-specific check goes here (cycle? middle?)
}
```

**Self-check:** why does the loop need BOTH `fast != null` and `fast.next != null`? Trace an even-length list and an odd-length list.

---

## 3. Partition / Dutch Flag

**How it works:** 3 categories, one pass, 3 pointers (`low`, `mid`, `high`). The array is always split into four zones:

```
[ done: 0s | done: 1s | unknown (mid → high) | done: 2s ]
```

**Real-life picture:** sorting laundry into three baskets (whites, colours, darks) in a single walk through the pile. Each item either goes into its basket or stays where it is, and nothing is handled twice.

**Spot it by:**
- "Array contains only 0, 1 and 2" (or 3 colours or 3 distinct values)
- "Sort in one pass", "without the library sort", "constant space"
- "Partition around a pivot"

**Memory hook:** *"Three zones, one walk."*

```java
// Skeleton: derive the three branches yourself
int low = 0, mid = 0, high = nums.length - 1;
while (mid <= high) {
    if (nums[mid] == 0) {
        // TODO
    } else if (nums[mid] == 1) {
        // TODO
    } else {              // nums[mid] == 2
        // TODO
    }
}
```

**To derive yourself:** what should happen when `mid` sees each of the three values? One of the three cases behaves differently from the other two. Find out which one and why.

**Self-check:** why `mid <= high` here and not `mid < high`? Think about which zone is unknown, and whether it still holds an element when `mid == high`.

---

## Complexity (all examples above)

| Style | Time | Space | Why |
|---|---|---|---|
| Opposite ends | O(n) | O(1) | Combined pointer movement covers the array once, so it is not nested |
| Reader and writer | O(n) | O(1) | One pass, only two index variables |
| Tortoise and hare | O(n) | O(1) | Fast reaches the end in at most n/2 steps |
| Dutch flag | O(n) | O(1) | Each element is examined once; `mid` never goes backward |

*Exception:* 3Sum is O(n²) because the two-pointer scan sits **inside** a `for` loop (multiply, not add).

---

## Pitfalls You Have Already Hit (and Fixed)

- **`left <= right` in opposite-ends problems:** when both pointers land on the same index, one element gets used twice. Use `left < right` whenever the problem needs two distinct positions. (`<=` is for cases like binary search, where the last remaining element still needs checking.)
- **Sentinel values that collide with real data:** starting `prevNumber = 0` broke when the array's first real value was 0. Compare against an actual array position instead, such as `arr[count - 1]`.
- **Duplicate results in 3Sum:** after a match, skip repeated values at `i`, `j` and `k`, not just at `i`.
- **Time vs space with recursion:** total work across all levels is time. The deepest chain of calls alive at once is space.

---

## Practice Set

| Pattern | Problem | Source |
|---|---|---|
| Opposite ends | Two Sum II (#167), Container With Most Water (#11), 3Sum (#15) | LeetCode (done) |
| Opposite ends | Valid Palindrome (#125) | LeetCode |
| Same direction (reader/writer) | Remove Duplicates from Sorted Array (#26) | LeetCode (done) |
| Same direction (reader/writer) | Move Zeroes (#283) | LeetCode |
| Same direction (tortoise and hare) | Linked List Cycle (#141), Middle of the Linked List (#876) | LeetCode |
| Dutch flag | Sort Colors (#75) | LeetCode |
| Dutch flag | Sort an array of 0s, 1s and 2s | TUF A2Z (Arrays), GeeksforGeeks |
| All styles | Two-Pointer section of the Array topic | RisingBrain (risingbrain.org/sheet) |
