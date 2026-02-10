# Whiteboard Practice Set (System / ML Interview)

> **Goal**: Be able to write a **correct baseline** for each problem on a whiteboard in **10–15 minutes**, and clearly explain the complexity and possible optimizations.
>
> **Universal whiteboard opener** (say this at the start of every problem):
> "Let me first write a correct baseline, then we can discuss optimizations and system constraints."

> **First-timer tips**:
> - Always **talk while you write**. Silence is the enemy in whiteboard interviews.
> - Start by **restating the problem** in your own words to confirm understanding.
> - Write **function signature first**, then fill in the body.
> - If you get stuck, say: "Let me think about the brute-force approach first."

---
---

# PART A — Data Structures & Algorithms (Problems 1–10, 10 problems)

---

## 1) LeetCode 1 — Two Sum (Hash Table)

### Problem
Given an array `nums` and an integer `target`, return the **indices** of the two numbers that add up to `target`.
- Each input has **exactly one** solution.
- You may **not** use the same element twice.

**Example**: `nums = [2, 7, 11, 15], target = 9` → `[0, 1]` (because `2 + 7 = 9`)

### Thought Process (say this out loud)
1. "The brute-force approach is to check every pair — that's O(n²). Can we do better?"
2. "If I've seen a number `x`, I just need to know if `target - x` appeared before."
3. "A hash table gives me O(1) lookup — so I can do this in one pass, O(n)."

### Whiteboard Template (O(n))
```python
def two_sum(nums, target):
    seen = {}          # value -> index
    for i, x in enumerate(nums):
        need = target - x
        if need in seen:
            return [seen[need], i]
        seen[x] = i    # store AFTER checking, to avoid using same element twice
    return None
```

### Line-by-Line Explanation
| Line | What it does | Why |
|------|-------------|-----|
| `seen = {}` | Create an empty hash map | We'll store numbers we've already visited |
| `for i, x in enumerate(nums)` | Loop through each element with its index | We need both the value and index |
| `need = target - x` | Calculate the complement | If `need` exists in `seen`, we found our pair |
| `if need in seen` | O(1) hash table lookup | This is where the speedup comes from |
| `return [seen[need], i]` | Return both indices | `seen[need]` is the earlier index, `i` is the current |
| `seen[x] = i` | Store current number AFTER checking | Prevents using the same element twice |

### Must-Say Points
- "I use a hash table to turn O(n²) brute-force into O(n) single-pass."
- **Time**: O(n) — one pass through the array
- **Space**: O(n) — hash table stores up to n elements

### Common Pitfalls
- Storing `seen[x] = i` **before** the check → might match an element with itself
- Forgetting to handle the case where no solution exists (depends on problem guarantee)

### Follow-up Questions the Interviewer Might Ask
- "What if there are multiple valid pairs?" → Return all pairs, or the first one found.
- "What if the array is sorted?" → Use two pointers instead (Problem 2 below).
- "What if the array is too large for memory?" → External sort + two pointers from disk.

---

## 2) Two Sum II — Sorted Array (Two Pointers)

### Problem
Given a **sorted** array `nums`, find two numbers that add up to `target`.

**Example**: `nums = [1, 3, 5, 7, 9], target = 10` → `[1, 3]` (indices of 3 and 7)

### Thought Process
1. "The array is sorted — that's useful information I should exploit."
2. "If I put one pointer at the start and one at the end, I can adjust based on the sum."
3. "Sum too small → move left pointer right. Sum too big → move right pointer left."

### Whiteboard Template (O(n))
```python
def two_sum_sorted(nums, target):
    left, right = 0, len(nums) - 1
    while left < right:
        current_sum = nums[left] + nums[right]
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1      # need a bigger number
        else:
            right -= 1     # need a smaller number
    return None
```

### Line-by-Line Explanation
| Line | What it does | Why |
|------|-------------|-----|
| `left, right = 0, len(nums) - 1` | Pointers at both ends | Start with smallest + largest |
| `while left < right` | Loop until pointers meet | If they cross, no solution |
| `current_sum < target` → `left += 1` | Move left pointer right | We need a larger value |
| `current_sum > target` → `right -= 1` | Move right pointer left | We need a smaller value |

### Must-Say Points
- "I exploit the sorted order: sum too small → move left; sum too big → move right."
- **Time**: O(n), **Space**: O(1) — no extra data structure needed
- "Compared to the hash table approach, this trades the O(n) space for the sorted-order assumption."

### Why Two Pointers Work (for your understanding)
```
Array: [1, 3, 5, 7, 9], target = 10

Step 1: left=0(1), right=4(9) → sum=10 ✓ Found!

If target were 8:
Step 1: left=0(1), right=4(9) → sum=10 > 8 → right--
Step 2: left=0(1), right=3(7) → sum=8 ✓ Found!
```
The key insight: moving `left` right **always increases** the sum; moving `right` left **always decreases** it. So we can systematically narrow down to the answer.

---

## 3) LeetCode 20 — Valid Parentheses (Stack)

### Problem
Given a string containing only `(`, `)`, `{`, `}`, `[`, `]`, determine if the input string is valid.
- Open brackets must be closed by the **same type**.
- Open brackets must be closed in the **correct order**.

**Example**: `"({[]})"` → `True`; `"({[}])"` → `False`

### Thought Process
1. "Every closing bracket must match the **most recent** unmatched opening bracket."
2. "Most recent = Last In, First Out = **Stack**."
3. "When I see an opener, push; when I see a closer, pop and check if it matches."

### Whiteboard Template (O(n))
```python
def is_valid(s):
    stack = []
    match = {')': '(', ']': '[', '}': '{'}   # closer -> opener
    for ch in s:
        if ch in match:                       # it's a closing bracket
            if not stack or stack[-1] != match[ch]:
                return False                  # mismatch or empty stack
            stack.pop()
        else:                                 # it's an opening bracket
            stack.append(ch)
    return len(stack) == 0                    # all openers must be matched
```

