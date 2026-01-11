---
description: Triggered by /logic. Provides technical implementation, C++ pseudocode, delegate bindings, and save/load system integration.
---

### COMMAND: /logic
**Goal:** Write the implementation. **CRITICAL:** Zero tolerance for memory churn or unoptimized loops.

**Optimization Rules to Enforce:**
1.  **Memory Churn:** When filling arrays, ALWAYS use `TArray::Reserve(n)` to prevent reallocation overhead.
2.  **Allocator Strategy:** For single-frame data (queries), use `FMemStack` (Linear Allocator) instead of Heap allocation.
3.  **Shadow Invalidation:** If spawning objects, set **Shadow Cache Invalidation** to `Rigid` or `Static` (especially for WPO foliage) to prevent VSM page invalidation.
4.  **Significance Manager:** If ticking is unavoidable, implement `USignificanceManager` logic to throttle distant entities.

**Output Action:**
Edit the target `.md` file to append:

## 4. Implementation Details

**A. Header Requirements**
* **Delegates:** List Events.

**B. Pseudocode Logic (.cpp)**
* **Loop Optimization:** Show how to iterate over contiguous memory (Structs).
* **Allocation:** Show `Array.Reserve()` syntax.
* **Timer Handles:** Use `FTimerManager` for non-frame-critical logic (e.g., Health Regen).

**C. Save/Load Logic**
* Explain how `F[Feature]State` is serialized.