# 🤖 Temi Robot: Multi-Agent Orchestration with CrewAI for Proactive Fall Prevention

> [!IMPORTANT]  
> Developed as a Bachelor's Thesis (TFG) at the Innovation Department of **Group Saltó**. This project implements a production-ready, multi-agent AI pipeline for domestic fall prevention. Source code and datasets remain private due to confidentiality agreements.

---

## 📌 Project Context & Motivation
Falls are the second leading cause of unintentional injury death globally, with adults over sixty being the most affected group. Floor-level domestic obstacles are among the most preventable contributing factors for elderly people living alone.

This project transforms the **Temi assistive robot** from a fundamentally reactive platform into a proactive healthcare agent. Utilizing a multi-agent computer vision system, the robot autonomously patrols domestic environments, detects floor-level hazards, evaluates their spatial risk, and communicates them in natural language before falls occur.

---

## 🧠 Architecture: Cognition on Demand

The system introduces a **"Cognition on Demand"** paradigm: a strict separation between brute force computer vision and expensive Large Language Model (LLM) reasoning. Heavy numerical work is delegated to specialized edge models, while higher-level cognition is handled by an orchestrator that never processes raw video.

### Multi-Agent System Schema (CrewAI)
<img width="1360" height="1240" alt="figure_5_1_multiagent" src="https://github.com/user-attachments/assets/250e8740-111e-4928-bc66-ad3632df68c2" />

### Orchestration Layer
The orchestration layer operates on a hierarchical star topology using **CrewAI**. Lateral delegation between workers is disabled (`allow_delegation=False`) to enforce a strict star topology where the orchestrator is the only node allowed to route data.

* **Manager Orchestrator**: Interprets user requests, dynamically delegates tasks at runtime without hardcoded routing logic, and handles zero-shot language adaptation (English, Spanish, Catalan).
* **Base Scan Technician**: Interfaces with the YOLO detector to extract the video inventory.
* **Topological Risk Specialist**: Interfaces with SegFormer to score hazard levels from 1 to 100 based on spatial geometry.
* **Visual Taxonomist**: Bridges vision and language by invoking Moondream2 to semantically identify objects.
* **Spatial Context Writer**: Uses Moondream2 on full-frame scenes to generate relational spatial descriptions.

---

## 👁️ Cascading Vision Pipeline

The pipeline is organized sequentially so the most expensive model (VLM) only receives a small, high-priority subset of everything originally detected, cutting the VLM workload by more than 80%.

1. **Instance Segmentation (YOLOv8-Seg):** High-speed anomaly detection operating strictly within a defined floor Region of Interest (ROI). Uses asymmetric resolution to run inference on downscaled frames while extracting VLM crops from native full-resolution images.
2. **Duplicate Filter:** A deterministic module using Discrete Cosine Transform (DCT) perceptual hashing to eliminate visually redundant object crops.
3. **Crop Expansion Agent:** Geometrically detects truncated bounding boxes at frame edges and triggers a secondary YOLO pass to reconstruct complete object silhouettes.
4. **Spatial Risk Assessor (SegFormer-B0):** Computes a perspective-corrected hazard score based on the object's geometric centrality within the walkable corridor. Objects scoring below 50 are discarded before any VLM inference is performed.
5. **Semantic Identification & Context (Moondream2 1.8B):** A two-call VLM design addressing edge-hardware attention constraints. Call one identifies the object from a tight crop; Call two generates spatial context from the full frame.

---

## 🛡️ LLM Integration & Reliability Mechanics

Bridging probabilistic LLMs with deterministic Python backends required specific engineering to ensure safety-critical reliability:

* **Two-Phase Memory Architecture:** Combines a permanent per-session JSON cache (preventing redundant vision model inference) with a 3-turn FIFO sliding window (preventing "lost in the middle" attention degradation during multi-turn chats).
* **Single Source of Truth (SSOT):** The orchestrator is explicitly forbidden from generating object IDs. All references are empirically retrieved via the internal memory tool (`consultar_memoria_interna`) to prevent hallucinated dictionary keys.
* **Uncertainty Normalization:** Scans VLM outputs for hedged phrases and replaces them with a controlled `Unknown` label to trigger precautionary alerts.

---

## 🔗 Robotic Integration Layer

The cognitive pipeline connects to the physical Temi hardware via a dedicated microservices architecture.

* **FastAPI Backend:** Handles asynchronous video uploads and multitenant conversational queries, isolating concurrent sessions into ephemeral, auto-deleting UUID workspaces.
* **Kotlin Android Module:** Implements a Finite State Machine (FSM) to manage Temi SDK callbacks, structured Kotlin coroutine concurrency, and CameraX video capture without blocking the robot's main navigation thread.

---

## 📊 Validation & Results

The system was validated via structured behavioral testing and ground-truth vision metrics during autonomous residential patrols.

* **Vision Pipeline:** Achieved 100% recall (13/13) and 100% post-filter precision for object detection. Semantic VLM identification reached 92% accuracy (12/13) on edge hardware.
* **Behavioral Suite:** 18 structured test cases across 6 categories (early stopping, agent skipping, object search, guardrails, language compliance, and conversational memory).
* **Score:** 17 PASS, 1 PARTIAL, 0 FAIL.

---
**Author:** Blai Gené Mora  
Degree in Computer Engineering, Escola Politècnica Superior (Universitat de Lleida)  
Developed at **Group Saltó** (Innovation Dept.)
