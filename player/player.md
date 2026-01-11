# Player Feature Spec

## 1. Context
*   **Summary:** The Player is the primary agent controlled by the user, capable of traversing the environment using a physics-based movement controller and engaging in combat with a health system. The system includes Inverse Kinematics (IK) for grounding and realistic foot placement.
*   **Core Loop Utility:** The player's survival (Health) and traversal (Movement) are central to the RPG experience, enabling exploration and combat encounters.
*   **Data Strategy:** We adhere to **Data-Oriented Design (DOD)**.
    *   **Structs over Actors:** We favor `FPlayerState` structs for runtime data over storing everything in the `APlayerCharacter` actor to minimize cache misses and overhead.
    *   **Soft References:** All asset references in configuration are `TSoftObjectPtr` to prevent cascade loading and reduce initial memory footprint.

## 2. Data Contract

### A. Configuration (`UDataAsset`)
**`UPlayerConfig`**
*   **Mesh Assets:**
    *   `TSoftObjectPtr<USkeletalMesh> CharacterMesh`
    *   `TSoftObjectPtr<UAnimBlueprint> AnimBlueprint`
*   **Health Config (`FPlayerHealthConfig`):**
    *   `float MaxHealth` (Default: 100.0f)
    *   `float HealthRegenRate` (Default: 0.0f)
*   **Movement Config (`FPlayerMovementConfig`):**
    *   `float WalkSpeed` (Default: 400.0f)
    *   `float SprintSpeed` (Default: 800.0f)
    *   `float JumpForce` (Default: 500.0f)
*   **IK Config (`FPlayerIKConfig`):**
    *   `float FootTraceDistance` (Default: 50.0f)
    *   `float PelvisOffsetSmoothing` (Default: 10.0f)

### B. Runtime State (`UStruct`)
**`FPlayerState`**
*   **Constraint:** Designed for high cache locality. If we were simulating thousands of players, we would split this into parallel arrays (SoA), but for a single local player, a unified struct is acceptable, provided it remains lightweight.

```cpp
struct FPlayerState
{
    // Health
    float CurrentHealth;
    bool bIsDead;

    // Movement Flags (Bitfield preference or Enum)
    uint8 bIsGrounded : 1;
    uint8 bIsSprinting : 1;
    uint8 bIsCrouching : 1;
    
    // IK State (Transient)
    FVector FootL_Offset;
    FVector FootR_Offset;
    float PelvisOffset;
};
```

## 3. Architecture

### A. Inheritance Strategy
*   **Base Class:** `APlayerCharacter` (inherits from `ACharacter`)
    *   **CRITICAL CONSTRAINT:** `PrimaryActorTick.bCanEverTick = false`. This actor **MUST NOT** tick individually. All logic is driven by the Subsystem.
*   **Tick Strategy:** "Centralized Manager"
    *   **Manager:** `UPlayerMovementSubsystem` (inherits `UWorldSubsystem`).
    *   **Reasoning:** Prevents ticking overhead for each actor. The Subsystem iterates over registered `FPlayerState` data in a tight loop, applying physics and logic more efficiently.

### B. File System
*   **Logic/Manager:**
    *   `Source/Game/Player/PlayerSubsystem.h`
    *   `Source/Game/Player/PlayerSubsystem.cpp`
*   **Representation/Actor:**
    *   `Source/Game/Player/PlayerCharacter.h`
    *   `Source/Game/Player/PlayerCharacter.cpp`
*   **Data:**
    *   `Source/Game/Player/PlayerConfig.h`
    *   `Source/Game/Player/PlayerState.h`

### C. Visual Flow
```mermaid
graph TD
    GameLoop[Game Loop] -->|Tick| Subsystem[UPlayerMovementSubsystem]
    Subsystem -->|Iterate| PlayerStates[Array<FPlayerState>]
    
    subgraph Update Logic
        PlayerStates -->|Calc| Physics[Movement Physics]
        PlayerStates -->|Calc| IK[IK Solvers]
        PlayerStates -->|Calc| Health[Health Regen]
    end
    
    Physics -->|Apply| CharacterAActor[APlayerCharacter \n(SetActorLocation/Rotation)]
    IK -->|Apply| CharacterAnim[AnimInstance \n(Update Control Rig)]
```