### Line-by-Line Explanation
| Line | What it does | Why |
|------|-------------|-----|
| `match = {')':'(', ...}` | Map each closer to its opener | Quick lookup for matching |
| `if ch in match` | Check if current char is a closer | Closers need to match the stack top |
| `not stack` | Stack is empty but we got a closer | No opener to match → invalid |
| `stack[-1] != match[ch]` | Top of stack doesn't match | Wrong type of bracket → invalid |
| `stack.pop()` | Remove the matched opener | Successfully paired |
| `else: stack.append(ch)` | Push openers onto stack | Wait for a future closer |
| `return len(stack) == 0` | Check if any openers remain | Leftover openers = invalid |

### Must-Say Points
- "When I see a closing bracket, I match it against the stack top; mismatch means invalid."
- **Time**: O(n), **Space**: O(n) (stack can hold up to n/2 openers)

### Common Pitfalls
- Forgetting `not stack` check → crash on `stack[-1]` when stack is empty
- Returning `True` immediately when a match is found → must check ALL characters
- Forgetting the final `len(stack) == 0` → `"((("` would incorrectly return `True`

---

## 4) LeetCode 933 — Number of Recent Calls (Queue / Sliding Window)

### Problem
Implement a `RecentCounter` class that counts the number of requests in the past 3000 milliseconds (inclusive).
- `ping(t)`: Adds a new request at time `t` and returns the number of requests in `[t - 3000, t]`.
- Calls to `ping` are guaranteed to have strictly increasing `t`.

### Thought Process
1. "I need to track a sliding window of timestamps."
2. "Old timestamps (< t - 3000) should be removed."
3. "A **queue** (FIFO) is perfect: new timestamps enter at the back, old ones leave from the front."

### Whiteboard Template
```python
from collections import deque

class RecentCounter:
    def __init__(self):
        self.q = deque()        # stores timestamps in order

    def ping(self, t):
        self.q.append(t)                    # add new request
        while self.q[0] < t - 3000:         # remove expired requests
            self.q.popleft()
        return len(self.q)                  # remaining = recent calls
```

### Visual Walkthrough
```
ping(1):     q = [1]           → return 1
ping(100):   q = [1, 100]     → return 2
ping(3001):  q = [1, 100, 3001]  → 1 >= 3001-3000=1 ✓ → return 3
ping(3002):  q = [1, 100, 3001, 3002] → 1 < 3002-3000=2 → pop 1
             q = [100, 3001, 3002]     → 100 >= 2 ✓     → return 3
```

### Must-Say Points
- "The queue always holds exactly the timestamps within the valid window [t-3000, t]."
- **Time**: Amortized O(1) per `ping` — each timestamp is enqueued and dequeued at most once.
- **Space**: O(W) where W = max window size (number of calls in any 3000ms window).

### Real-World Connection
- This is essentially a **rate limiter** — a common system design building block.
- In production: Redis sorted sets or sliding window counters implement this pattern.

---

## 5) LeetCode 206 — Reverse Linked List (Linked List)

### Problem
Reverse a singly linked list.

**Example**: `1 → 2 → 3 → 4 → 5` becomes `5 → 4 → 3 → 2 → 1`

### Thought Process
1. "I need to reverse the direction of every `next` pointer."
2. "I'll use three pointers: `prev`, `curr`, `nxt` to avoid losing references."
3. "At each step: save next, reverse the pointer, advance all three."

### Whiteboard Template — Iterative (safest for interviews)
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_list(head):
    prev = None
    curr = head
    while curr:
        nxt = curr.next      # 1. save next node (before we lose it)
        curr.next = prev     # 2. reverse the pointer
        prev = curr          # 3. advance prev
        curr = nxt           # 4. advance curr
    return prev              # prev is now the new head
```

### Visual Walkthrough
```
Initial:  None ← prev   curr → [1] → [2] → [3] → None

Step 1:   nxt = [2]
          [1].next = None  (reverse!)
          prev = [1], curr = [2]
          
          None ← [1]   curr → [2] → [3] → None
                 prev

Step 2:   nxt = [3]
          [2].next = [1]  (reverse!)
          prev = [2], curr = [3]
          
          None ← [1] ← [2]   curr → [3] → None
                       prev

Step 3:   nxt = None
          [3].next = [2]  (reverse!)
          prev = [3], curr = None
          
          None ← [1] ← [2] ← [3]
                             prev    curr = None → loop ends

Return prev = [3], which is the new head: [3] → [2] → [1] → None ✓
```

### Must-Say Points
- "Three pointers: prev / curr / nxt — save-next, reverse, advance."
- **Time**: O(n), **Space**: O(1)

### Bonus: Recursive Version (if interviewer asks)
```python
def reverse_list_recursive(head):
    if not head or not head.next:
        return head
    new_head = reverse_list_recursive(head.next)
    head.next.next = head     # the node after me should point back to me
    head.next = None           # I now point to nothing (I'm the new tail)
    return new_head
```
- Time O(n), Space O(n) due to recursion stack.
- The iterative version is preferred in interviews because it's O(1) space.

---

## 6) LeetCode 283 — Move Zeroes (Two Pointers / In-place)

### Problem
Given an array `nums`, move all `0`s to the **end** while maintaining the relative order of non-zero elements. Must be done **in-place**.

**Example**: `[0, 1, 0, 3, 12]` → `[1, 3, 12, 0, 0]`

### Thought Process
1. "I need to separate non-zeros from zeros, keeping non-zero order."
2. "This is like partitioning — similar to quicksort's partition step."
3. "Use a `slow` pointer to track where the next non-zero should go."

### Whiteboard Template (Fast-Slow Pointers)
```python
def move_zeroes(nums):
    slow = 0                            # position for next non-zero
    for fast in range(len(nums)):
        if nums[fast] != 0:
            nums[slow], nums[fast] = nums[fast], nums[slow]
            slow += 1
```

### Visual Walkthrough
```
nums = [0, 1, 0, 3, 12]
        s
        f

