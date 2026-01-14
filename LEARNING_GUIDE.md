# Learning Guide: Quick Navigation

This guide helps you navigate the complete syllabus in `LEARNING_SYLLABUS.md`.

## Quick Start (First Time Here?)

1. **Read the Overview** - Understand what you'll learn
2. **Start with Part 1: Foundational Concepts** - Learn architecture principles
3. **Move to Part 2: Memory Management** - Master arenas and strings
4. **Build up through Parts 3-6** - Learn data structures, async, parsing
5. **Practice with Part 8** - Build real projects
6. **Use the Checklist** - Verify mastery

---

## Part-by-Part Overview

### Part 1: Foundational Concepts (Setup)
- **Module 1.1**: Layered Architecture & Dependencies
  - Design a 3-layer project from scratch
  - Learn namespace conventions
  
- **Module 1.2**: Coding Conventions & Naming
  - Understand scope keywords (`internal`, `global`)
  - Master naming schemes (types, functions, macros)

### Part 2: Memory Management (Foundation)
- **Module 2.1**: Arena Allocation
  - Why arenas beat malloc
  - Implement a simple arena allocator
  - Practice with temporary scopes

- **Module 2.2**: Smart Strings (String8 Pattern)
  - {pointer, size} instead of null-terminated
  - Build a string library
  - Understand slicing and building

### Part 3: Collections & Data Structures
- **Module 3.1**: Linked Lists
  - Singly-linked (stacks)
  - Doubly-linked (flexible insertion)
  - Implement a queue

- **Module 3.2**: Hash Tables
  - Hash functions
  - Linear probing collision resolution
  - Add resizing capability

### Part 4: Asynchronous Processing & Caching
- **Module 4.1**: Work Queues
  - Producer-consumer pattern
  - Circular buffers
  - Async work processing

- **Module 4.2**: Caching
  - Content-addressed storage
  - LRU eviction
  - Access tracking

### Part 5: Parsing & Serialization
- **Module 5.1**: Binary File Parsing
  - Stream abstraction
  - Struct-based reading
  - Bounds checking and validation

### Part 6: Compiler Patterns
- **Module 6.1**: Lexer (Text → Tokens)
  - Tokenization with position tracking
  - Whitespace handling
  - Token classification

- **Module 6.2**: Parser (Tokens → AST)
  - Recursive descent parsing
  - Operator precedence
  - Building abstract syntax trees

### Part 7: Advanced Concepts
- **Module 7.1**: Type Systems & Abstraction
  - Discriminated unions (tagged unions)
  - Abstract interfaces
  - Polymorphism in C

- **Module 7.2**: Optimization Patterns
  - Pre-allocation strategies
  - Batch operations
  - Lazy evaluation

### Part 8: Practical Mastery
- **Module 8.1**: Building a Simple Project
  - Complete data processor example
  - File loading, parsing, statistics
  - Real-world patterns

- **Module 8.2**: Real-World Problem Solving
  - Exercise 1: Symbol Table (compiler)
  - Exercise 2: Event Dispatcher (UI)
  - Exercise 3: Configuration Parser (apps)

### Part 9: Study Checklist
- Core patterns you must know
- Practice you must complete
- Code reading you must do

### Part 10: Final Checklist & Principles
- Are you ready? (10-point checklist)
- Universal principles for ANY C project

---

## Learning Paths

### Path A: Quick Learner (3 Weeks)
- Week 1: Parts 1-2 (Architecture + Memory)
- Week 2: Parts 3-4 (Data structures + Async)
- Week 3: Parts 8 (Practice + Projects)

### Path B: Thorough Learner (8 Weeks)
- Weeks 1-2: Parts 1-2 (Foundations)
- Weeks 3-4: Parts 3-4 (Collections & Async)
- Weeks 5-6: Parts 5-6 (Parsing & Compilers)
- Weeks 7-8: Parts 7-8 (Advanced + Practice)

### Path C: Deep Mastery (12+ Weeks)
- Complete all exercises
- Read production code
- Build 3-5 real projects
- Study complex subsystems

---

## By Experience Level

### Beginners
1. Read all of Parts 1-2 carefully
2. Do exercises in each module
3. Don't skip the practice code—type it out!
4. Build the simple project in Module 8.1

### Intermediate Programmers
1. Skim Part 1-2 (likely familiar)
2. Deep dive into Parts 3-4 (async patterns)
3. Study Parts 5-6 (parsing is key)
4. Practice with all exercises in Part 8

### Advanced Programmers
1. Skim everything but focus on:
   - Different design patterns in each area
   - Trade-offs and alternatives
   - How patterns work together
