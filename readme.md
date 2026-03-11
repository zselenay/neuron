#  Biological Memory Simulation
### Modeling the Effect of Stress (Cortisol) on Memory Encoding and Recall

---

## 📌 Overview

This project simulates the interaction between the **Hippocampus** (memory center) and the **Amygdala** (emotion/stress center) in the human brain. It demonstrates why memories cannot be fully encoded or recalled during periods of high stress — using real neuroscience, differential equations, and associative memory networks.

The core question driving this project:

> *"A memory that remains 'hidden' in the system — with what minimum trigger data does it become visible again?"*

---

##  The Science Behind It

### Why Does Stress Break Memory?

When danger is perceived, the **Amygdala** fires an alarm and triggers the release of **cortisol** — the stress hormone. Cortisol travels to the hippocampus and does two things:

1. **Raises the firing threshold** of neurons → they need much more input to fire → fewer spikes → weaker memory encoding
2. **Reduces synaptic plasticity** → neurons can't strengthen their connections properly → the memory engram forms incompletely

A small amount of cortisol actually *enhances* memory (which is why emotional memories are vivid). But high cortisol *fragments* it — the memory is stored, but becomes inaccessible.

This project models the high-cortisol scenario: memory under stress.

---

##  How It Works — Layer by Layer

### Layer 1: Single Neuron Model (LIF)

Each neuron is modeled using the **Leaky Integrate-and-Fire (LIF)** equation:

```
τ · (dVm/dt) = -(Vm - V_rest) + R · I(t)
```

| Parameter | Description | Value |
|-----------|-------------|-------|
| `V_rest` | Resting potential | -70 mV |
| `V_threshold` | Firing threshold | -55 mV (baseline) |
| `V_reset` | Post-spike reset | -70 mV |
| `τ (tau)` | Time constant | 20 ms |
| `R` | Membrane resistance | 1 |
| `dt` | Time step (Euler method) | 0.1 ms |

The equation is solved numerically using the **Euler Method**:
```
Vm(t + dt) = Vm(t) + dt · (dVm/dt)
```

When `Vm ≥ V_threshold`, the neuron fires, voltage resets, and a **refractory period** (5 ms) begins.

---

### Layer 2: Cortisol (Stress) Mechanism

A `stress_level` parameter (0.0 – 1.0) modifies the firing threshold linearly:

```
V_threshold = -55 + (stress_level × 15)
```

| Stress Level | Threshold | Effect |
|-------------|-----------|--------|
| 0.0 | -55 mV | Normal firing |
| 0.5 | -47.5 mV | Reduced activity |
| 0.9 | -41.5 mV | Near silence |
| 1.0 | -40.0 mV | No firing |

At stress ≥ 0.7, neurons receiving standard input can no longer reach threshold — the hippocampus goes silent.

---

### Layer 3: Hopfield Network — Associative Memory

A **100-neuron Hopfield Network** is used to model memory storage and recall. Each neuron is in state +1 (active) or -1 (inactive).

**Memory encoding** uses **Hebbian Learning**:
```
W = (1/N) · (1 - stress_level) · x · xᵀ
```
Where `x` is the pattern to be stored (the letter "A" as a 10×10 binary matrix).

- Neurons that fire together strengthen their connection → positive weight
- Neurons with opposite states weaken their connection → negative weight
- Diagonal set to 0 (no self-connections)

**Stress effect on the weight matrix:**

Cortisol not only weakens connections but also introduces **synaptic noise**:
```
W = (1 - stress) · W_base + noise · 0.02
```

This models the real biology: under cortisol, synaptic transmission becomes both weaker and more error-prone.

---

### Layer 4: Memory Recall

Given a **corrupted cue** (some pixels flipped), the network iteratively updates each neuron:

```
s_new = sign(W · s)
```

This repeats until convergence (no change between iterations) — the network settles into the closest stored memory. Under low stress, it finds the original "A". Under high stress, it gets lost in noise.

---

##  Outputs

### 1. Voltage Plot
Shows the membrane potential over time for multiple stress levels. As stress increases, spikes become rarer and eventually disappear entirely.

### 2. Raster Plot
Displays spike times for all 100 neurons across different stress levels. Dense activity at stress=0.0 gradually gives way to silence at stress≥0.7.

### 3. Memory Matrix Comparison
Side-by-side heatmap comparison:
- **Original "A"** pattern
- **Stress-free recall** (stress=0.0) → clean, accurate
- **Stressed recall** (stress=0.9) → fragmented, distorted

### 4. Stress vs. Memory Accuracy
A performance curve showing how recall accuracy degrades as cortisol rises. The curve remains near 100% until stress ≈ 0.7, then drops sharply — a critical threshold effect.

### 5. Minimum Trigger Experiment
The core experiment: how many correct pixels are needed to recover a stress-encoded memory?

---

##  Experimental Finding

**Question:** *"What is the minimum trigger data needed to recover a hidden memory?"*

**Result (stress=0.7):**

The network shows a sharp phase transition around **55 correct pixels (out of 100)**:

- Below 55 pixels → recall accuracy near 0% — the memory cannot be recovered
- At 55 pixels → accuracy jumps to ~98% — the memory suddenly snaps back

This is not a gradual recovery. It is a **threshold phenomenon** — exactly as observed in real neuroscience and trauma research.

**Biological interpretation:** A stress-encoded memory is not erased. It exists in the network but is inaccessible. A sufficiently strong trigger — a smell, a sound, a location — that provides enough correct "pixels" of the original context can suddenly bring the memory back in full. This mirrors the clinical observation of trauma memories resurfacing after sensory triggers.

---

##  Project Structure

```
biological-memory-simulation/
│
├── simulation.py          # Main simulation file
└── README.md              # This file
```

---

##  How to Run

**Requirements:**
```
numpy
matplotlib
```

**Install:**
```bash
pip install numpy matplotlib
```

**Run:**
```bash
python simulation.py
```

The program will sequentially display all 5 output graphs.

---

##  Technologies Used

| Tool | Purpose |
|------|---------|
| Python | Core language |
| NumPy | Matrix operations, Euler integration |
| Matplotlib | All visualizations |
| LIF Model | Single neuron electrophysiology |
| Hopfield Network | Associative memory storage & recall |
| Hebbian Learning | Weight matrix computation |
| Euler Method | Numerical ODE solving |

---

##  Key Concepts

- **Leaky Integrate-and-Fire (LIF):** A simplified but biologically plausible neuron model based on RC circuit dynamics
- **Engram:** The physical trace of a memory — distributed across a population of neurons, not stored in a single location
- **Hebbian Learning:** "Neurons that fire together, wire together" — the biological basis of memory formation
- **Hopfield Network:** An associative memory model that can reconstruct complete patterns from partial or noisy inputs
- **Cortisol:** A stress hormone that raises neural firing thresholds and reduces synaptic plasticity, fragmenting memory encoding
- **Phase Transition:** The sharp jump in recall accuracy at the critical trigger threshold — a hallmark of attractor network dynamics

---