# 🤖 Temi Robot: Multi-Agent Orchestration with CrewAI for Proactive Fall Prevention

> [!IMPORTANT]
> **Status: Work in Progress (WIP)**: Developed as part of my Bachelor's Thesis (TFG) at the Innovation Department of **Group Saltó**. Source code and datasets are private due to confidentiality agreements.

---

## 📌 Project Context
This project transforms the **Temi assistive robot** into a proactive healthcare agent. A Multi-Agent System (MAS) identifies tripping hazards in domestic environments and evaluates their risk **before falls occur**, running entirely on edge hardware.

---

## 🧠 Multi-Agent Architecture (CrewAI)

The system is orchestrated by **CrewAI** using a Manager pattern: agents are only invoked when the previous stage confirms a risk threshold, saving up to **70% of computational cycles** in safe environments.

| Agent | Role | Model |
|---|---|---|
| **Manager Orchestrator** | Event-driven task delegation | CrewAI |
| **Perception Agent** | Real-time instance segmentation | YOLOv8-Seg |
| **Geometric Specialist** | Truncated object reconstruction | Custom geometry |
| **Safety Auditor** | 3D topology + Hazard Score | SegFormer |
| **Semantic Reasoner** | Natural language hazard justification | Moondream2 (1.8B VLM) |
| **UI Translator** | Report localisation (CA / ES / EN) | — |

---

## 👁️ Computer Vision Pipeline

### Instance Segmentation — YOLOv8-Seg
Real-time detection and pixel-level segmentation of domestic obstacles. Handles truncated objects at frame boundaries via the Geometric Specialist for complete hazard assessment.

### 3D Topology & Hazard Scoring — SegFormer
SegFormer computes depth-aware floor topology to derive a **Hazard Score** per detected object. Transformer-based architecture provides strong robustness against indoor reflections and parquet glare — a critical advantage over CNN baselines in domestic settings.

### Visual Language Reasoning — Moondream2
Activated only on high Hazard Score triggers. Uses **contextual cropping** (focused, padded ROI rather than full-frame) to maximise semantic quality within edge memory constraints.

### Temporal Stability
Robot motion introduces inference jitter. **Exponential Moving Average (EMA)** and temporal maturity filters stabilise detections across frames for consistent hazard tracking.

---

## ⚡ Edge Deployment Challenges

Running a MAS with multiple neural networks on Temi's restricted hardware requires:

* **Asymmetric resolution strategies**: high-quality reasoning only where hazard scores justify it.
* **Lazy loading + model switching**: keeping a 1.8B VLM viable within limited RAM.
* **Latency budgeting**: CrewAI overhead and inference times benchmarked against safety-relevant response limits.

---

## 🔗 Robot Integration

The AI backend (Python) communicates with the Temi robot via a **REST API** and a native **Kotlin Android app** built on Temi's SDK. The app handles physical navigation, on-demand camera capture, and asynchronous frame dispatch to the inference pipeline, fully integrated into Group Saltó's platform architecture.

---

## 🛠️ Technical Stack

| Layer | Technologies |
|---|---|
| **Orchestration** | CrewAI |
| **Vision & AI** | PyTorch, OpenCV, YOLOv8-Seg, SegFormer, Moondream2 |
| **Backend / API** | Python, REST API |
| **Robot App** | Kotlin, Android SDK, Temi SDK |
| **Robotics** | ROS2 (integration in progress) |

---

## 🚀 Roadmap
- [x] YOLOv8-Seg + SegFormer pipeline with Hazard Score
- [x] CrewAI orchestration with Manager pattern
- [x] REST API + Kotlin app + Physical robot integration
- [ ] Finalise CrewAI decision logic for complex obstacle scenarios
- [ ] Optimise VLM inference speed on Temi hardware
- [ ] Field testing in controlled domestic environments
- [ ] Full ROS2 navigation feedback integration

---
**Developed by:** Blai Gené Mora — AI Engineering Intern at **Group Saltó** (Innovation Dept.)  
Computer Engineering Student at **Escola Politècnica Superior (UdL)**
