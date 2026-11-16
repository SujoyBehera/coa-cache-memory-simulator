# SRM INSTITUTE OF SCIENCE AND TECHNOLOGY
### Department of Computing Technologies · College of Engineering and Technology
### 21CSS201T — Computer Organization and Architecture
### Group Project Deliverable & Comprehensive User Manual

---

# PROJECT TITLE:
## Cache Memory Architecture & Mapping Visual Simulator
### *(Direct, Associative, Set-Associative with FIFO/LRU Hit-Miss Tracker & AMAT Analyzer)*

**Academic Year**: 2026–2027 | **Semester**: III (2nd Year B.Tech)  
**Project Group**: **Group 9**

### **Team Members**:
1. **Sujoy Behera** — Reg No: `RA2511033010057`
2. **Laasika Manjari Fogla** — Reg No: `RA2511033010059`
3. **Dhruvil Bharat Chawda** — Reg No: `RA2511033010034`

---

## 1. Project Abstract & Objectives

Modern computer architectures rely heavily on hierarchical memory subsystems to bridge the widening speed gap between ultra-fast processor clock rates and relatively slow dynamic RAM (DRAM). Cache memory provides high-speed SRAM buffers near the CPU by exploiting the twin properties of **Temporal Locality** (reuse of recently referenced data) and **Spatial Locality** (accessing memory locations physically adjacent to recent data).

The objective of this project is to develop a production-ready, interactive, single-page web simulation workbench (**Cache Memory Architecture & Mapping Visual Simulator**) that visually and mathematically demonstrates:
1. Real-time decomposition of memory addresses into **Tag**, **Set Index**, and **Block Offset** fields across configurable address bit-widths (8, 12, 16, 32 bits).
2. Live simulation of the three fundamental cache mapping architectures:
   - **Direct-Mapped Cache** ($1$ line per set)
   - **Set-Associative Cache** ($k$-way: 2-way, 4-way, 8-way)
   - **Fully Associative Cache** ($1$ unified set across all cache lines)
3. Dynamic replacement policy behavior comparing **LRU (Least Recently Used)**, **FIFO (First-In, First-Out)**, **LFU (Least Frequently Used)**, and **Random Replacement**.
4. Memory write policy emulation: **Write-Back** with live Dirty Bit ($D$) state tracking vs. **Write-Through**.
5. Automated **Three C’s Miss Classification** (Compulsory/Cold, Conflict, Capacity Misses) on every memory access.
6. Real-time **Average Memory Access Time (AMAT)** computation based on live hit/miss ratios, hit latency ($T_{\text{hit}}$), and miss penalty ($T_{\text{penalty}}$).
7. One-click guided academic experiments demonstrating cache thrashing, policy divergence, block size scaling, and memory bus traffic reduction.

---

## 2. Theoretical Architecture & Mathematical Framework

### 2.1 The Principle of Locality
* **Temporal Locality**: If an address is accessed at time $t$, it is highly probable that the exact same address will be accessed again at time $t + \Delta t$ (e.g., loop control counters, function call frames, accumulator variables).
* **Spatial Locality**: If memory address $A$ is referenced, neighboring addresses $A+1, A+2, \dots$ are likely to be accessed soon (e.g., sequential instruction execution, sequential array traversals).

### 2.2 Mathematical Bit-Partitioning Formulations
Given:
* Address space size = $N$ bits (Addressable bytes = $2^N$)
* Total Cache Capacity = $C$ bytes
* Block / Line Size = $B$ bytes
* Associativity = $k$ ways (where $k=1$ is Direct-Mapped, $k=L$ is Fully Associative)

The cache dimensions and bit fields are derived as follows:

1. **Total Number of Cache Lines ($L$)**:
   $$\text{Total Lines } L = \frac{\text{Cache Capacity } C}{\text{Block Size } B}$$

2. **Total Number of Sets ($S$)**:
   $$\text{Number of Sets } S = \frac{\text{Total Lines } L}{\text{Associativity } k} = \frac{C}{B \times k}$$

