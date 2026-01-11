---
description: Triggered by /define. Establishes the feature concept, gameplay justification, and creates the C++ Data Contract (Configs & State).
---

### COMMAND: /define [Feature Name]
**Goal:** Define the feature. **CRITICAL:** You must enforce **Data-Oriented Design** from step one.

**Optimization Rules to Enforce during Definition:**
1.  **DOD over OOP:** Ask "Can this be a TArray of Structs (`FMyData`) instead of an Array of Actors?" to prevent Cache Misses.
2.  **Soft References:** Reject Hard Pointers (`UPROPERTY() AItem*`). Mandate `TSoftObjectPtr<T>` to prevent cascade loading.
3.  **Mass Entity Check:** If the feature involves >100 entities (crowds, traffic, projectiles), suggest **Mass Entity (ECS)** instead of Actors.

**Output Action:**
Edit the target `.md` file to include:

# [Feature Name] Spec

## 1. Context
* **Summary:** (Technical summary).
* **Core Loop Utility:** (Connection to pillars).
* **Data Strategy:** (Explain the choice of Structs/SoA vs Actors).

## 2. Data Contract
**A. Configuration (`UDataAsset`)**
* Create `F[Feature]Config`.
* **Constraint:** Use `TSoftObjectPtr` for all assets.

**B. Runtime State (`UStruct`)**
* Create `F[Feature]State`.
* **Constraint:** If iterating over many items, use Structure of Arrays (SoA) layout (e.g., separate arrays for `Health` vs `Position`) to maximize Cache Locality.