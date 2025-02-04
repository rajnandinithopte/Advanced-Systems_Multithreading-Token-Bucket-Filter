# 🔷 Advanced Systems: Multithreading Token Bucket Filter

## 🔶 Overview
This project implements a **multi-threaded traffic shaper** that simulates a **token bucket filter** for regulating packet transmission. The system consists of multiple **concurrent threads**, including a **packet arrival thread, token arrival thread, and server threads**, that work together to process and transmit packets in an efficient and synchronized manner. 

The implementation focuses on:
- **Multithreading using `pthread`** to manage packet arrivals, token generation, and processing.
- **Synchronization using mutexes (`pthread_mutex_t`) and condition variables (`pthread_cond_t`)** to manage shared resources.
- **A doubly-linked FIFO queue to manage packets awaiting service.**
- **Statistical tracking for packet delays, queueing times, and system efficiency.**

---

## 🔷 Libraries Used
- **stdio.h** – Standard input/output operations.
- **stdlib.h** – Memory allocation and process control.
- **string.h** – String handling and manipulation.
- **pthread.h** – Multithreading support (thread creation, mutexes, condition variables).
- **semaphore.h** – Synchronization primitives.
- **unistd.h** – Sleep, process control, and system calls.
- **errno.h** – Error handling.
- **sys/time.h** – High-precision time tracking for event timing.

---

## 🔷 System Architecture

### 🔶 Token Bucket Filter
The **token bucket filter** controls the rate at which packets are processed, preventing bursts of traffic from overwhelming the system.

- The **bucket holds up to `B` tokens** at any time.
- **Tokens arrive at a rate of `r` tokens per second**.
- If the bucket is **full**, incoming tokens are **discarded**.
- When a packet arrives, it **requests `P` tokens** from the bucket:
  - If enough tokens are available, the packet is moved to **Queue 2 (Q2)** for processing.
  - If not, the packet **waits in Queue 1 (Q1)** until enough tokens accumulate.

### 🔶 Packet Processing Flow
1. **Packet Arrival**:
   - Packets arrive at a rate of **λ packets per second**.
   - Each packet requires **P tokens** before it can proceed.
   - Packets initially wait in **Queue 1 (Q1)**.

2. **Token Bucket Check**:
   - Tokens are deposited in the bucket at a fixed rate **r**.
   - If a packet in **Q1** has enough tokens, it is moved to **Queue 2 (Q2)** for service.

3. **Server Processing**:
   - Two **server threads (`S1` and `S2`)** fetch packets from **Q2**.
   - Packets are processed **in a First-Come-First-Serve (FCFS) manner**.
   - Each packet has a **service time based on `μ` (mean service time)**.

4. **Completion and Statistics**:
   - Packet completion timestamps are recorded.
   - The system tracks **queueing times, service times, and total response times**.

---

## 🔷 Multi-Threading Implementation

### 🔶 Threads Used:
- **Packet Arrival Thread** (`pthread`):
  - Generates packets at the specified **arrival rate (λ)**.
  - Places packets in **Queue 1 (Q1)** and waits for token availability.

- **Token Bucket Thread** (`pthread`):
  - Deposits tokens into the bucket at a rate of **r tokens per second**.
  - Checks if any packet in **Q1** can proceed to **Q2**.

- **Server Threads (`S1 & S2`)**:
  - Dequeue packets from **Q2** and process them using **random service times**.
  - Maintain per-server statistics (average processing time, utilization).

- **Signal Handler Thread (`SIGINT`)**:
  - Captures **Ctrl+C** for **graceful shutdown**.
  - Ensures all resources are released before exiting.

---

## 🔷 Additional Details

### 🔶 1. More Information on Thread Synchronization and Mutex Usage
- The system uses **mutexes (`pthread_mutex_t`)** to ensure **thread-safe access** to shared resources like **Q1, Q2, and the token bucket**.
- **Condition variables (`pthread_cond_t`)** are used for **signaling between threads**, particularly for waiting on packets and tokens.
- The `pthread_cond_wait()` and `pthread_cond_broadcast()` calls ensure that **threads are woken up efficiently** when resources become available.

### 🔶 2. More Explanation on FIFO Queue (Doubly-Linked List)
- The **MyList doubly-linked list** is used as a FIFO queue for **Q1 and Q2**.
- Queue operations include:
  - `MyListAppend()` → Adds a new packet at the end of the queue.
  - `MyListUnlink()` → Removes a processed packet from the queue.
  - `MyListFirst()` and `MyListLast()` → Fetch the head and tail elements.
- This ensures **O(1) complexity** for **enqueue and dequeue operations**.

### 🔶 3. Packet and Token Arrival Handling
- Packet arrival follows a **Poisson process** with an **inter-arrival time λ** (adjusted using `usleep()`).
- Tokens arrive **periodically** at a rate **r**.
- **Overflow Conditions**:
  - If a **packet requires more tokens than available** in the bucket (`B`), it is **dropped**.
  - If the **token bucket is full** when a new token arrives, the token is **discarded**.

### 🔶 4. Trace-Driven vs. Deterministic Execution Mode
- The program can either generate packets **deterministically** (based on provided `λ, μ, r` values) or read packet events from a **trace file** (`-t tsfile` mode).
- The **trace file must be correctly formatted** with three values per line (**inter-arrival time, tokens required, and service time**).
- **Validations performed**:
  - **Line format correctness** (no missing or extra values).
  - **Correct sequence of packets** (monotonic timestamps).
  - **Leading/trailing whitespace issues**.

### 🔶 5. Signal Handling and Graceful Shutdown
- **SIGINT (Ctrl+C) handling** ensures that:
  - No new packets or tokens are generated after receiving **SIGINT**.
  - Remaining packets in **Q1 and Q2** are removed, and logs are updated.
  - Threads are **properly canceled** to prevent memory leaks.

### 🔶 6. More Details on Statistics Computation
- The system tracks **detailed performance metrics**, including:
  - **Average inter-arrival time** of packets.
  - **Average service time** and **total utilization** of each server.
  - **Time spent by packets in Q1, Q2, and the system overall**.
  - **Packet and token drop probabilities**.
  - **Variance and Standard Deviation Calculations**:
    - The program computes **system time variance** and derives the **standard deviation** to analyze **performance variability**.
    - This helps evaluate the **effectiveness of the token bucket filter** under different loads.

### 🔶 7. Error Handling and Robustness
- The program includes **extensive error handling**, such as:
  - **Invalid command-line arguments** (missing values, negative values).
  - **Incorrect trace file formats** (malformed lines, incorrect values).
  - **Memory allocation failures** (gracefully handled to prevent crashes).
  - Errors print **detailed messages** and **terminate execution safely** with **clear diagnostics**.

---

📌 **Note:**  
The code for this project **cannot be made publicly available** due to academic restrictions. However, it can be shared **privately upon request** for review or discussion.
