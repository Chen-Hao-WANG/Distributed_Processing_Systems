---
# Frontmatter based on user's rich example
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
title: 'Outatime: Using Speculation for Cloud Gaming'
info: |
  ## Outatime: Using Speculation to Enable Low-Latency Continuous Interaction for Cloud Gaming
  Presentation based on the OSDI '20 paper / Technical Report. Enhanced with on-slide explanations and progressive reveal for easier presentation without speaker notes. Simplified layout for better visibility. Final version.
class: text-center # Apply to the first slide
drawings:
  persist: false
transition: slide-left
mdc: true
---

# Outatime
## Using Speculation to Enable Low-Latency Continuous Interaction for Cloud Gaming

**Authors:** Kyungmin Lee, David Chu, Eduardo Cuervo, Johannes Kopf, Sergey Grizan, Alec Wolman, Jason Flinn
(University of Michigan, Microsoft Research, Siberian Federal University)

**Source:** OSDI '20 / Technical Report

**Presented By:** [您的姓名]
**Course:** [您的課程名稱/編號]
**Date:** [報告日期]

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
</div>

---
layout: default
---

# Introduction - The Promise and Problem of Cloud Gaming

### What is Cloud Gaming? [Source 2, 9]
<ul v-click-every>
<li>Servers execute & render games remotely.</li>
<li>Thin clients (your device) send input, display video stream.</li>
<li>Examples: GeForce Now, Xbox Cloud Gaming</li>
</ul>

### Advantages: [Source 10-14]
<ul v-click-every>
<li>High-end graphics on any device (no powerful hardware needed).</li>
<li>Solves platform compatibility & performance tuning for developers.</li>
<li>Easier server management (updates, fixes handled centrally).</li>
<li>Instant access to large game libraries (no downloads/installs).</li>
</ul>

<hr v-click/>

<div v-click>
### The MAJOR Problem: Latency! [Source 3, 16]
<ul v-click-every>
<li>WAN latencies often > 100ms (WiFi, Cellular, Wired). [Source 4, 22-24]</li>
<li>**Showstopper:** > 100ms feels sluggish, unacceptable for many games (esp. fast-paced ones requiring quick reflexes). [Source 4, 18, 19]</li>
<li>**Interactivity is Key:** Unlike video streaming, buffering doesn't work - actions need immediate feedback. [Source 25]</li>
</ul>

<div class="text-center pt-5">
<img src="https://i.imgur.com/hnJ3AYQ.png" alt="[雲端遊戲中的輸入延遲]" class="w-80 mx-auto" />
<p class="text-sm">Latency breaks the feeling of direct control and immersion.</p>
</div>
</div>

---
layout: default
---

# Outatime - Goal and Core Idea

### Paper's Goal: [Source 5, 26, 68]
- Mask network latency (up to 250ms).
- Provide a low-latency (<32ms frame time) gaming experience, similar to local play.

<hr v-click/>

<div v-click>
### Core Idea: Speculative Execution [Source 5, 27]
<ol v-click-every>
<li>*"Server guesses what you'll do next."*</li>
<li>Predicts future inputs OR explores possible outcomes.</li>
<li>Renders potential future frames *ahead of time*.</li>
<li>Sends frames *one RTT (Round Trip Time) early*. [Source 6, 70] (RTT: Signal travel time there-and-back)</li>
<li>Client displays the correct frame instantly upon input, effectively hiding network delay. [Source 7, 80, 82]</li>
</ol>
</div>

<div v-click class="flex items-center justify-center mt-4">
  <div class="flex flex-col items-center">
    <img src="https://i.imgur.com/5xV7LbL.png" alt="[傳統延遲 vs Outatime 延遲]" class="w-100 mx-auto" />
    <p class="text-sm text-center mt-2">Top: Standard Cloud Gaming (Input-to-Output delay includes RTT). Bottom: Outatime (Speculation hides RTT). [Source: Fig 1]</p>
  </div>
</div>

---
layout: default
---

# Outatime System Architecture

<div v-click class="text-center mb-4">
  <img src="https://i.imgur.com/ZZTIiUZ.png" alt="[Outatime 架構圖]" class="w-110 mx-auto" />
  <p class="text-sm text-center mt-1">Outatime Architecture: Shows different paths for Navigation and Impulse inputs. [Source: Fig 2 / Image 1]</p>
</div>

### Key Components:

**Server:** [Source 78]
<ul v-click-every>
<li>Receives client input (RX Input).</li>
<li>Performs **Input Prediction** (for Navigation) OR **Parallel Speculation** (for Impulse).</li>
<li>Applies techniques: Subsampling, Time Shifting, Rendering (Cube Maps), Joint Encoding.</li>
<li>Sends speculative bitstream (TX Bitstream).</li>
</ul>

**Client (Thin):** [Source 78]
<ul v-click-every>
<li>Receives bitstream (RX Bitstream).</li>
<li>Decodes video.</li>
<li>If misprediction occurs (Navigation), performs **View Interpolation** (Client-side correction).</li>
<li>Displays the final frame.</li>
<li>Sends user input (TX Input).</li>
<li>*(Kept thin to run on any device)*.</li>
</ul>

<hr v-click/>

<div v-click>
### Input Classification: [Source 84, 86, 92]
Handles different inputs differently. **Crucial insight!**
<ul v-click-every>
<li>**Navigation** (Movement/View): Continuous, predictable -> Use Prediction Model.</li>
<li>**Impulse** (Fire/Action): Discrete, unpredictable -> Use Parallel Exploration (explore multiple possibilities).</li>
<li>**Delay Tolerant** (HUD/Reload): Less critical, has cooldown -> Use Time Compression (play animation faster).</li>
</ul>
</div>

---
layout: default
---

# Speculation for Navigation Input (Movement/View)

### Challenge: [Source 85]
- Predict continuous view/movement changes accurately & smoothly.

### Prediction Model: Markov Chain [Source 100, 107]
<ul v-click-every>
<li>Predicts next state based only on current state (Simple but effective). [Source 101]</li>
<li>Uses *Supersampling* (higher input rate) for better accuracy. [Source 118]</li>
</ul>

### Two-tiered Error Handling:
<div v-click>
Based on expected error severity (related to RTT):
</div>
<ul v-click-every>
<li>**Low RTT (<40ms):** Minor errors -> Use **Kalman Filter** to smooth "video shake". (Purpose: Improve visual smoothness) [Source 105, 136]</li>
<li>**High RTT (>40ms):** Larger errors -> Use **Misprediction Compensation** (Client-side fix). (Purpose: Correct larger visual deviations locally) [Source 106, 130, 145]</li>
</ul>

<div class="grid grid-cols-2 gap-4 mt-4">
  <div v-click>
    <img src="https://i.imgur.com/azdx6gK.png" alt="[輸入預測準確度]" class="w-100 mx-auto" />
    <p class="text-sm text-center mt-1">Prediction Error CDF for Yaw (Left/Right Turn): Most errors are small (<4° imperceptible). [Source: Fig 4 / Image 2]</p>
  </div>
  <div v-click>
    <img src="https://i.imgur.com/qNF6TPw.png" alt="[導航預測隨時間變化]" class="w-100 mx-auto" />
    <p class="text-sm text-center">Example: Markov prediction (red) vs. actual user path (blue).</p>
  </div>
</div>

---
layout: default
---

# Misprediction Compensation: View Interpolation

### Goal: [Source 38, 146]
- If server's prediction was wrong, client corrects the received frame *locally* without waiting for server response. (**Client fixes it itself!**)

