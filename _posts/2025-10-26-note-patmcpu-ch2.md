---
title: Performance Analysis and Tuning on Modern CPU - Chapter 2
description: Note about Chapter 2 of Performance Analysis and Tuning on Modern CPU
author:
date: 2025-10-22 00:00:00 +0100
categories: [Note, Performance]
tags: [SW]
pin: true
math: true
mermaid: true
---

# 2 Measuring Performance

## Intrinsic Challenges of Performance Measurement

- **Counterintuitive nature**: Measurement results are often surprising. Seemingly unrelated code changes can have a major impact on performance.
- **Measurement bias**: Results may consistently overestimate or underestimate true performance, leading to distortion.
- **Poor reproducibility**: Unlike functional bugs, performance issues are nearly impossible to reproduce at the “cycle-level” accuracy since performance characteristics can vary between runs.

---

### Foundations of Successful Measurement

- **Conduct fair experiments**: This is the key to obtaining accurate and meaningful results. Ensure you are studying the right problem rather than debugging unrelated noise.
- **Careful design**: Designing performance test cases and configuring the measurement environment are two crucial parts of the evaluation process.

---

### Key Topics Covered in This Chapter

- **Measurement noise in modern systems**: An overview of why noise exists in performance measurement and how to mitigate it.
- **The importance of production measurements**: Explains the value of measuring performance in deployed systems.
- **General guidelines for performance measurement**: Provides general guidance on how to correctly collect and analyze performance data.
- **Automated performance regression detection**: Explores methods to automatically detect performance regressions after code changes.
- **Software and hardware timers**: Describes the various timing mechanisms available to developers for time-based measurement.
- **How to write good microbenchmarks**: Discusses best practices and common pitfalls when writing microbenchmarks.

---

## 2.1 Noise in Modern Systems

Many performance characteristics of modern computer systems (both hardware and software) are **non-deterministic**, which makes performance measurement inherently noisy and variable. Conducting fair and accurate performance experiments is extremely challenging.

### Major Sources of Noise

#### 1. Hardware Noise – Dynamic Frequency Scaling Example
- **Phenomenon**: CPU turbo mode (high-frequency operation) may not sustain due to thermal limits, leading to lower frequencies in subsequent runs.
- **Impact**: As shown in the graph, the first run on a “cold” CPU benefits from short-term high frequency and runs 200 ms faster than the second run, even with identical code.
- **Other factors**: Even running monitoring tools like Task Manager can activate additional CPU cores and affect the target program’s frequency.

#### 2. Software Noise – Filesystem Cache Example
- **Phenomenon**: The first run of a disk-accessing program (like `git status`) is slower because it must populate the filesystem cache; the second run is much faster once the cache is “warm.”
- **Deeper sources of noise**: Environment variable sizes, link order, memory layout, and other seemingly irrelevant factors can unpredictably affect performance.

---

### Mitigation Strategies and Trade-offs

#### 1. Control the Environment to Minimize Noise (for precise comparisons)
- **Scenario**: When you need to accurately measure relative improvements (before vs. after a code change).
- **Method**: Rigidly control hardware configuration, OS settings, and background processes. Disable non-deterministic features (see Appendix A). Tools like `temci` can help create low-variance environments.
- **Limitation**: It’s impossible to eliminate all noise sources (e.g., thermal fluctuations, power spikes, interrupts). This can be an endless process, especially for large distributed cloud systems.

#### 2. Embrace Noise to Measure Real-World Performance (for production evaluations)
- **Scenario**: When you need to evaluate performance impact in real-world settings.
- **Method**: Replicate the production environment as closely as possible. Non-deterministic features (like Turbo mode, file caching) are designed to enhance user experience — disabling them would make measurements unrepresentative.
- **Drawback**: Higher variance in results.

---

### Conclusion

Performance engineers must make informed trade-offs:
- To achieve **precise and repeatable micro-level comparisons**, control and reduce system noise as much as possible.
- To assess **real-world macro performance**, measure in representative, noisy, production-like environments.

---

## 2.2 Measuring Performance in Production

Did not fully understand the meaning of this section.

---

## 2.3 Continuous Benchmarking

Did not fully understand the meaning of this section.

---

## 2.4 Manual Performance Testing

When automated CI systems are unavailable, local **manual performance evaluation** can be conducted. The core process is:
**Measure baseline performance → Measure modified performance → Compare both.**

### Methods

- **Method 1**: Perform multiple measurements to obtain a distribution.
- **Method 2**: Collect *N* measurements for both “before” (baseline) and “after” versions to create two performance distributions.

Use professional benchmarking tools (e.g., **Hyperfine**) — they can automatically determine the number of runs, handle warm-up, and present results in tables or visual charts (means, extremes, boxplots, etc.), simplifying the process.

