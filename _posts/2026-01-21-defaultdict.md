---
title: "How to use defaultdict(list)"
date: 2025-01-21
categories: [Python]
tags: [python]     # TAG names should always be lowercase
---

## 1. What is `defaultdict(list)`?

A `defaultdict` is a specialized container found in Python's built-in `collections` module. It works exactly like a standard dictionary, but with one major advantage: **it never raises a `KeyError`.**

{: .prompt-info }
> When you initialize it as `defaultdict(list)`, the `list` constructor acts as a **default_factory**. If a requested key is not found, it automatically creates a new, empty list `[]` for that key.

---

## 2. Why Use It? (Efficiency & Readability)

`defaultdict` allows you to bypass manual key-existence checks, leading to cleaner and faster implementation.

### Standard `dict` Approach (Verbose)
```python
groups = {}
for item in data:
    if item.key not in groups:
        groups[item.key] = []  # Manual initialization
    groups[item.key].append(item.value)

```

### `defaultdict` Approach (Optimized)

```python
from collections import defaultdict

groups = defaultdict(list)
for item in data:
    groups[item.key].append(item.value) # Automatically initializes if key is missing

```

---

## 3. Real-World Example: Group Anagrams

This is a classic LeetCode Medium problem (LeetCode 49) often seen in **IBM Research** coding assessments.

```python
from collections import defaultdict

def group_anagrams(strs):
    # Initialize defaultdict with list factory
    anagram_map = defaultdict(list)
    
    for s in strs:
        # Create a unique key by sorting the string
        key = "".join(sorted(s))
        
        # No need to check if key exists; just append!
        anagram_map[key].append(s)
        
    return list(anagram_map.values())

# Input: ["eat","tea","tan","ate","nat","bat"]
# Output: [['eat', 'tea', 'ate'], ['tan', 'nat'], ['bat']]

```