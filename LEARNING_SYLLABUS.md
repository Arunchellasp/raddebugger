# Advanced C Systems Programming: Learning Patterns from Production Codebases

## Overview

This syllabus teaches **universal architectural patterns and coding principles** used in large-scale C systems projects (kernels, debuggers, compilers, game engines). By studying the RADDebugger as a reference implementation, you'll learn patterns that apply to ANY complex project.

Rather than debugger-specific knowledge, you'll master:
- **Memory management at scale**
- **Layered architecture & dependency management**
- **Binary file format parsing**
- **Asynchronous processing & caching**
- **Type systems & abstraction**
- **Performance optimization**
- **Code organization strategies**

Each concept includes **step-by-step practice code** you can write and test.

---

## Part 1: Foundational Concepts

### Module 1.1: Layered Architecture & Dependency Management

**Objective**: Learn how to organize code into independent, composable layers.

**Key Principles**:
1. **Directed Acyclic Graph (DAG) of Dependencies**
   - Layer A can depend on Layer B
   - Layer B cannot depend on Layer A (prevents circular deps)
   - Creates clear hierarchical structure
   - Makes testing and refactoring easier

2. **Layer Independence**
   - Bottom layers (base) have zero dependencies
   - Middle layers depend only on lower layers
   - Top layers (UI) depend on most layers
   - Allows reuse in different projects

3. **Namespace Conventions**
   - Every function/type has a prefix (e.g., `str8_`, `arena_`, `hash_`)
   - Makes grep-ability instant
   - Prevents naming collisions
   - Clarifies which layer owns the code

**Practice Exercise 1.1: Design a Layered Project**

```c
// Your goal: Create a structured project with 3 layers
// 
// LAYER STRUCTURE:
// ┌─────────────────────┐
// │   Application (APP_) │  <- Depends on: Core, Util
// ├─────────────────────┤
// │   Util (UT_)        │  <- Depends on: Core
// ├─────────────────────┤
// │   Core (CORE_)      │  <- Depends on: Nothing
// └─────────────────────┘

// FILE: core.h (zero dependencies)
// ================================
#ifndef CORE_H
#define CORE_H

#include <stdint.h>
#include <string.h>

// Integer types
typedef uint8_t  U8;
typedef uint16_t U16;
typedef uint32_t U32;
typedef uint64_t U64;
typedef int32_t  S32;
typedef int32_t  B32;

// Memory
typedef struct {
  U8 *data;
  U64 size;
} CORE_Memory;

// Allocate memory (basic)
CORE_Memory core_alloc(U64 size);
void core_free(CORE_Memory mem);

#endif // CORE_H

// FILE: core.c (implementation)
// ================================
#include "core.h"
#include <stdlib.h>

CORE_Memory core_alloc(U64 size) {
  CORE_Memory mem = {0};
  mem.data = (U8 *)malloc(size);
  mem.size = size;
  return mem;
}

void core_free(CORE_Memory mem) {
  free(mem.data);
}

// FILE: util.h (depends on core)
// ================================
#ifndef UTIL_H
#define UTIL_H

#include "core.h"

// String type: {pointer, size}
typedef struct {
  U8 *str;
  U64 size;
} UT_String;

// String operations
UT_String ut_string_from_cstr(char *cstr);
B32 ut_string_equal(UT_String a, UT_String b);
void ut_string_print(UT_String s);

#endif // UTIL_H

// FILE: util.c
// ================================
#include "util.h"
#include <stdio.h>

UT_String ut_string_from_cstr(char *cstr) {
  UT_String s = {0};
  if (cstr) {
    s.str = (U8 *)cstr;
    s.size = strlen(cstr);
  }
  return s;
}

B32 ut_string_equal(UT_String a, UT_String b) {
  if (a.size != b.size) return 0;
  return memcmp(a.str, b.str, a.size) == 0;
}

void ut_string_print(UT_String s) {
  printf("%.*s\n", (int)s.size, (char *)s.str);
}

// FILE: app.h (depends on util and core)
// ================================
#ifndef APP_H
#define APP_H

#include "util.h"

typedef struct {
  UT_String name;
  U32 age;
} APP_Person;

void app_init(void);
void app_print_person(APP_Person p);

#endif // APP_H

// FILE: app.c
// ================================
#include "app.h"
#include <stdio.h>

void app_init(void) {
  printf("App initialized\n");
}

void app_print_person(APP_Person p) {
  printf("Person: ");
  ut_string_print(p.name);
  printf("Age: %u\n", p.age);
}

// FILE: main.c
// ================================
#include "app.h"

int main(void) {
  app_init();
  
  APP_Person person = {
    .name = ut_string_from_cstr("Alice"),
    .age = 30
  };
  
  app_print_person(person);
  return 0;
}
```

**What You Learned**:
- ✅ Created 3-layer hierarchy without circular dependencies
- ✅ Each layer has a namespace prefix
- ✅ Lower layers have fewer dependencies
- ✅ Code is modular and reusable

**Exercises**:
1. Add a 4th layer (Data) between Util and App
2. Create a dependency graph showing which layer depends on which
3. Try to create a circular dependency and see why it fails

---

### Module 1.2: Coding Conventions & Naming

**Objective**: Master naming conventions that make code self-documenting.

**Key Conventions**:

```c
// 1. KEYWORDS & SCOPE
internal      // static (file-local scope)
global        // static (file-local global variable)
local_persist // static (persistent local variable)

// 2. NAMING SCHEME
// Types:      PascalCase with namespace (Hash_Table, Arena, String8)
// Functions:  snake_case with namespace (hash_get, arena_push)
// Macros:     UPPER_CASE (KB, MB, MAX, MIN)
// Variables:  snake_case (count, result, buffer)

// 3. INTEGER TYPES (standardized)
typedef uint8_t  U8;   // unsigned 8-bit
typedef uint16_t U16;  // unsigned 16-bit
typedef uint32_t U32;  // unsigned 32-bit
typedef uint64_t U64;  // unsigned 64-bit
typedef int8_t   S8;   // signed 8-bit
typedef int32_t  S32;  // signed 32-bit
typedef int32_t  B32;  // boolean (32-bit)

// 4. SECTION ORGANIZATION
// Header files: sections marked with //~ for grouping
// Implementation: //- for subsections

// FILE: example.h
// ================================
#ifndef EXAMPLE_H
#define EXAMPLE_H

// Code sections grouped with //~

////////////////////////////////
//~ EX: Type Definitions

typedef struct {
  U32 id;
  U64 value;
} EX_Item;

////////////////////////////////
//~ EX: Main API

EX_Item ex_item_create(U32 id, U64 value);
void ex_item_print(EX_Item item);

#endif // EXAMPLE_H

// FILE: example.c
// ================================
#include "example.h"
#include <stdio.h>

////////////////////////////////
//~ EX: Item Creation

EX_Item ex_item_create(U32 id, U64 value) {
  EX_Item item = {0};
  item.id = id;
  item.value = value;
  return item;
}

//- EX: Printing

void ex_item_print(EX_Item item) {
  printf("Item{id=%u, value=%llu}\n", item.id, item.value);
}
```

**Practice Exercise 1.2: Rewrite Bad Code with Good Conventions**

