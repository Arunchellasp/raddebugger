# Practice Project Templates

This file contains **ready-to-use project templates** to practice the patterns from the syllabus.

## Project 1: String Utility Library

**Goal**: Implement a complete string library using the String8 pattern

**Files to Create**:
```
string_lib/
  ├── string.h       (declarations)
  ├── string.c       (implementations)
  ├── test.c         (test cases)
  └── Makefile       (build)
```

**Implementation Checklist**:
- [ ] `str8_from_cstr(char *cstr)` - C string → String8
- [ ] `str8(U8 *data, U64 size)` - Create String8
- [ ] `str8_equal(String8 a, String8 b)` - Compare
- [ ] `str8_find_substring(String8 haystack, String8 needle)` - Search
- [ ] `str8_slice(String8 s, U64 start, U64 end)` - Substring
- [ ] `str8_split(String8 s, String8 sep)` - Split by delimiter
- [ ] `str8_concat(Arena *a, String8 s1, String8 s2)` - Concatenate
- [ ] `str8_trim(String8 s)` - Remove whitespace
- [ ] `str8_to_u64(String8 s)` - Parse number
- [ ] `str8_builder_*` - Dynamic string building

**Test Cases**:
```c
// Test compare
assert(str8_equal(str8_from_cstr("hello"), str8_from_cstr("hello")));

// Test find
String8 s = str8_from_cstr("hello world");
assert(str8_find_substring(s, str8_from_cstr("world")) == 6);

// Test slice
String8 slice = str8_slice(s, 0, 5);
assert(str8_equal(slice, str8_from_cstr("hello")));
```

---

## Project 2: Expression Evaluator

**Goal**: Build a simple expression evaluator using lexer + parser + evaluator

**Expression Grammar**:
```
expr   : term (('+' | '-') term)*
term   : factor (('*' | '/') factor)*
factor : number | '(' expr ')'
```

**Files to Create**:
```
calculator/
  ├── lexer.h
  ├── lexer.c
  ├── parser.h
  ├── parser.c
  ├── eval.h
  ├── eval.c
  ├── main.c
  └── Makefile
```

**Phases**:
1. **Lexer**: "2 + 3" → [Number(2), Plus, Number(3)]
2. **Parser**: [tokens] → AST(BinOp(Number(2), Plus, Number(3)))
3. **Evaluator**: AST → 5

**Extension Ideas**:
- Add variables: `x = 5; x + 3`
- Add functions: `sqrt(16)`, `max(5, 3)`
- Add comparison: `2 < 5`
- Add conditions: `if (x > 0) 10 else 20`

---

## Project 3: Configuration File Parser

**Goal**: Parse INI-style config files with sections and keys

**File Format**:
```
[database]
host=localhost
port=5432
name=mydb

[server]
port=8080
debug=true
```

**Files to Create**:
```
config_parser/
  ├── config.h
  ├── config.c
  ├── test_config.ini
  ├── main.c
  └── Makefile
```

**Data Structures**:
```c
typedef struct {
  String8 key;
  String8 value;
} ConfigEntry;

typedef struct {
  String8 section;
  ConfigEntry *entries;
  U64 count;
} ConfigSection;

typedef struct {
  ConfigSection *sections;
  U64 count;
} Config;
```

**Functions to Implement**:
- [ ] `config_parse(Arena *arena, char *filename)` - Read and parse
- [ ] `config_get(Config *c, String8 section, String8 key)` - Lookup
- [ ] `config_get_int(Config *c, String8 section, String8 key, S64 default_val)`
- [ ] `config_get_bool(Config *c, String8 section, String8 key, B32 default_val)`
- [ ] `config_section_count(Config *c)` - Query sections
- [ ] `config_free(Config *c)` - Cleanup

**Test Cases**:
```c
Config cfg = config_parse(arena, "test.ini");
String8 host = config_get(&cfg, str8("database"), str8("host"));
assert(str8_equal(host, str8("localhost")));
```

---

## Project 4: Simple Logger

**Goal**: Build a logging system with multiple output destinations

**Features**:
- [ ] Log levels: Debug, Info, Warning, Error, Fatal
- [ ] Multiple outputs: File, Console, Memory buffer
- [ ] Timestamping
- [ ] Format strings with arguments
- [ ] Thread-safe operations

**Files to Create**:
```
logger/
  ├── log.h
  ├── log.c
  ├── main.c
  └── Makefile
```

**API**:
```c
void log_init(void);
void log_debug(String8 fmt, ...);
void log_info(String8 fmt, ...);
void log_warning(String8 fmt, ...);
void log_error(String8 fmt, ...);
void log_fatal(String8 fmt, ...);
String8Array log_get_all_entries(void);
void log_cleanup(void);
```

---

## Project 5: Simple Profiler

**Goal**: Measure function execution time

**Features**:
- [ ] Named timing events
- [ ] Nested timing scopes
- [ ] Aggregate statistics (min, max, avg)
- [ ] CSV export

**Macro API**:
```c
PROF_BEGIN_FUNCTION(); // auto __FUNCTION__
PROF_END_FUNCTION();

PROF_SCOPED("operation_name");
// ... code ...
// Scope ends, timing recorded
```

**Files to Create**:
```
profiler/
  ├── profile.h
  ├── profile.c
  ├── main.c
  └── Makefile
```

---

## Project 6: JSON-like Configuration

**Goal**: Parse and serialize structured data (like Metadesk format)