The essence of manual testing is to build **performance distributions** through multiple measurements, visualize them (e.g., boxplots), and compare metrics (mean, median, standard deviation, percentiles). Testers must balance data reliability and measurement cost, understand each metric’s meaning, and make objective, data-driven performance judgments.

---

## 2.5 Software and Hardware Timers

Modern platforms provide two main timing mechanisms: **system high-resolution timers (software)** and **timestamp counters (hardware)**. For most situations not requiring cycle-level precision, system timers (e.g., C++ `std::chrono`) are sufficient, easier to use, and more portable.

| Feature | System High-Resolution Timer | Timestamp Counter (TSC) |
|----------|------------------------------|--------------------------|
| Type | Software timer (OS-provided) | Hardware timer (CPU register) |
| Implementation | System call (e.g., Linux `clock_gettime`) | Instruction (e.g., x86 `RDTSC`) |
| Precision | Nanoseconds | CPU cycles |
| Monotonic | Yes | Yes |
| Consistency | Same across all CPU cores | Each core has its own TSC |
| Access overhead | High (~500 ns) | Low (~5 ns or ~20 CPU cycles) |
| Suitable for | Events longer than microseconds | Short events from nanoseconds to minutes |
| Example | C++ `std::chrono` | `__rdtsc()` |

Timer choice depends on the measurement duration and required precision. For minimal overhead and maximal precision in short events, **TSC** is unmatched; for longer measurements, system timers offer sufficient accuracy and ease of use.

---

## 2.6 Microbenchmarks

Microbenchmarks are small, standalone programs for quickly validating performance hypotheses, typically comparing algorithmic or implementation differences. Modern languages provide frameworks (e.g., C++ Google Benchmark, C# BenchmarkDotNet).

### Key Challenges and Best Practices

1. **Prevent Compiler Over-Optimization**
   Compilers might optimize away the code you’re trying to measure, invalidating results. Use helper functions like `DoNotOptimize` to prevent unwanted optimizations.

2. **Ensure Realistic Testing**
   Use input data that reflects real-world conditions. Synthetic or atypical data may yield misleading results. Microbenchmarks often run in isolation, which hides contention issues that appear under real workloads. Always “measure one layer deeper” — confirm the target code is the actual bottleneck.

### Microbenchmarks vs. Unit Tests

- Unit test frameworks (e.g., GoogleTest) do **not** replace well-designed microbenchmarks.
- Microbenchmarks focus on performance under realistic conditions; unit tests focus on correctness in controlled environments.

### Summary

Writing effective microbenchmarks requires caution. Developers must:
- Guard against compiler optimizations that invalidate measurements.
- Simulate realistic workloads and environments.
- Understand the differences between microbenchmarks and unit tests.

A well-written microbenchmark is invaluable for assessing code performance, while a poorly designed one can be worse than none — leading optimization in the wrong direction.

---

## 2.7 Active Benchmarking

### Core Idea: Active vs. Passive Benchmarking

- **Passive Benchmarking**
  - *Approach*: “Run and forget.” Accepts favorable results and ignores unfavorable or abnormal ones.
  - *Consequence*: Leads to incomplete, misleading, or false conclusions. May overstate performance gains (e.g., quoting a one-off 30% speedup as typical).

- **Active Benchmarking**
  - *Approach*: Investigative and verification-oriented. Probes the underlying causes behind performance changes.
  - *Method*: Ensures result reliability through multidimensional validation.

### Active Benchmarking Practices

Active benchmarking requires working like a detective. Core practices include:
- **Environmental consistency**: Ensure identical hardware configurations (CPU, RAM) to avoid distortions.
- **Repetition and validation**: Run tests for longer periods and remove outliers.
- **Deep analysis**: “Always measure one layer deeper.” Inspect machine code differences, hardware events (instructions, cache misses, page faults, context switches).
- **Technical attribution**: Seek the technical root cause of observed results.
- **Skeptical mindset**: Treat surprisingly good results with suspicion — as John Ousterhout said, “All performance results are guilty until proven innocent.”
- **Reliability over convenience**: Active benchmarking demands more effort but yields trustworthy conclusions.
- **Comprehensive reporting**: Present well-explained results, not just percentage improvements.

---

# Additional Knowledge

## Active Benchmarking [1]

1. Analyze hardware and benchmark configuration beforehand.
2. Analyze performance *during* the benchmark (not just after), using tools (e.g., the USE Method).
3. Include limiting or suspected limiting factors in reports to explain results.

Example: [Active Benchmarking: Bonnie++](https://www.brendangregg.com/ActiveBenchmarking/bonnie++.html)

---

# References

1. [Active Benchmarking](https://www.brendangregg.com/activebenchmarking.html)
2. [Active Benchmarking for Better Performance Predictions](https://6083598.fs1.hubspotusercontent-na1.net/hubfs/6083598/Active%20Benchmarking%20for%20Better%20Performance%20Predictions.pdf)
