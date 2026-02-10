# OPPO Research Scientist Interview Whiteboard (English)

> **Purpose:** A single-page whiteboard you can open during the interview to stay structured.
>
> **Principle:** **Correct first → Explain clearly → Optimize if asked.**

---

## 0) Universal Whiteboard Script (Say This)

- **Opening:** “Let me start with a correct baseline solution, then we can optimize based on constraints.”
- **While coding:** “The key state variable here is …”
- **When stuck:** “I’ll write the simplest correct version first and refine after.”
- **Wrap-up:** “Time is O(n) and space is O(1)/(O(n)). We can further optimize/clean up if needed.”

---

## 1) 60-Min Interview Structure (Mental Map)

- **5–10 min:** Intro + why OPPO + your work (device + reliable LLM systems)
- **25–35 min:** Coding (DS template or ML loop)
- **15–20 min:** LLM systems: long context / RAG / eval / edge tradeoffs
- **5–10 min:** Your questions

---

## 2) Coding Templates (Most Likely)

### A) Hash Map (Two Sum)
```python
# Time: O(n), Space: O(n)
seen = {}  # value -> index
for i, x in enumerate(nums):
    need = target - x
    if need in seen:
        return [seen[need], i]
    seen[x] = i
```

### B) Two Pointers (Sorted Two Sum)
```python
# Time: O(n), Space: O(1)
l, r = 0, len(nums) - 1
while l < r:
    s = nums[l] + nums[r]
    if s == target: return [l, r]
    if s < target: l += 1
    else: r -= 1
```

### C) Stack (Valid Parentheses)
```python
# Time: O(n), Space: O(n)
stack = []
match = {')':'(', ']':'[', '}':'{'}
for ch in s:
    if ch in match:
        if not stack or stack[-1] != match[ch]:
            return False
        stack.pop()
    else:
        stack.append(ch)
return len(stack) == 0
```

### D) Queue / Sliding Window (Recent Calls)
```python
from collections import deque

class RecentCounter:
    def __init__(self):
        self.q = deque()

    def ping(self, t: int) -> int:
        self.q.append(t)
        while self.q[0] < t - 3000:
            self.q.popleft()
        return len(self.q)
```

### E) Linked List (Reverse List, Iterative)
```python
# Time: O(n), Space: O(1)
prev = None
curr = head
while curr:
    nxt = curr.next
    curr.next = prev
    prev = curr
    curr = nxt
return prev
```

### F) In-place Two Pointers (Move Zeroes)
```python
# Time: O(n), Space: O(1)
slow = 0
for fast in range(len(nums)):
    if nums[fast] != 0:
        nums[slow], nums[fast] = nums[fast], nums[slow]
        slow += 1
```

---

## 3) ML Coding Templates (OPPO-Style)

### A) Mini-batch Linear Regression (with bias)

**Model:** \(\hat{y} = Xw + b\)

```python
# X: [N, d], y: [N]
# w: [d], b: scalar
w = zeros(d)
b = 0.0

for epoch in range(num_epochs):
    for i in range(0, N, batch_size):
        Xb = X[i:i+batch_size]
        yb = y[i:i+batch_size]

        pred = Xb @ w + b
        err = pred - yb

        grad_w = (Xb.T @ err) / len(yb)
        grad_b = sum(err) / len(yb)

        w -= lr * grad_w
        b -= lr * grad_b
```

**One-liner to say:** “\(b\)’s gradient is just the **mean error** over the mini-batch.”

---

### B) 1-Hidden-Layer Neural Net Backprop (vectorized skeleton)

**Forward:**
- \(z_1 = XW_1 + b_1\)
- \(h = \mathrm{ReLU}(z_1)\)
- \(\hat{y} = hW_2 + b_2\)
- \(L=\mathrm{MSE}(\hat{y}, y)\)

```python
# Forward
z1 = X @ W1 + b1
h  = relu(z1)
yhat = h @ W2 + b2

# Loss gradient (MSE)
err = (yhat - y)           # [B, out]
dyhat = 2 * err / B

# Layer 2
# yhat = hW2 + b2

dW2 = h.T @ dyhat
# db2: sum over batch

db2 = sum(dyhat, axis=0)

# Backprop to hidden
# dh = dyhat W2^T

dh = dyhat @ W2.T

# ReLU backprop
# relu'(z1) = 1(z1>0)

dz1 = dh * (z1 > 0)

# Layer 1
# z1 = XW1 + b1

dW1 = X.T @ dz1
# db1: sum over batch

db1 = sum(dz1, axis=0)

# Update
W1 -= lr * dW1
b1 -= lr * db1
W2 -= lr * dW2
b2 -= lr * db2
```

**One-liners to say:**
- “I’ll write forward first, then apply chain rule backward.”
- “I’ll sanity-check shapes at each step.”

---

## 4) LLM Systems Talking Points (Device + 1B)

### A) 1B LLM Tradeoffs (edge)
- Bottlenecks: **KV cache**, memory bandwidth, TTFT, token/s, power/thermal
- Knobs: quantization (INT8/INT4), MQA/GQA, cache eviction/compression, speculative decoding

### B) Long Context / Memory
- Why it breaks: position extrapolation, attention cost, retrieval noise
- Solutions: sliding/block attention, RoPE scaling, chunking + summary, external memory (RAG)

### C) RAG / In-context Retrieval
- Components: embed → retrieve → rerank → pack context
- Failure modes: wrong retrieval, prompt injection, topic drift
- Edge angle: latency + on-device index size + caching

### D) Benchmarking / LLM-as-Judge
- Risks: judge bias, scale drift, poor reproducibility
- Mitigation: rubric, pairwise eval, gold sets, multi-judge agreement, calibration

---

## 5) Quick “Why OPPO” Answer (30 seconds)

“I’m excited about OPPO because you’re building **real device-constrained GenAI** (AI glasses/earphones), where success depends on **system-level tradeoffs**—latency, memory, power, reliability—not just larger models. My work focuses on **reliable LLM systems**, including routing/evaluation and deployment-oriented optimization, which aligns well with building ~1B multimodal models with long-context and retrieval on edge hardware.”

---

## 6) Questions to Ask (Pick 2–3)

- “What are the primary device constraints you’re optimizing for—TTFT, token/s, memory, or power?”
- “How do you evaluate long-context + retrieval quality for your target scenarios?”
- “What does success look like in the first 3 months for this role?”
- “How is the work split between architecture research vs. training data/evaluation vs. deployment integration?”

---

## 7) Final Checklist (2 minutes before the call)

- Confirm time zone: **2:30–3:30 PM PT**
- Open this whiteboard
- Water + quiet room
- Start with: **baseline solution**
- Keep talking while writing