3. **Block Offset Field ($b$ bits)**:
   $$b = \log_2(B)$$
   *Identifies the specific byte within a multi-byte cache line.*

4. **Set Index Field ($s$ bits)**:
   $$s = \log_2(S)$$
   *Selects the exact set (row) within the cache table. (In Fully Associative, $S=1 \implies s = \log_2(1) = 0$ bits).*

5. **Tag Field ($t$ bits)**:
   $$t = N - s - b$$
   *Uniquely verifies whether the block stored in that cache set belongs to the requested memory address.*

---

### 2.3 Cache Mapping Techniques Comparison

| Characteristic | Direct-Mapped ($k=1$) | $k$-Way Set-Associative | Fully Associative ($k=L$) |
| :--- | :--- | :--- | :--- |
| **Set Count ($S$)** | $S = L$ sets | $S = L / k$ sets | $S = 1$ unified set |
| **Set Index Bits ($s$)** | $\log_2(L)$ bits | $\log_2(L/k)$ bits | **$0$ bits** |
| **Lines per Set** | Exactly $1$ line | Exactly $k$ lines | All $L$ lines |
| **Placement Freedom** | Strict: $\text{Block} \pmod L$ | Any of the $k$ ways in $\text{Block} \pmod S$ | Any line anywhere in cache |
| **Hardware Comparators** | $1$ comparator (fastest, cheapest) | $k$ parallel comparators | $L$ parallel comparators (expensive CAM) |
| **Conflict Misses** | High (Severe Thrashing Risk) | Low (Conflict largely eliminated for $k \ge 4$) | **Zero Conflict Misses** |

---

### 2.4 The Three C's of Cache Misses (Mark Hill Taxonomy)
1. **Compulsory Miss (Cold Miss)**:
   * Occurs on the very first reference to a memory block. The data has never been loaded into the cache before.
   * *Mitigation*: Larger block sizes (exploits spatial locality) or hardware prefetching.
2. **Conflict Miss (Collision Miss)**:
   * Occurs in Direct-Mapped or Set-Associative caches when multiple distinct memory blocks map to the exact same set index, evicting an active block even though other sets in the cache are completely empty.
   * *Mitigation*: Increasing associativity ($k$-way) or skewing hash functions.
3. **Capacity Miss**:
   * Occurs when the working set of the running program exceeds the total physical capacity of the cache ($C$). The block was previously in the cache, but was evicted due to space exhaustion.
   * *Mitigation*: Increasing the total cache size ($C$).

---

### 2.5 Replacement Policies
When a cache miss occurs and all $k$ lines in the designated set are full, a victim block must be chosen for eviction:
* **LRU (Least Recently Used)**: Evicts the line whose last reference timestamp is oldest. Exploits temporal locality optimal for loops.
* **FIFO (First-In, First-Out)**: Evicts the line that entered the cache earliest, regardless of how frequently or recently it was accessed.
* **LFU (Least Frequently Used)**: Evicts the line with the smallest reference counter.
* **Random**: Evicts a pseudo-randomly selected line, requiring zero tracking hardware.

---

### 2.6 Cache Write Policies
* **Write-Through**: Every write operation updates both the cache line and main memory simultaneously. Maintains immediate memory consistency, but incurs high memory bus write traffic.
* **Write-Back**: CPU writes only to the cache line and marks the **Dirty Bit ($D=1$)**. Main memory is updated only when that dirty block is evicted to make room for a new block. Slashes memory bus traffic by over 80%.

---

### 2.7 Average Memory Access Time (AMAT)
$$\text{AMAT} = T_{\text{hit}} + (1 - \text{Hit Rate}) \times T_{\text{penalty}}$$
*Where $T_{\text{hit}}$ is the cache lookup time (e.g., 1 clock cycle) and $T_{\text{penalty}}$ is the latency to fetch a block from main memory (e.g., 20–50 clock cycles).*

---

