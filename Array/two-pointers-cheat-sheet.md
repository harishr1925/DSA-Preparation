# Two Pointers: Theory + Code Cheat Sheet

**Core idea:** use two indices instead of a nested loop, so O(n²) becomes **O(n) time and O(1) space**.
The three styles differ in **where the pointers start** and **how they move**.

**Quick decision flow (use this on any new problem):**

1. Comparing the two ends, or searching pairs in sorted data → **Opposite ends**
2. Filtering or rearranging in place, or a linked list cycle/middle → **Same direction (fast and slow)**
3. Only 3 categories, or a pivot partition in a single pass → **Dutch flag**
4. Count (not just find) pairs in a sorted array with repeated values → **Group counting** (section 4)

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

## 4. Counting Pairs When Values Repeat (Group Counting)

An extension of **opposite ends**. Use it when the question asks for the **number** of pairs (not just whether one exists) and the sorted array has **repeated values**.

**Why the usual rule fails:** with distinct values, each value has exactly one partner, so moving either pointer at a match is safe. With duplicates, one value can have many partners, and many copies can share the same partner. Test on `[1,1,1,1]`, target 2 (expected answer **6**):

| What moves at a match | Pairs counted | Count |
|---|---|---|
| `right--` only | (0,3), (0,2), (0,1) | 3 |
| `left++` only | (0,3), (1,3), (2,3) | 3 (different pairs) |
| both pointers | (0,3), (1,2) | 2 |

**Mindset shift:** at a match, "which pointer do I move?" is the wrong question. Treat **equal values as a group**: count the whole group at once, then jump both pointers past it.

### The two counting rules

- **Ends equal → `n × (n − 1) / 2`**, where `n = right - left + 1`. Everything between two equal ends (in a sorted array) is the same value, so every pair works.
  *Real-life picture:* handshakes. 5 people each shake hands once with everyone else: 4 + 3 + 2 + 1 = 10. Check: 4 equal elements give 4 × 3 / 2 = 6.
- **Ends different → `leftCopies × rightCopies`.** Each left copy pairs with each right copy.
  *Real-life picture:* 2 boys and 2 girls, every boy dances with every girl: 2 × 2 = 4. Boys don't pair with boys, because the sum of two left values is not the target.

**Spot it by:** "count the number of pairs", sorted array, "may contain duplicates", target sum.

**Memory hook:** *"Count the crowd, then jump the crowd."*

### Code

```java
int countPairs(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    int count = 0;

    while (left < right) {
        int sum = arr[left] + arr[right];

        if (sum < target) {
            left++;                               // need a bigger sum
        } else if (sum > target) {
            right--;                              // need a smaller sum
        } else {                                  // sum == target
            if (arr[left] == arr[right]) {
                // everything from left to right is the same value
                int n = right - left + 1;
                count += n * (n - 1) / 2;
                break;                            // every pair in that stretch is counted, nothing left
            }

            int leftVal = arr[left], rightVal = arr[right];
            int leftCount = 0, rightCount = 0;

            while (left < right && arr[left] == leftVal) {    // count copies on the left, and jump past them
                leftCount++;
                left++;
            }
            while (right >= left && arr[right] == rightVal) { // count copies on the right, and jump past them
                rightCount++;
                right--;
            }
            count += leftCount * rightCount;
        }
    }
    return count;
}
```

### Walkthrough: `[1,1,2,2,3,3]`, target 4

| Step | left → right | sum | What happens | count |
|---|---|---|---|---|
| 1 | 0 → 5 (values 1, 3) | 4 | Match, ends differ. Two 1s, two 3s: add 2 × 2 = 4. Jump: left → 2, right → 3 | 4 |
| 2 | 2 → 3 (values 2, 2) | 4 | Match, ends equal. n = 2, add 2 × 1 / 2 = 1. Stop | **5** |

Check by hand: (1,3) gives 4 combinations, (2,2) gives 1 combination, total **5**.

### The counting block, line by line

