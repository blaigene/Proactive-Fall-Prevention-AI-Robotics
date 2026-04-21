# 🤖 Temi Robot: Multi-Agent Orchestration with CrewAI for Proactive Fall Prevention

> [!IMPORTANT]
> **Status: Work in Progress (WIP)** > This project is currently under active development as part of my Bachelor's Thesis (TFG). Features, agents, and hardware optimizations are being iteratively refined and tested.

**Disclaimer:** This repository serves as an architectural overview and technical documentation for the project developed at the Innovation Department of **Group Saltó**. Due to corporate confidentiality agreements, the source code and raw datasets are private.

---

## 📌 Project Context
This project aims to transform the **Temi assistive robot** into a proactive healthcare agent. Using a sophisticated Multi-Agent System (MAS), the robot identifies tripping hazards in domestic environments and evaluates their risk to prevent falls before they occur.

## 🧠 Advanced Multi-Agent Orchestration (CrewAI)
The core of this architecture is powered by **CrewAI**, which manages the collaboration between specialized agents. The system is designed to transition from static scripts to a dynamic team of agents, each with specific roles, backgrounds, and sets of tools.

### 🧩 The "Crew" & Agent Fleet (Under Development)
* **Manager Orchestrator (CrewAI Driven):** Handles complex task delegation and event-driven triggers.
* **Perception Agent:** Implements **YOLOv8-Seg** for real-time instance segmentation.
* **Geometric Specialist:** Developed to handle on-demand reconstruction of truncated objects.
* **Safety Auditor:** Utilizes **SegFormer** to calculate 3D topology and Hazard Scores.
* **Semantic Reasoner (Moondream VLM):** A lightweight VLM integrated to provide natural language reasoning for high-risk obstacles.
* **UI Translator:** Localizes reports into the user's language (Catalan, Spanish, etc.).

---

## ⚡ The Challenge: Edge AI on Restricted Hardware
Deploying complex MAS and multiple neural networks on a restricted edge device like Temi presents significant engineering hurdles.

### Ongoing Technical Challenges:
* **Resource Constraints:** Developing **asymmetric resolution strategies** to balance high-quality visual reasoning with low-latency detection.
* **Memory Optimization:** Implementing "Lazy Loading" and efficient model switching to run 1.8B parameter models on limited RAM.
* **System Latency:** Benchmarking CrewAI overhead and model inference times to remain within safety limits.

---

## 💡 Lessons Learned (AI & Robotics Engineering)

Developing this architecture has provided key insights into the reality of deploying state-of-the-art AI in production environments:

* **VLM Scale vs. Edge Reality:** While 1.8B models like Moondream are "small" by LLM standards, they are "massive" for robotics hardware. I learned that **contextual cropping** is more efficient than increasing resolution; feeding the model a focused, padded image yields better semantic results than a full high-res frame.
* **Orchestration Over Sequence:** Using **CrewAI** taught me that a "Manager" approach is superior to a linear pipeline. By assigning "roles," the system avoids running the VLM if the Safety Auditor hasn't confirmed a high hazard score, saving up to 70% of computational cycles in safe environments.
* **The "Jittering" Problem in Real-World Data:** In contrast to static datasets, robot movement introduces noise. Implementing **Exponential Moving Average (EMA)** and temporal maturity filters is essential to stabilize AI inferences for consistent tracking.
* **Semantic Segmentation Robustness:** Transformers (SegFormer) are significantly more resilient to indoor reflections and parquet floor "glares" than traditional CNNs, which is critical for accurate 3D topology in domestic settings.

---

## 🛠️ Technical Stack
* **Orchestration:** CrewAI
* **Vision & AI:** PyTorch, OpenCV, YOLOv8-Seg, SegFormer (Transformers), Moondream2 (VLM)
* **Hardware:** Temi Robot (Edge Computing)
* **Environment:** ROS2 (Integration in progress)

## 🚀 Future Roadmap
- [ ] Finalize the CrewAI decision-making logic for complex obstacle scenarios.
- [ ] Optimize VLM inference speed on Temi's hardware.
- [ ] Conduct field testing in controlled domestic environments at Group Saltó.
- [ ] Integrate full ROS2 communication for real-time robot navigation feedback.

---
**Developed by:** Blai Gené Mora – AI Engineering Intern at **Group Saltó** (Innovation Dept)  
Computer Engineering Student at **Escola Politècnica Superior (UdL)**