f=0: nums[0]=0, skip
f=1: nums[1]=1≠0, swap(s=0, f=1) → [1, 0, 0, 3, 12], s=1
f=2: nums[2]=0, skip
f=3: nums[3]=3≠0, swap(s=1, f=3) → [1, 3, 0, 0, 12], s=2
f=4: nums[4]=12≠0, swap(s=2,f=4) → [1, 3, 12, 0, 0], s=3

Result: [1, 3, 12, 0, 0] ✓
```

### Must-Say Points
- "`slow` points to the position where the next non-zero element should be placed."
- **Time**: O(n), **Space**: O(1)
- "This is essentially the same idea as quicksort's partition step."

---

## 7) LeetCode 121 — Best Time to Buy and Sell Stock (Greedy / One-Pass) ★ NEW — REAL INTERVIEW QUESTION

### Problem
Given an array `prices` where `prices[i]` is the stock price on day `i`, find the **maximum profit** you can achieve by buying on one day and selling on a later day. If no profit is possible, return 0.

- You must **buy before you sell** (can't sell on an earlier day).
- You can only make **one transaction** (one buy + one sell).

**Example 1**: `prices = [7, 1, 5, 3, 6, 4]` → `5` (buy at 1, sell at 6)
**Example 2**: `prices = [7, 6, 4, 3, 1]` → `0` (prices only go down, no profit possible)

### Thought Process (say this out loud)
1. "Brute force: check every (buy, sell) pair — that's O(n²). Can I do better?"
2. "Key insight: as I scan left to right, I only care about the **minimum price so far**."
3. "At each day, the best profit I could make is `today's price - min price so far`."
4. "So I track two things: the running minimum and the running maximum profit."

### Why This Works — The Core Idea
```
For any selling day i, the best buying day is the day with the
lowest price BEFORE day i.

So instead of checking all pairs, I just remember the lowest
price I've seen so far, and check if selling today beats my
best profit.

This turns an O(n²) two-loop problem into an O(n) one-pass problem.
```

### Whiteboard Template (O(n))
```python
def max_profit(prices):
    min_price = float('inf')    # smallest price seen so far
    max_profit = 0              # best profit so far

    for price in prices:
        if price < min_price:
            min_price = price           # found a new lowest buying point
        else:
            profit = price - min_price  # what if I sell today?
            if profit > max_profit:
                max_profit = profit     # update best profit
    
    return max_profit
```

### Cleaner Pythonic Version (same logic)
```python
def max_profit(prices):
    min_price = float('inf')
    max_profit = 0
    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)
    return max_profit
```

### Line-by-Line Explanation
| Line | What it does | Why |
|------|-------------|-----|
| `min_price = float('inf')` | Initialize to infinity | Any real price will be smaller |
| `max_profit = 0` | Initialize to 0 | If no profit possible, return 0 |
| `if price < min_price` | Found a cheaper buying point | Update the running minimum |
| `profit = price - min_price` | Calculate profit if selling today | Using the best buying point so far |
| `if profit > max_profit` | Check if this is the best profit | Update running maximum |

### Visual Walkthrough
```
prices = [7, 1, 5, 3, 6, 4]

Day 0: price=7, min_price=7, profit=0,  max_profit=0
Day 1: price=1, min_price=1, profit=0,  max_profit=0   ← new min!
Day 2: price=5, min_price=1, profit=4,  max_profit=4   ← sell at 5, bought at 1
Day 3: price=3, min_price=1, profit=2,  max_profit=4
Day 4: price=6, min_price=1, profit=5,  max_profit=5   ← sell at 6, bought at 1 ✓ BEST
Day 5: price=4, min_price=1, profit=3,  max_profit=5

Answer: 5 (buy at day 1 for $1, sell at day 4 for $6)
```

### Must-Say Points
- "I track the minimum price seen so far and the maximum profit achievable at each step."
- "This is a one-pass greedy approach — at each step I make the locally optimal decision."
- **Time**: O(n) — single pass through the array
- **Space**: O(1) — only two variables

### Common Pitfalls
- Initializing `min_price = prices[0]` without checking if array is empty → crash
- Forgetting that we **cannot sell before buying** (must scan left-to-right)
- Confusing this with the "multiple transactions" variant (LeetCode 122)

### Follow-up Questions the Interviewer Might Ask
- "What if you can make multiple transactions?" → Track every upward slope: `max_profit += max(0, prices[i] - prices[i-1])` (LeetCode 122)
- "What if there's a cooldown between transactions?" → DP with states (LeetCode 309)
- "What if there's a transaction fee?" → DP with fee subtracted (LeetCode 714)
- "Can you find the actual buy and sell days?" → Store the index when updating `min_price` and `max_profit`