### Technique: View Interpolation [Source 40, 147]
<ul v-click-every>
<li>Server sends: Predicted Frame (f') + **Depth Info** (f^Δ). [Source 148] (Depth: Distance of each pixel)</li>
<li>Client uses actual input to calculate correct viewpoint.</li>
<li>Client generates corrected frame (f'') by **re-projecting/warping** f' using f^Δ. [Source 149] (Uses depth to "stretch" image to new view)</li>
</ul>

### Requirement & Solution: [Source 157, 160, 161]
<div v-click>
- Needs depth data & a wider original view (to avoid missing pixels after warping).
- **Solution:** Server renders a **Cube Map** (panoramic view) + Depth Map.
  - *(Analogy: Like being inside a box and seeing all 6 walls.)*
</div>

<div class="grid grid-cols-2 gap-4 mt-4">
  <div v-click>
    <img src="https://i.imgur.com/RYSb5wF.png" alt="[視點內插範例]" class="w-100 mx-auto" />
    <p class="text-sm text-center">View interpolation example: Client corrects viewpoint locally using depth. [Source: Fig 6]</p>
  </div>
  <div v-click>
    <img src="https://i.imgur.com/Yjf37VY.png" alt="[立方體貼圖渲染]" class="w-100 mx-auto" />
    <p class="text-sm text-center">Cube map rendering provides the necessary wider field of view. [Source: Fig 7]</p>
  </div>
</div>

---
layout: default
---

# Optimizing View Interpolation: Clipped Cube Maps

### Problem: [Source 164-167]
- Full Cube Map + Depth = Huge Overhead (~12x Bandwidth/Render Cost).

### Observation: [Source 169, 170, 178]
- Prediction errors usually stay within a limited angular range. (Prediction isn't usually *wildly* wrong).

### Solution: Clipped Cube Map [Source 171]
<ul v-click-every>
<li>Render only the *necessary portion* of the Cube Map.</li>
<li>Covers expected range of prediction errors (e.g., 99% coverage). [Source 176, 177]</li>
<li>**Key Benefit:** Significantly reduces rendering and bandwidth overhead while still correcting most errors. [Source 178]</li>
</ul>

<div class="grid grid-cols-2 gap-4 mt-4">
  <div v-click>
    <img src="https://i.imgur.com/qJjvBD8.png" alt="[預測錯誤分佈]" class="w-100 mx-auto" />
    <p class="text-sm text-center">Prediction errors cover limited range (99% coverage shown). [Source: Fig 8 / Image 3 right]</p>
  </div>
  <div v-click>
    <img src="https://i.imgur.com/lnOtbMc.png" alt="[裁剪過的立方體貼圖]" class="w-100 mx-auto" />
    <p class="text-sm text-center">Clipped cube map only renders the likely needed area. [Source: Fig 7 / Image 3 left]</p>
  </div>
</div>

---
layout: default
---

# Speculation for Impulse Input (Actions)

### Challenge: [Source 33, 86-89, 189]
- Sporadic events (fire!) are hard to predict.
- Misprediction causes jarring visual/semantic errors (enemy reappears!). (Prediction errors are very noticeable and break gameplay).

### Approach: Parallel Timeline Speculation [Source 34, 89, 190]
<ul v-click-every>
<li>Server explores *multiple possible futures* in parallel (e.g., Fire vs. No Fire).</li>
<li>Renders the final frame for *each* possible timeline.</li>
<li>Sends *all* potential outcomes to the client.</li>
<li>Client picks the frame matching the actual input. [Source 191, 192]</li>
</ul>

### Problem: State Space Explosion! [Source 35, 193, 195]
<div v-click>
- *(Analogy: If every step has 2 choices, possibilities explode quickly!)*
- Too many futures ($2^{\lambda}$) to simulate efficiently.
</div>

<div v-click class="flex items-center justify-center mt-4">
  <div class="flex flex-col items-center">
    <img src="https://i.imgur.com/uL1fk1I.png" alt="[平行時間線推測]" class="w-80 mx-auto" />
    <p class="text-sm text-center mt-2">Exploring multiple possible futures (timelines) in parallel for impulse events.</p>
  </div>
</div>

---
layout: default
---

# Taming State Explosion: Subsampling & Time-Shifting

### Goal:
- Balance perceived responsiveness vs. server load. (Find a sweet spot!)

### Technique 1: Subsampling [Source 196, 197]
<ul v-click-every>
<li>Only allow impulse events to register at specific intervals ($\sigma > 1$ tick). (Limit *when* branches can occur).</li>
<li>Reduces state space ($2^{\lambda/\sigma}$). [Source 198]</li>
</ul>

### Technique 2: Time-Shifting [Source 199, 200]
<ul v-click-every>
<li>Shift actual event time slightly (forward/backward) to align with the nearest interval tick. [Source 202] ("Nudge" events to fit the grid).</li>
<li>Preserves the *feeling* of responsiveness (small shifts are often imperceptible).</li>
</ul>

### Result: [Source 205]
<div v-click>
- Bounds speculation (e.g., max 4 timelines for RTT ≤ 256ms). (Keeps computation manageable).
</div>

<div v-click class="flex items-center justify-center mt-4">
  <div class="flex flex-col items-center">
    <img src="https://i.imgur.com/VU5jK6L.png" alt="[子採樣與時間平移]" class="w-100 mx-auto" />
    <p class="text-sm text-center mt-2">Subsampling & Time-shifting limit branches. Events (X) are shifted to interval ticks (arrows). [Source: Fig 9 / Image 4]</p>
  </div>
</div>

---
layout: default
---

# Handling Delay Tolerant Input & Reducing Bandwidth

### Delay Tolerant Events [Source 92, 211]
<ul v-click-every>
<li>Events with long cool-downs > RTT (e.g., Reload). [Source 213]</li>
<li>**Approach: Time Compression** [Source 217-220]
  - Tolerate missing first RTT's consequences.
  - Play animation faster during cooldown to "catch up".</li>
</ul>

<hr v-click/>

<div v-click>
### Bandwidth Challenge [Source 42, 238]
- Sending multiple speculative frames = expensive.
- **Observation:** Speculative frames look very similar. [Source 241, 243, 245] (High redundancy between frames).
</div>

<div v-click>
### Solution: Joint Video Encoding [Source 43, 246]
- *"Encode only the differences across frames."*
<ul v-click-every>
<li>Extend standard video encoding (H.264).</li>
<li>Find similar blocks (macroblocks) *across* different speculative streams.</li>
<li>Use pointers for redundancy -> lower bitrate. [Source 248]</li>
</ul>
</div>

<div v-click class="flex items-center justify-center mt-4">
  <img src="https://i.imgur.com/8hQvSjv.png" alt="[聯合視訊編碼]" class="w-100 mx-auto" />
  <p class="text-sm text-center">Joint encoding finds similarities across speculative frames to save bandwidth.</p>
</div>

---
layout: default
---

# Implementation Highlights

### Key Challenges & Solutions
<ul v-click-every>
<li>**Game Modification:** Done for Doom 3, Fable 3; deemed generalizable (can be applied to other games). [Source 250, 252]</li>
<li>**Parallel Execution:** Master-Slave architecture used for running speculative timelines concurrently. [Source 259]</li>
<li>**Efficient Checkpoint/Restore:** *Critical* for performance! Needed custom hybrid Page-Level (CoW) + Object-Level approach for speed. [Source 41, 266, 269, 270, 274, 275, 284]</li>
<li>**GPU Acceleration:** Leveraged heavily for rendering, video encoding/decoding, and client-side view interpolation. [Source 287-292]</li>
</ul>

<div v-click class="flex items-center justify-center mt-4">
  <img src="https://i.imgur.com/DG3kmPV.png" alt="[實作架構]" class="w-80 mx-auto" />
  <p class="text-sm text-center">Implementation uses master-slave processes for parallel speculation.</p>
</div>

---
layout: default
---

# Evaluation - Methodology & Baselines

### Methods [Source 293]
- User Studies (Subjective Experience)
- Performance Benchmarks (Objective Data)

### Games Tested [Source 295, 296]
- Doom 3 (Latency-sensitive FPS)
- Fable 3 (Action RPG - Verify results)

### Systems Compared [Source 302, 303]
<ul v-click-every>
<li>**Outatime**: Proposed system (with speculation)</li>
<li>**Standard Thin Client**: Traditional cloud gaming (baseline - experiences full latency; 傳統雲端遊戲，承受完整延遲)</li>
<li>**Standard Fat Client**: Local execution (ideal - zero network latency; 遊戲在本地跑，理想對照組)</li>
</ul>

<hr v-click/>

<div v-click>
### User Study Metrics [Source 315-318]
<ul v-click-every>
<li>**Mean Opinion Score (MOS)**
  - Subjective rating (1-5) *(5=Best)* - How did it feel?</li>
<li>**Skill Impact**
  - Player health remaining - Did latency affect performance?</li>
<li>**Task Completion Time**
  - Time to complete standard tasks - Were players slower?</li>
</ul>
</div>

<div v-click class="flex items-center justify-center mt-4">
  <img src="https://i.imgur.com/vCDR9fZ.png" alt="[測試設置]" class="w-80 mx-auto" />
  <p class="text-sm text-center">User study setup with controlled network conditions.</p>
</div>

---
layout: default
---

# Evaluation - Key User Study Results (Impulse)

*(Focus: How did speculation impact user experience and performance?)*

<div class="grid grid-cols-3 gap-4">

<div>
<h4 class="text-center">Mean Opinion Score (MOS)</h4>
<div v-click>
- Outatime stays high (~4.0-4.5) up to 256ms RTT.
- Thin Client drops rapidly.
*(MOS > 4 = acceptable/good experience)* [Source 328-333]
</div>
<img src="https://i.imgur.com/kx2XoCV.png" alt="[MOS 結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">MOS vs. RTT: Outatime maintains good subjective experience. [Source: Fig 10]</p>
</div>

<div>
<h4 class="text-center">Player Health Remaining</h4>
<div v-click>
- Outatime players maintain performance (health).
- Thin Client players suffer significantly. [Source 340-343]
</div>
<img src="https://i.imgur.com/m1yITHD.png" alt="[生命值結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">Health vs. RTT: Outatime preserves player skill/performance. [Source: Fig 11]</p>
</div>

<div>
<h4 class="text-center">Task Completion Time</h4>
<div v-click>
- Outatime stable (fast).
- Thin Client slower with RTT. [Source 347]
</div>
<img src="https://i.imgur.com/XCJuQfm.png" alt="[任務時間結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">Task Time vs. RTT: Outatime maintains task efficiency. [Source: Fig 12]</p>
</div>

</div>

<div v-click class="text-center mt-4">
<strong>Key Takeaway</strong>: Users strongly prefer Outatime and perform better with it at higher latencies.
</div>

---
layout: default
---

# Evaluation - System Performance & Overhead

*(Focus: What are the costs of using Outatime?)*

<div class="grid grid-cols-3 gap-4">

<div>
<h4 class="text-center">Client Frame Time</h4>
<div v-click>
- Outatime = Low & Stable (<32ms), like local execution.
- Thin Client = Depends directly on RTT. [Source 361, 364]
</div>
<img src="https://i.imgur.com/8YWJrxT.png" alt="[客戶端畫面時間結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">Client Frame Time CDF: Outatime achieves target frame time. [Source: Fig 13]</p>
</div>

<div>
<h4 class="text-center">Server Load</h4>
<div v-click>
- Outatime adds overhead (speculation, checkpointing, rendering).
- **Result:** Manageable within frame budget (avg < 32ms). [Source 366-369]
</div>
<img src="https://i.imgur.com/tZrD1Py.png" alt="[伺服器負載結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">Server Processing Time: Overhead increases with RTT but is manageable. [Source: Fig 16 left]</p>
</div>

<div>
<h4 class="text-center">Bandwidth</h4>
<div v-click>
- Outatime uses more bandwidth (1.5x - 4.5x).
- **Mitigation:** Joint Encoding & Clipping significantly reduce cost. [Source 51, 301, 371-376]
</div>
<img src="https://i.imgur.com/xnRVj2Q.png" alt="[頻寬結果]" class="w-full mt-2" />
<p class="text-xs text-center mt-1">Bitrate CDF: Optimizations reduce bandwidth overhead. [Source: Fig 17 right]</p>
</div>

</div>

<div v-click class="text-center mt-4">
<strong>Overall Trade-off</strong>: Higher resource cost (server CPU, bandwidth) for *much* better user experience. [Source 52] (**Worthwhile trade-off!**)
</div>

---
layout: default
---

# Conclusion

### Problem Recap [Source 3, 16]
- Network latency is a major blocker for high-quality cloud gaming.

### Outatime's Contribution [Source 5, 26]
<div v-click>
- **Demonstrates feasibility:** Speculative execution *can effectively mask* high latency (up to 250ms) for interactive cloud gaming.
</div>

### Key Techniques Recap
<div v-click>
- Input Prediction (Navigation)
- Parallel Timelines (Impulse - using Subsampling/Time-Shifting)
- Misprediction Compensation (View Interpolation/Clipping)
- Joint Encoding (Bandwidth)
</div>

<hr v-click/>

<div v-click>
### Results Summary [Source 394, 395]
- Dramatically improves perceived responsiveness & user satisfaction.
- Cost (CPU/Bandwidth) is manageable due to optimizations.
</div>

<div v-click>
### Impact [Source 395]
- **Makes high-latency cloud gaming practical and enjoyable.**
- Demonstrates the power of application-specific speculation for latency hiding.
</div>

<div v-click>
### Discussion Point / Future
- Applicability to other interactive remote apps (VR/AR, remote desktop)?
- Further reducing resource costs?
</div>

<div v-click class="flex items-center justify-center mt-4">
  <img src="https://i.imgur.com/5xV7LbL.png" alt="[傳統延遲 vs Outatime 延遲]" class="w-80 mx-auto" />
</div>

---
layout: center
class: text-center
---

# Thank You!

[Paper Link (OSDI '20 Version)](https://www.usenix.org/system/files/osdi20-lee.pdf)

Questions?