## 3. Simulator User Manual

### 3.1 Launching the Application
1. Locate `Cache_Memory_Simulator.html` (in your Downloads folder or project repository).
2. Double-click the file to open it in any modern web browser (Google Chrome, Microsoft Edge, Mozilla Firefox, Brave, Safari).
3. The simulator has **zero external software dependencies** and runs offline.

### 3.2 Main Workspace Navigation
The application features 4 dedicated tabs accessible from the top navigation bar:
* **Tab 1: 🎛️ Interactive Simulator**: The primary workbench featuring live address bit partitioning, step-by-step trace stepping, interactive cache structure table, and live AMAT analytics.
* **Tab 2: ⚖️ Architecture Shootout**: Executes your address trace simultaneously across Direct-Mapped, 2-Way Set-Associative, and Fully Associative caches side-by-side.
* **Tab 3: 🧪 Guided Lab Experiments**: 4 pre-packaged academic experiments that configure the simulator and execute live demonstrations with one click.
* **Tab 4: 📚 COA Theory & Viva Cheat-Sheet**: Comprehensive syllabus equations, architectural diagrams, and top 10 viva exam questions with answers.

### 3.3 Operating the Interactive Simulator (Tab 1)
1. **Configure Architecture Parameters**:
   * Set **Address Width**: Select 8, 12, 16, or 32 bits.
   * Set **Cache Size**: Select from 16 Bytes to 512 Bytes.
   * Set **Block Size**: Select from 2 Bytes to 16 Bytes.
   * Set **Associativity**: Choose Direct Mapped, 2-Way, 4-Way, or Fully Associative.
   * Select **Replacement Policy**: Choose LRU, FIFO, LFU, or Random.
   * Select **Write Policy**: Choose Write-Back (WB) or Write-Through (WT).
2. **Observe the Bit Decomposition Box**:
   * The top widget immediately recalculates and visualizes the exact proportional bit-widths of the **Tag (Purple)**, **Set Index (Blue)**, and **Block Offset (Emerald)** fields along with live logarithmic formulas.
3. **Enter Memory Address Trace**:
   * Enter comma- or space-separated byte addresses in the input box (e.g., `0, 4, 8, 12, 0, 4, 16, 20` or hex `0x00, 0x10, 0x20`).
   * Prefix with `W:` for write operations (e.g., `W:0, R:4`).
   * Or click any **Quick Preset Chip**:
     * `📦 Spatial Locality`
     * `🔄 Temporal Loop`
     * `⚠️ Conflict Thrashing`
     * `⚔️ LRU vs FIFO`
     * `📊 Matrix Access`
4. **Execution Controls**:
   * Click **`Next ▶`**: Advance the simulation by exactly one memory access.
   * Click **`◀ Back`**: Step backward in time (full state undo).
   * Click **`▶ Play`**: Auto-play through the sequence at adjustable speeds (200ms to 1800ms).
   * Click **`↺ Reset`**: Flush the cache and reset all metrics to zero.
5. **Observing the Cache Table & Metrics**:
   * Active lines pulse with high-contrast animations:
     * **Green Glow (`row-hit`)**: Cache Hit.
     * **Red/Pink Pulse (`row-miss-fill`)**: Cache Miss loaded into empty line.
     * **Amber Pulse (`row-miss-evict`)**: Cache Miss triggering line eviction.
   * Monitor the **Hit Ratio gauge**, **Miss Classification (Compulsory, Conflict, Capacity)**, and live **AMAT cycle counter**.
   * Click **`Export CSV`** to download a spreadsheet record of the full access trace.

---

## 4. Laboratory Experiments & Verification Test Cases

