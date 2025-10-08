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
```

## 2️⃣ Prepare the Model Directory

```bash
mkdir models
cd models
```

copy the repository name of the model .
<img width="763" height="439" alt="image" src="https://github.com/user-attachments/assets/1dcd2a64-d1a8-44b0-b370-81c54c273818" />


Download the model from Hugging Face:

```bash
huggingface-cli download gaunernst/gemma27b-it-int4-awq --local-dir ./gemma27b-it-int4-awq
```
## 3️⃣ Create the PBS Job Script

Prepare a directory to store your logs:

```bash
mkdir logs
```
Save the following as run_model_1.pbs:

```bash
#!/bin/bash
#PBS -N 1gpu
#PBS -l select=1:ncpus=20:mem=180gb:ngpus=1
#PBS -q gpu_1w
#PBS -o logs/output.log
#PBS -e logs/error.log

# === Load modules & activate conda ===
module use /app/common/modules
module load anaconda3-2024.10
source activate gptoss-vllm

# === Move to working directory ===
cd /home/skiredj.abderrahman/models

# === Confirm node ===
echo "==== Job running on node: $(hostname -s) ===="

# === Start vLLM server ===
vllm serve /home/skiredj.abderrahman/models/gemma27b-it-int4-awq \
  --tensor-parallel-size 1 \
  --port 9998
```

## 4️⃣ Submit the Job

Submit the PBS job:

```bash
qsub run_model_1.pbs
```
Check if it’s running:

```bash
qstat
```
<img width="774" height="186" alt="image" src="https://github.com/user-attachments/assets/8afc7952-f269-437c-ae43-25e30fff53fe" />

## 📋 Useful PBS Commands

* qstat -Qf → Show all queue details

* qstat -f <job_id> → Show detailed info for a specific job

* pbsnodes -a → List all nodes and see where your model is running

## 🔍 Verifying the Deployment

Once the job is running, check which node it’s on:

```bash
qstat -f <job_id> | grep exec_host
```
Example:

```bash
qstat -f 5950.head1 | grep exec_host
```
You’ll see something like exec_host = gpu08/..., meaning the model is running on gpu08.
<img width="921" height="128" alt="image" src="https://github.com/user-attachments/assets/5e68c58f-a9a4-445b-af02-bb5de6c56ec9" />


## ✅ Test the Model Inside the HPC Cluster

Execute this command on the login node (replace gpu08 with your actual node):

```bash
curl http://gpu08:9998/v1/completions -H "Content-Type: application/json" -d '{"model":"/home/skiredj.abderrahman/models/gemma27b-it-int4-awq","prompt":"hi how are u","max_tokens":50}'
```
example of output : 

<img width="1653" height="146" alt="image" src="https://github.com/user-attachments/assets/f21df5e2-50a9-4235-8677-9c3a34ba0234" />

## 🌐 Accessing the Model Locally (SSH Tunneling)

To access the model from your local machine, create an SSH tunnel to the GPU node:

```bash
ssh -N -L 127.0.0.1:9998:gpu08:9998 skiredj.abderrahman@172.30.30.11
```
Enter your HPC password when prompted.
Keep this terminal open while the tunnel is active.

<img width="1071" height="401" alt="image" src="https://github.com/user-attachments/assets/6c6c4b83-9076-4578-ab16-2bf4c780f7c7" />


## 💬 Test a Prompt Locally

Open a new CMD window and run:

```bash
curl -X POST http://localhost:9998/v1/completions -H "Content-Type: application/json" -d "{\"model\":\"/home/skiredj.abderrahman/models/gemma27b-it-int4-awq\",\"prompt\":\"do u have a wish\",\"max_tokens\":50}"
```
If the setup is correct, you’ll receive a JSON response containing the model’s output.
<img width="1092" height="632" alt="image" src="https://github.com/user-attachments/assets/53852d1d-3b20-474c-a5a5-2191e13569f9" />


















