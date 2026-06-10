# 🤖 Temi Robot: Multi-Agent Orchestration with CrewAI for Proactive Fall Prevention

---

## 📌 Project Context & Motivation
[cite_start]Falls are the second leading cause of unintentional injury death globally, with adults over sixty being the most affected group[cite: 93, 94]. [cite_start]Floor-level domestic obstacles are among the most preventable contributing factors for elderly people living alone[cite: 26].

[cite_start]This project transforms the **Temi assistive robot** from a fundamentally reactive platform into a proactive healthcare agent[cite: 82, 87]. [cite_start]Utilizing a multi-agent computer vision system, the robot autonomously patrols domestic environments, detects floor-level hazards, evaluates their spatial risk, and communicates them in natural language before falls occur[cite: 104].

---

## 🧠 Architecture: Cognition on Demand

[cite_start]The system introduces a **"Cognition on Demand"** paradigm: a strict separation between brute force computer vision and expensive Large Language Model (LLM) reasoning[cite: 478, 482]. [cite_start]Heavy numerical work is delegated to specialized edge models, while higher-level cognition is handled by an orchestrator that never processes raw video[cite: 479, 481].

### Multi-Agent System Schema (CrewAI)

### Orchestration Layer
[cite_start]The orchestration layer operates on a hierarchical star topology using **CrewAI**[cite: 283, 505]. [cite_start]Lateral delegation between workers is disabled (`allow_delegation=False`) to enforce a strict star topology where the orchestrator is the only node allowed to route data[cite: 527].

<img width="1360" height="1240" alt="figure_5_1_multiagent" src="https://github.com/user-attachments/assets/0e90dfc5-d188-4942-850a-24d6003e7381" />

* [cite_start]**Manager Orchestrator**: Interprets user requests, dynamically delegates tasks at runtime without hardcoded routing logic, and handles zero-shot language adaptation (English, Spanish, Catalan)[cite: 132, 284, 350].
* [cite_start]**Base Scan Technician**: Interfaces with the YOLO detector to extract the video inventory[cite: 517, 531].
* [cite_start]**Topological Risk Specialist**: Interfaces with SegFormer to score hazard levels from 1 to 100 based on spatial geometry[cite: 521, 531].
* [cite_start]**Visual Taxonomist**: Bridges vision and language by invoking Moondream2 to semantically identify objects[cite: 522, 531].
* [cite_start]**Spatial Context Writer**: Uses Moondream2 on full-frame scenes to generate relational spatial descriptions[cite: 523, 531].

---

## 👁️ Cascading Vision Pipeline

[cite_start]The pipeline is organized sequentially so the most expensive model (VLM) only receives a small, high-priority subset of everything originally detected, cutting the VLM workload by more than 80%[cite: 483, 487].

1. [cite_start]**Instance Segmentation (YOLOv8-Seg):** High-speed anomaly detection operating strictly within a defined floor Region of Interest (ROI)[cite: 213, 737]. [cite_start]Uses asymmetric resolution to run inference on downscaled frames while extracting VLM crops from native full-resolution images[cite: 739, 741].
2. [cite_start]**Duplicate Filter:** A deterministic module using Discrete Cosine Transform (DCT) perceptual hashing to eliminate visually redundant object crops[cite: 680, 804].
3. [cite_start]**Crop Expansion Agent:** Geometrically detects truncated bounding boxes at frame edges and triggers a secondary YOLO pass to reconstruct complete object silhouettes[cite: 831, 835].
4. [cite_start]**Spatial Risk Assessor (SegFormer-B0):** Computes a perspective-corrected hazard score based on the object's geometric centrality within the walkable corridor[cite: 29, 860]. [cite_start]Objects scoring below 50 are discarded before any VLM inference is performed[cite: 683].
5. [cite_start]**Semantic Identification & Context (Moondream2 1.8B):** A two-call VLM design addressing edge-hardware attention constraints[cite: 263, 974]. Call one identifies the object from a tight crop; [cite_start]Call two generates spatial context from the full frame[cite: 975, 976].

---

## 🛡️ LLM Integration & Reliability Mechanics

Bridging probabilistic LLMs with deterministic Python backends required specific engineering to ensure safety-critical reliability:

* [cite_start]**Two-Phase Memory Architecture:** Combines a permanent per-session JSON cache (preventing redundant vision model inference) with a 3-turn FIFO sliding window (preventing "lost in the middle" attention degradation during multi-turn chats)[cite: 599, 601, 611].
* [cite_start]**Single Source of Truth (SSOT):** The orchestrator is explicitly forbidden from generating object IDs[cite: 301]. [cite_start]All references are empirically retrieved via the internal memory tool (`consultar_memoria_interna`) to prevent hallucinated dictionary keys[cite: 299, 1007].
* [cite_start]**Uncertainty Normalization:** Scans VLM outputs for hedged phrases and replaces them with a controlled `Unknown` label to trigger precautionary alerts[cite: 945, 950].

---

## 🔗 Robotic Integration Layer

[cite_start]The cognitive pipeline connects to the physical Temi hardware via a dedicated microservices architecture[cite: 496].

* [cite_start]**FastAPI Backend:** Handles asynchronous video uploads and multitenant conversational queries, isolating concurrent sessions into ephemeral, auto-deleting UUID workspaces[cite: 353, 629, 634].
* [cite_start]**Kotlin Android Module:** Implements a Finite State Machine (FSM) to manage Temi SDK callbacks, structured Kotlin coroutine concurrency, and CameraX video capture without blocking the robot's main navigation thread[cite: 364, 365, 372, 647].

---

## 📊 Validation & Results

[cite_start]The system was validated via structured behavioral testing and ground-truth vision metrics during autonomous residential patrols[cite: 104, 1272].

* [cite_start]**Vision Pipeline:** Achieved 100% recall (13/13) and 100% post-filter precision for object detection[cite: 1342]. [cite_start]Semantic VLM identification reached 92% accuracy (12/13) on edge hardware[cite: 1365].
* [cite_start]**Behavioral Suite:** 18 structured test cases across 6 categories (early stopping, agent skipping, object search, guardrails, language compliance, and conversational memory)[cite: 112].
* [cite_start]**Score:** 17 PASS, 1 PARTIAL, 0 FAIL[cite: 1459].

---
[cite_start]**Author:** Blai Gené Mora [cite: 3]  
[cite_start]Degree in Computer Engineering, Escola Politècnica Superior (Universitat de Lleida) [cite: 8]  
[cite_start]Developed at **Group Saltó** (Innovation Dept.) [cite: 74]
