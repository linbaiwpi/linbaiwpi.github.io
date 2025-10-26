---
title: Performance Ninja - Lab 0 Warmup
description: Note about Lab 0 Warmup of of Performance Ninja
author:
date: 2025-10-22 00:00:00 +0100
categories: [Note, Performance]
tags: [SW]
pin: true
math: true
mermaid: true
---
Recently, I started learning software optimization and discovered the treasure blogger [Easyperf](https://easyperf.net/) and his book *Performance Analysis and Tuning on Modern CPUs*.

This time I recorded notes from the first two *Performance Ninja* videos:
- [Welcome Video](https://www.youtube.com/watch?v=2tzdkC6IDbo)
- [Warmup Lab Assignment](https://www.youtube.com/watch?v=jFRwAcIoLgQ&list=PLRWO2AL1QAV6bJAU2kgB4xfodGID43Y5d&index=2)

The main focus was environment setup and optimizing a toy program to get familiar with the overall workflow.

Since the `perf` command needs to be installed, if you want to set up the environment on your own machine, you’ll need to check your OS kernel version.
Here’s my kernel version:
```sh
xxx@G15-5511-Ubuntu-005:~$ uname -a
Linux G15-5511-Ubuntu-005 6.8.0-85-generic #85~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Sep 19 16:18:59 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```
P.S. It’s a bit odd that my OS is Ubuntu 22.04, but the kernel version is 6.8.0. However, this doesn’t affect the setup process.

---

# Create a Docker Container
```bash
docker run --privileged --cap-add SYS_ADMIN --pid=host -it -v /home/xxx:/home/xxx --name perf-ninja ubuntu:22.04 /bin/bash
```

Install necessary software inside the container:
```bash
apt-get update
apt-get upgrade -y
apt-get install build-essential git-core wget cmake -y
mkdir /workspace
cd /workspace
```

Follow [perf-ninja/GetStarted.md](https://github.com/dendibakh/perf-ninja/blob/main/GetStarted.md) and install Google Benchmark using [make_benchmark_library.sh](https://github.com/dendibakh/perf-ninja/blob/main/tools/make_benchmark_library.sh):
```bash
# Check out the library.
git clone https://github.com/google/benchmark.git
# Benchmark requires Google Test as a dependency. Add the source tree as a subdirectory.
git clone https://github.com/google/googletest.git benchmark/googletest
# Go to the library root directory
cd /workspace/benchmark
# Make a build directory to place the build output.
cmake -E make_directory "build"
# Generate build system files with cmake.
cmake -E chdir "build" cmake -DCMAKE_BUILD_TYPE=Release ../
# or, starting with CMake 3.13, use a simpler form:
# cmake -DCMAKE_BUILD_TYPE=Release -S . -B "build"
# Build the library.
cmake --build "build" --config Release --parallel 4

# Build release version of google benchmark library
cd /workspace/benchmark
cmake --build "build" --config Release --target install
```

Install Clang-17:
```bash
cd /workspace/
wget https://apt.llvm.org/llvm.sh
chmod +x llvm.sh
apt install lsb-release wget software-properties-common gnupg -y
./llvm.sh 17 all
# Enable clang-17 compiler for building labs. If you want to make clang-17 the default on the system, do the following:
update-alternatives --install /usr/bin/cc cc /usr/bin/clang-17 30
update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++-17 30
```

---

# Build the Lab Program
```bash
# build lab assignment
cd /workspace
git clone https://github.com/dendibakh/perf-ninja.git
cd /workspace/perf-ninja/labs/misc/warmup
cmake -E make_directory build
cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build . --config Release --parallel 8
cmake --build . --target validateLab
cmake --build . --target benchmarkLab
```

If you want to compile in debug mode:
```bash
cmake -DCMAKE_BUILD_TYPE=Debug .. -DCMAKE_C_FLAGS="-g" -DCMAKE_CXX_FLAGS="-g"
cmake --build . --config Debug --parallel 8
cmake --build . --target validateLab
cmake --build . --target benchmarkLab
```

Example output:
```bash
2025-10-22T21:19:16+00:00
Running ./lab
Run on (16 X 4600 MHz CPU s)
CPU Caches:
  L1 Data 48 KiB (x8)
  L1 Instruction 32 KiB (x8)
  L2 Unified 1280 KiB (x8)
  L3 Unified 24576 KiB (x1)
Load Average: 0.18, 0.15, 0.16
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1           23.7 ns         23.7 ns    119575979
```

You can use the `perf record` command to collect performance data (which will give similar results to the above), and then use `perf report` for performance analysis and to view corresponding assembly code:
```bash
perf record ./lab
perf report
```

---

# Wrapup Lab
In the directory `/workspace/perf-ninja/labs/misc/warmup/`,  
`bench.cpp` is the framework file defining an optimization task: summing numbers from 1 to 1000.
```cpp
#include "solution.h"
#include <iostream>

static void bench1(benchmark::State &state) {
  // problem: count sum of all the numbers up to N
  constexpr int N = 1000;
  int arr[N];
  for (int i = 0; i < N; i++) {
    arr[i] = i + 1;
  }

  int result = 0;

  // benchmark
  for (auto _ : state) {
    result = solution(arr, N);
    benchmark::DoNotOptimize(arr);
  }
}

// Register the function as a benchmark
BENCHMARK(bench1);

// Run the benchmark
BENCHMARK_MAIN();
```

The `solution` function is the one we need to optimize, defined in `solution.cpp` and `solution.h`:
```cpp
#include "solution.h"
int solution(int *arr, int N) {
  int res = 0;
  for (int i = 0; i < N; i++) {
    res += arr[i];
  }
  return res;
}
```

Compile, validate, and display benchmark results:
```bash
root@c68f663b1972:/workspace/perf-ninja/labs/misc/warmup/build# cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build . --config Debug --parallel 8 && cmake --build . --target validateLab && cmake --build . --target benchmarkLab
```

As we know from Gauss’s story, the sum can be computed mathematically as:

$$(1 + N) * N / 2$$

So we can modify `solution.cpp` as:
```cpp
#include "solution.h"

int solution(int *arr, int N) {
  return (N+1)*N/2;
}
```

Compile and run again:
```bash
root@c68f663b1972:/workspace/perf-ninja/labs/misc/warmup/build# cmake -DCMAKE_BUILD_TYPE=Release .. && cmake --build . --config Debug --parallel 8 && cmake --build . --target validateLab && cmake --build . --target benchmarkLab
```

You’ll get output like:
```bash
-- Configuring done
-- Generating done
-- Build files have been written to: /workspace/perf-ninja/labs/misc/warmup/build
Consolidate compiler generated dependencies of target validate
Consolidate compiler generated dependencies of target lab
[ 16%] Building CXX object CMakeFiles/validate.dir/solution.cpp.o
[ 33%] Building CXX object CMakeFiles/lab.dir/solution.cpp.o
[ 66%] Linking CXX executable lab
[ 66%] Linking CXX executable validate
[ 83%] Built target validate
[100%] Built target lab
Consolidate compiler generated dependencies of target validate
[100%] Built target validate
Validation Successful
[100%] Built target validateLab
Consolidate compiler generated dependencies of target lab
[100%] Built target lab
2025-10-22T21:19:33+00:00
Running ./lab
Run on (16 X 4600 MHz CPU s)
CPU Caches:
  L1 Data 48 KiB (x8)
  L1 Instruction 32 KiB (x8)
  L2 Unified 1280 KiB (x8)
  L3 Unified 24576 KiB (x1)
Load Average: 0.14, 0.14, 0.15
***WARNING*** CPU scaling is enabled, the benchmark real time measurements may be noisy and will incur extra overhead.
-----------------------------------------------------
Benchmark           Time             CPU   Iterations
-----------------------------------------------------
bench1          0.692 ns        0.692 ns   4007876365
[100%] Built target benchmarkLab
```

We can see that the result is correct (since `cmake --build . --target validateLab` did not fail), and the execution time improved from **23.7 ns to 0.692 ns**.

If you want to upload the work to a private GitHub branch, refer to the latter part of the video tutorial [Warmup Lab Assignment](https://www.youtube.com/watch?v=jFRwAcIoLgQ&list=PLRWO2AL1QAV6bJAU2kgB4xfodGID43Y5d&index=2).

---

# Troubleshoot
## Installing the `perf` tool
Since the Docker container uses the host kernel, you need to install the corresponding tools for kernel version 6.8.0:
```bash
echo "deb http://archive.ubuntu.com/ubuntu noble main restricted" | tee /etc/apt/sources.list.d/noble-perf.list
apt update
apt install linux-tools-6.8.0-85-generic linux-cloud-tools-6.8.0-85-generic -y
```http://0.0.0.0:4000/tags/typography/
