# Plan: Replace `child` with `tagged` (blockless) API

## Summary

Make `child` a private method and use `tagged` without a block as the public API for creating child loggers with instance named tags.

## Changes

### Phase 1: Update Tests (will be red until code changes)

1. **test/logger_test.rb** - Replace `.child` calls with `.tagged` (blockless)
   - Rename describe block from `.child` to `.tagged without block`
   - Change `logger.child(**tags)` to `logger.tagged(**tags)`

2. **test/loggable_test.rb** - Update `TestChildClassLogger` test class
   - Replace `logger_child` with `logger_tagged` in class definition
   - Update test description

### Phase 2: Update Code

3. **lib/semantic_logger/base.rb**
   - Modify `tagged` method to return `child(**named_tags)` when no block is given and only named tags are passed
   - Move `child` to private section

4. **lib/semantic_logger/loggable.rb**
   - Rename `logger_child` to `logger_tagged`

### Phase 3: Update Documentation

5. **docs/api.md**
   - Update "Child Logger" section to show `tagged` usage instead of `child`
   - Possibly rename section to "Instance Tagged Logger" or similar

6. **CHANGELOG.md**
   - Update entry to reflect the API change

### Phase 4: Add Instance Positional Tags Support

Support positional arguments (instance tags) in `tagged` without a block.

**Steps:**

1. **Write new tests** - Add tests for `tagged` with positional instance tags
   - `logger.tagged('tag1', 'tag2')` returns a child logger with instance tags
   - Instance tags are prefixed to positional tags passed to log methods
   - Combined usage: `logger.tagged('tag1', user: 'alice')` supports both positional and named instance tags

2. **Execute tests** (Expected: Failure)

3. **Update code** - Modify `tagged` and child logger to support instance tags
   - Store positional instance tags in child logger
   - Prefix instance tags to the tags array used by log methods

4. **Execute tests** (Expected: Success)

### Phase 5: Block Inherits Instance Tags from Child Logger

When `tagged` is called with a block on a child logger, push the child logger's instance tags and instance named tags to the thread for the duration of the block.

**Steps:**

1. **Write new tests** - Add tests for `tagged` block behavior on child loggers
   - `child_logger.tagged { }` pushes instance tags to thread
   - `child_logger.tagged { }` pushes instance named tags to thread
   - `child_logger.tagged('extra') { }` pushes both instance tags and block tags to thread
   - `child_logger.tagged(extra: 'value') { }` merges instance named tags with block named tags
   - `child_logger.tagged('extra', extra: 'value') { }` pushes instance tags + block tags and merges instance named tags + block named tags

2. **Execute tests** (Expected: Failure)

3. **Update code** - Modify `tagged` block behavior to include instance tags
   - When block given, prepend instance tags to the tags pushed to thread
   - When block given, merge instance named tags with the named tags pushed to thread

4. **Execute tests** (Expected: Success)

## Behavior specification for `tagged`

When `tagged` is called:
- **With a block (on root logger)**: Current behavior (thread-local tags, yields to block)
- **With a block (on child logger)**: Push instance tags + block tags to thread, push instance named tags + block named tags to thread
- **Without a block + named tags only**: Return a child logger with instance named tags
- **Without a block + positional tags only**: Return a child logger with instance tags
- **Without a block + both positional and named tags**: Return a child logger with both instance tags and instance named tags