**Format Example**:
```
{
  name: "MyApp"
  version: "1.0"
  features: {
    debug: true
    optimization: 2
    plugins: [
      "audio.so"
      "render.so"
    ]
  }
}
```

**Data Type**:
```c
typedef enum {
  ValueKind_Null,
  ValueKind_Bool,
  ValueKind_Integer,
  ValueKind_Float,
  ValueKind_String,
  ValueKind_Array,
  ValueKind_Object,
} ValueKind;

typedef struct Value {
  ValueKind kind;
  union {
    B32 boolean;
    S64 integer;
    F64 floating;
    String8 string;
    struct {
      struct Value *elements;
      U64 count;
    } array;
    struct {
      struct {
        String8 key;
        struct Value value;
      } *pairs;
      U64 count;
    } object;
  } data;
} Value;
```

---

## Project 7: In-Memory Database

**Goal**: Simple key-value store with persistence

**Features**:
- [ ] Get/Set/Delete operations
- [ ] In-memory hash table storage
- [ ] Save to binary format
- [ ] Load from binary format
- [ ] Transactions (batch operations)

**API**:
```c
DB *db_open(char *filename);
void db_set(DB *db, String8 key, String8 value);
String8 db_get(DB *db, String8 key);
void db_delete(DB *db, String8 key);
void db_save(DB *db);
void db_close(DB *db);
```

---

## Project 8: Plugin System

**Goal**: Load external code dynamically

**Features**:
- [ ] Load .so/.dll files
- [ ] Look up function pointers
- [ ] Initialize/shutdown plugins
- [ ] Query plugin info

**Plugin Interface**:
```c
typedef struct {
  String8 name;
  String8 version;
  void (*init)(void);
  void (*shutdown)(void);
  void (*process)(void *data);
} PluginInterface;

// Plugins must export this function:
// PluginInterface *plugin_get_interface(void);
```

---

## Building & Testing Your Projects

### Minimal Makefile Template

```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -Wextra -g
LDFLAGS = 

SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)
TARGET = app

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

clean:
	rm -f $(OBJS) $(TARGET)

test: $(TARGET)
	./$(TARGET)

.PHONY: all clean test
```

### Testing Pattern

```c
// In your test file or main:

#define ASSERT(cond, msg) do { \
  if (!(cond)) { \
    printf("FAIL: %s\n", msg); \
    return 0; \
  } \
} while(0)

int test_basic_functionality(void) {
  // Setup
  Arena arena = arena_create(MB(1));
  
  // Test
  String8 s = str8_from_cstr("hello");
  ASSERT(s.size == 5, "String size mismatch");
  
  // Cleanup
  arena_free(&arena);
  return 1;
}

int main(void) {
  int passed = 0, failed = 0;
  
  if (test_basic_functionality()) passed++;
  else failed++;
  
  printf("Tests passed: %d, Failed: %d\n", passed, failed);
  return failed == 0 ? 0 : 1;
}
```

---

## Project Completion Criteria

For each project, you're done when:

1. ✅ All functions implemented
2. ✅ All test cases pass
3. ✅ Follows naming conventions from Part 1
4. ✅ No memory leaks (use arenas!)
5. ✅ Handles edge cases (empty input, NULL, etc.)
6. ✅ Code is documented with section markers
7. ✅ Compiles with `-Wall -Wextra`
8. ✅ You could explain it to someone else

---

## Progression Suggestion

1. **Start**: Project 1 (Strings) - Foundations
2. **Continue**: Project 2 (Evaluator) - Parsing skills
3. **Expand**: Project 3 (Config) - Real-world problem
4. **Advance**: Project 5 (Profiler) - System understanding
5. **Challenge**: Project 7 (Database) - Integration test

Each project reuses code from previous ones. Project 7 uses arenas (Project 1 patterns), parsing (Project 2), configuration (Project 3), and logging (Project 4).

---

## Project Variations

Once you complete a project, try these variations:

**Project 1 (Strings)**:
- Add printf-like formatting: `str8_printf(arena, "%s: %d", name, value)`
- Add UTF-8 validation
- Add regex matching

**Project 2 (Calculator)**:
- Add variables and assignment
- Add built-in functions: sqrt, sin, cos, etc.
- Add binary operators: %, &, |, ^

**Project 3 (Config)**:
- Add environment variable expansion: `${HOME}`
- Add type conversion: `config_get_int()`, `config_get_bool()`
- Add validation rules

**Project 4 (Logger)**:
- Add log rotation (when file gets too big)
- Add filtering (log only certain levels)
- Add colorized console output

**Project 7 (Database)**:
- Add indexing on frequently-searched keys
- Add range queries: `keys between "a" and "z"`
- Add transactions: all-or-nothing operations

---

## Production-Quality Additions

For any project, production-readiness requires:

1. **Error Handling**
   - Return status codes or error structs
   - Never assert on user input
   - Handle resource exhaustion gracefully

2. **Performance**
   - Profile before optimizing
   - Use arenas for allocation-heavy code
   - Batch operations when possible

3. **Testing**
   - Unit tests for each module
   - Integration tests for components
   - Boundary condition tests

4. **Documentation**
   - Function comments explaining purpose
   - Example usage in comments
   - API design rationale

5. **Compatibility**
   - Works on Windows, Linux, macOS
   - Handles different endianness (if needed)
   - 32-bit and 64-bit compatible

---

Good luck building! 🚀
