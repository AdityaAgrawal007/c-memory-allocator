# Heap Allocator in C

A memory allocator built from scratch on a static 1000-byte heap. Implements `malloc` and `free` with block splitting, forward coalescing, and backward coalescing using an explicit free list.

## How it works

The heap is a static `unsigned char` array. Every allocation carves out a region with a header tracking block size, allocation status, and whether the previous block is free. The header is padded to maintain alignment.

`allocate_heap_memory` uses first-fit. It walks the block list until it finds a free block large enough to satisfy the request. The raw size is rounded up to the nearest alignment boundary with `ROUND_UP`. If the leftover space after carving out the requested block is large enough to hold a header plus a `free_node`, the remainder is split into a separate free block rather than wasted as internal fragmentation.

`free` marks the block as unallocated, then coalesces. It checks the next block first: if it's free, the two blocks merge. For backward coalescing, it checks the `previous_block_status` flag in the header. If the previous block is free, it walks from the start of the heap to find it and merges backward. The `previous_block_status` flag on the block after the newly freed region is updated so future frees stay correct.

The epilogue block at the end of the heap has `size_of_block = 0` and `allocation_status = true`. This lets the allocation loop detect the end of the heap without a bounds check.

## Build and test

```bash
make
./exe/test_basic
```

The other tests (`test_block_splitting`, `test_header_sanity`, `test_monotonic_growth`) can be added to the Makefile as additional targets following the same pattern.

## Design notes

`ALIGNMENT` is set to 6 in the header. This is non-standard since power-of-two alignment is conventional, and the `ROUND_UP` macro (`((x) + (a-1)) & ~(a-1)`) is only mathematically correct when `a` is a power of two. This is a known limitation.

Free list nodes (`free_node`) live inside the payload region of free blocks. This means the minimum usable payload is `sizeof(free_node)`, which is two pointers. Any allocation smaller than that gets rounded up silently.

Backward coalescing walks from `first` to find the previous block, making it O(n) in the number of live blocks. A boundary tag at the end of each block would make it O(1).
