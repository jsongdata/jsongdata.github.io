---
title: "Python Multiprocessing: Functional vs Class-based"
categories: [LLMOps, Python]
tags: [Multiprocessing, Programming, OOP]
---

## 1. Functional Process Creation (`process_func.py`)
This approach is the most straightforward and is ideal for **one-off, independent tasks** that don't require complex state management.

```python
from multiprocessing import Process # Import the Process class for creating sub-processes

# 1. Define the target function to be executed in parallel
def hello(arg1, arg2):
    print('Hello Process') # Output the task result

# 2. Instantiate the Process object
# target: Specifies the function name to run
# args: Positional arguments for the target (must be a tuple)
# kwargs: Keyword arguments for the target (dictionary format)
p1 = Process(target=hello, args=('123',), kwargs=dict(arg2='456'))

p1.start()     # 3. Start the child process (asynchronous execution)
p1.join()      # 4. Block the main process until the child process terminates (synchronization)
p1.terminate() # 5. Forcefully stop the process (used for cleanup or emergency stops)
```
Important Note on "Zombie Processes" Always call join() or terminate(). If a child process finishes but the parent doesn't acknowledge it, it remains in the system as a Zombie Process, wasting resources.


```python
from multiprocessing import Process # Import base Process class to inherit its functionality

# 1. Define a custom class by inheriting from 'Process'
class HelloProcess(Process):
    # The __init__ method: Used to set up initial state/data before execution
    def __init__(self):
        # Mandatory: Call the parent class (Process) constructor to initialize internal mechanisms
        super(HelloProcess, self).__init__() 

    # 2. Overriding the run() method: This is where the actual parallel logic goes
    # Internal trigger: When you call .start(), the OS executes this .run() method
    def run(self):
        print("Hello Process") # Business logic of the sub-process

# 3. Instantiate the process object
p1 = HelloProcess()

p1.start()     # 4. Trigger the OS to create the process and execute .run()
p1.join()      # 5. Wait for the process to complete before the main process continues
p1.terminate() # 6. Safety check: Forcefully kill the process if it's still hanging
```

Technical Insight: The Class-based Way
State Persistence: Classes allow you to store persistent objects like Database connections or Model configurations within self, making them accessible across different internal methods.

Encapsulation: Grouping data and logic into a single object prevents "Global Variable" issues and improves code maintainability.

Standardization: Most enterprise-grade frameworks (like Ray or vLLM mentioned in your roadmap) utilize class inheritance to ensure all worker processes follow a specific protocol.

---

## Overview

In Python's `multiprocessing` module, there are three primary ways to start a new process.

---

## 1. Process Start Methods

### **Spawn**
- **Definition**: Starts a fresh Python interpreter process from scratch.
- **Characteristics**: 
  - Only the necessary resources (code) are sent to the new process.
  - Slower than `fork` but highly stable and clean.
- **Platform**: Default for **Windows** and **macOS**.

### **Fork**
- **Definition**: Copies the existing parent process (including memory and data).
- **Characteristics**: 
  - Extremely fast as it duplicates the current state.
  - Can lead to issues with shared resources (like file handles or DB connections) and high memory overhead during the copy.
- **Platform**: Default for **Linux/Unix**.

### **Forkserver**
- **Definition**: A server process is started, and all new processes are requested from this server.
- **Characteristics**: 
  - Avoids inheriting unnecessary resources compared to `fork`.
  - Used in large-scale server environments for safety.
- **Platform**: Available on **Linux/Unix**.

---

## 2. OS Support & Compatibility

| OS | Supported Methods | Default | Note |
| :--- | :--- | :--- | :--- |
| **Windows** | `spawn` | `spawn` | Does not support `fork` |
| **Linux** | `spawn`, `fork`, `forkserver` | `fork` | `spawn` is becoming the safer trend |
| **macOS** | `spawn`, `fork` | `spawn` | Changed from `fork` to `spawn` recently |

---

## 3. Implementation Example

You can check or change the start method using the following code. Note that `set_start_method()` should be called only once and protected by `if __name__ == '__main__':`.

