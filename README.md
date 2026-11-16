# ⚡ Cache Memory Architecture & Mapping Visual Simulator
### SRM Institute of Science and Technology · Department of Computing Technologies
### 21CSS201T — Computer Organization and Architecture · Group 9 Project

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![HTML5 / CSS3 / Vanilla JS](https://img.shields.io/badge/Stack-HTML5%20%7C%20CSS3%20%7C%20Vanilla%20JS-brightgreen)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero%20(Pure%20Offline)-success)](index.html)
[![Course](https://img.shields.io/badge/Course-21CSS201T%20COA-orange)](https://www.srmist.edu.in)

> An interactive, high-performance web-based simulation workbench designed to visualize cache memory architectures, real-time address bit decomposition, hit/miss classification (The Three C's), replacement policies (LRU, FIFO, LFU, Random), and Average Memory Access Time (AMAT) analysis.

---

## 👥 Project Team (Group 9)

| Name | Registration Number | Department | Role / Contribution |
| :--- | :--- | :--- | :--- |
| **Sujoy Behera** | `RA2511033010057` | Computing Technologies, SRMIST | Core Simulation Engine, Address Bit Partitioning, 3 C's Classification |
| **Laasika Manjari Fogla** | `RA2511033010059` | Computing Technologies, SRMIST | Replacement & Write Policies, Multi-Architecture Shootout Studio |
| **Dhruvil Bharat Chawda** | `RA2511033010034` | Computing Technologies, SRMIST | Reactive UI/CSS Grid Layout, AMAT Analytics Engine, User Manual |

---

## 🚀 Key Features

* **Real-Time Address Bit Decomposition**:
  * Live visual bit-box displaying proportional allocation for **Tag (Purple)**, **Set Index (Blue)**, and **Block Offset (Emerald)**.
  * Dynamically computes formulas:
    $$\text{Offset Bits } (b) = \log_2(\text{Block Size})$$
    $$\text{Set Index Bits } (s) = \log_2(\text{Number of Sets})$$
    $$\text{Tag Bits } (t) = \text{Address Bits} - s - b$$
* **Multi-Architecture Support**:
  * **Direct-Mapped Cache** ($1$ line per set)
  * **$k$-Way Set-Associative Cache** (2-way, 4-way, 8-way)
  * **Fully Associative Cache** ($1$ unified set across all cache lines, zero conflict misses)
* **Configurable Cache Parameters**:
  * Address Widths: 8-bit, 12-bit, 16-bit, 32-bit
  * Cache Size: 16 Bytes to 512 Bytes
  * Line/Block Size: 2 Bytes to 16 Bytes
  * Replacement Policies: **LRU**, **FIFO**, **LFU**, **Random**
  * Write Policies: **Write-Back** (with live Dirty Bit $D=1$ tracking) vs. **Write-Through**
* **The Three C's Miss Classification (Mark Hill Model)**:
  * Automatically tags every miss as **Compulsory (Cold)**, **Conflict (Collision)**, or **Capacity**.
* **Interactive Step-Through & Reversibility**:
  * Forward stepping, backward stepping (full state undo snapshots), auto-play with adjustable speed (200ms–1800ms), and state reset.
* **Architecture Shootout Studio (Tab 2)**:
  * Runs any memory address trace simultaneously across Direct-Mapped, 2-Way, and Fully Associative caches side-by-side.
* **Guided Lab Experiments (Tab 3)**:
  * Pre-packaged one-click benchmarks demonstrating cache thrashing, policy divergence, block size scaling, and memory bus write traffic reduction.
* **Live AMAT Latency Calculator**:
  * Dynamically evaluates Average Memory Access Time:
    $$\text{AMAT} = T_{\text{hit}} + (1 - \text{Hit Rate}) \times T_{\text{penalty}}$$

---

## 📂 Repository Structure

```text
coa-cache-memory-simulator/
├── index.html                                        # Standalone web simulator (GitHub Pages ready)
├── Cache_Memory_Simulator.html                       # Standalone offline simulator backup
├── COA_Group9_Project_Report_and_User_Manual.docx    # Formatted Word Document for submission
├── COA_Group9_Project_Report_and_User_Manual.md      # Comprehensive technical report & viva guide
├── README.md                                         # Project documentation & overview
└── .gitignore                                        # Standard gitignore
```

---

## 🛠️ How to Run Locally

### Option 1: Direct Browser Launch (No Installation Required)
Simply clone the repository and open `index.html` in any web browser:

```bash
git clone https://github.com/SujoyBehera/coa-cache-memory-simulator.git
cd coa-cache-memory-simulator
# On Windows, open index.html directly:
start index.html
```

### Option 2: Live GitHub Pages
You can enable **GitHub Pages** in the repository settings (`Settings -> Pages -> Branch: main -> / (root) -> Save`) to host the interactive simulator live on the web at:
`https://sujoybehera.github.io/coa-cache-memory-simulator/`

---

## 🧪 Laboratory Experiments Included

1. **Experiment 1: Cache Thrashing in Direct-Mapped Caches**  
   * Demonstrates how alternating between addresses mapping to the same set index (`0, 64, 0, 64...`) causes a 0% hit rate, and how 2-Way Set Associativity resolves it (75% hit rate).
2. **Experiment 2: Replacement Policy Divergence (FIFO vs. LRU)**  
   * Demonstrates why LRU achieves superior hit rates over FIFO on cyclic memory access loops.
3. **Experiment 3: Spatial Locality & Block Size Scaling**  
   * Compares 2-byte, 4-byte, and 8-byte blocks on sequential memory access streams.
4. **Experiment 4: Write-Back (Dirty Bit) vs. Write-Through Bus Traffic**  
   * Measures physical memory bus transactions to show a 75%+ reduction in memory traffic via dirty-line buffering.

---

## 📚 Academic Grounding & Syllabus Alignment

Designed strictly for the **SRMIST 21CSS201T Computer Organization & Architecture** syllabus (Unit 5: Memory System). Covers:
* Cache Memory mapping functions (Direct, Associative, Set-Associative)
* Cache replacement algorithms & write strategies
* Performance evaluation & AMAT numerical formulations

---

## 📄 License
This project is open-source under the [MIT License](LICENSE).