### Experiment 1: Direct Mapping Thrashing & Conflict Miss Resolution
* **Scenario**: Two memory blocks, Block 0 (Address 0) and Block 16 (Address 64), are repeatedly accessed in an alternating loop: `0, 64, 0, 64, 0, 64, 0, 64`.
* **Configuration**: Cache Size = 64 Bytes, Block Size = 4 Bytes ($L = 16$ lines).
* **Direct-Mapped Tracing ($k=1, S=16$)**:
  * Address 0 $\implies$ Block 0 $\implies \text{Set } 0 \pmod{16} = 0$.
  * Address 64 $\implies$ Block 16 $\implies \text{Set } 16 \pmod{16} = 0$.
  * **Result**: Every single access evicts the previous one from Set 0!
  * **Hit Rate: 0.0%** (Catastrophic Thrashing) despite 15 empty cache lines.
* **2-Way Set-Associative Resolution ($k=2, S=8$)**:
  * Set 0 now holds 2 ways!
  * Step 1 (Addr 0): Cold Miss $\implies$ loaded into Set 0, Way 0.
  * Step 2 (Addr 64): Cold Miss $\implies$ loaded into Set 0, Way 1.
  * Steps 3–8: Both blocks reside in cache simultaneously $\implies$ **100% Hits**!
  * **Overall Hit Rate: 75.0%** (Saves 6 out of 8 accesses).

---

### Experiment 2: Replacement Policy Divergence (FIFO vs. LRU)
* **Configuration**: Cache Size = 16 Bytes, Block Size = 4 Bytes, 2-Way Set-Associative ($L=4, S=2, \text{Ways}=2$).
* **Access Trace**: `0, 4, 8, 12, 0, 16, 0, 4, 8`
* **Observation**:
  * In Set 0, Blocks 0, 2, and 4 compete for 2 ways.
  * When Block 0 is re-accessed, LRU refreshes its timestamp. When Block 4 (Addr 16) arrives, LRU evicts Block 2 instead of Block 0.
  * FIFO does not update line age on hits; it blindly evicts Block 0 because it arrived first, causing an immediate subsequent miss on the next access to Block 0.
  * **Conclusion**: LRU achieves higher hit rates by respecting temporal locality.

---

### Experiment 3: Spatial Locality & Block Size Scaling
* **Access Trace**: Sequential byte access `0, 1, 2, 3, 4, 5, 6, 7` (e.g., streaming through an array of 8 bytes).
* **Comparison**:
  * With **2-Byte Blocks**: Addresses 0, 2, 4, 6 each cause a compulsory miss (4 misses / 8 accesses) $\implies$ **Hit Rate = 50.0%**.
  * With **4-Byte Blocks**: Addresses 0 and 4 cause misses; addresses 1, 2, 3 and 5, 6, 7 hit in cache $\implies$ **Hit Rate = 75.0%**.
  * With **8-Byte Blocks**: Address 0 causes 1 cold miss, prefetching all 8 bytes $\implies$ **Hit Rate = 87.5%**.

---

### Experiment 4: Write-Back Dirty Bit vs. Write-Through Bus Traffic
* **Access Trace**: `W:0, W:0, W:0, W:0, R:32` (Repeatedly writing to Address 0, followed by an eviction by Address 32).
* **Write-Through Analysis**: Every `W:0` instruction immediately writes to the main memory bus $\implies$ **4 Main Memory Bus Transactions**.
* **Write-Back Analysis**: The four `W:0` instructions update the cache line in SRAM and set $D=1$. Only when Address 32 evicts the block is **1 single writeback to RAM** performed $\implies$ **75% reduction in memory bus traffic**!

---

## 5. Viva-Voce Examination Preparation (Top 12 Questions & Answers)

1. **Q: Why does a cache memory access time depend on associativity?**  
   *A*: Increasing associativity reduces conflict misses, but slightly increases hit time ($T_{\text{hit}}$) because parallel comparators and an output multiplexer are required to select the matching way.
2. **Q: How do you calculate the tag bit width for a 16-bit address with a 1 KB cache, 16-byte block size, and 4-way set-associativity?**  
   *A*:  
   * Offset $b = \log_2(16) = 4$ bits.  
   * Total lines $L = 1024 / 16 = 64$.  
   * Sets $S = 64 / 4 = 16 \implies s = \log_2(16) = 4$ bits.  
   * Tag $t = 16 - 4 - 4 = 8$ bits.