```python
import multiprocessing

def worker_task():
    print("Child process is running.")

if __name__ == '__main__':
    # Check current method
    print(f"Initial Method: {multiprocessing.get_start_method()}")

    # Change to 'spawn' for stability (especially in LLM workloads)
    try:
        multiprocessing.set_start_method('spawn', force=True)
    except RuntimeError:
        pass

    print(f"Changed Method: {multiprocessing.get_start_method()}")

    # Initialize and start process
    p1 = multiprocessing.Process(target=worker_task)
    p1.start()
    p1.join()
```

---

## Overview

In multiprocessing, each process has its own memory space. To share data between them, we need a communication channel. Python provides the `multiprocessing.Queue`, which is a thread and process-safe way to exchange objects.

---

## 1. Concept of Queue

- **Purpose**: Safely exchange data/objects between multiple processes.
- **Mechanism**: Based on a pipe and shared memory with locks (semaphores) to ensure data integrity.
- **Main Actions**:
    - `put()`: Inserts data into the queue.
    - `get()`: Removes and returns an item from the queue (blocks if the queue is empty).



---

## 2. Key Workflow

1.  **Initialize**: Create a `multiprocessing.Queue()` in the parent process.
2.  **Pass**: Send the queue object as an argument (`args`) to the child processes.
3.  **Produce/Consume**: One process puts data in, and another takes it out.

---

## 3. Implementation Example

This example demonstrates a **Producer-Consumer** pattern where one process generates data and another processes it.

```python
import multiprocessing
import time

def producer(q):
    """Generates data and puts it into the queue."""
    for i in range(10):
        item = f"Task-{i}"
        q.put(item)
        print(f"[Producer] Pushed: {item}")
        time.sleep(0.5)  # Simulate work

def consumer(q):
    """Retrieves and processes data from the queue."""
    while True:
        # get() blocks until an item is available
        item = q.get()
        if item is None: break  # Sentinel value to stop
        print(f"[Consumer] Processed: {item}")

if __name__ == '__main__':
    # 1. Create a shared queue
    queue = multiprocessing.Queue()

    # 2. Define processes
    p1 = multiprocessing.Process(target=producer, args=(queue,))
    p2 = multiprocessing.Process(target=consumer, args=(queue,))

    # 3. Start processes
    p1.start()
    p2.start()

    p1.join()
    # Add logic to stop consumer if needed (e.g., queue.put(None))
```

---

## Overview

A `SimpleQueue` is a simplified version of a queue designed for basic data exchange between processes without complex synchronization features like timeouts.

---

## 1. Core Logic

- **Producer (`pro1`)**: Iterates through a range and pushes data into the queue using `q.put()`.
- **Consumer (`pro2`)**: Runs an infinite loop and retrieves data using `q.get()`, which blocks until an item is available.
- **Initialization**: Created via `multiprocessing.SimpleQueue()`.


---

## Overview
`SimpleQueue` is a stripped-down version of `Queue`, optimized for basic data passing with minimal overhead.

## 1. Comparison

| Feature | Standard `Queue` | `SimpleQueue` |
| :--- | :--- | :--- |
| **Complexity** | High (supports timeouts, maxsize) | Low (only put/get) |
| **Size Limit** | Can be limited | Infinite |
| **Performance** | Feature-rich but heavier | Lightweight and fast |


## 2. Implementation Code

```python
import multiprocessing
from time import sleep

def pro1(q):
    """Producer: Pushes integers 0-9 into the queue."""
    for i in range(10):
        q.put(i)
        sleep(0.5)

def pro2(q):
    """Consumer: Continuously pulls and prints data from the queue."""
    while True:
        item = q.get()
        print(item)

if __name__ == '__main__':
    # 1. Initialize the SimpleQueue
    queue = multiprocessing.SimpleQueue()

    # 2. Define processes and pass the queue as an argument
    p1 = multiprocessing.Process(target=pro1, args=(queue,))
    p2 = multiprocessing.Process(target=pro2, args=(queue,))

    # 3. Start execution
    p1.start()
    p2.start()

    p1.join()
    p2.terminate() # Explicitly stop the infinite loop consumer
```

---

## Overview

A `JoinableQueue` is a specialized version of `Queue` that allows the parent process to track the completion of tasks. It ensures that all data sent to the queue has been fully processed before the program proceeds.

---

## 1. Key Concepts & Methods

