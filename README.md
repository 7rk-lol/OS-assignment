# Distributed Producer-Consumer Transfer & Parallel Analytics

This project implements a multi-process socket-based file transfer (Producer-Consumer model) and a multi-threaded parallel file analytics engine in C, targeted for Ubuntu 22.04+.

## Project Structure

```
project_root/
├── src/
│   ├── common.h         # Common definitions and TCP helpers
│   ├── server.c         # Part 1: TCP Server (Producer)
│   ├── client.c         # Part 1: TCP Client (Consumer)
│   ├── operations.c     # Part 2: Parallel Analytics (Min, Max, Sort)
│   ├── generate_data.c  # Helper: 1GB random integer generator
│   ├── verify.c         # Helper: Correctness verification tool
│   └── page_simulator.c # Helper: Page replacement algorithm simulator
├── Makefile             # Compilation rules
├── run_test.sh          # Test suite runner
└── README.md            # Project description
```

## Mandatory Primitives Used

1. **Network IPC**: TCP sockets (`socket`, `bind`, `listen`, `accept`, `connect`, `send`, `recv`).
2. **Process Management**: `fork`, `exec`, `wait`/`waitpid`.
3. **Thread Management**: `pthread_create`, `pthread_join`.
4. **Synchronization**:
   - `pthread_mutex_t`: Protecting process-shared reassembly buffers and thread global reductions.
   - `sem_t` (POSIX): Semaphore coordination for chunk delivery counting and sorting thread pool size limiting.
   - `pthread_cond_t`: Condition variables for chunk arrival signaling and bottom-up thread merge step transitions.
5. **Memory Mapping**: `mmap` for sharing client reassembly buffers and IPC coordination structures.

## Build Instructions

To compile the server, client, operations binary, verification tools, and page simulator:
```bash
make
```

To clean build artifacts and output files:
```bash
make clean
```

## Running the Automated Test Suite

To run the automated pipeline (generates 1GB test file, starts server/client, runs analytics, and verifies output values/log formats):
```bash
make test
```

## Usage

### Part 1: Server (Producer)
```bash
./server <input_binary_file>
```
Starts the server listening on port `9090`. It accepts connections, determines chunk count, and streams data chunks concurrently to client children.

### Part 1: Client (Consumer)
```bash
./client -p <num_processes> [-h <server_ip>] [-t <num_threads>]
```
- `-p`: Number of processes and chunks to divide the file into.
- `-h`: IP address of the server (defaults to `127.0.0.1`).
- `-t`: Number of threads to run the operations analytics with (defaults to `8`).

### Part 2: Operations (Analytics Engine)
The client will automatically trigger `./operations` upon successful transfer, but it can also be run standalone:
```bash
./operations -t <num_threads> -f <reassembled_file>
```
- `-t`: Number of worker threads.
- `-f`: Path to the reassembled binary file.

### Bonus: Page Replacement Simulator
An interactive simulator implementing Page Replacement algorithms covered in **Lecture 9 & 10**:
```bash
./page_simulator
```
Upon execution, it prompts you to:
1. Enter the number of physical frames (default: 3).
2. Enter the page reference string (or press Enter to use the default slide-based string: `7 0 1 2 0 3 0 4 2 3 0 3 2 1 2 0 1 7 0 1`).
It outputs step-by-step frame contents, faults, and hit/fault rates for FIFO, LRU, and Optimal.