```java
int leftVal = arr[left], rightVal = arr[right];   // save the values: left/right are about to move
int leftCount = 0, rightCount = 0;

while (left < right && arr[left] == leftVal) {    // loop 1: count the left group, and step past it
    leftCount++;
    left++;
}
while (right >= left && arr[right] == rightVal) { // loop 2: count the right group, and step past it
    rightCount++;
    right--;
}
count += leftCount * rightCount;
```

After the block, `left` and `right` sit on the first values that are **not** part of either group, so no pair is ever counted twice.

### Why loop 2 uses `right >= left` (not `right > left`)

Think of elements as seats in a row, with the **left loop going first**:

- **Loop 1** must **not** take the seat at `right`, because that seat belongs to the right group. So its condition is strict: `left < right`.
- **Loop 2** goes second. `left` now points at the first seat the left group did **not** take. That seat is still unclaimed, so the right group may take it. So its condition is inclusive: `right >= left`.

Proof with `[1,3]`, target 4 (expected 1): after loop 1, `left = 1 = right`, and that element is the 3 (it belongs to the right group). With `>=` loop 2 counts it, so `count = 1 × 1 = 1`. With `>` loop 2 never runs, `rightCount = 0`, and the answer is a wrong 0.

It isn't only the `left == right` case. In `[1,3,3]`, target 4 (expected 2), loop 2 must count the 3 at index 1 as well, which only happens with `>=`.

*Note:* in this branch the two ends have different values, so loop 2 would stop on its own at the last left-group element. The guard is mostly defensive, but if you write it, it must be `>=`.

### Complexity

**O(n) time** (each pointer only moves inward, never back) and **O(1) space**. With large inputs `n * (n - 1) / 2` can overflow `int` (n = 100,000 already does), so use `long` for `count` and the product when the problem allows.

### Self-checks (trace by hand)

1. `[1,3]`, target 4: after the counting block, what are `left`, `right`, `leftCount`, `rightCount`, `count`? (Expect `1, 0, 1, 1, 1`.)
2. `[1,3,3]`, target 4: run the block once with `>=` and once with `>`. One gives 2, the other gives 1.
3. `[1,2,2,3,3,3]`, target 5: work out the expected answer first by listing the value pairs (expect 6), then trace the code.

---

## Complexity (all examples above)

| Style | Time | Space | Why |
|---|---|---|---|
| Opposite ends | O(n) | O(1) | Combined pointer movement covers the array once, so it is not nested |
| Reader and writer | O(n) | O(1) | One pass, only two index variables |
| Tortoise and hare | O(n) | O(1) | Fast reaches the end in at most n/2 steps |
| Dutch flag | O(n) | O(1) | Each element is examined once; `mid` never goes backward |
| Group counting (pairs with duplicates) | O(n) | O(1) | Each pointer only moves inward; every group of equal values is jumped over once |

*Exception:* 3Sum is O(n²) because the two-pointer scan sits **inside** a `for` loop (multiply, not add).

---

## Pitfalls You Have Already Hit (and Fixed)

- **`left <= right` in opposite-ends problems:** when both pointers land on the same index, one element gets used twice. Use `left < right` whenever the problem needs two distinct positions. (`<=` is for cases like binary search, where the last remaining element still needs checking.)
- **Sentinel values that collide with real data:** starting `prevNumber = 0` broke when the array's first real value was 0. Compare against an actual array position instead, such as `arr[count - 1]`.
- **Duplicate results in 3Sum:** after a match, skip repeated values at `i`, `j` and `k`, not just at `i`.
- **Time vs space with recursion:** total work across all levels is time. The deepest chain of calls alive at once is space.
- **Counting pairs with duplicates:** moving only one pointer (or both) at a match always misses pairs when values repeat. Count the whole group of equal values at once, then jump past it.
- **Inclusive vs strict loop bounds in two loops that share a boundary:** the loop that runs second may need `>=` because the first loop already stopped on an element the second one still owns.

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
| Group counting | Count pairs with a given sum in a sorted array with duplicates (re-attempt without looking) | Brought by Harish, platform not named |
| All styles | Two-Pointer section of the Array topic | RisingBrain (risingbrain.org/sheet) |
