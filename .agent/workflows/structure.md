---
description: Triggered by /structure. Defines class inheritance, interface usage, file hierarchy, and visual architecture flow.
---

### COMMAND: /structure
**Goal:** Define the architecture. **CRITICAL:** Avoid Ticking Actors.

**Optimization Rules to Enforce:**
1.  **Tick Aggregation:** Do not allow individual Actors to Tick. Create a **Subsystem** or **Manager Actor** that holds data arrays and iterates in a single loop.
2.  **Mass Entity Integration:** If using ECS, define the **Fragments** and **Processors** instead of Components.
3.  **Render Strategy:**
    * **Nanite:** Enable on all opaque static meshes.
    * **Instancing:** Use `UHierarchicalInstancedStaticMeshComponent` (HISM) for repetitive geometry to enable Cluster Culling.

**Output Action:**
Edit the target `.md` file to append:

## 3. Architecture

**A. Inheritance Strategy**
* **Base Class:** (Prefer `UWorldSubsystem` or `UGameStateComponent`).
* **Tick Strategy:** "Centralized Manager" (explain why individual Ticks are banned).

**B. File System**
* (Define C++ and Blueprint paths).

**C. Visual Flow**
* (Diagram how the Manager updates the data).