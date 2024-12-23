# Memory Allocator for x86-64 Architecture

## Overview
This project implements a memory allocator for the x86-64 architecture with the following features:

- **Segregated Free Lists**: Free blocks are segregated by size class, using a first-fit policy within each class.
- **Immediate Coalescing**: Large blocks are coalesced with adjacent free blocks immediately upon being freed.
- **Boundary Tags**: Efficient coalescing is supported by boundary tags, with an optimization that omits footers from allocated blocks.
- **Block Splitting**: Blocks are split without creating splinters.
- **32-Byte Alignment**: Allocated blocks are aligned to 32-byte boundaries.
- **LIFO Free List Management**: Free lists are maintained using a last-in-first-out (LIFO) discipline.
- **Prologue and Epilogue**: A prologue and epilogue are used to avoid edge cases at the ends of the heap.
- **Wilderness Preservation**: A heuristic prevents unnecessary heap growth by treating the "wilderness block" as larger than any other block.

---

## Key Features and Policies

### Free List Management
- Free blocks are organized in a fixed array of segregated free lists, indexed by size class.
- The size classes follow a Fibonacci sequence:
  - Minimum block size: `M = 32 bytes`
  - Each subsequent class is a multiple of `M`, with size intervals based on Fibonacci numbers.
- The last free list contains only the wilderness block, which extends the heap when necessary.

### Block Placement
- **Segregated Fits Policy**:
  - Allocations start by searching the smallest size class that can accommodate the request.
  - Within a size class, the first-fit policy is used.
- If no suitable block exists, the heap is extended using the `sf_mem_grow` function.

### Splitting and Splinters
- Blocks are split during allocation to reduce internal fragmentation.
- Splinters (blocks smaller than 32 bytes) are avoided by over-allocating if necessary.

### Freeing and Coalescing
- Freed blocks are coalesced with adjacent free blocks to reduce external fragmentation.
- Coalesced blocks are inserted at the front of their size class's free list.

---

## Block Format

### Header
Each block's header includes:
- Block size (aligned to 32 bytes).
- Allocation status (free/allocated).
- Status of the previous block (allocated/free).

### Footer
- Free blocks include a footer that mirrors the header.
- Allocated blocks omit the footer, using the space for payload.

### Alignment
- Blocks are aligned to 32-byte boundaries.
- Minimum block size: 32 bytes (header + footer + payload).

---

## Implemented Functions

### `sf_malloc(size_t size)`
- Allocates memory of the specified size.
- Returns a pointer to a valid memory region or `NULL` if allocation fails.

### `sf_realloc(void *ptr, size_t size)`
- Resizes a previously allocated memory region to the specified size.
- Returns a pointer to the resized region or `NULL` if resizing fails.

### `sf_free(void *ptr)`
- Frees the memory region pointed to by `ptr`.
- Coalesces the block with adjacent free blocks if possible.

### `sf_memalign(size_t size, size_t alignment)`
- Allocates memory of the specified size aligned to the specified boundary.
- Returns a pointer to the aligned region or `NULL` if allocation fails.

---
