# Zero Trust Adaptive Security Gateway (ZTASG) for IoT Networks

## Overview
The **Zero Trust Adaptive Security Gateway (ZTASG)** is a machine-learning-driven security framework designed to analyze IoT network traffic and automatically derive enforceable security policies at the gateway level.

The project applies **Zero Trust principles** (“never trust, always verify”) to IoT environments by:
- Continuously evaluating device behavior using ML-based threat and trust scores
- Building **device-specific, protocol-aware security policies**
- Translating learned policies into **Snort IDS rules** that can be deployed for detection or enforcement

The system is implemented as an **offline / batch analysis pipeline**, intended as a research and prototyping foundation for adaptive and real-time IoT security gateways.

---

## Motivation
IoT devices are typically resource-constrained, long-lived, and difficult to patch, making traditional endpoint security impractical. Static network rules are often too permissive or too brittle.

ZTASG addresses this by:
- Learning *normal vs malicious behavior* from real IoT traffic
- Using ML outputs to derive **data-driven security boundaries**
- Enforcing Zero Trust at the **network edge**, rather than on devices

---

## Dataset
This project uses the **IEEE IoT Network Intrusion Dataset**, which provides real PCAP-based IoT traffic containing both benign and attack scenarios.

- Original dataset: IEEE DataPort – IoT Network Intrusion Dataset  
- PCAP files include **benign traffic** and **multiple attack types**
- Network flows were extracted and labeled:
  - `label = 0` → benign
  - `label = 1` → attack
- A custom dataset was created by combining benign and attack flows into a single labeled CSV

> **Note:** Due to licensing and size constraints, the dataset is not included in this repository. Instructions and references are provided for reproducibility.

---

## Feature Representation
Traffic is represented using **flow-level features** similar to Zeek `conn.log`, including:
- Protocol, ports, services
- Flow duration
- Packet and byte counts
- Connection metadata

These features allow the model to reason about **behavioral patterns**, not payload content.

---

## ML-Driven Threat and Trust Scoring
A **Random Forest classifier** is trained to distinguish benign vs malicious traffic.

For each network flow:
- **Threat Score** = `P(attack)`
- **Trust Score** = `1 − P(attack)`

This dual-score design allows the system to:
- Quantify risk probabilistically
- Avoid binary “allow/deny” decisions
- Support adaptive policy thresholds

Feature relevance is analyzed using:
- Mutual Information (pre-selection)
- Random Forest feature importance (policy guidance)

---

## Adaptive Policy Generation
Instead of relying on static rules, ZTASG derives policies from **trusted behavior**:

1. For a given IoT device:
   - Select flows with `trust_score ≥ threshold`
2. For each observed protocol:
   - Extract allowed destination ports
   - Compute acceptable ranges for traffic volume and timing features
3. Construct a **device- and protocol-specific policy**

This results in policies such as:
- Allowed port lists
- Expected traffic volume ranges
- Behavioral baselines per protocol

---

## Automated Snort Rule Synthesis
Derived policies are automatically converted into **Snort IDS rules**, including:
- **Port allow-list violations**  
  Alerts when a device communicates on unexpected ports
- **Traffic volume anomalies**  
  Alerts when response sizes exceed learned thresholds

Generated rules are exported to:
