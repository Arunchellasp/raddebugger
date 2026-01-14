# RADDebugger Codebase: Advanced Learning Syllabus

## Overview

This syllabus is designed to enable deep understanding and mastery of the RADDebugger codebase—a sophisticated, production-grade debugger built by Epic Games. By following this structured learning path, you will internalize the architectural principles, coding conventions, design patterns, and the original author's mindset to confidently implement new features in the exact style and quality of the original code.

The RADDebugger is a native, user-mode, multi-process graphical debugger supporting Windows x64 debugging with PDBs, with plans to expand to Linux with DWARF. It also includes the RAD Debug Info (RDI) format and the RAD Linker—a high-performance linker optimized for massive executables.

---

## Part 1: Foundational Concepts & Philosophy

### Module 1.1: Codebase Organization & Layer Architecture

**Objective**: Understand how the codebase is organized into isolated, composable layers.

**Key Concepts**:
- **Layered Architecture**: The codebase uses a Directed Acyclic Graph (DAG) of dependencies. Layers isolate problems and allow selective inclusion without pulling in unrelated code.
- **Namespace Convention**: Layers use short 1-3 character prefixes (e.g., `RDI_`, `CTRL_`, `E_`, `UI_`). Types use capitalized namespaces; functions and globals use lowercase.
- **Layer Independence**: Some layers (prefixed with `lib_`) are designed to be completely independent and relocatable to other projects (e.g., `lib_rdi`, `lib_rdi_make`, `lib_raddbg_markup`).

**Learning Activities**:
1. Read [README.md](README.md#layer-descriptions) section on "Layer Descriptions" to understand all 50+ layers
2. Create a visual dependency graph:
   - Map base layers that have no dependencies
   - Trace how higher-level layers depend on lower layers
   - Identify entry points (raddbg, radbin, linker)
3. Study the [src/base](src/base/) folder structure—this is the foundation

**Key Files to Study**:
- [README.md](README.md) - Complete layer documentation
- [src/base/base_inc.h](src/base/base_inc.h) - Foundation layer includes
- [src/base/base_core.h](src/base/base_core.h) - Core macros and utilities

**Questions to Answer**:
- What are the 3 foundational layers with zero dependencies?
- How does the dependency DAG prevent circular dependencies?
- Why would you choose to put code in `lib_rdi` vs `rdi`?

---

### Module 1.2: The RAD Debugger Architecture

**Objective**: Understand the high-level system design and how components communicate.

**Key Concepts**:
- **Multi-Threaded Architecture**: The debugger uses a producer-consumer pattern with separate threads for control, engine, and UI
- **Asynchronous Design**: Many operations (debug info loading, file streaming, caching) happen asynchronously
- **Cache-Driven Data Flow**: Information flows through multiple cache layers before reaching the UI
- **Plugin-Like Extensibility**: The `eval` layer acts as a compiler for a custom expression language

**Component Overview**:

| Layer | Purpose | Pattern |
|-------|---------|---------|
| `demon` (`DMN_`) | Low-level OS abstraction for process control | Abstract interface |
| `ctrl` (`CTRL_`) | Asynchronous debugger control system | Producer/consumer |
| `dbg_engine` (`D_`) | Core debugger logic (stepping, breakpoints) | Synchronous orchestration |
| `dbg_info` (`DI_`) | Debug info loading & caching | Async queue processing |
| `eval` (`E_`) | Expression compiler & evaluator | Compiler phases |
| `raddbg` (`RD_`) | Main GUI & command-line interface | Immediate-mode UI |

**Learning Activities**:
1. Create a sequence diagram showing how a breakpoint hit flows through layers:
   - `demon` detects breakpoint hit
   - `ctrl` processes the event
   - `dbg_engine` manages state
   - UI updates display
2. Study how async operations are queued and processed
3. Understand the cache architecture: `content`, `artifact_cache`, `mutable_text`

**Key Files to Study**:
- [src/dbg_engine/dbg_engine_core.h](src/dbg_engine/dbg_engine_core.h) - Core types
- [src/ctrl/ctrl_core.h](src/ctrl/ctrl_core.h) - Control system
- [src/raddbg/raddbg_core.h](src/raddbg/raddbg_core.h) - Main app structure

**Questions to Answer**:
- What is the relationship between `ctrl` and `demon`?
- How does the `eval` layer integrate with the debug engine?
- What role does the cache play in debugger performance?

---

### Module 1.3: Coding Conventions & Style

**Objective**: Learn the distinctive C coding style and naming conventions used throughout.

**Key Conventions**:

1. **Keywords & Linkage**:
   ```c
   internal      // static (file-local scope)
   global        // static (file-local global variable)
   local_persist // static (persistent local variable)
   shared_function // extern + dllexport (exported function)
   ```

2. **Naming Scheme**:
   - Types: `PascalCase` with namespace prefix (e.g., `CTRL_Entity`, `E_Expr`)
   - Functions: `snake_case` with namespace prefix (e.g., `ctrl_entity_from_id`, `e_expr_parse`)
   - Macros: `SCREAMING_SNAKE_CASE` or namespace_specific
   - Structure members: `snake_case` (lowercase)
   - Global variables: `snake_case` with optional namespace prefix

3. **Integer Types**:
   ```c
   U8, U16, U32, U64   // unsigned
   S8, S16, S32, S64   // signed
   B32                 // boolean (32-bit)
   ```
   These are defined in `base_core.h`

4. **Comments & Sections**:
   ```c
   ////////////////////////////////
   //~ rjf: Section Name
   
   // NOTE(rjf): Important observation
   // TODO(rjf): Future work
   // BUG(rjf): Known issue
   
   //- rjf: Subsection
   ```
   The `~` prefix groups related sections; `-` marks subsections. Author initials indicate who wrote it.

5. **Operator Conventions**:
   - `^` = XOR
   - `<<`, `>>` = bitwise shifts
   - `|`, `&` = bitwise OR, AND
   - Macros use `()` extensively for safety

**Learning Activities**:
1. Read the first 300 lines of [src/base/base_core.h](src/base/base_core.h#L1-L300)
2. Scan through [src/base/base_core.c](src/base/base_core.c) to see function implementations
3. Rewrite 5 existing functions in your own style, then compare with the original
4. Create a "style guide" document with examples from your reading

**Key Files to Study**:
- [src/base/base_core.h](src/base/base_core.h) - Core type definitions and macros
- [src/base/base_strings.h](src/base/base_strings.h) - String handling patterns
- Any `.c` file in `src/base/` - Implementation style

**Questions to Answer**:
- What does `internal` mean, and when would you use it?
- How are sections organized in a header file?
- What's the significance of the author initials in comments?

---

## Part 2: Foundation Layers

### Module 2.1: The `base` Layer - Universal Constructs

**Objective**: Master the foundational utilities used everywhere in the codebase.

**Key Components**:

1. **Memory Management - Arenas**:
   - `Arena` struct provides growing memory allocation
   - Supports chaining arenas together
   - Temporary scopes with `temp_begin()/temp_end()`
   - Greatly reduces memory fragmentation

2. **String Handling - `String8`, `String16`, etc.**:
   - Strings are `{pointer, size}` pairs (not null-terminated by default)
   - `String8List` for accumulating strings
   - `String8Array` for random access
   - Builder pattern for string concatenation

3. **Collections**:
   - Linked lists (doubly-linked, singly-linked queues, stacks)
   - Hash tables with linear probing
   - Arrays (dynamic, based on arenas)

4. **Utility Macros**:
   - `KB(n)`, `MB(n)`, `GB(n)` - Memory size macros
   - `Min()`, `Max()`, `Clamp()` - Math macros
   - `for i in range(0, 10)` - Loop macros
   - `push_array()` - Memory allocation
   - `SLLStackPush()`, `DLLInsert()` - List manipulation

**Learning Activities**:
1. Study arena allocation:
   - Create an arena, allocate memory, use temp scopes
   - Understand why arenas are better than malloc for this use case
2. Master string operations:
   - Convert between String8 variants
   - Build and manipulate string lists
   - Understand null-termination implications
3. Implement a small data structure (e.g., a symbol table) using base layer primitives

**Key Files to Study**:
- [src/base/base_arena.h](src/base/base_arena.h) - Arena allocation
- [src/base/base_arena.c](src/base/base_arena.c) - Arena implementation
- [src/base/base_strings.h](src/base/base_strings.h) - String types (150+ lines of reference)
- [src/base/base_strings.c](src/base/base_strings.c) - String implementation
- [src/base/base_core.h](src/base/base_core.h) - Linked list macros (lines 300-350)

**Practical Exercises**:
1. Write a function that reads a text file into a `String8List`, splitting by newlines
2. Implement a simple symbol table using `base` layer primitives
3. Create temporary scratch arenas for different computation phases

**Questions to Answer**:
- How does arena allocation prevent memory fragmentation?
- What's the difference between `push_array` and `push_array_no_zero`?
- Why are strings represented as `{pointer, size}` instead of null-terminated?

---

### Module 2.2: Memory Profiling, Hashing & Metadata

**Objective**: Understand performance measurement, content hashing, and type metadata systems.

**Key Components**:

1. **Profiling** (`base_profile.h`):
   - `Prof*` macros for timing critical sections
   - Integration with debugger's performance view
   - Named profiling events

2. **Hashing** (`base_hash.h`):
   - Fast hash functions for lookup tables
   - Content hashing (MD5, SHA256) for debug info caching
   - Hash-based content deduplication

3. **Metadata** (`base_meta.h`):
   - Type introspection metadata (generated by metagen)
   - Used for serialization, hashing, copying
   - Intrinsic function wrappers

**Learning Activities**:
1. Add profiling to your arena-based program
2. Understand how content hashing enables the cache system
3. Study how metadata generation works with the `metagen` tool

**Key Files to Study**:
- [src/base/base_profile.h](src/base/base_profile.h)
- [src/base/base_hash.h](src/base/base_hash.h)
- [src/base/base_meta.h](src/base/base_meta.h)

---

## Part 3: File Format Parsing & Debug Information

### Module 3.1: Binary File Format Parsing Patterns

**Objective**: Learn how the codebase parses complex binary formats (PDB, DWARF, ELF, COFF).

**Key Patterns**:

1. **Structured Reading**:
   ```c
   // Rather than manual byte-by-byte parsing, use struct-based reading
   typedef struct {
     U32 magic;
     U32 version;
     U64 timestamp;
   } FileHeader;
   
   // Read entire struct at once
   FileHeader header = {0};
   read_data(file, &header, sizeof(header));
   ```

2. **Lazy Parsing**:
   - Read structure without immediately parsing all nested data
   - Parse on-demand when fields are accessed
   - Reduces memory usage for huge files

3. **Stream Abstraction**:
   - `MSF_Context` for Microsoft Stream Format
   - Provides seek/read operations abstracting file format
   - Enables parsing from memory or disk

4. **Type Hierarchies**:
   - Use discriminated unions with kind/type field
   - Example: `CV_LeafKind` with corresponding `CV_Leaf*` structures

**Learning Activities**:
1. Study [src/pdb/pdb.h](src/pdb/pdb.h) - Type definitions for PDB format
2. Read [src/pdb/pdb_parse.c](src/pdb/pdb_parse.c) - Parsing implementation
3. Create a mini-parser for a simple binary format using these patterns

**Key Files to Study**:
- [src/pdb/pdb.h](src/pdb/pdb.h) - PDB types
- [src/pdb/pdb_parse.c](src/pdb/pdb_parse.c) - PDB parsing
- [src/coff/coff.h](src/coff/coff.h) - COFF types
- [src/elf/elf.h](src/elf/elf.h) - ELF types
- [src/dwarf/dwarf.h](src/dwarf/dwarf.h) - DWARF types

---

### Module 3.2: The RDI (RAD Debug Info) Format

**Objective**: Understand the custom debug info format and its advantages.

**Key Concepts**:

1. **Why RDI Exists**:
   - PDB format has limitations (32-bit overflow on huge projects)
   - DWARF is fragmented across different producers
   - RDI is a simplified, unified debug info format
   - On-demand conversion from PDB/DWARF to RDI

2. **RDI Structure**:
   - Tables for source files, line info, types, variables, etc.
   - Index-based references instead of pointers
   - Efficient for binary searching and caching

3. **Conversion Pipeline**:
   - `rdi_from_pdb` (`P2R_`) - PDB → RDI
   - `rdi_from_dwarf` (`D2R_`) - DWARF → RDI (in progress)
   - `rdi_from_elf` (`E2R_`) - ELF → RDI
   - `rdi_from_coff` (`C2R_`) - COFF → RDI

**Learning Activities**:
1. Study the RDI format specification in [src/lib_rdi/rdi.h](src/lib_rdi/rdi.h)
2. Trace through a PDB → RDI conversion in [src/rdi_from_pdb/rdi_from_pdb.c](src/rdi_from_pdb/rdi_from_pdb.c)
3. Understand the `RDI_Parsed` structure and its helper functions

**Key Files to Study**:
- [src/lib_rdi/rdi.h](src/lib_rdi/rdi.h) - RDI format specification (1000+ lines)
- [src/lib_rdi/rdi_parse.h](src/lib_rdi/rdi_parse.h) - RDI parsing helpers
- [src/rdi_from_pdb/rdi_from_pdb.c](src/rdi_from_pdb/rdi_from_pdb.c) - PDB conversion (2000+ lines)
- [src/rdi_make/rdi_make_local.h](src/rdi_make/rdi_make_local.h) - RDI construction

**Questions to Answer**:
- What are the main sections of an RDI file?
- How does RDI avoid PDB's 32-bit overflow issues?
- How is the debugger able to load debug info asynchronously?

---

## Part 4: Core Debugger Systems

### Module 4.1: Process Control (`demon` & `ctrl`)

**Objective**: Understand how the debugger attaches to and controls processes.

**Key Components**:

1. **`demon` Layer** (`DMN_`):
   - OS abstraction for low-level process control
   - Windows: Uses WinDbg API (Windows Debug Events, breakpoints via INT3)
   - Linux: Uses ptrace system call
   - Exposes: Process creation, attachment, stepping, register access

2. **`ctrl` Layer** (`CTRL_`):
   - Higher-level asynchronous control system
   - Entity model: machines, processes, threads, modules
   - Breakpoint management (code + data breakpoints)
   - Stepping operations (in, over, out)
   - Trap nets: predictive breakpoints to resume execution safely

**Key Abstractions**:
- `CTRL_Entity` - Universal abstraction for machines/processes/threads
- `CTRL_Event` - Debugger events (process created, breakpoint hit, etc.)
- `CTRL_TrapNet` - Set of breakpoints for predictive stepping

**Learning Activities**:
1. Study the entity model:
   - Understand parent-child relationships
   - How handles refer to entities
2. Trace a stepping operation:
   - User clicks "step into"
   - Breakpoint is set predictively
   - Process resumes
   - Breakpoint is hit and processed
3. Understand trap nets for safe stepping

**Key Files to Study**:
- [src/demon/demon_core.h](src/demon/demon_core.h) - Low-level interface
- [src/demon/demon_core.c](src/demon/demon_core.c) - Implementation
- [src/demon/win32/](src/demon/win32/) - Windows-specific code
- [src/demon/linux/](src/demon/linux/) - Linux-specific code
- [src/ctrl/ctrl_core.h](src/ctrl/ctrl_core.h) - Control system types
- [src/ctrl/ctrl_core.c](src/ctrl/ctrl_core.c) - Control system implementation

**Questions to Answer**:
- How does `demon` abstract OS differences?
- What is a "trap net" and why is it needed?
- How does the debugger handle multi-threading?

---

### Module 4.2: Debug Info Integration (`dbg_info`)

**Objective**: Understand how debug info is loaded and cached.

**Key Components**:

1. **Asynchronous Loading**:
   - Debug info is loaded in background threads
   - Doesn't block UI or debuggee
   - Fuzzy searching enables fast symbol lookups

2. **On-Demand Conversion**:
   - PDB/DWARF files trigger RDI conversion if needed
   - Conversion happens asynchronously in separate process
   - Converted data is cached

3. **Caching Strategy**:
   - Uses `content` layer for hash-based caching
   - Maintains `DI_Key` references to loaded debug info
   - Automatic eviction of unused debug info

**Learning Activities**:
1. Study how debug info is keyed and accessed
2. Understand the queue-based processing model
3. Trace through a fuzzy search operation for a symbol

**Key Files to Study**:
- [src/dbg_info/dbg_info.h](src/dbg_info/dbg_info.h) - Types and API
- [src/dbg_info/dbg_info.c](src/dbg_info/dbg_info.c) - Implementation

---

### Module 4.3: Expression Evaluation (`eval`)

**Objective**: Master the expression compiler and evaluator.

**Key Concepts**:

1. **Compiler Architecture**:
   - Lexer: Text → tokens
   - Parser: Tokens → AST (Abstract Syntax Tree)
   - Type-checker: AST + debug info → typed IR
   - IR generation: Typed AST → evaluation plan
   - Evaluator: Executes plan in debuggee context

2. **Type System**:
   - Extracts types from debug info
   - Supports: primitives, pointers, structs, unions, arrays, functions
   - Type reconstruction from RDI

3. **Value Space Abstraction**:
   - `E_Space` - Abstract notion of where values live
   - Register space
   - Memory space
   - Debugger state space

4. **Caching**:
   - Results cached at each compiler phase
   - Re-evaluation only recomputes changed phases

**Learning Activities**:
1. Study the expression grammar by reading the lexer/parser
2. Trace a simple expression through all compiler phases:
   - `x` (variable)
   - `x + 5` (binary operation)
   - `foo.bar` (struct member access)
   - `array[i]` (indexing)
   - Function calls with custom visualizers
3. Understand how variables are resolved from debug info
4. Study auto-visualization hooks

**Key Files to Study**:
- [src/eval/eval_core.h](src/eval/eval_core.h) - Types and API (1200+ lines)
- [src/eval/eval_core.c](src/eval/eval_core.c) - Implementation
- [src/eval/eval_parse.c](src/eval/eval_parse.c) - Parser
- [src/eval/eval_type_check.c](src/eval/eval_type_check.c) - Type checking

**Questions to Answer**:
- What are the 5 phases of expression compilation?
- How does the evaluator read memory from the debuggee?
- How are auto-visualization hooks registered?

---

## Part 5: User Interface & Visualization

### Module 5.1: Immediate-Mode UI System

**Objective**: Understand the graphical UI framework.

**Key Concepts**:

1. **Immediate-Mode UI**:
   - UI is rebuilt every frame (not retained)
   - Simpler logic: build UI → process input → render
   - State lives in application, not UI framework

2. **Hierarchical Structure**:
   - UI elements form a tree
   - Parent clips/constrains children
   - Input events bubble up

3. **Core Components**:
   - `UI_Box` - Basic rectangular element
   - `UI_Signal` - User interaction results
   - Automatic layout (horizontal, vertical)

**Learning Activities**:
1. Study the UI building API in [src/ui/ui_core.h](src/ui/ui_core.h)
2. Examine a simple panel implementation
3. Understand the event processing pipeline

**Key Files to Study**:
- [src/ui/ui_core.h](src/ui/ui_core.h) - UI types and API
- [src/ui/ui_core.c](src/ui/ui_core.c) - UI implementation
- [src/raddbg/raddbg_panels.c](src/raddbg/raddbg_panels.c) - Panel implementations

---

### Module 5.2: Visualization System

**Objective**: Understand value visualization and custom visualizers.

**Key Components**:

1. **Base Visualizers**:
   - Built-in visualizations for common types (arrays, strings, etc.)
   - Color gradients for numerical data
   - Memory layouts for structures

2. **Visualization Engine** (`eval_visualization`):
   - Converts evaluated values into visual representations
   - Supports: tables, trees, geometries, custom renderers

3. **Auto-Visualization System**:
   - Type hooks that auto-apply visualizations
   - Custom evaluation code via `raddbg_*` markup
   - Nested visualizations

**Learning Activities**:
1. Study a built-in visualizer implementation
2. Understand how custom visualization markup works
3. Create a simple custom visualizer type hook

**Key Files to Study**:
- [src/eval_visualization/eval_visualization_core.h](src/eval_visualization/eval_visualization_core.h)
- [src/eval_visualization/eval_visualization_core.c](src/eval_visualization/eval_visualization_core.c)

---

## Part 6: Advanced Patterns & Systems

### Module 6.1: Async Processing & Caching

**Objective**: Master the asynchronous processing patterns used throughout.

**Key Systems**:

1. **Content Cache** (`content`):
   - Hash-based content store
   - Automatic cleanup of unreferenced data
   - Downstream tracking (who's using this data?)

2. **Artifact Cache** (`artifact_cache`):
   - Asynchronously computed artifacts
   - Automatic eviction
   - Progress tracking

3. **Mutable Text** (`mutable_text`):
   - Asynchronously-updated text buffers
   - Used for Output log

4. **Work Queues**:
   - Producer thread queues work
   - Consumer thread processes asynchronously
   - Lock-free queues for efficiency

**Learning Activities**:
1. Understand queue-based work dispatch
2. Study progress tracking for long operations
3. Implement a simple async operation (file read)

**Key Files to Study**:
- [src/content/content.h](src/content/content.h)
- [src/artifact_cache/artifact_cache.h](src/artifact_cache/artifact_cache.h)
- [src/mutable_text/mutable_text.h](src/mutable_text/mutable_text.h)

---

### Module 6.2: Code Generation with Metagen

**Objective**: Understand how code is generated from metadata.

**Key Concepts**:

1. **Metadesk Files** (`.mdesk`):
   - JSON-like configuration format
   - Defines enums, tables, and code structures
   - Parsed by `mdesk` layer

2. **Metagen Tool** (`metagen`):
   - Reads `.mdesk` files
   - Generates C code
   - Produces enums, lookup tables, serialization code

3. **Generated Code**:
   - Type metadata (for serialization)
   - String tables for UI
   - Command/feature registrations

**Learning Activities**:
1. Find and study a `.mdesk` file
2. Understand what code it generates
3. Examine the generated `.h` files in `generated/` folders

**Key Files to Study**:
- [src/mdesk/mdesk.h](src/mdesk/mdesk.h) - Metadesk format
- [src/metagen/](src/metagen/) - Code generation
- Look for `.mdesk` files in `src/*/`

---

## Part 7: Practical Mastery

### Module 7.1: Hands-On: Understanding a Feature End-to-End

**Objective**: Trace a complete feature through the entire codebase.

**Example: Setting a Breakpoint**

Trace the following user action: "User clicks in editor margin to set a breakpoint"

1. **UI Layer** (`raddbg`):
   - Click detected in source view panel
   - Converts pixel coordinates to file path + line number

2. **Engine Layer** (`dbg_engine`):
   - Receives breakpoint request
   - Looks up source line in RDI
   - Determines executable address

3. **Control Layer** (`ctrl`):
   - Queues breakpoint installation
   - Gets trap point from `demon`

4. **OS Layer** (`demon`):
   - Writes INT3 instruction at target address

5. **Result**:
   - Breakpoint stored in `dbg_engine`
   - UI displays breakpoint marker
   - When hit, process stops and event flows back up

**Learning Activities**:
1. Pick a feature from the debugger
2. Identify all layers involved
3. Create a sequence diagram
4. Read relevant code in each layer
5. Document the complete flow

**Example Features to Trace**:
- Setting a breakpoint
- Evaluating a watch expression
- Loading debug info asynchronously
- Stepping into a function
- Visualizing a complex data structure

---

### Module 7.2: Hands-On: Implementing a Simple Feature

**Objective**: Write new code in the debugger's style.

**Feature Ideas** (from simplest to complex):

1. **Simple**: Add a new built-in type visualizer
   - Pick a standard library type (e.g., `std::vector`)
   - Add visualization rules in [src/eval_visualization/](src/eval_visualization/)
   - Register visualization hook in `eval` layer

2. **Medium**: Add a new command to the UI
   - Add command definition in `dbg_engine`
   - Implement in `raddbg` UI layer
   - Add keyboard binding

3. **Complex**: Add support for custom visualization markup
   - Extend expression evaluator
   - Parse custom attributes
   - Generate visualization data

**Implementation Guidelines**:

1. **Follow the layer structure**:
   - Keep logic in the lowest possible layer
   - UI depends on engine, not vice versa

2. **Use existing patterns**:
   - Use arenas for memory
   - Use `String8` for strings
   - Use entity model for entities

3. **Match the coding style**:
   - Use proper keywords (`internal`, `global`)
   - Follow naming conventions
   - Add comments with `//~` section markers

4. **Async by default**:
   - Avoid blocking the UI
   - Use async patterns for long operations

**Practical Exercise**:
1. Implement a new source file filter in the debug info browser
   - Add UI panel
   - Add filtering logic
   - Use fuzzy matching from debug info layer

---

### Module 7.3: Debugging the Debugger

**Objective**: Learn how to debug the RADDebugger itself.

**Techniques**:

1. **Logging**:
   - Use `log_*` functions in `base_log.h`
   - Logs appear in Output panel

2. **Profiling**:
   - Add `Prof*` macros to trace execution
   - View timeline in profiler panel

3. **Assertions**:
   - Use `Assert()`, `AssertAlways()` for invariants
   - Assertions break to debugger

4. **Self-Hosting**:
   - Run debugger under debugger
   - Debug UI by stepping through frame rendering
   - Debug async operations by inspecting queues

**Learning Activities**:
1. Add logging to a feature you implemented
2. Profile a slow operation
3. Use assertions to document invariants
4. Self-host the debugger (run under VS or self)

---

## Part 8: Deep Dives

### Module 8.1: Memory Management Mastery

**Key Topics**:
- Arena allocation strategy
- Scratch arena usage patterns
- Memory lifetime design
- Profiling memory usage

**Advanced Study**:
- How arenas are chained for hierarchical allocation
- Why `temp_begin()/temp_end()` is preferred over malloc/free
- How memory usage scales with debuggee size
- Using large pages for performance (linker feature)

---

### Module 8.2: Type System Internals

**Key Topics**:
- Type reconstruction from RDI
- Handling forward declarations
- Pointer/reference type handling
- Function type handling
- Overload resolution

**Advanced Study**:
- How the `eval` layer resolves types
- Custom type hook system
- Visualization hints stored with types
- Debug info limitations (forward decls, templates)

---

### Module 8.3: Concurrency & Synchronization

**Key Topics**:
- Lock-free work queues
- Atomic operations
- Thread-local storage
- OS abstraction for threading

**Advanced Study**:
- How multiple threads coordinate without locks
- Handling race conditions in production code
- Testing concurrent code paths

---

## Part 9: Code Reading Strategies

### Strategy 9.1: The "Follow the Flow" Technique

Start with a user action and trace upward:

1. **Identify UI element** that responds to action
2. **Find signal handler** that processes response
3. **Trace function calls** down through layers
4. **Identify key data structures** that flow through
5. **Map to persistent state** in engine layer

### Strategy 9.2: The "Grep & Pattern" Technique

1. Grep for a key type or function
2. Understand all call sites
3. Identify patterns in usage
4. Build mental model of "how this is used"

### Strategy 9.3: The "Experiment" Technique

1. Make a small change to code
2. Rebuild and test
3. Observe the effect
4. Reason backward to understand impact

### Strategy 9.4: The "Reverse Design" Technique

1. Identify a feature in the UI
2. Search for related functions
3. Reverse-engineer the design
4. Document the architecture

---

## Part 10: Study Roadmap

### Phase 1: Foundation (Weeks 1-2)
- [ ] Read [README.md](README.md) completely
- [ ] Study `base` layer thoroughly
- [ ] Set up development environment
- [ ] Build the debugger successfully
- [ ] Learn the coding style conventions

### Phase 2: File Formats (Weeks 3-4)
- [ ] Study PDB parsing (look at `pdb` layer)
- [ ] Understand RDI format
- [ ] Trace PDB → RDI conversion
- [ ] Read sample binary format parsing code

### Phase 3: Core Systems (Weeks 5-7)
- [ ] Deep dive into `demon` (process control)
- [ ] Master `ctrl` (control system)
- [ ] Understand `dbg_info` (debug info loading)
- [ ] Study `eval` (expression evaluation)

### Phase 4: UI & Visualization (Weeks 8-9)
- [ ] Learn immediate-mode UI system
- [ ] Understand visualization engine
- [ ] Study a complete panel implementation
- [ ] Understand async operations

### Phase 5: Advanced Patterns (Weeks 10-11)
- [ ] Master async patterns and caching
- [ ] Learn code generation system
- [ ] Understand concurrency model
- [ ] Study memory management strategies

### Phase 6: Hands-On (Weeks 12+)
- [ ] Implement a simple feature
- [ ] Debug the debugger itself
- [ ] Read complex parts of codebase
- [ ] Contribute improvements

---

## Recommended Resources

### Primary
- [RADDebugger Repository](https://github.com/EpicGamesExt/raddebugger)
- README.md - Comprehensive technical documentation
- Source code with `rjf:` comments by original author

### Secondary
- VS Code with RADDebugger running
- Debugger's built-in help and documentation
- MSDN/Apple docs for Windows/Linux APIs being abstracted

---

## Key Takeaways for Authors' Mindset

### Philosophy
1. **Simplicity over cleverness**: Code should be readable and debuggable
2. **Explicit over implicit**: Clear intent in every line
3. **Performance matters**: Caching and async are pervasive
4. **Layers provide freedom**: Abstraction enables independent evolution
5. **Tests prove understanding**: Mule tests validate features

### Practices
1. **Namespace everything**: Reduces cognitive load, aids grep searches
2. **Document section structure**: `//~` sections group related code
3. **Be consistent**: Style consistency aids reading
4. **Profile relentlessly**: Optimize what matters, not what you guess
5. **Plan for scale**: Design for 10GB executables from day one

### Patterns
1. **Arenas for allocation**: Eliminates fragmentation, aids cleanup
2. **Async by default**: Never block the UI
3. **Cache aggressively**: Hash-based caching throughout
4. **Abstract OS details**: `demon` enables Windows/Linux support
5. **Compile & evaluate separately**: Expression compiler has clear phases

---

## Conclusion

The RADDebugger is a masterclass in systems programming. By following this syllabus, studying the code carefully, and implementing features, you'll internalize the architectural principles and coding style that make this codebase exceptional. The goal is not just to understand the code, but to think like the original author—designing clean layers, optimizing for scale, and writing code that's a pleasure to maintain and extend.

Success looks like: writing new code that's indistinguishable from the original author's work, understanding trade-offs made in the design, and extending the debugger with confidence.

Happy learning!