```c
// BAD CODE (what NOT to do):
// ================================
int _global_count = 0;

struct item {  // Wrong: lowercase
  int ID;      // Wrong: CAPS and inconsistent
  int val;     // Wrong: abbreviated
};

struct item get_item() {  // Wrong: no namespace
  struct item i;
  i.ID = _global_count++;
  i.val = 100;
  return i;
}

void pr(struct item i) {  // Wrong: abbreviated function name
  printf("%d %d\n", i.ID, i.val);
}

// GOOD CODE (what you should write):
// ================================
global U32 item_counter = 0;

typedef struct {
  U32 id;
  U64 value;
} ITEM_Record;

internal ITEM_Record item_record_create(void) {
  ITEM_Record record = {0};
  record.id = item_counter++;
  record.value = 100;
  return record;
}

internal void item_record_print(ITEM_Record record) {
  printf("Item{id=%u, value=%llu}\n", record.id, record.value);
}
```

**Your Task**:
Rewrite this bad code using proper conventions:

```c
// Fix this:
int count;
char* str;
struct data { int a; int b; };
void do_work(struct data* d);
int compute(int x, int y);

// Should become:
// [Write your answer here]
```

---

---

## Part 2: Memory Management Mastery

### Module 2.1: Arena Allocation (Better than malloc)

**Objective**: Learn arena-based memory allocation used in all systems programming.

**Why Arenas?**
- `malloc` is slow (every allocation hits the heap)
- `malloc` fragments memory
- `malloc` is hard to clean up (easy to leak)
- **Arenas**: Pre-allocate large block, divide into regions, free all at once

**How It Works**:

```c
// FILE: arena.h
#ifndef ARENA_H
#define ARENA_H

#include <stdint.h>

typedef uint64_t U64;
typedef uint8_t U8;
typedef int B32;

// Arena: large pre-allocated memory region
typedef struct {
  U8 *data;      // start of memory
  U64 capacity;  // total size
  U64 used;      // how much we've used
} Arena;

// Create an arena
Arena arena_create(U64 size);

// Allocate memory from arena
void *arena_push(Arena *a, U64 size);

// Clear arena (for reuse)
void arena_clear(Arena *a);

// Free arena
void arena_free(Arena *a);

// Temporary scope: auto-cleanup
typedef struct {
  Arena *arena;
  U64 pos;
} Temp;

Temp temp_begin(Arena *a);
void temp_end(Temp t);

#endif // ARENA_H

// FILE: arena.c
#include "arena.h"
#include <stdlib.h>

Arena arena_create(U64 size) {
  Arena a = {0};
  a.data = (U8 *)malloc(size);
  a.capacity = size;
  a.used = 0;
  return a;
}

void *arena_push(Arena *a, U64 size) {
  if (a->used + size > a->capacity) {
    return 0; // out of memory
  }
  void *result = a->data + a->used;
  a->used += size;
  return result;
}

void arena_clear(Arena *a) {
  a->used = 0;
}

void arena_free(Arena *a) {
  free(a->data);
  a->data = 0;
  a->capacity = 0;
  a->used = 0;
}

Temp temp_begin(Arena *a) {
  Temp t = {.arena = a, .pos = a->used};
  return t;
}

void temp_end(Temp t) {
  t.arena->used = t.pos;
}

// FILE: main.c
#include "arena.h"
#include <stdio.h>

int main(void) {
  // Create arena: allocate 1 MB upfront
  Arena arena = arena_create(1024 * 1024);
  
  // Allocate many small objects (fast, no fragmentation!)
  for (int i = 0; i < 1000; i++) {
    int *ptr = (int *)arena_push(&arena, sizeof(int));
    if (ptr) *ptr = i;
  }
  
  printf("Used %llu bytes\n", arena.used);
  
  // Temporary scope: all allocations freed when scope ends
  {
    Temp temp = temp_begin(&arena);
    char *buffer = (char *)arena_push(&arena, 256);
    // ... use buffer ...
    temp_end(temp); // buffer memory reclaimed!
  }
  
  printf("After temp: Used %llu bytes\n", arena.used);
  
  // Clear entire arena for reuse
  arena_clear(&arena);
  
  // All memory released
  arena_free(&arena);
  
  return 0;
}
```

