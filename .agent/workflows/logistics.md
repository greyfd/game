---
description: Triggered by /logistics. Lists Build.cs dependencies, plugins, MVP success criteria, and potential edge cases/risks.
---

### COMMAND: /logistics
**Goal:** Roadmap and Performance validation. **CRITICAL:** Define the "Potato Mode" fallback for low-end hardware.

**Optimization Rules to Enforce:**
1.  **Potato Mode (Scalability):**
    * `r.ScreenPercentage=50` + TSR/FSR Upscaling.
    * `r.DynamicGlobalIlluminationMethod=0` (Disable Lumen).
    * `r.ForwardShading=1` (If target is Integrated Graphics/Mobile).
2.  **Lumen Optimization:** Set `r.Lumen.ScreenProbeGather.DownsampleFactor` for mid-range.

**Output Action:**
Edit the target `.md` file to append:

## 5. Logistics

**A. Dependencies**
* **Modules:** (Add `MassEntity` if ECS is used).

**B. Scalability Settings (The "Potato" Config)**
* List specific Console Variables (`r.ShadowQuality=0`, `r.VolumetricFog=0`) to ensure 60FPS on low-end hardware.

**C. Profiling Checklist**
- [ ] **Stat Unit:** Verify Game Thread vs GPU bottleneck.
- [ ] **Unreal Insights:** Check for Garbage Collection spikes.
- [ ] **GPU Visualizer:** Check "ShadowDepths" cost (verify VSM Invalidation).