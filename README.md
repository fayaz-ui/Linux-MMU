# Memory Management: Processes, Threads, and the MMU

Understanding how programs use memory and how the operating system manages memory isolation between processes.

## Table of Contents
- [Overview](#overview)
- [Virtual vs Physical Memory](#virtual-vs-physical-memory)
- [The Memory Management Unit (MMU)](#the-memory-management-unit-mmu)
- [Processes vs Threads](#processes-vs-threads)
- [Memory Isolation](#memory-isolation)
- [Practical Examples](#practical-examples)
- [Key Concepts](#key-concepts)
- [Why This Matters](#why-this-matters)

## Overview

Modern operating systems use virtual memory to provide each process with its own isolated address space. The Memory Management Unit (MMU) translates virtual addresses that programs use into physical addresses in RAM, enabling memory protection and efficient memory management.

## Virtual vs Physical Memory

### Virtual Memory
- Each process sees a consistent, private address space starting from `0x0000`
- Programs always work with virtual addresses
- Virtual address space can be larger than available physical RAM
- Same virtual address can exist in multiple processes simultaneously

### Physical Memory
- Actual locations in RAM chips
- Shared resource managed by the operating system
- Can be allocated non-contiguously while appearing contiguous to programs
- Physical addresses change as the OS moves pages around

## The Memory Management Unit (MMU)

The MMU is hardware that automatically translates every memory access from virtual to physical addresses.

### How Translation Works

```
Virtual Address → Page Table Lookup → Physical Address
    0x1000     →   MMU Hardware    →     0x8000
```

### Page Tables
- Each process has its own page table
- Maps virtual page numbers to physical page numbers
- Maintained by the operating system
- Used by MMU hardware for address translation

**Example Translation:**
```
Process A Page Table:
Virtual 0x1000 → Physical 0x8000
Virtual 0x2000 → Physical 0xC000

Process B Page Table:
Virtual 0x1000 → Physical 0xA000  // Same virtual, different physical!
Virtual 0x3000 → Physical 0xE000
```

## Processes vs Threads

### Processes
- **Isolation**: Each process has its own page table
- **Memory Space**: Completely separate virtual address spaces
- **Communication**: Must use IPC mechanisms (pipes, sockets, shared memory)
- **Security**: Cannot access each other's memory directly

```
Process A (PID 1234):
├── Page Table A
├── Virtual Address Space: 0x0000 - 0xFFFF
└── Physical Pages: 0x8000, 0xC000

Process B (PID 5678):
├── Page Table B
├── Virtual Address Space: 0x0000 - 0xFFFF
└── Physical Pages: 0xA000, 0xE000
```

### Threads
- **Sharing**: Threads within a process share the same page table
- **Memory Space**: Share heap, global variables, and code
- **Private Areas**: Each thread has its own stack
- **Communication**: Can directly access shared memory

```
Process C (Multi-threaded):
├── Shared Page Table
├── Shared Virtual Memory:
│   ├── Code Segment: 0x1000 → Physical 0x10000
│   ├── Heap: 0x2000 → Physical 0x12000
│   └── Global Data: 0x3000 → Physical 0x14000
├── Thread 1 Stack: 0xF000 → Physical 0x16000
└── Thread 2 Stack: 0xE000 → Physical 0x18000
```

## Memory Isolation

### How Isolation Works
1. Each process gets its own page table during creation
2. Page tables only contain mappings to pages allocated to that process
3. Attempting to access unmapped virtual addresses triggers page faults
4. OS handles page faults (usually by terminating the offending process)

### Example of Isolation
```
Process A tries to access virtual address 0x1000:
1. MMU looks up 0x1000 in Process A's page table
2. Finds mapping: 0x1000 → 0x8000
3. Access proceeds to physical address 0x8000

Process A tries to access Process B's data:
1. Process B's data is at physical address 0xA000
2. Process A has no virtual address that maps to 0xA000
3. Process A cannot access Process B's data
```

## Practical Examples

### C Code Example: Process Memory Layout
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_var = 42;          // Global data segment
static int static_var = 100;  // Global data segment

int main() {
    int stack_var = 10;       // Stack
    int *heap_ptr = malloc(sizeof(int)); // Heap
    
    printf("PID: %d\n", getpid());
    printf("Global variable address: %p\n", &global_var);
    printf("Static variable address: %p\n", &static_var);
    printf("Stack variable address: %p\n", &stack_var);
    printf("Heap allocation address: %p\n", heap_ptr);
    printf("Function address: %p\n", main); // Code segment
    
    free(heap_ptr);
    return 0;
}
```

### Thread Example
```c
#include <pthread.h>
#include <stdio.h>

int shared_counter = 0; // Shared between threads

void* thread_function(void* arg) {
    int local_var = 99; // Each thread has its own copy
    shared_counter++;   // Shared memory access
    
    printf("Thread %ld: local_var at %p, shared_counter = %d\n",
           (long)arg, &local_var, shared_counter);
    
    return NULL;
}
```

## Key Concepts

### Virtual Memory Benefits
- **Isolation**: Processes cannot interfere with each other
- **Abstraction**: Programs see consistent address space
- **Efficiency**: Physical memory can be allocated optimally
- **Security**: Access control through page table permissions

### MMU Functions
- **Address Translation**: Virtual to physical mapping
- **Access Control**: Read/write/execute permissions
- **Page Fault Handling**: Triggers OS intervention for invalid accesses
- **TLB Management**: Caches recent translations for performance

### Memory Layout (Typical Process)
```
High Addresses (0xFFFFFFFF)
┌─────────────────────────┐
│        Stack            │ ← Grows downward
│         ↓               │
│                         │
│      (Unused)           │
│                         │
│         ↑               │
│        Heap             │ ← Grows upward
├─────────────────────────┤
│    Global Variables     │
├─────────────────────────┤
│    Code Segment         │
└─────────────────────────┘
Low Addresses (0x00000000)
```

## Why This Matters

### Security Implications
- **Process Isolation**: Malicious or buggy programs cannot corrupt other processes
- **Privilege Separation**: Kernel and user space are strictly separated
- **Buffer Overflow Protection**: Attacks are contained within the victim process

### Performance Benefits
- **Efficient Memory Use**: Physical memory allocated only when needed
- **Caching**: TLB caches translations for faster access
- **Swapping**: Unused pages can be moved to disk transparently

### System Stability
- **Fault Containment**: Process crashes don't affect other processes
- **Memory Fragmentation**: Virtual memory eliminates external fragmentation
- **Dynamic Loading**: Programs and libraries can be loaded at any physical address

### Development Advantages
- **Simplified Programming**: Developers work with predictable virtual addresses
- **Debugging**: Memory layout is consistent across runs
- **Portability**: Same virtual memory model across different hardware

## Advanced Topics

### Copy-on-Write (COW)
When a process forks, parent and child initially share physical pages. Pages are copied only when modified.

### Memory-Mapped Files
Files can be mapped into virtual address space, allowing file I/O through memory operations.

### Shared Memory
Multiple processes can map the same physical pages into their virtual address spaces for efficient IPC.

### Non-Uniform Memory Access (NUMA)
On multi-CPU systems, memory access costs vary based on CPU and memory location.

---

*This README provides a comprehensive overview of memory management concepts. For hands-on learning, try writing programs that examine their own memory layout and experiment with process creation and threading.*
