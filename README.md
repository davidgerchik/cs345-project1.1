# Project 1.1: Building a Sequential MapReduce Library in Go

## Overview

In this project, you will build a simple MapReduce library to gain hands-on experience with the Go programming language and understand the fundamentals of MapReduce.

This project will serve as the foundation for Project 1.2, where you will extend this implementation to a distributed version.

Project 1.1 is structured into two parts:
- **Part A:** Implement the core sequential MapReduce functions.
- **Part B:** Develop a simple MapReduce application by implementing a word counter.

## The Go Programming Language

All assignments in this course are implemented in Go. Follow the instructions at [https://go.dev/doc/install](https://go.dev/doc/install) to install Go.

Reference materials:
- [Effective Go](https://go.dev/doc/effective_go)
- [Go by Example](https://gobyexample.com)
- *The Go Programming Language* (recommended book)

For concurrency background:
- Asynchronous Go for Beginners
- Concurrency is not Parallelism

## Project Setup

```bash
tar -zxvf lab1_1_mapreduce.tgz
```

The extracted folder contains:
```
lab1_1_mapreduce/
  go.mod
  main/
  mapreduce/
```

Verify setup:
```bash
go run main/wc.go
```
If you see `Word Counter: see usage comments in file`, everything is set up correctly.

---

## Part A: Implementing Sequential MapReduce

### Objective

Complete the core functions to enable sequential MapReduce processing. This mode processes tasks one at a time and is useful for debugging before moving to a distributed version.

### Tasks

Implement the following:
- `mapreduce/common_map.go` — implement `doMap()`
- `mapreduce/common_reduce.go` — implement `doReduce()`

### Workflow

1. The master assigns input files to map tasks.
2. Each map task:
   - Reads its input file
   - Applies the map function
   - Writes intermediate key/value pairs into multiple files (one per reduce task)
3. The master collects intermediate results and assigns reduce tasks.
4. Each reduce task:
   - Collects intermediate files
   - Applies the reduce function to aggregate results
   - Writes final output

### Testing

```bash
go test -race -run Sequential ./mapreduce
```

For verbose output, set `debugEnabled = true` in `mapreduce/common.go` and add `-v`:
```bash
go test -race -run Sequential -v ./mapreduce
```

---

## Part B: Implementing a Word Counter Using MapReduce

### Objective

Use the sequential MapReduce framework to implement a word count application.

### Tasks

Implement in `main/wc.go`:

**`mapF(file string, contents string) []mapreduce.KeyValue`**
- Splits input text into words using `unicode.IsLetter`
- Outputs `KeyValue{Key: word, Value: "1"}` for each word

**`reduceF(key string, values []string) string`**
- Sums the occurrences of each word
- Returns the final count as a string

### Testing

Single file:
```bash
go run main/wc.go master sequential main/pg-huckleberry_finn.txt
```

All test files:
```bash
bash main/test-wc.sh
```

---

## Submission Instructions

1. Move to the parent directory of the project folder and run:
   ```bash
   tar -zcvf lab1_1_mapreduce_<GROUP_NUMBER>.tgz lab1_1_mapreduce
   ```
2. Verify that `mapreduce/common_map.go`, `mapreduce/common_reduce.go`, and `main/wc.go` are included.
3. Remove any debugging print statements before submitting.
4. Only one group member submits to Canvas.

---

## Grading

### Part A

Each test case is run twice — once normally and once with race condition checks:

| Test Case           | TA Command       | Pass (ok) | Pass (no race) | Fail |
|---------------------|------------------|-----------|----------------|------|
| TestSequentialSingle | SequentialSingle | 2pt       | 1.4pt          | 0pt  |
| TestSequentialMany   | SequentialMany   | 2pt       | 1.4pt          | 0pt  |

A test that passes both runs earns 100%. Failing the race check deducts 30%. Failing both earns 0%.

### Part B

| Test Case  | TA Command           | Pass | Fail |
|------------|----------------------|------|------|
| Word Count | `bash main/test-wc.sh` | 4pt  | 0pt  |

All members of the group receive the same grade.