### Pattern Recognition
This problem belongs to the **"track running min/max"** pattern:
- Best Time to Buy and Sell Stock → track min price
- Maximum Subarray (Kadane's) → track running sum, reset when negative
- Trapping Rain Water → track left max and right max
If you master this problem, the same mental model applies to many others.

---

## 8) Binary Tree Traversals + Lowest Common Ancestor (Tree) ★ NEW

### 8a) Binary Tree Traversals

### Problem
Given a binary tree, perform **inorder**, **preorder**, and **postorder** traversals.

```
        1
       / \
      2   3
     / \
    4   5

Preorder  (Root-Left-Right): [1, 2, 4, 5, 3]
Inorder   (Left-Root-Right): [4, 2, 5, 1, 3]
Postorder (Left-Right-Root): [4, 5, 2, 3, 1]
```

### Thought Process
1. "Tree traversals are naturally recursive — the tree itself is a recursive structure."
2. "The three orders differ only in **when** we process the root node."

### Whiteboard Template — Recursive (simplest to write)
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def preorder(root):
    """Root → Left → Right"""
    if not root:
        return []
    return [root.val] + preorder(root.left) + preorder(root.right)

def inorder(root):
    """Left → Root → Right"""
    if not root:
        return []
    return inorder(root.left) + [root.val] + inorder(root.right)

def postorder(root):
    """Left → Right → Root"""
    if not root:
        return []
    return postorder(root.left) + postorder(root.right) + [root.val]
```

### Whiteboard Template — Iterative Inorder (interviewers love this)
```python
def inorder_iterative(root):
    result = []
    stack = []
    curr = root
    while curr or stack:
        while curr:             # go as far left as possible
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()      # backtrack
        result.append(curr.val) # visit node
        curr = curr.right       # go right
    return result
```

### Why Iterative Inorder Works (step by step)
```
Tree:     1
         / \
        2   3
       / \
      4   5

Step 1: Push 1, 2, 4 (go left)     stack=[1,2,4]
Step 2: Pop 4, visit 4, go right(None)  result=[4]
Step 3: Pop 2, visit 2, go right(5)     result=[4,2]
Step 4: Push 5 (go left of 5 = None)    stack=[1,5]
Step 5: Pop 5, visit 5, go right(None)  result=[4,2,5]
Step 6: Pop 1, visit 1, go right(3)     result=[4,2,5,1]
Step 7: Push 3 (go left of 3 = None)    stack=[3]
Step 8: Pop 3, visit 3, go right(None)  result=[4,2,5,1,3]
```

### Must-Say Points
- "Recursive is clean but uses O(h) call stack. Iterative uses an explicit stack."
- **Time**: O(n) for all traversals — visit every node once.
- **Space**: O(h) where h = tree height. Worst case O(n) for skewed tree, O(log n) for balanced.

### Level-Order Traversal (BFS) — Bonus
```python
from collections import deque

def level_order(root):
    if not root:
        return []
    result = []
    queue = deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):    # process one level at a time
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    return result
# For the tree above: [[1], [2, 3], [4, 5]]
```

---

### 8b) LeetCode 236 — Lowest Common Ancestor of a Binary Tree (LCA)

### Problem
Given a binary tree and two nodes `p` and `q`, find their **Lowest Common Ancestor** (the deepest node that has both `p` and `q` as descendants).

```
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4

LCA(5, 1) = 3    (root is their ancestor)
LCA(5, 4) = 5    (5 is ancestor of 4, and 5 is ancestor of itself)
```

### Thought Process
1. "If the current node is `p` or `q`, it could be the LCA."
2. "Recursively search left and right subtrees."
3. "If both sides return non-null, the current node is the LCA (p and q are in different subtrees)."
4. "If only one side returns non-null, propagate that result up."

### Whiteboard Template
```python
def lowest_common_ancestor(root, p, q):
    # Base case: reached a leaf's child, or found p or q
    if not root or root == p or root == q:
        return root

    # Search both subtrees
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)

    # If both sides found something, current node is the LCA
    if left and right:
        return root

    # Otherwise, return whichever side found something (or None)
    return left if left else right
```

### Why This Works — Detailed Walkthrough
```
Find LCA(5, 4) in the tree above:

Call on node 3:
  left  = call on node 5
    → node 5 == p → return 5       ← found p!
  right = call on node 1
    → call on 0 → None
    → call on 8 → None
    → return None                   ← q not in right subtree of 3? 
    
Wait — node 4 is under node 5, not under node 1.