Unlike a standard `Queue`, `JoinableQueue` uses an internal counter to track unfinished tasks. This counter increases when you `put()` and decreases when you call `task_done()`.

### **`task_done()`**
- **Purpose**: Signals that a formerly enqueued task is complete.
- **Usage**: Must be called by the consumer process after it finishes processing an item retrieved via `get()`.
- **Risk**: If omitted, the internal counter never reaches zero, causing the program to hang.

### **`join()`**
- **Purpose**: Blocks the calling process (usually the main process) until all items in the queue have been processed.
- **Condition**: It unblocks only when the number of `task_done()` calls equals the number of `put()` calls.



---

## 2. Implementation Code

This code combines the logic from your screenshots with the necessary synchronization methods explained in the lecture.

```python
import multiprocessing
from time import sleep

def pro1(q):
    """Producer: Pushes 10 items into the queue."""
    for item in range(10):
        q.put(item)
        print(f"Produced: {item}")
        sleep(0.5)

def pro2(q):
    """Consumer: Processes items and signals completion."""
    while True:
        item = q.get()
        # Process the data
        print(f"Consumed: {item}")
        # Notify the queue that the specific task is finished
        q.task_done()

if __name__ == '__main__':
    # Initialize JoinableQueue
    queue = multiprocessing.JoinableQueue()

    p1 = multiprocessing.Process(target=pro1, args=(queue,))
    p2 = multiprocessing.Process(target=pro2, args=(queue,))
    
    # Set consumer as a daemon so it exits when the main process finishes
    p2.daemon = True 

    p1.start()
    p2.start()

    # Wait for producer to finish putting all items
    p1.join()      
    
    # Block main process until all queue items are processed via task_done()
    queue.join()
    
    print("All tasks are completed successfully.")
```

---

## Overview

A `Pipe` is the most fundamental 1-to-1 communication channel between two processes. When calling `multiprocessing.Pipe()`, it returns a pair of connection objects representing the ends of the pipe.

---

## 1. Key Concepts & Features

- **Connection Objects**: Data sent at one end of the pipe is received at the other.
- **Serialization (Pickling)**: Objects sent through the pipe must be picklable; non-picklable objects will cause errors.
- **Blocking Mechanism**: The `recv()` method blocks the process until an item is available. If the sender never sends data, the receiver will hang indefinitely.
- **Capacity Limits**: Sending very large objects (typically over 32MB) might cause issues depending on the OS.

---

## 2. Communication Modes

| Mode | Configuration | Description |
| :--- | :--- | :--- |
| **Full Duplex** | `Pipe(True)` (Default) | Allows simultaneous two-way communication where both ends can send and receive. |
| **Half Duplex** | `Pipe(False)` | Creates a unidirectional pipe; one end can only receive, and the other can only send. |

---

## 3. Implementation Example

Based on the lecture logic and your provided code structure, here is an example of Pipe communication.

```python
import multiprocessing

def sender_task(conn):
    """Sender Process: Sends data through the connection."""
    message = "1234"
    conn.send(message) 
    conn.close()

def receiver_task(conn):
    """Receiver Process: Waits for and prints data."""
    try:
        data = conn.recv() # Blocks until data arrives
        print(f"Received from Pipe: {data}")
    except EOFError:
        print("Connection closed.")
    finally:
        conn.close()

if __name__ == '__main__':
    # Initialize Pipe (Set False for Half Duplex as per your screenshot)
    ps, pr = multiprocessing.Pipe(False)

    p1 = multiprocessing.Process(target=sender_task, args=(ps,))
    p2 = multiprocessing.Process(target=receiver_task, args=(pr,))

    p1.start()
    p2.start()
    
    p1.join()
    p2.join()
```
---

## Overview

Unlike threads, processes in Python have independent memory spaces. To share data between them, we must use shared memory objects provided by the `multiprocessing` module: **`Value`** and **`Array`**.

---

## 1. Why Shared Memory?

- **Isolation**: Child processes cannot access variables in the parent process's memory directly.
- **Solution**: Shared memory objects reside in a special memory area accessible by all participating processes.

---

## 2. The `Value` Class

The `Value` class is used to share a single piece of data (like an integer or a float).