2. Build the real-world exercises (Part 8.2)
3. Compare with RADDebugger source code

---

## Practice Exercises by Difficulty

### Easy (Do First)
- [ ] Module 1.1: Design 3-layer project
- [ ] Module 1.2: Rewrite bad code with conventions
- [ ] Module 2.1: Use arenas instead of malloc
- [ ] Module 2.2: Implement str8_starts_with/ends_with
- [ ] Module 3.1: Implement queue on top of linked list
- [ ] Module 3.2: Add hash table resizing

### Medium (Do Next)
- [ ] Module 4.1: Build work queue system
- [ ] Module 4.2: Extend cache with expiration
- [ ] Module 5.1: Parse section-based format
- [ ] Module 6.1: Add comments and strings to lexer
- [ ] Module 6.2: Add assignment operator to parser
- [ ] Module 8.1: Build complete data processor

### Hard (Do Last)
- [ ] Module 8.2, Exercise 1: Build symbol table
- [ ] Module 8.2, Exercise 2: Build event dispatcher
- [ ] Module 8.2, Exercise 3: Build config parser
- [ ] Combine all: Build a mini-compiler or interpreter

---

## Key Concepts by Area

### Memory & Allocation
- Arenas instead of malloc
- Temporary scopes
- Pre-allocation vs reallocation
- Memory lifetime design

### Data Structures
- Linked lists (singly, doubly)
- Hash tables with probing
- Heaps and priority queues
- Trees and search structures

### Async & Concurrency
- Work queues (producer-consumer)
- Circular buffers
- Atomic operations
- Lock-free patterns

### Parsing & Compilation
- Lexical analysis (lexer)
- Syntactic analysis (parser)
- Type checking
- Code generation (basic)

### Performance
- Batch operations
- Cache locality
- Lazy evaluation
- Content-addressed caching

### Design Patterns
- Layered architecture
- Abstract interfaces
- Discriminated unions
- Observer/event patterns

---

## Building Your First Project

### Minimal Project (1-2 weeks)
1. Implement arena + string library (Part 2)
2. Add hash table (Part 3)
3. Parse a simple format (Part 5)
4. Result: basic data processor

### Medium Project (3-4 weeks)
1. All of above
2. Add parser (Part 6.1-6.2)
3. Implement symbol table (Module 8.2)
4. Build evaluator
5. Result: simple expression calculator or config processor

### Full Project (6-8 weeks)
1. All of above
2. Add async processing (Part 4)
3. Implement multiple data formats
4. Add caching system
5. Build CLI interface
6. Result: production-quality tool

---

## Study Tips

1. **Type out the code**: Don't copy-paste. Typing reinforces learning.

2. **Modify examples**: Change parameters, add features, break it on purpose.

3. **Draw diagrams**: Visual understanding complements code.

4. **Explain to others**: Teaching is the best way to learn.

5. **Compare approaches**: Why use hash table vs linked list?

6. **Read production code**: Study RADDebugger source while learning.

7. **Time your execution**: Measure performance of your implementations.

8. **Build incrementally**: Each module builds on previous ones.

---

## FAQ

**Q: Do I need to read everything in order?**
A: Parts 1-2 are foundational. After that, you can jump around, but each module references earlier concepts.

**Q: How much code should I write?**
A: Plan to write 2000-3000 lines of practice code. This includes all exercises.

**Q: Should I compare my code with RADDebugger?**
A: Yes, constantly. Look for different approaches and optimizations.

**Q: What if I get stuck?**
A: Re-read the explanation, type the example code, modify it, then try the exercise again.

**Q: How long does mastery take?**
A: 3-4 weeks for good understanding, 8-12 weeks for deep mastery, 6+ months to internalize completely.

**Q: Can I skip exercises?**
A: No. Exercises are where learning happens. Even if you think you know the concept.

---

## Success Criteria

You've mastered the material when you can:

- [ ] Explain layered architecture to someone else
- [ ] Design a system with clean layers
- [ ] Choose appropriate data structures
- [ ] Implement a small parser from scratch
- [ ] Build an async work queue system
- [ ] Write memory-safe C with arenas
- [ ] Implement a cache with proper eviction
- [ ] Read a 5000-line codebase and understand it
- [ ] Identify patterns in unfamiliar code
- [ ] Extend existing code in its own style

When all checkboxes are ✅, you've achieved mastery.

---

## Next Steps

1. **Start**: Read Part 1 overview + Module 1.1
2. **Practice**: Do exercise 1.1 (design 3-layer project)
3. **Build**: Complete Module 1.2 code review exercise
4. **Continue**: Follow the learning path for your experience level

**Happy learning!**