3. **Q: What is the purpose of the Valid Bit?**  
   *A*: It indicates whether the cache line contains valid, coherent data copied from main memory. On system boot or cache flush, all valid bits are set to 0.
4. **Q: What is the purpose of the Dirty Bit?**  
   *A*: Used exclusively in Write-Back caches to track whether data in the cache line was modified by the CPU. If $D=1$, the line must be written back to RAM upon eviction.
5. **Q: Why are there zero Set Index bits in a Fully Associative cache?**  
   *A*: Because there is only $1$ unified set ($S=1$). Since $\log_2(1) = 0$, all address bits (excluding block offset) serve as the Tag.
6. **Q: What is cache thrashing?**  
   *A*: A failure mode where two or more actively used memory blocks map to the exact same cache set, causing them to repeatedly evict each other and driving the hit rate toward 0%.
7. **Q: How does increasing block size affect the miss rate?**  
   *A*: Increasing block size initially reduces compulsory misses due to spatial locality. However, if the block size becomes too large relative to cache capacity, the total number of lines drops, causing conflict and capacity misses to rise.
8. **Q: What is a write buffer in a Write-Through cache?**  
   *A*: A small FIFO hardware buffer that holds store data awaiting transmission to main memory, allowing the CPU to proceed immediately without stalling for DRAM write completion.
9. **Q: Distinguish between Write-Allocate and No-Write-Allocate on a write miss.**  
   *A*: In Write-Allocate (common with Write-Back), the missing block is loaded into the cache and modified. In No-Write-Allocate (common with Write-Through), the write is sent straight to main memory without bringing the block into cache.
10. **Q: What is Belady’s Anomaly and can it occur in cache replacement?**  
    *A*: Belady's Anomaly (where increasing capacity increases miss count) can occur in FIFO replacement, but is mathematically impossible in LRU because LRU belongs to the family of **Stack Algorithms**.
11. **Q: Define AMAT and explain how a second-level (L2) cache improves it.**  
    *A*: $\text{AMAT} = T_{\text{hit\_L1}} + \text{MissRate}_{\text{L1}} \times (T_{\text{hit\_L2}} + \text{MissRate}_{\text{L2}} \times T_{\text{penalty\_RAM}})$. L2 cache intercepts L1 misses, replacing a slow DRAM penalty (e.g., 50 cycles) with a fast SRAM lookup (e.g., 8 cycles).
12. **Q: Why is true LRU impractical for 16-way or 32-way caches?**  
    *A*: Maintaining true LRU requires $O(k^2)$ bits or sorting counters per set, requiring excessive hardware. Modern microprocessors use **Pseudo-LRU (Tree-PLRU or Bit-PLRU)** requiring only $k-1$ bits per set.

---

## 6. Team Contributions & Division of Work

* **Sujoy Behera (`RA2511033010057`)**:
  * Core Cache Simulation Engine architecture and state machine implementation.
  * Dynamic address partitioning algorithms (Tag, Index, Offset) and bitwise manipulation.
  * Three C's miss classification logic (Compulsory vs. Conflict vs. Capacity).
* **Laasika Manjari Fogla (`RA2511033010059`)**:
  * Replacement policy implementations (LRU timestamp ordering, FIFO queue, LFU frequency counters).
  * Write policy handlers (Write-Through bus metrics and Write-Back dirty bit tracking).
  * Architecture Shootout comparison studio and experimental verification test cases.
* **Dhruvil Bharat Chawda (`RA2511033010034`)**:
  * High-performance reactive UI layout, CSS grid architecture, and pulsating visual state animations.
  * AMAT real-time calculator and SVG metric gauge data binding.
  * Comprehensive project documentation, user manual, and viva preparation guide.

---
*SRM Institute of Science and Technology · Department of Computing Technologies · 2026*
