# Experiment 04: Analysis of Synchronization Mechanisms in Parallel Programs

## 1. Executive Summary

This experiment investigates synchronization mechanisms used in shared-memory parallel programs using C, OpenMP, and POSIX threads on Ubuntu Linux. I implemented and compared race conditions, critical sections, atomic operations, barriers, locks, and semaphores to understand their impact on correctness and performance. A benchmark was conducted using 1, 2, 4, and 8 threads to measure execution time, lost updates, and scalability. The results demonstrate the trade-offs between synchronization overhead, data consistency, and parallel performance.

## 2. Table of Contents

- [Objectives](#3-objectives)
- [Theoretical and Architectural Comparison](#4-theoretical-and-architectural-comparison)
- [Technologies Used](#5-technologies-used)
- [Execution Steps](#6-execution-steps)
- [Experimental Results](#7-experimental-results)
- [Performance Comparison](#8-performance-comparison)
- [Conclusion](#9-conclusion)
- [Repository Structure](#10-repository-structure)

## 3. Objectives

- Understand why synchronization is required in shared-memory parallel programming.
- Demonstrate race conditions caused by concurrent access to shared variables.
- Implement critical sections, atomic operations, barriers, locks, and semaphores.
- Verify correctness by comparing the final counter with the expected value.
- Measure execution time for different thread counts.
- Analyze synchronization overhead, scalability, and the trade-off between correctness and performance.

## 4. Theoretical and Architectural Comparison

### 4.1 Why Synchronization Is Required

When multiple threads access and modify the same shared variable concurrently, their operations can interleave unpredictably.

For example:

```c
counter++;
```

This operation involves reading the current value, incrementing it, and writing the result back. If two threads read the same value before either writes its update, one increment can be lost.

Synchronization mechanisms coordinate access to shared resources or coordinate execution between threads.

### 4.2 Synchronization Mechanisms

| Mechanism | Working principle | Correctness | Main limitation |
|---|---|---|---|
| No synchronization | Threads update shared data without coordination | Not guaranteed | Race conditions and lost updates |
| Critical section | Allows only one thread at a time into the protected block | Guaranteed for protected updates | Serializes access and increases waiting |
| Atomic operation | Performs a supported shared-variable operation indivisibly | Guaranteed for the protected atomic operation | Limited to supported operations; contention remains possible |
| OpenMP lock | A thread acquires a lock before accessing shared data and releases it afterward | Guaranteed when used correctly | Lock acquisition and release overhead |
| Barrier | All participating threads wait until every thread reaches the synchronization point | Synchronizes execution phases | Faster threads wait for slower threads |
| Semaphore | Uses a counter to control access to a resource | Depends on correct semaphore usage | Waiting and synchronization overhead |

### 4.3 Architectural Comparison

The programs demonstrate two different forms of synchronization.

**OpenMP-based synchronization**

- Uses compiler directives such as `#pragma omp critical`, `#pragma omp atomic`, and `#pragma omp barrier`.
- Uses OpenMP locks through `omp_lock_t`.
- Supports parallel loops and configurable thread counts.
- Was used to benchmark shared-counter updates.

**POSIX threads and semaphore synchronization**

- Uses `pthread_create()` to create threads.
- Uses `pthread_join()` to wait for thread completion.
- Uses `sem_wait()` and `sem_post()` to control access to the shared counter.
- Demonstrates synchronization through the POSIX thread and semaphore libraries.

### 4.4 Execution Flow

```text
                 Start
                   |
          Initialize shared data
                   |
             Create threads
                   |
          Execute parallel work
                   |
       +-----------+-----------+
       |                       |
   No protection         Synchronization
       |                       |
   Race condition       +------+------+
   may occur            |             |
                     OpenMP        POSIX
                     mechanisms    semaphore
       |                       |
       +-----------+-----------+
                   |
          Wait for completion
                   |
        Check counter and timing
                   |
                  End
```

### 4.5 Synchronization Architecture Diagram

The following diagrams illustrate the logical difference between unprotected and synchronized shared-data access.

**Without synchronization**

```text
Thread 1 --------+
                 |
                 +----> Shared Counter
                 |
Thread 2 --------+

Concurrent updates may overwrite one another.
```

**With synchronization**

```text
Thread 1 ----+
             |
Thread 2 ----+----> Synchronization ----> Shared Counter
             |          Mechanism
Thread 3 ----+

Access is coordinated according to the mechanism.
```

## 5. Technologies Used

| Technology | Purpose |
|---|---|
| C | Implementation of synchronization programs |
| GCC | Compilation of C source files |
| OpenMP | Parallel execution, atomic operations, critical sections, barriers, and locks |
| POSIX Threads (pthreads) | Thread creation and management |
| POSIX Semaphores | Controlled access to shared resources |
| Ubuntu 22.04 LTS | Execution environment |
| VMware Workstation Pro | Virtual machine environment |
| Git and GitHub | Version control and experiment documentation |

## 6. Execution Steps

### 6.1 Prerequisites

The experiment was designed for Ubuntu Linux with GCC, OpenMP, and POSIX threads.

Open a terminal and install the required compiler tools:

```bash
# Update the Ubuntu package index.
sudo apt update

# Install GCC, Make, and the OpenMP runtime.
sudo apt install build-essential -y

# Verify that GCC is installed.
gcc --version
```

Check OpenMP support:

```bash
# Preprocess a small program that includes the OpenMP header.
echo '#include <omp.h>' | gcc -E -fopenmp -xc - > /dev/null \
    && echo "OpenMP support available"
```

### 6.2 Navigate to the Project Directory

```bash
# Navigate to the directory containing the C programs.
cd ~/Exp5_Synchronization

# List the available source files and result files.
ls -la
```


### 6.3 Compile the Individual Programs

```bash
# Compile the unsynchronized shared-counter program.
gcc race.c -fopenmp -o race

# Compile the critical-section program.
gcc critical.c -fopenmp -o critical

# Compile the atomic-operation program.
gcc atomic.c -fopenmp -o atomic

# Compile the barrier synchronization program.
gcc barrier.c -fopenmp -o barrier

# Compile the POSIX semaphore program.
gcc semaphore.c -pthread -o semaphore

# Compile the synchronization comparison benchmark.
gcc compare_sync.c -fopenmp -o compare_sync
```

**Why are the compilation flags different?**

- `-fopenmp` enables OpenMP directives and links the OpenMP runtime.
- `-pthread` enables the POSIX threads compilation and linking support.
- `-o` specifies the name of the executable.

### 6.4 Run the Individual Programs

```bash
# Run the race-condition program.
./race

# Run the critical-section program.
./critical

# Run the atomic-operation program.
./atomic

# Run the barrier program.
./barrier

# Run the semaphore program.
./semaphore
```

The race-condition program may produce an incorrect counter because multiple threads update the same variable without protection.

The critical-section and atomic programs should produce the expected counter value of `10000000` when executed successfully. The semaphore program should produce `200000`, since two threads each perform `100000` protected increments.

The barrier program prints messages before and after the synchronization point. Message order can vary, but all participating threads must reach the barrier before any of them proceeds beyond it.

### 6.5 Run the Comparison Benchmark

```bash
# Run the benchmark for 1, 2, 4, and 8 threads.
./compare_sync
```

The benchmark executes the same workload of `10000000` increments using four strategies: no synchronization, a critical section, an atomic operation, and an OpenMP lock.

For each strategy and thread count, it reports:

- Final counter value
- Correctness status
- Execution time in seconds

The benchmark uses `omp_set_num_threads()` to configure thread counts and `omp_get_wtime()` to measure elapsed time.

### 6.6 Save the Benchmark Output

```bash
# Execute the benchmark and save its output to results.txt.
./compare_sync | tee results.txt

# Display the saved output.
cat results.txt
```

The `tee` command displays the benchmark output in the terminal while saving it to a file for later comparison.

### 6.7 Run an OpenMP Program with a Specific Thread Count

```bash
# Run the atomic program with four OpenMP threads.
OMP_NUM_THREADS=4 ./atomic

# Run the critical-section program with eight OpenMP threads.
OMP_NUM_THREADS=8 ./critical
```

The standalone programs use OpenMP parallel loops. The comparison benchmark explicitly configures its thread counts internally, so the benchmark should be run using `./compare_sync`.

## 7. Experimental Results

### 7.1 Individual Program Results

The following results were observed.

| Program | Expected result | Observed result | Interpretation |
|---|---:|---:|---|
| Race condition | 100,000,000 | 36,207,46* | Incorrect; lost updates |
| Critical section | 10,000,000 | 10,000,000 | Correct |
| Atomic operation | 10,000,000 | 10,000,000 | Correct |
| Barrier | All threads synchronize | Before/after messages displayed | Synchronization point demonstrated |
| Semaphore | 200,000 | 200,000 | Correct |

*The race-condition screenshot displays `Counter=3620746`. The expected count for the loop is `100000000`, not `10000000`; therefore, the result is incorrect.*

### 7.2 Benchmark Results

The workload is `10000000` shared increments.
<img width="786" height="568" alt="result" src="https://github.com/user-attachments/assets/ae65eba3-2b62-4047-a63f-dff602c1978b" />

| Mechanism | Threads | Final Count | Correctness | Time (s) |
|---|---:|---:|---|---:|
| None | 1 | 10,000,000 | Correct | 0.0064 |
| Critical | 1 | 10,000,000 | Correct | 0.1153 |
| Atomic | 1 | 10,000,000 | Correct | 0.0672 |
| Lock | 1 | 10,000,000 | Correct | 0.1171 |
| None | 2 | 7,650,266 | Incorrect | 0.0115 |
| Critical | 2 | 10,000,000 | Correct | 0.3513 |
| Atomic | 2 | 10,000,000 | Correct | 0.1837 |
| Lock | 2 | 10,000,000 | Correct | 0.3681 |
| None | 4 | 3,018,766 | Incorrect | 0.0136 |
| Critical | 4 | 10,000,000 | Correct | 0.6616 |
| Atomic | 4 | 10,000,000 | Correct | 0.1980 |
| Lock | 4 | 10,000,000 | Correct | 1.1201 |
| None | 8 | 1,995,492 | Incorrect | 0.0140 |
| Critical | 8 | 10,000,000 | Correct | 0.7128 |
| Atomic | 8 | 10,000,000 | Correct | 0.2119 |
| Lock | 8 | 10,000,000 | Correct | 1.1376 |

### 7.3 Individual Observations

**Race condition**

- The single-thread run produced the correct result because there was no concurrent update race.
- With multiple threads, the final count became incorrect.
<img width="1600" height="633" alt="race" src="https://github.com/user-attachments/assets/b59405c4-99be-4b50-823a-f4b4e4d817cb" />



**Critical section**

- Produced the correct final count for all tested thread counts.
- Execution time increased substantially with additional threads because access to the shared counter was serialized.
<img width="786" height="287" alt="critical" src="https://github.com/user-attachments/assets/97b148ec-d316-473a-8f83-72d300f00a64" />




**Atomic operation**

- Produced the correct result for all tested thread counts.
- Was faster than both the critical section and the OpenMP lock in the recorded multithreaded benchmark.

<img width="749" height="301" alt="atomic" src="https://github.com/user-attachments/assets/34419550-ee9d-4909-937c-87f33767aa86" />


**Barrier**

- Demonstrated synchronization between execution phases.
- The output order varied because thread scheduling is nondeterministic.
<img width="786" height="567" alt="barrier" src="https://github.com/user-attachments/assets/6edd6918-13a2-40d5-b648-bfbfe7047859" />



**Semaphore**

- Produced the correct result of `200000`.
- Demonstrated mutual exclusion using a binary semaphore initialized to one.
<img width="786" height="512" alt="semaphore" src="https://github.com/user-attachments/assets/767823ad-f7dd-4ba3-823e-88299429e89c" />



## 8. Performance Comparison

### 8.1 Execution Time by Thread Count

| Threads | None (s) | Critical (s) | Atomic (s) | Lock (s) |
|---:|---:|---:|---:|---:|
| 1 | 0.0064 | 0.1153 | 0.0672 | 0.1171 |
| 2 | 0.0115 | 0.3513 | 0.1837 | 0.3681 |
| 4 | 0.0136 | 0.6616 | 0.1980 | 1.1201 |
| 8 | 0.0140 | 0.7128 | 0.2119 | 1.1376 |
<img width="474" height="291" alt="exec_vs_no-threads" src="https://github.com/user-attachments/assets/f2d92975-4c12-4598-8304-b5a9dee18e8b" />

The unsynchronized program is the fastest in the recorded runs, but its multithreaded results are incorrect. Therefore, its low execution time does not represent a valid performance improvement.

The atomic operation provides the best balance of correctness and execution time among the four strategies tested.

### 8.2 Performance Analysis

**Atomic operation vs. critical section**

At eight threads:

- Critical-section time: `0.7128 s`
- Atomic-operation time: `0.2119 s`

The atomic implementation was approximately **3.36 times faster** than the critical-section implementation for the recorded run.

**Atomic operation vs. OpenMP lock**

At eight threads:

- Lock time: `1.1376 s`
- Atomic-operation time: `0.2119 s`

The atomic implementation was approximately **5.37 times faster** than the lock implementation.

These ratios describe the measured execution times for this particular workload and environment; they are not universal performance guarantees.

### 8.3 Scalability Analysis

The benchmark shows that adding threads does not necessarily improve performance.

- **No synchronization:** Low overhead, but lost updates make the result unreliable.
- **Critical section:** Correctness is maintained, but threads spend more time waiting for exclusive access.
- **Atomic operation:** Correctness is maintained with lower overhead for this simple counter update.
- **OpenMP lock:** Correctness is maintained, but repeated lock acquisition and release become expensive under contention.

The benchmark measures elapsed time and correctness, not CPU utilization or hardware-counter events. Consequently, the results demonstrate observed performance and contention but do not independently establish the exact hardware-level cause of every timing difference.

### 8.4 Overall Comparison
<img width="535" height="291" alt="execution_time" src="https://github.com/user-attachments/assets/9d7b367f-7db4-4f52-84dc-35591d74a665" />

| Mechanism | Correctness | Relative performance in this experiment | Suitability |
|---|---|---|---|
| None | Not guaranteed | Fastest, but incorrect under contention | Only when shared updates are not involved |
| Critical section | Correct when used properly | Slow | Protecting larger code blocks |
| Atomic operation | Correct for the protected operation | Best among the tested synchronization strategies | Simple shared-variable updates |
| OpenMP lock | Correct when used properly | Slowest at four and eight threads in this run | Explicit lock-controlled access |
| Barrier | Synchronizes execution phases | Adds waiting overhead | Coordinating parallel computation phases |
| Semaphore | Correct when used properly | Moderate overhead; separate workload | Resource access control and signaling |

## 9. Conclusion

This experiment demonstrated the importance of synchronization in shared-memory parallel programs. The unsynchronized implementation had the lowest execution time but produced incorrect results as the number of threads increased. Critical sections, atomic operations, and locks maintained correctness, but their execution times differed considerably.

For the tested shared-counter workload, atomic operations provided the best balance between correctness and performance. Critical sections and locks introduced greater contention, while barriers were useful for coordinating execution phases rather than protecting shared variables directly. The semaphore implementation demonstrated controlled access using POSIX threads.

Overall, the experiment established that a synchronization mechanism must be selected based on the required correctness guarantees, the type of shared operation, synchronization overhead, and scalability—not execution time alone.

## 10. Repository Structure

```text
PGC-04-Synchronization-Mechanisms-Analysis/
│
├── src/
│   ├── atomic.c
│   ├── barrier.c
│   ├── compare_sync.c
│   ├── critical.c
│   ├── race.c
│   └── semaphore.c
│
├── performance-analysis/
│   ├── exec_vs_no-threads.png   
│   ├── execution_time.png
│   ├── racecondition.png
│   └── result.jpeg
│
├── screenshots/
│   ├── atomic.jpeg
│   ├── barrier.jpeg
│   ├── comparison.jpeg
│   ├── critical.jpeg
│   ├── race.jpeg
│   ├── result.jpeg
│   └── semaphore.jpeg
│
└── README.md
```