### **Key Features**
- **Type Codes**: You must specify the data type using codes from the `array` module (e.g., `'i'` for signed int, `'d' 'for double).
- **`.value` Access**: Always use the `.value` property to read or write the actual data.

### **Common Type Codes**
| Code | C Type | Python Type |
| :--- | :--- | :--- |
| `'i'` | signed int | int |
| `'d'` | double | float |
| `'b'` | signed char | int |

---

## 3. Implementation Code

This example shows how `Process 2` can modify a value that `Process 1` is reading.

```python
import multiprocessing
from time import sleep

def reader(v):
    """Prints the shared value, waits, and prints it again to see changes."""
    print(f"Initial Value: {v.value}")
    sleep(3)
    print(f"Updated Value: {v.value}")

def writer(v):
    """Modifies the shared value."""
    sleep(1) # Ensure the reader starts first
    v.value = 10
    print("Value modified to 10")

if __name__ == '__main__':
    # Initialize shared memory with type 'i' and initial value 0
    shared_val = multiprocessing.Value('i', 0)

    p1 = multiprocessing.Process(target=reader, args=(shared_val,))
    p2 = multiprocessing.Process(target=writer, args=(shared_val,))

    p1.start()
    p2.start()

    p1.join()
    p2.join()
```

---


## Overview

While `Value` and `Array` are great for simple data types, the `multiprocessing.Manager` provides a way to share higher-level Python objects like **lists** and **dictionaries** across multiple processes.

---

## 1. How the Manager Works

- **Server Process**: The `Manager()` starts a separate server process that manages the shared objects.
- **Proxies**: When you create a list or dict through a manager, other processes access them via proxies.
- **Flexibility**: It is slower than shared memory but supports almost all standard Python operations on collections.

---

## 2. Key Methods

- `manager.list()`: Creates a shared list.
- `manager.dict()`: Creates a shared dictionary.
- Supports shared synchronization primitives like `Lock`, `Semaphore`, and `Queue`.

---

## 3. Implementation Example

This code demonstrates how two independent processes can concurrently update a single shared list.

```python
import multiprocessing

def task_one(shared_list):
    """Adds 'p1' to the shared collection."""
    shared_list.append('p1')

def task_two(shared_list):
    """Adds 'p2' to the shared collection."""
    shared_list.append('p2')

if __name__ == '__main__':
    # Initialize the Manager
    with multiprocessing.Manager() as manager:
        # Create a shared list
        m_list = manager.list()

        # Define and start processes
        p1 = multiprocessing.Process(target=task_one, args=(m_list,))
        p2 = multiprocessing.Process(target=task_two, args=(m_list,))

        p1.start()
        p2.start()
        p1.join()
        p2.join()

        # Final output will contain items from both processes
        print(f"Final List: {list(m_list)}")
```

---

## Lock

When multiple processes access a shared object concurrently, a race condition can occur. Python provides the `Lock` class to ensure that only one process can access the shared resource at any given time.

---

## 1. Key Concepts of Lock

- **Mutual Exclusion**: Prevents multiple processes from modifying the same data simultaneously.
- **`acquire()`**: Grabs the lock. Other processes must wait until the lock is released.
- **`release()`**: Releases the lock so that the next waiting process can proceed.



---

## 2. Implementation from Source

The following code is implemented exactly as shown in the provided screenshots (`multiprocessing_lock.py`).

```python
import multiprocessing
import random
from time import sleep

def pro1(arg, lock):
    """Process 1: Safely increments the shared value up to 10."""
    lock.acquire()
    while arg.value < 10:
        print('Current Increment Value', arg.value)
        arg.value += 1
        sleep(random.randint(0, 1))
    lock.release()

def pro2(arg, lock):
    """Process 2: Safely decrements the shared value down to -10."""
    lock.acquire()
    while arg.value > -10:
        print('Current Decrement Value', arg.value)
        arg.value -= 1
        sleep(random.randint(0, 1))
    lock.release()

if __name__ == '__main__':
    # Initialize shared memory and the lock object
    shared_obj = multiprocessing.Value('i', 0)
    lock = multiprocessing.Lock()

    # Create processes and pass the shared object and lock
    p1 = multiprocessing.Process(target=pro1, args=(shared_obj, lock))
    p2 = multiprocessing.Process(target=pro2, args=(shared_obj, lock))

    # Start the processes
    p1.start()
    p2.start()
```