**Key Benefits**:
- ✅ All allocations in one big block (CPU cache friendly)
- ✅ No fragmentation (allocations don't scatter)
- ✅ Easy cleanup (free entire arena at once)
- ✅ Temporary scopes (auto-cleanup)
- ✅ 100x faster than malloc for many allocations

**Practice Exercise 2.1: Use Arenas in Your Code**

Your task: Rewrite this malloc-heavy code using arenas:

```c
// BAD: Using malloc everywhere
struct Item {
  int *data;
  int count;
};

struct Item *items = (struct Item *)malloc(sizeof(struct Item) * 100);
for (int i = 0; i < 100; i++) {
  items[i].data = (int *)malloc(sizeof(int) * 50);
  items[i].count = 50;
}

// ... use items ...

// CLEANUP NIGHTMARE:
for (int i = 0; i < 100; i++) {
  free(items[i].data);
}
free(items);

// YOUR TASK: Rewrite using arena allocation
// Hint: Pre-allocate one big arena, use arena_push for everything
```

**Solution**:

```c
// GOOD: Using arenas
Arena arena = arena_create(10 * 1024 * 1024); // 10 MB

struct Item {
  int *data;
  int count;
};

// All allocations come from one arena
struct Item *items = (struct Item *)arena_push(&arena, sizeof(struct Item) * 100);
for (int i = 0; i < 100; i++) {
  items[i].data = (int *)arena_push(&arena, sizeof(int) * 50);
  items[i].count = 50;
}

// ... use items ...

// ONE cleanup:
arena_free(&arena);
```

---

### Module 2.2: Smart Strings (String8 Pattern)

**Objective**: Learn the {pointer, size} string pattern used everywhere.

**Why Not Null-Terminated?**
- Can have embedded nulls
- No need to scan for terminator
- Easy to slice/substring
- More efficient

**Implementation**:

```c
// FILE: string.h
#ifndef STRING_H
#define STRING_H

#include <stdint.h>

typedef uint8_t U8;
typedef uint64_t U64;
typedef int B32;

// String: {pointer, size} pair
typedef struct {
  U8 *str;
  U64 size;
} String8;

// Create string from C string
String8 str8_from_cstr(char *cstr);

// Create string from data + size
String8 str8(U8 *data, U64 size);

// Compare strings
B32 str8_equal(String8 a, String8 b);

// Find substring
U64 str8_find_substring(String8 haystack, String8 needle);

// Slice substring
String8 str8_slice(String8 s, U64 start, U64 end);

// String builder (dynamic)
typedef struct {
  U8 *data;
  U64 size;
  U64 capacity;
} String8Builder;

String8Builder str8_builder_create(U64 initial_capacity);
void str8_builder_append(String8Builder *b, String8 s);
String8 str8_builder_get(String8Builder *b);
void str8_builder_free(String8Builder *b);

#endif // STRING_H

// FILE: string.c
#include "string.h"
#include <string.h>
#include <stdlib.h>

String8 str8_from_cstr(char *cstr) {
  String8 s = {0};
  if (cstr) {
    s.str = (U8 *)cstr;
    s.size = strlen(cstr);
  }
  return s;
}

String8 str8(U8 *data, U64 size) {
  return (String8){.str = data, .size = size};
}

B32 str8_equal(String8 a, String8 b) {
  if (a.size != b.size) return 0;
  return memcmp(a.str, b.str, a.size) == 0;
}

U64 str8_find_substring(String8 haystack, String8 needle) {
  for (U64 i = 0; i + needle.size <= haystack.size; i++) {
    if (memcmp(haystack.str + i, needle.str, needle.size) == 0) {
      return i;
    }
  }
  return (U64)-1; // not found
}

String8 str8_slice(String8 s, U64 start, U64 end) {
  if (start > s.size) start = s.size;
  if (end > s.size) end = s.size;
  if (start > end) start = end;
  return str8(s.str + start, end - start);
}

String8Builder str8_builder_create(U64 initial_capacity) {
  String8Builder b = {0};
  b.data = (U8 *)malloc(initial_capacity);
  b.capacity = initial_capacity;
  b.size = 0;
  return b;
}

void str8_builder_append(String8Builder *b, String8 s) {
  if (b->size + s.size > b->capacity) {
    b->capacity *= 2;
    b->data = (U8 *)realloc(b->data, b->capacity);
  }
  memcpy(b->data + b->size, s.str, s.size);
  b->size += s.size;
}

String8 str8_builder_get(String8Builder *b) {
  return str8(b->data, b->size);
}

void str8_builder_free(String8Builder *b) {
  free(b->data);
  b->data = 0;
  b->capacity = 0;
  b->size = 0;
}

// FILE: main.c
#include "string.h"
#include <stdio.h>

int main(void) {
  // Create strings
  String8 s1 = str8_from_cstr("Hello, World!");
  String8 s2 = str8((U8 *)"Hello", 5);
  
  // Compare
  if (str8_equal(s1, s2)) {
    printf("Equal\n");
  }
  
  // Find substring
  U64 pos = str8_find_substring(s1, str8_from_cstr("World"));
  printf("Found at position: %llu\n", pos);
  
  // Slice
  String8 slice = str8_slice(s1, 0, 5); // "Hello"
  printf("Slice: %.*s\n", (int)slice.size, (char *)slice.str);
  
  // Build strings dynamically
  String8Builder builder = str8_builder_create(64);
  str8_builder_append(&builder, str8_from_cstr("Hello"));
  str8_builder_append(&builder, str8_from_cstr(" "));
  str8_builder_append(&builder, str8_from_cstr("World"));
  String8 result = str8_builder_get(&builder);
  printf("%.*s\n", (int)result.size, (char *)result.str);
  str8_builder_free(&builder);
  
  return 0;
}
```

**Practice Exercise 2.2: Implement String Functions**

Implement these functions:
```c
// Return true if string starts with prefix
B32 str8_starts_with(String8 s, String8 prefix);

// Return true if string ends with suffix
B32 str8_ends_with(String8 s, String8 suffix);

// Replace all occurrences of 'from' with 'to'
String8 str8_replace(Arena *arena, String8 s, String8 from, String8 to);
```

---

## Part 3: Collections & Data Structures

### Module 3.1: Linked Lists (The Foundation)

**Objective**: Master linked lists—the universal building block.

**Why Linked Lists?**
- Dynamic size without reallocation
- O(1) insertion/deletion (with pointer)
- Build queues, stacks, deques easily
- The foundation for many advanced structures

**Implementation**:

```c
// FILE: list.h
#ifndef LIST_H
#define LIST_H

#include <stdint.h>

typedef uint64_t U64;
typedef int B32;

// Singly-linked list node
typedef struct SLLNode {
  struct SLLNode *next;
  int value; // payload
} SLLNode;

SLLNode *sll_push(SLLNode *head, int value);
SLLNode *sll_pop(SLLNode *head, int *out);
void sll_print(SLLNode *head);

// Doubly-linked list node
typedef struct DLLNode {
  struct DLLNode *prev;
  struct DLLNode *next;
  int value;
} DLLNode;

void dll_insert(DLLNode *pos, int value);
void dll_remove(DLLNode *node);
void dll_print(DLLNode *head, DLLNode *tail);

#endif // LIST_H

// FILE: list.c
#include "list.h"
#include <stdlib.h>
#include <stdio.h>

// SINGLY-LINKED: Stack (LIFO)
SLLNode *sll_push(SLLNode *head, int value) {
  SLLNode *node = (SLLNode *)malloc(sizeof(SLLNode));
  node->value = value;
  node->next = head;
  return node; // new head
}

SLLNode *sll_pop(SLLNode *head, int *out) {
  if (!head) return head;
  *out = head->value;
  SLLNode *new_head = head->next;
  free(head);
  return new_head;
}

void sll_print(SLLNode *head) {
  printf("Stack: ");
  for (SLLNode *n = head; n; n = n->next) {
    printf("%d -> ", n->value);
  }
  printf("NULL\n");
}

// DOUBLY-LINKED: Flexible insertion/removal
void dll_insert(DLLNode *pos, int value) {
  DLLNode *node = (DLLNode *)malloc(sizeof(DLLNode));
  node->value = value;
  node->prev = pos->prev;
  node->next = pos;
  if (pos->prev) pos->prev->next = node;
  pos->prev = node;
}

void dll_remove(DLLNode *node) {
  if (node->prev) node->prev->next = node->next;
  if (node->next) node->next->prev = node->prev;
  free(node);
}

void dll_print(DLLNode *head, DLLNode *tail) {
  printf("DLL: ");
  for (DLLNode *n = head; n; n = n->next) {
    printf("%d <-> ", n->value);
  }
  printf("NULL\n");
}

// FILE: main.c
#include "list.h"

int main(void) {
  // Stack example
  SLLNode *stack = 0;
  stack = sll_push(stack, 10);
  stack = sll_push(stack, 20);
  stack = sll_push(stack, 30);
  sll_print(stack); // 30 -> 20 -> 10 -> NULL
  
  int val;
  stack = sll_pop(stack, &val);
  printf("Popped: %d\n", val); // 30
  
  return 0;
}
```

**Key Patterns**:
- Singly-linked: Fast push, good for stacks
- Doubly-linked: Flexible insertion/removal
- Sentinel nodes: Simplify edge cases

**Practice Exercise 3.1: Implement a Queue**

A queue is FIFO (First-In, First-Out). Implement these:

```c
typedef struct {
  SLLNode *head;
  SLLNode *tail;
} Queue;

Queue queue_create(void);
void queue_enqueue(Queue *q, int value);
int queue_dequeue(Queue *q);
```

---

### Module 3.2: Hash Tables (Fast Lookup)

**Objective**: Build a hash table for O(1) lookups.

**Key Idea**: Hash key → find bucket → search bucket

```c
// FILE: hashtable.h
#ifndef HASHTABLE_H
#define HASHTABLE_H

#include <stdint.h>

typedef uint64_t U64;
typedef uint32_t U32;

// Entry in hash table
typedef struct {
  U32 hash;
  char *key;
  int value;
  int occupied; // 1 if slot is in use
} HashEntry;

// Hash table
typedef struct {
  HashEntry *entries;
  U64 capacity;
  U64 count;
} HashTable;

HashTable ht_create(U64 capacity);
void ht_insert(HashTable *ht, char *key, int value);
int ht_lookup(HashTable *ht, char *key);
void ht_free(HashTable *ht);

#endif // HASHTABLE_H

// FILE: hashtable.c
#include "hashtable.h"
#include <stdlib.h>
#include <string.h>

// Simple hash function
static U32 hash_string(char *str) {
  U32 hash = 5381;
  for (char c = *str; c; c = *(++str)) {
    hash = ((hash << 5) + hash) ^ c; // hash*33^c
  }
  return hash;
}

HashTable ht_create(U64 capacity) {
  HashTable ht = {0};
  ht.entries = (HashEntry *)calloc(capacity, sizeof(HashEntry));
  ht.capacity = capacity;
  ht.count = 0;
  return ht;
}

void ht_insert(HashTable *ht, char *key, int value) {
  U32 hash = hash_string(key);
  U64 index = hash % ht->capacity;
  
  // Linear probing: find empty slot
  while (ht->entries[index].occupied) {
    index = (index + 1) % ht->capacity;
  }
  
  ht->entries[index].hash = hash;
  ht->entries[index].key = (char *)malloc(strlen(key) + 1);
  strcpy(ht->entries[index].key, key);
  ht->entries[index].value = value;
  ht->entries[index].occupied = 1;
  ht->count++;
}

int ht_lookup(HashTable *ht, char *key) {
  U32 hash = hash_string(key);
  U64 index = hash % ht->capacity;
  
  // Linear probing: find matching entry
  while (ht->entries[index].occupied) {
    if (strcmp(ht->entries[index].key, key) == 0) {
      return ht->entries[index].value;
    }
    index = (index + 1) % ht->capacity;
  }
  
  return -1; // not found
}

void ht_free(HashTable *ht) {
  for (U64 i = 0; i < ht->capacity; i++) {
    if (ht->entries[i].occupied) {
      free(ht->entries[i].key);
    }
  }
  free(ht->entries);
}

// FILE: main.c
#include "hashtable.h"
#include <stdio.h>

int main(void) {
  HashTable ht = ht_create(16);
  
  ht_insert(&ht, "apple", 5);
  ht_insert(&ht, "banana", 3);
  ht_insert(&ht, "orange", 7);
  
  printf("apple: %d\n", ht_lookup(&ht, "apple")); // 5
  printf("banana: %d\n", ht_lookup(&ht, "banana")); // 3
  printf("grape: %d\n", ht_lookup(&ht, "grape")); // -1 (not found)
  
  ht_free(&ht);
  return 0;
}
```

**Key Concepts**:
- Hash function: key → number
- Linear probing: find slot if occupied
- Collision handling: probe forward
- Load factor: count/capacity (resize if > 0.75)

**Practice Exercise 3.2: Add Resize**

The hash table will fail if too full. Implement:

```c
void ht_resize(HashTable *ht, U64 new_capacity);
```

When load factor exceeds 0.75, double the capacity and rehash all entries.

---

## Part 4: Asynchronous Processing & Caching

### Module 4.1: Work Queues (Producer-Consumer Pattern)

**Objective**: Process work asynchronously without blocking the main thread.

**Key Idea**:
- Main thread queues work
- Worker thread processes asynchronously
- Main thread checks for results

**Implementation**:

```c
// FILE: queue.h
#ifndef QUEUE_H
#define QUEUE_H

#include <stdint.h>

typedef uint64_t U64;
typedef uint32_t U32;

// Work item
typedef struct {
  U32 id;
  char *data;
  U64 data_size;
} WorkItem;

// Work queue
typedef struct {
  WorkItem *items;
  U64 capacity;
  U64 write_idx; // where to write next
  U64 read_idx;  // where to read next
} WorkQueue;

WorkQueue wq_create(U64 capacity);
void wq_push(WorkQueue *wq, WorkItem item); // Add work
WorkItem wq_pop(WorkQueue *wq);              // Get work
U64 wq_count(WorkQueue *wq);                 // How much work?
void wq_free(WorkQueue *wq);

#endif // QUEUE_H

// FILE: queue.c
#include "queue.h"
#include <stdlib.h>
#include <string.h>

WorkQueue wq_create(U64 capacity) {
  WorkQueue wq = {0};
  wq.items = (WorkItem *)malloc(sizeof(WorkItem) * capacity);
  wq.capacity = capacity;
  wq.write_idx = 0;
  wq.read_idx = 0;
  return wq;
}

void wq_push(WorkQueue *wq, WorkItem item) {
  // Circular buffer: wrap around if needed
  U64 next = (wq->write_idx + 1) % wq->capacity;
  if (next == wq->read_idx) {
    // Queue full - resize
    U64 new_capacity = wq->capacity * 2;
    WorkItem *new_items = (WorkItem *)malloc(sizeof(WorkItem) * new_capacity);
    
    // Copy items in order
    for (U64 i = 0; i < wq_count(wq); i++) {
      new_items[i] = wq->items[(wq->read_idx + i) % wq->capacity];
    }
    
    free(wq->items);
    wq->items = new_items;
    wq->capacity = new_capacity;
    wq->read_idx = 0;
    wq->write_idx = wq_count(wq);
  }
  
  wq->items[wq->write_idx] = item;
  wq->write_idx = (wq->write_idx + 1) % wq->capacity;
}

WorkItem wq_pop(WorkQueue *wq) {
  WorkItem item = {0};
  if (wq->read_idx != wq->write_idx) {
    item = wq->items[wq->read_idx];
    wq->read_idx = (wq->read_idx + 1) % wq->capacity;
  }
  return item;
}

U64 wq_count(WorkQueue *wq) {
  if (wq->write_idx >= wq->read_idx) {
    return wq->write_idx - wq->read_idx;
  } else {
    return wq->capacity - wq->read_idx + wq->write_idx;
  }
}

void wq_free(WorkQueue *wq) {
  free(wq->items);
}

// FILE: main.c (Simple async processing)
#include "queue.h"
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

WorkQueue input_queue;
WorkQueue output_queue;

// Simulate work (reads from input_queue, writes to output_queue)
void *worker_thread(void *arg) {
  while (1) {
    WorkItem item = wq_pop(&input_queue);
    if (item.id == 0) break; // sentinel: exit
    
    // Simulate processing
    printf("Worker processing item %u\n", item.id);
    usleep(100000); // 100ms
    
    // Queue result
    WorkItem result = item;
    wq_push(&output_queue, result);
  }
  return 0;
}

int main(void) {
  input_queue = wq_create(10);
  output_queue = wq_create(10);
  
  // Start worker thread
  pthread_t thread;
  pthread_create(&thread, 0, worker_thread, 0);
  
  // Main thread: queue work
  for (int i = 1; i <= 5; i++) {
    WorkItem item = {.id = i, .data = "test", .data_size = 4};
    wq_push(&input_queue, item);
    printf("Queued item %d\n", i);
  }
  
  // Check for results
  usleep(500000); // 500ms
  while (wq_count(&output_queue) > 0) {
    WorkItem result = wq_pop(&output_queue);
    printf("Got result for item %u\n", result.id);
  }
  
  // Stop worker
  WorkItem sentinel = {0};
  wq_push(&input_queue, sentinel);
  pthread_join(thread, 0);
  
  wq_free(&input_queue);
  wq_free(&output_queue);
  
  return 0;
}
```

**Key Pattern**: 
- Producer queues work items
- Consumer processes asynchronously
- Main thread checks output queue periodically

---

### Module 4.2: Simple Caching (Content-Addressed Storage)

**Objective**: Cache expensive-to-compute results.

**Key Idea**: Hash(data) → location of cached result

```c
// FILE: cache.h
#ifndef CACHE_H
#define CACHE_H

#include <stdint.h>

typedef uint64_t U64;
typedef uint32_t U32;

// 128-bit hash (like MD5)
typedef struct {
  U64 a;
  U64 b;
} Hash128;

// Cached entry
typedef struct {
  Hash128 hash;
  void *data;
  U64 data_size;
  U64 access_count; // how many times accessed
} CacheEntry;

// Simple cache
typedef struct {
  CacheEntry *entries;
  U64 capacity;
  U64 count;
} Cache;

Cache cache_create(U64 capacity);
void cache_store(Cache *c, Hash128 hash, void *data, U64 size);
void *cache_lookup(Cache *c, Hash128 hash, U64 *out_size);
void cache_evict_lru(Cache *c); // evict least recently used
void cache_free(Cache *c);

// Hash data
Hash128 hash_data(void *data, U64 size);

#endif // CACHE_H

// FILE: cache.c
#include "cache.h"
#include <stdlib.h>
#include <string.h>

// Simple hash function for data
static U64 fnv1a(void *data, U64 size) {
  U64 hash = 14695981039346656037ULL;
  for (U64 i = 0; i < size; i++) {
    hash ^= ((U8 *)data)[i];
    hash *= 1099511628211ULL;
  }
  return hash;
}

Hash128 hash_data(void *data, U64 size) {
  U64 h1 = fnv1a(data, size);
  U64 h2 = fnv1a(&h1, sizeof(h1));
  return (Hash128){.a = h1, .b = h2};
}

Cache cache_create(U64 capacity) {
  Cache c = {0};
  c.entries = (CacheEntry *)calloc(capacity, sizeof(CacheEntry));
  c.capacity = capacity;
  c.count = 0;
  return c;
}

void cache_store(Cache *c, Hash128 hash, void *data, U64 size) {
  if (c->count >= c->capacity) {
    cache_evict_lru(c);
  }
  
  // Find free slot
  for (U64 i = 0; i < c->capacity; i++) {
    if (c->entries[i].data == 0) {
      c->entries[i].hash = hash;
      c->entries[i].data = malloc(size);
      memcpy(c->entries[i].data, data, size);
      c->entries[i].data_size = size;
      c->entries[i].access_count = 0;
      c->count++;
      return;
    }
  }
}

void *cache_lookup(Cache *c, Hash128 hash, U64 *out_size) {
  for (U64 i = 0; i < c->capacity; i++) {
    if (c->entries[i].data && 
        c->entries[i].hash.a == hash.a &&
        c->entries[i].hash.b == hash.b) {
      c->entries[i].access_count++;
      if (out_size) *out_size = c->entries[i].data_size;
      return c->entries[i].data;
    }
  }
  return 0; // cache miss
}

void cache_evict_lru(Cache *c) {
  // Find least recently used (lowest access_count)
  U64 lru_idx = 0;
  U64 min_count = (U64)-1;
  
  for (U64 i = 0; i < c->capacity; i++) {
    if (c->entries[i].data && c->entries[i].access_count < min_count) {
      lru_idx = i;
      min_count = c->entries[i].access_count;
    }
  }
  
  free(c->entries[lru_idx].data);
  c->entries[lru_idx].data = 0;
  c->count--;
}

void cache_free(Cache *c) {
  for (U64 i = 0; i < c->capacity; i++) {
    if (c->entries[i].data) {
      free(c->entries[i].data);
    }
  }
  free(c->entries);
}

// FILE: main.c
#include "cache.h"
#include <stdio.h>

int main(void) {
  Cache c = cache_create(10);
  
  // Store some data
  char data1[] = "Hello, World!";
  Hash128 h1 = hash_data(data1, sizeof(data1));
  cache_store(&c, h1, data1, sizeof(data1));
  printf("Stored: %s\n", data1);
  
  // Retrieve it
  U64 size;
  char *result = (char *)cache_lookup(&c, h1, &size);
  if (result) {
    printf("Retrieved from cache: %.*s\n", (int)size, result);
  }
  
  // Miss on different data
  char data2[] = "Different";
  Hash128 h2 = hash_data(data2, sizeof(data2));
  if (!cache_lookup(&c, h2, 0)) {
    printf("Cache miss (expected)\n");
  }
  
  cache_free(&c);
  return 0;
}
```

**Key Concepts**:
- Content-addressed: data hash → cache location
- Access tracking: LRU eviction
- Automatic cleanup: when cache is full, remove least-used

**Practice Exercise 4.2: Add Expiration Time**

Extend the cache to expire entries after a timeout:

```c
typedef struct {
  // ... existing fields ...
  U64 timestamp; // when was this cached?
} CacheEntry;

// Evict entries older than max_age_ms
void cache_evict_expired(Cache *c, U64 max_age_ms);
```

---

## Part 5: Parsing & Serialization

### Module 5.1: Binary File Parsing

**Objective**: Learn how to parse binary file formats safely and efficiently.

**Key Principles**:
1. Use struct-based parsing (not byte-by-byte)
2. Validate sizes and offsets
3. Handle endianness carefully
4. Design for extensibility

**Implementation**:

```c
// FILE: parser.h
#ifndef PARSER_H
#define PARSER_H

#include <stdint.h>

typedef uint8_t U8;
typedef uint32_t U32;
typedef uint64_t U64;

// Stream abstraction: read from data buffer
typedef struct {
  U8 *data;
  U64 size;
  U64 offset;
} Stream;

Stream stream_create(U8 *data, U64 size);
B32 stream_read(Stream *s, void *out, U64 amount);
B32 stream_seek(Stream *s, U64 offset);
U64 stream_pos(Stream *s);
B32 stream_has(Stream *s, U64 amount);

// Binary file format example: SimpleFile
// Format: [Header][Data...]
typedef struct {
  U32 magic;      // 0x12345678
  U32 version;    // 1
  U32 data_size;
} SimpleFileHeader;

typedef struct {
  SimpleFileHeader header;
  U8 *data;
  U64 data_size;
} SimpleFile;

SimpleFile parse_simple_file(U8 *file_data, U64 file_size);

#endif // PARSER_H

// FILE: parser.c
#include "parser.h"
#include <string.h>

Stream stream_create(U8 *data, U64 size) {
  return (Stream){.data = data, .size = size, .offset = 0};
}

B32 stream_read(Stream *s, void *out, U64 amount) {
  if (!stream_has(s, amount)) return 0;
  memcpy(out, s->data + s->offset, amount);
  s->offset += amount;
  return 1;
}

B32 stream_seek(Stream *s, U64 offset) {
  if (offset > s->size) return 0;
  s->offset = offset;
  return 1;
}

U64 stream_pos(Stream *s) {
  return s->offset;
}

B32 stream_has(Stream *s, U64 amount) {
  return s->offset + amount <= s->size;
}

SimpleFile parse_simple_file(U8 *file_data, U64 file_size) {
  SimpleFile sf = {0};
  Stream s = stream_create(file_data, file_size);
  
  // Read and validate header
  if (!stream_read(&s, &sf.header, sizeof(SimpleFileHeader))) {
    return sf; // Failed to read header
  }
  
  if (sf.header.magic != 0x12345678) {
    return sf; // Invalid magic
  }
  
  if (sf.header.version != 1) {
    return sf; // Unsupported version
  }
  
  // Bounds check
  if (!stream_has(&s, sf.header.data_size)) {
    return sf; // Not enough data
  }
  
  // Read data section
  sf.data = s.data + s.offset;
  sf.data_size = sf.header.data_size;
  
  return sf;
}

// FILE: main.c
#include "parser.h"
#include <stdio.h>
#include <string.h>

int main(void) {
  // Create a test file in memory
  U8 buffer[256];
  SimpleFileHeader header = {
    .magic = 0x12345678,
    .version = 1,
    .data_size = 10
  };
  
  // Write header and data
  memcpy(buffer, &header, sizeof(header));
  memcpy(buffer + sizeof(header), "Hello!!!!!", 10);
  
  // Parse it
  SimpleFile sf = parse_simple_file(buffer, sizeof(buffer));
  
  if (sf.data) {
    printf("Parsed file: magic=0x%x, version=%u, data_size=%u\n",
           sf.header.magic, sf.header.version, sf.header.data_size);
    printf("Data: %.*s\n", (int)sf.data_size, (char *)sf.data);
  } else {
    printf("Failed to parse file\n");
  }
  
  return 0;
}
```

**Key Patterns**:
- Stream abstraction: read/seek/validate
- Header validation: magic + version
- Bounds checking: always verify sizes
- Lazy parsing: don't parse everything upfront

**Practice Exercise 5.1: Extend the Parser**

Add a section-based format:

```c
// New format: [Header][Section1][Section2]...
// Each section: [kind][size][data]

typedef struct {
  U32 kind;
  U32 size;
} SectionHeader;

// Parse and return array of sections
typedef struct {
  U32 kind;
  U8 *data;
  U64 size;
} Section;

Section *parse_sections(Stream *s, U64 *out_count);
```

---

## Part 6: Compiler Patterns (Lexing, Parsing, Evaluating)

### Module 6.1: Lexer (Text → Tokens)

**Objective**: Learn how to tokenize text input.

**Implementation**:

```c
// FILE: lexer.h
#ifndef LEXER_H
#define LEXER_H

#include <stdint.h>

typedef uint8_t U8;
typedef uint32_t U32;

// Token types
typedef enum {
  TokenKind_EOF,
  TokenKind_Identifier,
  TokenKind_Number,
  TokenKind_Plus,
  TokenKind_Minus,
  TokenKind_Star,
  TokenKind_Slash,
  TokenKind_LParen,
  TokenKind_RParen,
  TokenKind_COUNT
} TokenKind;

// Token
typedef struct {
  TokenKind kind;
  U8 *str;
  U32 size;
  U32 line;
  U32 col;
} Token;

// Lexer state
typedef struct {
  U8 *input;
  U32 pos;
  U32 line;
  U32 col;
} Lexer;

Lexer lexer_create(U8 *input);
Token lexer_next(Lexer *lex);

#endif // LEXER_H

// FILE: lexer.c
#include "lexer.h"
#include <ctype.h>

Lexer lexer_create(U8 *input) {
  return (Lexer){.input = input, .pos = 0, .line = 1, .col = 1};
}

static void skip_whitespace(Lexer *lex) {
  while (isspace(lex->input[lex->pos])) {
    if (lex->input[lex->pos] == '\n') {
      lex->line++;
      lex->col = 1;
    } else {
      lex->col++;
    }
    lex->pos++;
  }
}

Token lexer_next(Lexer *lex) {
  Token token = {0};
  token.line = lex->line;
  token.col = lex->col;
  
  skip_whitespace(lex);
  
  U8 c = lex->input[lex->pos];
  
  // EOF
  if (c == 0) {
    token.kind = TokenKind_EOF;
    return token;
  }
  
  // Identifiers and keywords
  if (isalpha(c) || c == '_') {
    token.kind = TokenKind_Identifier;
    token.str = lex->input + lex->pos;
    while (isalnum(lex->input[lex->pos]) || lex->input[lex->pos] == '_') {
      lex->col++;
      lex->pos++;
    }
    token.size = lex->input + lex->pos - token.str;
    return token;
  }
  
  // Numbers
  if (isdigit(c)) {
    token.kind = TokenKind_Number;
    token.str = lex->input + lex->pos;
    while (isdigit(lex->input[lex->pos])) {
      lex->col++;
      lex->pos++;
    }
    token.size = lex->input + lex->pos - token.str;
    return token;
  }
  
  // Operators and punctuation
  token.str = lex->input + lex->pos;
  token.size = 1;
  
  switch (c) {
    case '+': token.kind = TokenKind_Plus; break;
    case '-': token.kind = TokenKind_Minus; break;
    case '*': token.kind = TokenKind_Star; break;
    case '/': token.kind = TokenKind_Slash; break;
    case '(': token.kind = TokenKind_LParen; break;
    case ')': token.kind = TokenKind_RParen; break;
    default: return token; // Unknown
  }
  
  lex->col++;
  lex->pos++;
  return token;
}

// FILE: main.c (Test lexer)
#include "lexer.h"
#include <stdio.h>

int main(void) {
  U8 *input = (U8 *)"2 + 3 * 4";
  Lexer lex = lexer_create(input);
  
  Token token;
  while ((token = lexer_next(&lex)).kind != TokenKind_EOF) {
    printf("Token: kind=%d, '%.*s' at %u:%u\n",
           token.kind, token.size, (char *)token.str, 
           token.line, token.col);
  }
  
  return 0;
}
```

**Key Concepts**:
- State machine for recognizing tokens
- Track position for error reporting
- Handle identifiers, numbers, operators
- Skip whitespace

**Practice Exercise 6.1: Add More Token Types**

Add support for:
- Strings (quoted with `"`)
- Comments (starting with `//`)
- Comparison operators (`==`, `!=`, `<`, `>`)

---

### Module 6.2: Parser (Tokens → AST)

**Objective**: Build an Abstract Syntax Tree from tokens.

```c
// FILE: parser.h (Recursive descent parser)
#ifndef PARSER_H
#define PARSER_H

#include "lexer.h"

// AST Node kinds
typedef enum {
  ASTKind_Number,
  ASTKind_BinOp,
  ASTKind_Identifier,
} ASTKind;

// AST Node
typedef struct ASTNode {
  ASTKind kind;
  union {
    U32 number;
    struct {
      struct ASTNode *left;
      TokenKind op; // +, -, *, /
      struct ASTNode *right;
    } binop;
    struct {
      U8 *name;
      U32 name_size;
    } identifier;
  } data;
} ASTNode;

typedef struct {
  Lexer lex;
  Token current;
} Parser;

Parser parser_create(U8 *input);
ASTNode *parser_parse(Parser *p);

#endif // PARSER_H

// FILE: parser.c (Simplified version)
#include "parser.h"
#include <stdlib.h>

Parser parser_create(U8 *input) {
  Parser p = {.lex = lexer_create(input)};
  p.current = lexer_next(&p.lex);
  return p;
}

static void parser_advance(Parser *p) {
  p->current = lexer_next(&p->lex);
}

// Forward declarations for recursive descent
static ASTNode *parse_expr(Parser *p);
static ASTNode *parse_term(Parser *p);
static ASTNode *parse_factor(Parser *p);

// parse_factor: number | identifier | '(' expr ')'
static ASTNode *parse_factor(Parser *p) {
  ASTNode *node = (ASTNode *)malloc(sizeof(ASTNode));
  
  if (p->current.kind == TokenKind_Number) {
    node->kind = ASTKind_Number;
    node->data.number = 0;
    for (U32 i = 0; i < p->current.size; i++) {
      node->data.number = node->data.number * 10 + 
                          (p->current.str[i] - '0');
    }
    parser_advance(p);
  } else if (p->current.kind == TokenKind_Identifier) {
    node->kind = ASTKind_Identifier;
    node->data.identifier.name = p->current.str;
    node->data.identifier.name_size = p->current.size;
    parser_advance(p);
  } else if (p->current.kind == TokenKind_LParen) {
    parser_advance(p);
    ASTNode *expr = parse_expr(p);
    parser_advance(p); // consume ')'
    return expr;
  }
  
  return node;
}

// parse_term: factor ((* | /) factor)*
static ASTNode *parse_term(Parser *p) {
  ASTNode *node = parse_factor(p);
  
  while (p->current.kind == TokenKind_Star || 
         p->current.kind == TokenKind_Slash) {
    ASTNode *binop = (ASTNode *)malloc(sizeof(ASTNode));
    binop->kind = ASTKind_BinOp;
    binop->data.binop.left = node;
    binop->data.binop.op = p->current.kind;
    parser_advance(p);
    binop->data.binop.right = parse_factor(p);
    node = binop;
  }
  
  return node;
}

// parse_expr: term ((+ | -) term)*
static ASTNode *parse_expr(Parser *p) {
  ASTNode *node = parse_term(p);
  
  while (p->current.kind == TokenKind_Plus || 
         p->current.kind == TokenKind_Minus) {
    ASTNode *binop = (ASTNode *)malloc(sizeof(ASTNode));
    binop->kind = ASTKind_BinOp;
    binop->data.binop.left = node;
    binop->data.binop.op = p->current.kind;
    parser_advance(p);
    binop->data.binop.right = parse_term(p);
    node = binop;
  }
  
  return node;
}

ASTNode *parser_parse(Parser *p) {
  return parse_expr(p);
}

// FILE: main.c (Test parser)
#include "parser.h"
#include <stdio.h>

void print_ast(ASTNode *node, int indent) {
  for (int i = 0; i < indent; i++) printf("  ");
  
  if (node->kind == ASTKind_Number) {
    printf("Number(%u)\n", node->data.number);
  } else if (node->kind == ASTKind_Identifier) {
    printf("Identifier(%.*s)\n", node->data.identifier.name_size,
           (char *)node->data.identifier.name);
  } else if (node->kind == ASTKind_BinOp) {
    printf("BinOp(%c)\n", node->data.binop.op == TokenKind_Plus ? '+' : '*');
    print_ast(node->data.binop.left, indent + 1);
    print_ast(node->data.binop.right, indent + 1);
  }
}

int main(void) {
  U8 *input = (U8 *)"2 + 3 * 4";
  Parser p = parser_create(input);
  ASTNode *ast = parser_parse(&p);
  
  printf("AST:\n");
  print_ast(ast, 0);
  
  return 0;
}
```

**Key Concepts**:
- Recursive descent parsing
- Operator precedence (multiplication before addition)
- Left-associativity handled by loop structure

**Practice Exercise 6.2: Add More Operations**

Support assignment: `x = 5`

```c
// Add to ASTKind
ASTKind_Assignment,

// parse_stmt: expr | expr '=' expr
```

---

## Part 7: Advanced Concepts

### Module 7.1: Type Systems & Abstraction

**Objective**: Learn how to design flexible type systems.

**Pattern: Discriminated Unions**

```c
// Instead of multiple types, use a kind field + union
typedef enum {
  ValueKind_Integer,
  ValueKind_Float,
  ValueKind_String,
} ValueKind;

typedef struct {
  ValueKind kind;
  union {
    S64 i;
    F64 f;
    String8 s;
  } data;
} Value;

// Safely access values
S64 value_as_integer(Value v) {
  if (v.kind != ValueKind_Integer) return 0;
  return v.data.i;
}
```

**Pattern: Abstract Interfaces**

```c
// Define abstract interface
typedef struct {
  void (*init)(void);
  void (*process)(void *data);
  void (*cleanup)(void);
} Algorithm;

// Multiple implementations
Algorithm algorithm_fast = {
  .init = fast_init,
  .process = fast_process,
  .cleanup = fast_cleanup,
};

Algorithm algorithm_accurate = {
  .init = accurate_init,
  .process = accurate_process,
  .cleanup = accurate_cleanup,
};

// Use polymorphically
void run_algorithm(Algorithm *algo, void *data) {
  algo->init();
  algo->process(data);
  algo->cleanup();
}
```

---

### Module 7.2: Optimization Patterns

**Objective**: Learn performance-critical patterns.

**Pattern 1: Pre-allocation Instead of Reallocation**

```c
// BAD: Reallocates on each growth
void *data = malloc(1);
for (...) {
  data = realloc(data, new_size); // expensive!
}

// GOOD: Allocate with headroom
void *data = malloc(capacity);
used = 0;
for (...) {
  if (used >= capacity) {
    capacity *= 2; // exponential growth
    data = realloc(data, capacity);
  }
  // use data[used++]
}
```

**Pattern 2: Batch Operations**

```c
// BAD: Process one at a time
for (int i = 0; i < 1000; i++) {
  process_item(items[i]); // cache miss on each call
}

// GOOD: Process in batches
for (int batch = 0; batch < 1000; batch += 64) {
  for (int i = batch; i < batch + 64; i++) {
    process_item(items[i]); // warm cache
  }
}
```

**Pattern 3: Lazy Evaluation**

```c
// BAD: Compute everything upfront
struct Report {
  char *title;
  char *long_description;
  char *detailed_analysis; // expensive!
};

// GOOD: Compute on demand
struct Report {
  char *title;
  Cache cache; // cache for expensive computations
  void (*compute_analysis)(Cache *);
};

char *get_analysis(Report *r) {
  if (!cache_lookup(&r->cache)) {
    r->compute_analysis(&r->cache);
  }
  return cache_get(&r->cache);
}
```

---

## Part 8: Practical Mastery

### Module 8.1: Building a Simple Project

**Objective**: Apply all patterns to build a complete project.

**Project: Simple Data Processor**

Build a command-line tool that:
1. Reads a binary file format
2. Parses data structures
3. Caches computation results
4. Outputs formatted results

**Complete Example**:

```c
// FILE: processor.h
#ifndef PROCESSOR_H
#define PROCESSOR_H

#include <stdint.h>

typedef uint8_t U8;
typedef uint32_t U32;
typedef uint64_t U64;
typedef int B32;

// Data structures
typedef struct {
  U32 id;
  U64 value;
} Record;

typedef struct {
  Record *records;
  U64 count;
} Dataset;

// Processing
typedef struct {
  U64 sum;
  U64 avg;
  U64 min;
  U64 max;
} Stats;

// Pipeline
Dataset load_file(char *filename);
Stats compute_stats(Dataset *data);
void print_stats(Stats s);

#endif // PROCESSOR_H

// FILE: processor.c
#include "processor.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

Dataset load_file(char *filename) {
  Dataset ds = {0};
  FILE *f = fopen(filename, "rb");
  if (!f) return ds;
  
  // Read count
  fread(&ds.count, sizeof(U64), 1, f);
  ds.records = (Record *)malloc(sizeof(Record) * ds.count);
  
  // Read records
  fread(ds.records, sizeof(Record), ds.count, f);
  fclose(f);
  return ds;
}

Stats compute_stats(Dataset *data) {
  Stats s = {0};
  if (data->count == 0) return s;
  
  s.min = (U64)-1;
  s.max = 0;
  
  for (U64 i = 0; i < data->count; i++) {
    s.sum += data->records[i].value;
    if (data->records[i].value < s.min) s.min = data->records[i].value;
    if (data->records[i].value > s.max) s.max = data->records[i].value;
  }
  
  s.avg = s.sum / data->count;
  return s;
}

void print_stats(Stats s) {
  printf("Statistics:\n");
  printf("  Sum: %llu\n", s.sum);
  printf("  Avg: %llu\n", s.avg);
  printf("  Min: %llu\n", s.min);
  printf("  Max: %llu\n", s.max);
}

// FILE: main.c
#include "processor.h"
#include <stdio.h>

int main(int argc, char **argv) {
  if (argc < 2) {
    printf("Usage: processor <file.bin>\n");
    return 1;
  }
  
  Dataset data = load_file(argv[1]);
  if (data.count == 0) {
    printf("Failed to load file\n");
    return 1;
  }
  
  Stats stats = compute_stats(&data);
  print_stats(stats);
  
  free(data.records);
  return 0;
}
```

**Your Tasks**:
1. Extend with arena-based allocation
2. Add cache for statistics (don't recompute)
3. Support multiple input formats
4. Add async file loading with work queue

---

### Module 8.2: Real-World Problem Solving

**Exercise 1: Symbol Table**

Build a symbol table for a compiler:

```c
// Requirements:
// - Store symbol name → value mapping
// - Support nested scopes (function scope, block scope)
// - Look up symbols (search current scope, then parent)
// - Multiple implementations: hash table, binary search

typedef struct Scope {
  struct Scope *parent;
  HashTable symbols;
} Scope;

Scope *scope_create(Scope *parent);
void scope_insert(Scope *s, char *name, U64 value);
B32 scope_lookup(Scope *s, char *name, U64 *out);
void scope_destroy(Scope *s);
```

**Exercise 2: Event Dispatcher**

Build a publish-subscribe system:

```c
// Requirements:
// - Register handlers for event types
// - Dispatch events to all registered handlers
// - Handler can unsubscribe
// - No memory leaks

typedef void (*EventHandler)(void *event, void *user_data);

typedef struct {
  int event_type;
  EventHandler handler;
  void *user_data;
} Subscription;

typedef struct {
  Subscription *subs;
  U64 count;
} Dispatcher;

void dispatcher_subscribe(Dispatcher *d, int event_type, EventHandler h, void *ud);
void dispatcher_dispatch(Dispatcher *d, int event_type, void *event);
void dispatcher_unsubscribe(Dispatcher *d, int event_type, EventHandler h);
```

**Exercise 3: Configuration Parser**

Parse INI-style config files:

```c
// [section]
// key=value
// key2=value2
//
// [another]
// ...

typedef struct {
  char *key;
  char *value;
} ConfigEntry;

typedef struct {
  char *section;
  ConfigEntry *entries;
  U64 count;
} ConfigSection;

typedef struct {
  ConfigSection *sections;
  U64 count;
} Config;

Config parse_config(char *filename);
char *config_get(Config *c, char *section, char *key);
```

---

## Part 9: Study Checklist

### Core Patterns You Must Know
- [ ] Arena allocation
- [ ] String8 pattern
- [ ] Linked lists & hash tables
- [ ] Work queues
- [ ] Content caching with hashing
- [ ] Lexer/Parser/Evaluator
- [ ] Discriminated unions
- [ ] Abstract interfaces
- [ ] Layered architecture
- [ ] Async processing

### Practice You Must Complete
- [ ] Implement at least 3 data structures
- [ ] Build a parser for some format
- [ ] Create an async system with work queue
- [ ] Implement a cache system
- [ ] Write a lexer/parser combo
- [ ] Build a complete small project (processor)

### Code Reading You Must Do
- [ ] Read 2000+ lines of well-written C
- [ ] Trace 3 features end-to-end
- [ ] Understand 2 different design patterns
- [ ] Study 5 different error handling approaches
- [ ] Analyze performance characteristics

---

## Final Checklist: Are You Ready?

You're ready when you can:

- ✅ Explain layered architecture and why it matters
- ✅ Choose between data structures (list vs hash table)
- ✅ Design an async system using work queues
- ✅ Implement caching for expensive operations
- ✅ Build a parser for binary data
- ✅ Write memory-safe C (arenas, bounds checks)
- ✅ Apply naming conventions consistently
- ✅ Design clean interfaces that hide complexity
- ✅ Read a large, complex codebase and understand it
- ✅ Implement a feature in the exact style of the original authors

---

## Key Universal Principles

These principles apply to **ANY** large C project:

### 1. **Organize by Layers**
Clear dependencies → easier refactoring, testing, reuse

### 2. **Name Everything**
Consistent, searchable names → easier to understand what code does

### 3. **Use Abstractions**
Hide complexity behind simple interfaces → easier to maintain

### 4. **Make it Fast by Design**
- Arena allocation (not malloc)
- Batch operations (not one-at-a-time)
- Cache results (not recompute)
- Lazy evaluation (not upfront)

### 5. **Be Explicit**
- Validate inputs
- Handle edge cases
- Check bounds
- Clear error reporting

### 6. **Document Intent**
- Section markers group related code
- Comments explain "why", not "what"
- Type names are self-documenting
- Function names tell their purpose

### 7. **Design for Scale**
- Assume 100x bigger dataset tomorrow
- Precompute what you can
- Stream what you can't
- Cache aggressively

---

## Conclusion

This syllabus teaches **universal patterns for building large systems in C**. The RADDebugger is the reference implementation, but these patterns apply equally to:

- Operating systems & kernels
- Compilers & interpreters
- Game engines
- Database systems
- Embedded systems
- Real-time processing

By mastering these patterns and practices, you'll be able to:
1. **Understand** any large C codebase quickly
2. **Extend** existing systems confidently
3. **Design** new systems with clean architecture
4. **Optimize** for performance and maintainability
5. **Teach** others to write better code

The goal isn't to copy the RADDebugger—it's to internalize the principles that make it excellent, so you can apply them everywhere.

**Happy coding!**