## 4. Implementation Details

### A. Header Requirements
*   **Delegates:**
    *   `DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FOnPlayerHealthChanged, float, NewHealth, float, MaxHealth);`
    *   `DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnPlayerDied);`

### B. Pseudocode Logic (.cpp)

#### 1. Optimization: Subsystem Loop
*   **Goal:** Iterate over contiguous memory to update logic.

```cpp
// PlayerSubsystem.cpp

void UPlayerMovementSubsystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);
    // Reserve memory to prevent early reallocations if we spuriously add/remove players
    RegisteredPlayers.Reserve(10); 
    PlayerStates.Reserve(10);
}

void UPlayerMovementSubsystem::Tick(float DeltaTime)
{
    // Iterate over contiguous structs for maximum cache locality
    for (int32 i = 0; i < PlayerStates.Num(); ++i)
    {
        FPlayerState& State = PlayerStates[i];
        
        // 1. Update Health (Simple Logic)
        if (State.CurrentHealth < MaxHealth && !State.bIsDead)
        {
            // Simple regen logic inline, or offload to a Timer/Manager if complex
            State.CurrentHealth += HealthRegenRate * DeltaTime;
        }

        // 2. Resolve Movement (Physics)
        // ... Solve velocity, collision sliding ...

        // 3. Resolve IK (Raycasts)
        // ... Raycast from foot positions ...
        
        // 4. Sync to Actor (The expensive part, keep minimal)
        if (APlayerCharacter* Actor = RegisteredPlayers[i])
        {
            Actor->SetActorLocation(CalculatedPosition);
        }
    }
}
```

#### 2. Timer Handles
For non-critical logic like **long-term Health Regen** (e.g. only regen when out of combat for 5s), do not check every frame in the tick loop. Use `FTimerManager`.
```cpp
// In PlayerCharacter or Subsystem
GetWorld()->GetTimerManager().SetTimer(RegenTimerHandle, this, &UPlayerMovementSubsystem::OnRegenTick, 1.0f, true);
```

### C. Save/Load Logic
*   **Serialization:** `FPlayerState` is a struct, making it trivial to serialize directly into an archive.

```cpp
// Serialization
FArchive& operator<<(FArchive& Ar, FPlayerState& State)
{
    Ar << State.CurrentHealth;
    Ar << State.bIsDead;
    // Do not serialize transient data like IK offsets
    return Ar;
}
```

## 5. Logistics

### A. Dependencies
*   **Modules:** `Core`, `CoreUObject`, `Engine`, `InputCore`
*   **Plugins:**
    *   `EnhancedInput` (Standard for Input)
    *   `ControlRig` (For IK solvers)

### B. Scalability Settings (The "Potato" Config)
**Target:** 60FPS on Integrated Graphics / Low-end Hardware.

| Setting | Console Variable | Value | Notes |
| :--- | :--- | :--- | :--- |
| **Upscaling** | `r.ScreenPercentage` | `50` | Use TSR or FSR |
| **Lumen** | `r.DynamicGlobalIlluminationMethod` | `0` | Disable Lumen |
| **Shadows** | `r.ShadowQuality` | `0` | Blob shadows only |
| **Fog** | `r.VolumetricFog` | `0` | Disable volumetric fog |
| **Shading** | `r.ForwardShading` | `1` | Consider for mobile/integrated |

### C. Profiling Checklist
- [ ] **Stat Unit:** Verify Frame Time (aim < 16.6ms). Ensure Game Thread is not the bottleneck (due to ticking actors).
- [ ] **Unreal Insights:** Check `UPlayerMovementSubsystem::Tick` cost. Check for Garbage Collection spikes (ensure no hard pointer cycles).
- [ ] **GPU Visualizer:** Check `ShadowDepths` cost. If high, ensure Character uses `bCastDynamicShadow` wisely or relies on cheap blob shadows in Potato Mode.