Actually let me retrace:
Call on node 3:
  left  = call on node 5:
    → root == p (5), so return 5 immediately
    (We don't recurse further — 4 is below 5 but we already found p)
  right = call on node 1:
    → neither p nor q found → returns None
  
  left=5, right=None → return left = 5

Answer: LCA(5, 4) = 5 ✓
(Because node 5 is an ancestor of node 4, and also IS node p)
```

### Must-Say Points
- "Post-order traversal: check left, check right, then decide at current node."
- "If both children return non-null, this node is the split point → LCA."
- **Time**: O(n), **Space**: O(h)

### Common Pitfalls
- Trying to find exact paths first (works but is more complex and uses more space)
- Forgetting that a node can be its own ancestor

---

## 9) BFS Shortest Path + Topological Sort (Graph) ★ NEW

### 9a) BFS Shortest Path in Unweighted Graph

### Problem
Given an unweighted graph and a source node, find the shortest path (minimum edges) from source to all other nodes.

### Thought Process
1. "In an unweighted graph, BFS naturally finds shortest paths."
2. "BFS explores nodes layer by layer — layer 0 is distance 0, layer 1 is distance 1, etc."
3. "I need a queue for BFS and a distance/visited structure."

### Whiteboard Template
```python
from collections import deque

def bfs_shortest_path(graph, source):
    """
    graph: adjacency list, e.g. {0: [1,2], 1: [0,3], ...}
    Returns: dict of {node: shortest_distance_from_source}
    """
    dist = {source: 0}
    queue = deque([source])

    while queue:
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor not in dist:         # not visited yet
                dist[neighbor] = dist[node] + 1
                queue.append(neighbor)

    return dist
```

### Visual Walkthrough
```
Graph (adjacency list):
  0 — 1 — 3
  |       |
  2 ——————4

graph = {0:[1,2], 1:[0,3], 2:[0,4], 3:[1,4], 4:[2,3]}
source = 0

Step 0: queue=[0],        dist={0:0}
Step 1: pop 0 → neighbors 1,2
        queue=[1,2],      dist={0:0, 1:1, 2:1}
Step 2: pop 1 → neighbors 0(visited),3
        queue=[2,3],      dist={0:0, 1:1, 2:1, 3:2}
Step 3: pop 2 → neighbors 0(visited),4
        queue=[3,4],      dist={0:0, 1:1, 2:1, 3:2, 4:2}
Step 4: pop 3 → neighbors 1(visited),4(visited)
Step 5: pop 4 → neighbors 2(visited),3(visited)
        queue empty → done!
```

### To Reconstruct the Actual Path
```python
def bfs_with_path(graph, source, target):
    dist = {source: 0}
    parent = {source: None}
    queue = deque([source])

    while queue:
        node = queue.popleft()
        if node == target:
            break
        for neighbor in graph[node]:
            if neighbor not in dist:
                dist[neighbor] = dist[node] + 1
                parent[neighbor] = node
                queue.append(neighbor)

    # Reconstruct path by following parent pointers
    path = []
    curr = target
    while curr is not None:
        path.append(curr)
        curr = parent[curr]
    return path[::-1]    # reverse to get source → target order
```

### Must-Say Points
- "BFS guarantees shortest path in unweighted graphs because it explores layer by layer."
- **Time**: O(V + E), **Space**: O(V)
- "For weighted graphs, I'd use Dijkstra's algorithm instead."

---

### 9b) Topological Sort (DAG)

### Problem
Given a **Directed Acyclic Graph (DAG)**, return a valid topological ordering — an ordering where for every edge u → v, u appears before v.

**Use case**: course prerequisites, build dependencies, task scheduling.

### Thought Process
1. "Topological sort only works on DAGs (no cycles)."
2. "Two main approaches: **Kahn's algorithm** (BFS with in-degree) or **DFS-based**."
3. "Kahn's is easier to write on a whiteboard and naturally detects cycles."

### Whiteboard Template — Kahn's Algorithm (BFS-based)
```python
from collections import deque

def topological_sort(num_nodes, edges):
    """
    num_nodes: number of nodes (0 to num_nodes-1)
    edges: list of (u, v) meaning u must come before v
    Returns: topological order, or [] if cycle exists
    """
    # Step 1: Build adjacency list and in-degree count
    graph = {i: [] for i in range(num_nodes)}
    in_degree = {i: 0 for i in range(num_nodes)}

    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1

    # Step 2: Start with all nodes that have in-degree 0
    queue = deque([n for n in range(num_nodes) if in_degree[n] == 0])
    order = []

    # Step 3: BFS — process nodes with in-degree 0
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    # Step 4: If we processed all nodes, it's a valid DAG
    if len(order) == num_nodes:
        return order
    else:
        return []    # cycle detected! Not all nodes reached in-degree 0
```

### Visual Walkthrough
```
Courses: 0,1,2,3
Prerequisites: 0→1, 0→2, 1→3, 2→3
(Must take 0 before 1 and 2; must take 1 and 2 before 3)

  0 → 1 → 3
  ↓       ↑
  2 ——————┘

in_degree: {0:0, 1:1, 2:1, 3:2}

Step 1: queue=[0] (only node with in-degree 0)
        pop 0, order=[0]
        → reduce in_degree of 1 (now 0) and 2 (now 0)
        queue=[1, 2]

Step 2: pop 1, order=[0, 1]
        → reduce in_degree of 3 (now 1)
        queue=[2]

Step 3: pop 2, order=[0, 1, 2]
        → reduce in_degree of 3 (now 0)
        queue=[3]

Step 4: pop 3, order=[0, 1, 2, 3]
        queue empty, len(order)==4==num_nodes ✓

Valid topological order: [0, 1, 2, 3]
(Also valid: [0, 2, 1, 3] — topological orders aren't unique)
```

### Must-Say Points
- "Kahn's algorithm: repeatedly remove nodes with in-degree 0."
- "If we can't process all nodes, there's a cycle in the graph."
- **Time**: O(V + E), **Space**: O(V + E)

### Cycle Detection Tip
- "The cycle detection is built into Kahn's: if `len(order) < num_nodes`, a cycle exists because some nodes could never reach in-degree 0."

---

## 10) LeetCode 200 — Number of Islands (BFS/DFS on Grid) ★ NEW BONUS

### Problem
Given a 2D grid of `'1'`s (land) and `'0'`s (water), count the number of islands. An island is surrounded by water and connected horizontally/vertically.

### Whiteboard Template (BFS)
```python
from collections import deque

def num_islands(grid):
    if not grid:
        return 0

    rows, cols = len(grid), len(grid[0])
    count = 0

    def bfs(r, c):
        queue = deque([(r, c)])
        grid[r][c] = '0'              # mark as visited
        while queue:
            row, col = queue.popleft()
            for dr, dc in [(0,1),(0,-1),(1,0),(-1,0)]:  # 4 directions
                nr, nc = row + dr, col + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == '1':
                    grid[nr][nc] = '0'  # mark before enqueuing to avoid duplicates
                    queue.append((nr, nc))

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1':
                count += 1
                bfs(r, c)         # sink the entire island

    return count
```

### Must-Say Points
- "Scan the grid; when I find a '1', that's a new island — BFS to mark all connected land."
- "I modify the grid in-place as a visited marker. Could use a separate `visited` set instead."
- **Time**: O(rows × cols), **Space**: O(min(rows, cols)) for BFS queue

---
---

# PART B — Machine Learning Core (Problems 11–13)

---

## 11) Linear Regression — Mini-batch Gradient Descent

### Model
**ŷ = Xw + b** (linear prediction with bias term)

**Loss**: MSE = (1/N) Σ(ŷᵢ - yᵢ)²

### Thought Process
1. "Linear regression minimizes mean squared error."
2. "I'll derive the gradient of MSE w.r.t. `w` and `b`, then iterate."
3. "For mini-batch: sample a batch, compute gradient on that batch, update."

### Gradient Derivation (be able to explain this)
```
Loss L = (1/B) ||Xw + b - y||²

∂L/∂w = (2/B) Xᵀ(Xw + b - y)    — shape: [d]  (same as w)
∂L/∂b = (2/B) Σ(Xw + b - y)      — shape: scalar (same as b)

Simplified (absorb the 2 into learning rate):
∂L/∂w = (1/B) Xᵀ · err
∂L/∂b = mean(err)
```

### Whiteboard Template
```python
import numpy as np

def linear_regression_sgd(X, y, lr=0.01, num_epochs=100, batch_size=32):
    """
    X: [N, d] feature matrix
    y: [N]    target vector
    """
    N, d = X.shape
    w = np.zeros(d)      # weight vector
    b = 0.0              # bias (scalar)

    for epoch in range(num_epochs):
        # Optional: shuffle data each epoch
        indices = np.random.permutation(N)
        X, y = X[indices], y[indices]

        for i in range(0, N, batch_size):
            Xb = X[i:i+batch_size]          # batch features [B, d]
            yb = y[i:i+batch_size]          # batch targets  [B]

            pred = Xb @ w + b               # predictions    [B]
            err  = pred - yb                # errors         [B]

            grad_w = Xb.T @ err / len(yb)  # [d, B] @ [B] → [d]
            grad_b = np.mean(err)           # scalar

            w -= lr * grad_w
            b -= lr * grad_b

    return w, b
```

### Shape Check (always verify on whiteboard!)
| Variable | Shape | Matches |
|----------|-------|---------|
| `Xb` | [B, d] | — |
| `pred` | [B] | ✓ same as `yb` |
| `err` | [B] | — |
| `grad_w` = `Xb.T @ err / B` | [d, B] @ [B] → [d] | ✓ same as `w` |
| `grad_b` = `mean(err)` | scalar | ✓ same as `b` |

### Must-Say Points
- "`w` and `b` are both learnable parameters; `b`'s gradient is just the mean error."
- **Time per epoch**: O(N × d)
- "This is the baseline. Possible improvements: L2 regularization, learning rate scheduling, data shuffling, early stopping."

### Follow-up: Adding L2 Regularization
```python
# Just add lambda * w to the gradient:
grad_w = Xb.T @ err / len(yb) + reg_lambda * w
# This penalizes large weights, preventing overfitting
```

### Closed-Form vs Gradient Descent
- Closed-form: `w = (XᵀX)⁻¹Xᵀy` — exact but O(d³) matrix inversion
- Gradient descent: iterative, O(Nd) per step, scales better for large d
- "In practice, if d < 10000 and N is moderate, closed-form is fine. Otherwise, use SGD."

---

## 12) Logistic Regression + Cross-Entropy Loss ★ NEW

### Model
**Binary classification**: predict probability that y = 1

```
z = Xw + b                         (linear combination)
ŷ = σ(z) = 1 / (1 + exp(-z))      (sigmoid squashes to [0, 1])
```

### Loss Function — Binary Cross-Entropy
```
L = -(1/N) Σ [yᵢ log(ŷᵢ) + (1 - yᵢ) log(1 - ŷᵢ)]
```

**Why this loss?**
- When y=1: L penalizes small ŷ (we want ŷ close to 1)
- When y=0: L penalizes large ŷ (we want ŷ close to 0)
- This is derived from maximum likelihood estimation under a Bernoulli model.

### Gradient Derivation (key insight!)
```
∂L/∂w = (1/N) Xᵀ(ŷ - y)     — shape [d], same as w
∂L/∂b = mean(ŷ - y)          — scalar, same as b

The sigmoid + cross-entropy gradients have a beautifully simple form:
the gradient is the SAME as linear regression's MSE gradient,
just with ŷ = σ(Xw+b) instead of ŷ = Xw+b.
```

### Whiteboard Template
```python
import numpy as np

def sigmoid(z):
    return 1.0 / (1.0 + np.exp(-z))

def logistic_regression_sgd(X, y, lr=0.01, num_epochs=100, batch_size=32):
    """
    X: [N, d] feature matrix
    y: [N]    binary labels (0 or 1)
    """
    N, d = X.shape
    w = np.zeros(d)
    b = 0.0

    for epoch in range(num_epochs):
        indices = np.random.permutation(N)
        X, y = X[indices], y[indices]

        for i in range(0, N, batch_size):
            Xb = X[i:i+batch_size]
            yb = y[i:i+batch_size]

            z    = Xb @ w + b          # linear [B]
            yhat = sigmoid(z)          # probabilities [B]
            err  = yhat - yb           # error [B]

            grad_w = Xb.T @ err / len(yb)   # [d]
            grad_b = np.mean(err)            # scalar

            w -= lr * grad_w
            b -= lr * grad_b

    return w, b

def predict(X, w, b, threshold=0.5):
    probs = sigmoid(X @ w + b)
    return (probs >= threshold).astype(int)
```

### Must-Say Points
- "Sigmoid maps the linear output to a probability in [0, 1]."
- "Cross-entropy + sigmoid gives the same gradient form as MSE + linear: `(ŷ - y)`."
- "The prediction threshold is typically 0.5 but can be tuned for precision/recall trade-off."
- **Time per epoch**: O(N × d)

### Comparison Table: Linear Regression vs Logistic Regression

| Aspect | Linear Regression | Logistic Regression |
|--------|------------------|-------------------|
| Task | Regression (continuous) | Classification (binary) |
| Output | ŷ = Xw + b (any real number) | ŷ = σ(Xw + b) (probability in [0,1]) |
| Loss | MSE | Binary Cross-Entropy |
| Gradient of w | (1/B) Xᵀ(ŷ - y) | (1/B) Xᵀ(ŷ - y) — same form! |
| Decision boundary | N/A | σ(Xw+b) = 0.5, i.e., Xw+b = 0 |

### Numerical Stability Tip
```python
# Naive sigmoid can overflow for large negative z:
# 1 / (1 + exp(1000)) → exp overflows!

# Stable version:
def sigmoid_stable(z):
    return np.where(z >= 0,
                    1 / (1 + np.exp(-z)),
                    np.exp(z) / (1 + np.exp(z)))
```

---

## 13) Neural Network Backpropagation (1-Hidden-Layer, Interview Skeleton)

> **Purpose**: Show that you can write forward + backward pass with clear chain-rule steps (no autograd).

### Network Architecture
```
Input:   X        [B, d]
Hidden:  h = ReLU(X @ W1 + b1)    [B, hidden]
Output:  ŷ = h @ W2 + b2          [B, out]
Loss:    MSE = mean((ŷ - y)²)
```

### Thought Process (say this out loud)
1. "I'll first write the forward pass clearly."
2. "Then I'll derive each gradient using the chain rule, working backwards."
3. "The key rule: **gradient shape must match parameter shape**."

### Whiteboard Template (Vectorized)
```python
# ============ FORWARD PASS ============
z1   = X @ W1 + b1          # pre-activation  [B, hidden]
h    = np.maximum(0, z1)    # ReLU activation  [B, hidden]
yhat = h @ W2 + b2          # output           [B, out]

# ============ LOSS ============
err = yhat - y               # [B, out]
loss = np.mean(err ** 2)     # scalar (for monitoring)

# ============ BACKWARD PASS ============
# Start from loss, work backwards through each operation

# dL/dyhat = 2*(yhat - y)/B
dyhat = 2 * err / B          # [B, out]

# --- Layer 2 gradients ---
# yhat = h @ W2 + b2
# ∂L/∂W2 = hᵀ @ dyhat
# ∂L/∂b2 = sum(dyhat, axis=0)

dW2 = h.T @ dyhat            # [hidden, B] @ [B, out] → [hidden, out] ✓
db2 = np.sum(dyhat, axis=0)  # [out] ✓

# --- Backprop through Layer 2 to hidden ---
# ∂L/∂h = dyhat @ W2ᵀ

dh = dyhat @ W2.T            # [B, out] @ [out, hidden] → [B, hidden] ✓

# --- ReLU backward ---
# ReLU'(z) = 1 if z > 0, else 0

dz1 = dh * (z1 > 0)          # element-wise mask [B, hidden] ✓

# --- Layer 1 gradients ---
# z1 = X @ W1 + b1
# ∂L/∂W1 = Xᵀ @ dz1
# ∂L/∂b1 = sum(dz1, axis=0)

dW1 = X.T @ dz1              # [d, B] @ [B, hidden] → [d, hidden] ✓
db1 = np.sum(dz1, axis=0)    # [hidden] ✓

# ============ UPDATE ============
W1 -= lr * dW1
b1 -= lr * db1
W2 -= lr * dW2
b2 -= lr * db2
```

### Complete Shape Verification Table
| Gradient | Computation | Shape | Matches Parameter? |
|----------|-------------|-------|--------------------|
| `dyhat` | `2*err/B` | [B, out] | ✓ same as `yhat` |
| `dW2` | `h.T @ dyhat` | [hidden, out] | ✓ same as `W2` |
| `db2` | `sum(dyhat, axis=0)` | [out] | ✓ same as `b2` |
| `dh` | `dyhat @ W2.T` | [B, hidden] | ✓ same as `h` |
| `dz1` | `dh * (z1>0)` | [B, hidden] | ✓ same as `z1` |
| `dW1` | `X.T @ dz1` | [d, hidden] | ✓ same as `W1` |
| `db1` | `sum(dz1, axis=0)` | [hidden] | ✓ same as `b1` |

### Must-Say Points (what the interviewer cares about most)
- "I write the forward pass first, then backward using the chain rule."
- "Every gradient's shape must match its corresponding parameter."
- "This is hand-written backprop; in practice we use autograd (PyTorch/TensorFlow), but the principle is the same."

### Follow-up: What if We Use Sigmoid Instead of ReLU?
```python
# Forward: h = sigmoid(z1)
# Backward: sigmoid'(z) = sigmoid(z) * (1 - sigmoid(z))
dz1 = dh * h * (1 - h)    # instead of dh * (z1 > 0)
```

### Follow-up: What About Softmax + Cross-Entropy Output?
```python
# For multi-class classification:
# Forward: probs = softmax(yhat)    [B, C]
# Loss: L = -mean(Σ y_true * log(probs))
# Beautiful gradient: dyhat = probs - y_one_hot    [B, C]
# (Same simple form as sigmoid + binary cross-entropy!)
```

---
---

# PART C — System Design Sketch (Problem 14)

---

## 14) Design a TF-IDF Pipeline + Simple Recommender ★ NEW

> **Context**: In ML interviews, you may be asked to sketch a lightweight system on the whiteboard.
> This is NOT a full system design — it's a "can you architect an ML pipeline?" question.

### 14a) TF-IDF Computation Pipeline

### Problem
"Design a pipeline that computes TF-IDF for a corpus of documents and supports querying the most relevant documents for a given query."

### Step 1: Define TF-IDF (say this first)
```
TF(t, d)  = (count of term t in document d) / (total terms in d)
IDF(t, D) = log(N / df(t))    where N = total docs, df(t) = docs containing t
TF-IDF(t, d) = TF(t, d) × IDF(t, D)
```

### Step 2: Sketch the Pipeline Architecture
```
[Raw Documents]
      ↓
[Preprocessing]  → lowercase, tokenize, remove stopwords, optional stemming
      ↓
[Build Vocabulary] → unique terms across all documents
      ↓
[Compute TF]     → for each (term, doc) pair
      ↓
[Compute IDF]    → for each term across the corpus
      ↓
[TF-IDF Matrix]  → sparse matrix [num_docs × vocab_size]
      ↓
[Query Service]  → convert query to TF-IDF vector → cosine similarity → top-K docs
```

### Step 3: Whiteboard Code Sketch
```python
import math
from collections import Counter

def build_tfidf(corpus):
    """
    corpus: list of strings (documents)
    Returns: tfidf_matrix (list of dicts), vocabulary
    """
    N = len(corpus)

    # Step 1: Tokenize
    tokenized = [doc.lower().split() for doc in corpus]

    # Step 2: Compute document frequency (df) for each term
    df = Counter()
    for doc_tokens in tokenized:
        unique_terms = set(doc_tokens)
        for term in unique_terms:
            df[term] += 1

    # Step 3: Compute IDF
    idf = {term: math.log(N / freq) for term, freq in df.items()}

    # Step 4: Compute TF-IDF for each document
    tfidf_matrix = []
    for doc_tokens in tokenized:
        tf = Counter(doc_tokens)
        doc_len = len(doc_tokens)
        tfidf = {}
        for term, count in tf.items():
            tfidf[term] = (count / doc_len) * idf[term]
        tfidf_matrix.append(tfidf)

    return tfidf_matrix, idf

def query_tfidf(query, tfidf_matrix, idf, top_k=3):
    """Find most relevant documents for a query using cosine similarity."""
    # Convert query to TF-IDF vector
    tokens = query.lower().split()
    tf = Counter(tokens)
    query_vec = {}
    for term, count in tf.items():
        if term in idf:
            query_vec[term] = (count / len(tokens)) * idf[term]

    # Compute cosine similarity with each document
    scores = []
    for doc_idx, doc_vec in enumerate(tfidf_matrix):
        # dot product
        dot = sum(query_vec.get(t, 0) * doc_vec.get(t, 0)
                  for t in set(query_vec) | set(doc_vec))
        # norms
        norm_q = math.sqrt(sum(v**2 for v in query_vec.values()))
        norm_d = math.sqrt(sum(v**2 for v in doc_vec.values()))
        if norm_q > 0 and norm_d > 0:
            scores.append((doc_idx, dot / (norm_q * norm_d)))
        else:
            scores.append((doc_idx, 0.0))

    # Return top-K
    scores.sort(key=lambda x: x[1], reverse=True)
    return scores[:top_k]
```

### Must-Say Points
- "TF captures local importance; IDF captures global rarity."
- "The TF-IDF matrix is sparse — most terms don't appear in most documents."
- "For large-scale: use inverted index for fast query, and sparse matrix representation."

### 14b) System Design Sketch: Simple Recommender

### Problem
"Sketch a content-based recommender system for articles."

### Architecture (draw this on whiteboard)
```
┌─────────────────────────────────────────────────┐
│                  OFFLINE PIPELINE                │
│                                                  │
│  [Article DB] → [Preprocessing] → [TF-IDF]     │
│                                   → [Item Vectors] → [Vector Store]  │
│                                                  │
│  [User History] → [Aggregate User Profile]      │
│                   (avg of article vectors        │
│                    the user liked)               │
└──────────────────────┬──────────────────────────┘
                       ↓
┌──────────────────────┴──────────────────────────┐
│                  ONLINE SERVING                  │
│                                                  │
│  [User Request] → [Get User Profile Vector]     │
│                → [Cosine Similarity with all     │
│                   article vectors]               │
│                → [Filter already-read articles]  │
│                → [Top-K Recommendations]         │
│                → [Return to User]               │
└─────────────────────────────────────────────────┘
```

### Key Design Decisions to Discuss
| Decision | Options | Trade-off |
|----------|---------|-----------|
| Feature representation | TF-IDF / Word2Vec / BERT embeddings | Complexity vs quality |
| Similarity metric | Cosine / Dot product / Euclidean | Cosine is scale-invariant |
| Serving speed | Brute-force / ANN (FAISS, Annoy) | Accuracy vs latency |
| Cold start | Popular items / Ask preferences | Coverage vs personalization |
| Update frequency | Real-time / Hourly / Daily | Freshness vs cost |

### Must-Say Points
- "Content-based: recommend items similar to what the user liked before."
- "Collaborative filtering would be the alternative: recommend based on similar users."
- "In production, we'd use approximate nearest neighbors (FAISS/Annoy) for speed."
- "Cold-start problem: new users have no history → fall back to popularity-based."

---
---

# Quick Reference: Complexity Cheat Sheet

| Problem | Time | Space | Key Technique |
|---------|------|-------|---------------|
| Two Sum (Hash) | O(n) | O(n) | Hash table lookup |
| Two Sum Sorted | O(n) | O(1) | Two pointers |
| Valid Parentheses | O(n) | O(n) | Stack matching |
| Recent Calls | O(1) amortized | O(W) | Queue sliding window |
| Reverse Linked List | O(n) | O(1) | Three pointers |
| Move Zeroes | O(n) | O(1) | Fast-slow pointers |
| **Buy & Sell Stock** | **O(n)** | **O(1)** | **Track running min** |
| Tree Traversal | O(n) | O(h) | Recursion / Stack |
| LCA | O(n) | O(h) | Post-order recursion |
| BFS Shortest Path | O(V+E) | O(V) | Queue + visited |
| Topological Sort | O(V+E) | O(V+E) | In-degree + BFS |
| Number of Islands | O(R×C) | O(R×C) | BFS/DFS flood fill |
| Linear Regression | O(Nd) / epoch | O(Nd) | Gradient descent |
| Logistic Regression | O(Nd) / epoch | O(Nd) | Sigmoid + CE |
| Backpropagation | O(Nd·h) / step | O(Bh) | Chain rule |
| TF-IDF | O(N·L) | O(N·V) | Term frequency + IDF |

---

# Study Plan (Suggested Daily Routine)

> **Week 1–2: Build muscle memory**
> - Day goal: **2 problems** (1 data structure + 1 ML)
> - Timer: **15 minutes** per problem, on paper or whiteboard
> - After writing: say the complexity and one optimization out loud

> **Week 3–4: Speed + depth**
> - Day goal: **3 problems** (2 DS + 1 ML or system design)
> - Timer: **10 minutes** per problem
> - Practice follow-up questions with a friend or rubber duck

### Suggested 14-Day Schedule

| Day | Problem 1 (DS/Algo) | Problem 2 (ML/System) |
|-----|---------------------|----------------------|
| 1 | Two Sum (Hash) | Linear Regression |
| 2 | Two Sum Sorted (Pointers) | Logistic Regression |
| 3 | Valid Parentheses (Stack) | Backpropagation |
| 4 | Recent Calls (Queue) | Linear Regression (with L2) |
| 5 | Reverse Linked List | Logistic Regression |
| 6 | Move Zeroes + **Buy & Sell Stock** | Backpropagation |
| 7 | **Review Day** — redo the 2 hardest | TF-IDF Pipeline |
| 8 | Tree Traversals | Linear Regression |
| 9 | LCA + **Buy & Sell Stock (redo)** | Logistic Regression |
| 10 | BFS Shortest Path | Backpropagation (Sigmoid) |
| 11 | Topological Sort | Recommender System sketch |
| 12 | Number of Islands | Backpropagation (Softmax) |
| 13 | **Timed drill**: 3 random DS problems | **Timed drill**: 2 ML problems |
| 14 | **Mock interview**: pick 2 random, full walkthrough with talking | |

---

> **Emergency interview phrases**:
> - "Let me start with a correct baseline, then optimize if time allows."
> - "Let me verify the shape/type of each variable as I go."
> - "The brute force would be O(n²), but we can do O(n) using ___."
> - "In production, I would use ___, but for this whiteboard I'll keep it simple."
> - "Let me trace through a small example to verify."

