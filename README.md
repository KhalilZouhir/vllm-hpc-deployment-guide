# 🚀 Gemma 27B (IT INT4 AWQ) Deployment with vLLM in HPC

This repository provides a complete guide for deploying and serving the **Gemma 27B INT4 AWQ** model on an HPC environment using **vLLM**, including PBS job setup, SSH tunneling, and local prompt testing.

---

## 🧰 Prerequisites

Before doing anything, make sure you have the following tools installed:

- **FortiClient VPN** → *(link and details shared privately)*
- **MobaXterm** → [Download here](https://mobaxterm.mobatek.net/download-home-edition.html)

These tools are required to connect to the HPC environment and manage sessions.

---

## 1️⃣ Setup Conda Environment

> **Note:** The environment and model directories already exist — no need to recreate them.  
> Just activate and use the existing setup if available.

Create and activate the environment (if not already created):

```bash
conda create --name gptoss-vllm python=3.10
conda activate gptoss-vllm
