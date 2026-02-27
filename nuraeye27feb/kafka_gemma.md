# NuraEye AI: Kafka & Gemma Integration Guide

This document explains the architecture, components, and usage of the NuraEye VMS AI system, which combines real-time data streaming (Kafka) with Generative AI (Gemma 2b).

## 1. System Components

The system follows a **Hybrid Intelligence Architecture**:

### A. The "Nervous System" (Apache Kafka)
*   **Role:** Real-time data pipeline.
*   **Function:** Ingests high-speed alerts from cameras (Motion, Intrusion, Weapons) and streams them to the backend.
*   **Topic:** `alerts`

### B. The "Brain" (Gemma 2b via Ollama)
*   **Role:** Reasoning & Natural Language Engine.
*   **Function 1 (Ingestion):** Analyzes incoming raw alerts to assign "Priority" and recommend "Actions".
*   **Function 2 (Querying):** Translates user questions (e.g., "Show me glitches in Zone A") into SQL queries.

### C. The "Memory" (SQLite)
*   **Role:** Persistent Storage.
*   **Function:** Stores alert history in `.alerts.db`. This allows the AI to "remember" past events and answer questions about them without hallucinating.

### D. The "Body" (Python Backend & Frontend)
*   **Backend (`kafka-backend.py`):** Orchestrates the flow. It listens to Kafka, talks to Gemma, saves to SQLite, and pushes updates to the UI via WebSockets.
*   **Frontend (AngularJS):** Displays real-time alert cards and provides the Chat Interface for asking questions.

---

## 2. How to Run the System

You need 3 separate terminal windows designed to run concurrently.

### Step 1: Start the AI Engine (Terminal 1)
This starts the local LLM server.
```bash
ollama serve
# Ensure model is ready: ollama pull gemma:2b
```

### Step 2: Start the Backend Orchestrator (Terminal 2)
This script connects everything (Kafka, DB, AI, UI).
```bash
python3 kafka-backend.py
```
*Wait until you see:* `Server running on ws://0.0.0.0:8081`

### Step 3: Start the Data Producer (Terminal 3)
Simulates cameras sending alerts.
```bash
python3 kafka-producer.py
```

### Step 4: Launch the Interface
Open `index.html` in your web browser and navigate to the **NuraAi** tab.

---

## 3. Use Cases

### ✅ Use Case 1: Real-Time Monitoring
**Scenario:** A security guard sits in front of the screen.
*   **Action:** Producer sends a "Weapon Detected" alert.
*   **System:** Kafka streams it -> Gemma analyzes it (assigns "Critical" priority) -> Frontend displays a red flashing card instantly.

### ✅ Use Case 2: Natural Language Forensics
**Scenario:** A theft occurred yesterday in the parking lot (Camera 3).
*   **User Asks:** "Show me all alerts from Camera 3 with high severity."
*   **System:** Gemma converts this to `SELECT * FROM alerts WHERE camera_name LIKE '%Camera 3%' AND severity='High'`.
*   **Result:** The user sees exactly the footage/logs they need without manual filtering.

### ✅ Use Case 3: Automated Triage
**Scenario:** Hundreds of motion alerts are triggered by a storm (false positives).
*   **User Asks:** "Summarize the motion alerts from the last hour."
*   **System:** Gemma reads the data and responds: "There were 150 motion alerts, but most appear to be low-severity wind triggers in Zone 4."
