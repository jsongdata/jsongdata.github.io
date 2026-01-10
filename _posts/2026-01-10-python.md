---
title: "My Python Diary"
date: 2025-01-10
categories: [Python]
tags: [python]     # TAG names should always be lowercase
---

### This page is where I record new things I've learned about Python while playing with it day by day.

### Accessing Dictionary Values: `m[key]` vs. `m.get(key)`

The main difference is how they handle **missing keys**.

* **`m[key]` (Direct Access):**
    * Raises a **`KeyError`** if the key is missing.

* **`m.get(key)` (Safe Access):**
    * Returns **`None`** (or a custom default) if the key is missing.

