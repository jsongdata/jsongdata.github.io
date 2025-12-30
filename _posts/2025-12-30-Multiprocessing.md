---
title: "How to Choose the Right LLM"
date: 2025-12-30
categories: [LLMOps]
tags: [llmops, python, multiprocessing]     # TAG names should always be lowercase
---


### 1. Overview of Multiprocessing

### Key Concepts
* **Definition**: A technique that replicates processes to run independently within separate memory spaces.
* **Structure**: Each process independently maintains its own **Code, Heap, Data, and Stack** sections.
* **Advantages**: Optimizes **CPU-bound operations** and ensures stability through process isolation.
* **Disadvantages**: Overhead from **context switching** and challenges in Inter-Process Communication (IPC).

---

## 2. Synchronization using Event

### Operating Principle
* **`event.wait()`**: Blocks process execution until the event flag is `set()` to true.
* **`event.set()`**: Signals all waiting processes to resume execution simultaneously.

### Code Implementation
```python
import multiprocessing
from time import sleep

def task(event, name):
    print(f'Current Process {name} waiting...')
    event.wait() # Wait for signal
    print(f'Current Process {name} started!')

if __name__ == "__main__":
    event = multiprocessing.Event()
    
    p1 = multiprocessing.Process(target=task, args=(event, 'pro1'))
    p2 = multiprocessing.Process(target=task, args=(event, 'pro2'))
    
    p1.start()
    p2.start()
    
    sleep(3) # 3-second delay
    event.set() # Wake up all processes
