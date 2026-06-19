# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Run the word count test (sequential mode):
```bash
bash main/test-wc.sh
```

Run all MapReduce unit tests:
```bash
go test ./mapreduce/
```

Run a single test:
```bash
go test ./mapreduce/ -run TestSequentialSingle
```

Run distributed mode manually:
```bash
# Terminal 1 - Master:
go run main/wc.go master localhost:7777 main/pg-*.txt
# Terminal 2+ - Workers:
go run main/wc.go worker localhost:7777 localhost:7778 &
```

## Architecture

This is a CS 345 MapReduce implementation in Go (`module cs345`). The framework is split across two packages:

### `mapreduce/` — Framework (mostly read-only)
- **`master.go`** — `Master` struct, `Sequential()` and `Distributed()` entrypoints. `Sequential` runs map/reduce inline; `Distributed` starts an RPC server and calls `schedule()` to dispatch tasks to workers.
- **`schedule.go`** — **Student TODO (Part 2):** `schedule()` must assign all map/reduce tasks to available workers over RPC, handling worker failures.
- **`common_map.go`** — **Student TODO (Part 1):** `doMap()` reads an input file, calls `mapF`, partitions output into `nReduce` intermediate files using `ihash(key) % nReduce`. File names come from `reduceName(jobName, mapTask, r)`.
- **`common_reduce.go`** — **Student TODO (Part 1):** `doReduce()` reads all intermediate files for a reduce task, sorts by key, calls `reduceF` per key, writes JSON-encoded `KeyValue` output to `outFile`.
- **`common_rpc.go`** — RPC types (`DoTaskArgs`, `RegisterArgs`, `ShutdownReply`) and `call()` helper (UNIX-domain sockets).
- **`worker.go`** — `Worker` struct and `RunWorker()`. Workers receive `DoTask` RPCs from the master and call `doMap`/`doReduce`.
- **`common.go`** — `KeyValue` struct, `reduceName()`/`mergeName()` filename helpers, `debug()`.
- **`master_splitmerge.go`**, **`master_rpc.go`** — Merge final outputs, RPC server lifecycle.

### `main/` — Application
- **`wc.go`** — **Student TODO (Part 1B):** Implement `mapF` (tokenize words → `KeyValue{word, ""}`) and `reduceF` (count occurrences → string). Supports sequential and distributed invocation modes.

### Data flow
1. Master splits input files among map tasks.
2. `doMap` writes intermediate files `mrtmp.<job>-<mapTask>-<reduceTask>` (one per reduce bucket).
3. `doReduce` reads all intermediate files for its reduce task, sorts, reduces, writes `mrtmp.<job>-res-<reduceTask>` as JSON.
4. Master merges reduce outputs into final `mrtmp.<job>`.

### Key constraints
- Intermediate and output files use JSON encoding (`encoding/json`).
- The `call()` function uses UNIX-domain sockets — worker addresses are socket paths under `/var/tmp/`.
- Do not modify files marked `// Please do not modify this file.` (`master.go`, `worker.go`).
- Enable `debugEnabled = true` in `common.go` for verbose logging during development.
