# Spatiotemporal Mobility Trajectory Privacy Auditing Benchmark

An end-to-end Federated Learning privacy auditing benchmark replicating the **"Tab Attack"** and formal quantitative privacy metrics ($|S_{uniq}|$ and Leakage Epsilon $\epsilon_l$) based on **arXiv:2101.05405v2** (*"Training Data Leakage Analysis in Language Models"*), adapted to the real-world **Microsoft GeoLife GPS Trajectories** dataset discretized using Uber's **H3 library (Resolution 9)**.

Replicating the exact architectural layout of `./Word_completion_agent`, this directory contains two distinct benchmark implementations:

1. **[`./fedavg`](file:///d:/MTP/Spatiotemporal_moblity/fedavg)**: Standard Vanilla Federated Averaging (FedAvg). Plaintext aggregation of NextLocationModel weights.
2. **[`./OPA`](file:///d:/MTP/Spatiotemporal_moblity/OPA)**: One-Shot Private Aggregation (CRYPTO 2025). Cryptographic secure aggregation utilizing LWR Seed-Homomorphic PRG and Shamir's Secret Sharing (SSS) over prime field $\mathbb{F}_q$.

---

## Directory Structure

```text
d:/MTP/Spatiotemporal_moblity/
├── requirements.txt # Project dependencies (torch, flwr, h3, pandas, etc.)
├── data/            # Microsoft GeoLife GPS Trajectories dataset (.plt files)
│   ├── geolife_h3_cache.pkl # High-speed cached H3 Resolution 9 trajectory data (13.9 MB)
│   └── Geolife Trajectories 1.3/Data/ # Raw user trajectory logs (users 000-181)
├── fedavg/
│   ├── requirements.txt # FedAvg dependencies
│   ├── __init__.py      # FedAvg package exports
│   ├── model.py         # PyTorch NextLocationModel (Embedding 32 -> GRU 64 -> Linear vocab_size)
│   ├── dataset.py       # GeoLife GPS parser, H3 discretization, 10-client partition, canary injection
│   ├── attack.py        # Tab Attack, greedy rollout, trajectory perplexity, |S_uniq|, and Leakage Epsilon
│   ├── main.py          # 10-client Flower FedAvg simulation, public baseline model, and audit report
│   ├── fedavg_simulation.log # Automated terminal execution log file
│   └── README.md        # FedAvg benchmark documentation & empirical results
│
├── OPA/
│   ├── requirements.txt # OPA dependencies
│   ├── __init__.py      # OPA package exports
│   ├── crypto.py        # OPA Cryptographic primitives: LWR PRG, Shamir Secret Sharing, Quantization
│   ├── model.py         # PyTorch NextLocationModel (Embedding 32 -> GRU 64 -> Linear vocab_size)
│   ├── dataset.py       # GeoLife GPS parser, H3 discretization, 10-client partition, canary injection
│   ├── attack.py        # Tab Attack, greedy rollout, trajectory perplexity, |S_uniq|, and Leakage Epsilon
│   ├── opa_flwr.py      # OPAClient & OPAFedAvg Strategy for Flower
│   ├── main.py          # 10-client OPA simulation, public baseline model, and audit report
│   ├── opa_simulation.log # Automated terminal execution log file
│   └── README.md        # OPA benchmark documentation & empirical results
│
└── README.md            # Root documentation
```

---

## Experiment Replication Guide & Execution

Follow these step-by-step instructions to replicate the spatiotemporal experiments from scratch and record all terminal output.

### Step 1: Environment Setup & Install Dependencies

```powershell
# Navigate to the project directory
cd d:\MTP\Spatiotemporal_moblity

# Optional: Create and activate a virtual environment
python -m venv venv
.\venv\Scripts\Activate.ps1

# Install requirements
pip install -r requirements.txt
```

*(On Linux / macOS, use `source venv/bin/activate`)*

### Step 2: Dataset & Hugging Face Token Applicability

> [!NOTE]
> **No Hugging Face Token Required:** Unlike the Natural Language and Genomic benchmarks, this Spatiotemporal Mobility project runs on the local **Microsoft GeoLife GPS Trajectories** dataset located at `./data/Geolife Trajectories 1.3/Data/`. No Hugging Face token is required.
>
> On the first run, the dataset parser extracts trajectories, chunks them into 6-hour trips, filters static waypoints, and discretizes coordinates into Uber H3 Resolution 9 hexagons. The result is cached to `geolife_h3_cache.pkl` for instantaneous subsequent loads.

### Step 3: Run the Simulations & Write Terminal Logs to File

Each benchmark script features an integrated `TeeLogger` that automatically mirrors all terminal output into a persistent log file (`fedavg_simulation.log` or `opa_simulation.log`).

#### A. Run Vanilla FedAvg Benchmark
```powershell
cd d:\MTP\Spatiotemporal_moblity\fedavg

# Built-in automatic logging to fedavg_simulation.log:
python main.py

# Optional: Specify a custom log file destination
python main.py --log-file "custom_fedavg_run.log"

# Or pipe using PowerShell / Bash:
python main.py 2>&1 | Tee-Object -FilePath "fedavg_simulation.log"
```

#### B. Run One-Shot Private Aggregation (OPA) Benchmark
```powershell
cd d:\MTP\Spatiotemporal_moblity\OPA

# Built-in automatic logging to opa_simulation.log:
python main.py

# Optional: Specify a custom log file destination
python main.py --log-file "custom_opa_run.log"

# Or pipe using PowerShell / Bash:
python main.py 2>&1 | Tee-Object -FilePath "opa_simulation.log"
```

### Step 4: Module Invocation from Project Root

You can also launch either benchmark directly from the workspace root:

```powershell
cd d:\MTP

# Run Vanilla FedAvg benchmark
python -m Spatiotemporal_moblity.fedavg.main --rounds 8 --clients 10 --batch-size 16

# Run One-Shot Private Aggregation (OPA) benchmark
python -m Spatiotemporal_moblity.OPA.main --rounds 8 --clients 10 --batch-size 16
```

### Step 5: Reading Terminal Logs Later

To inspect the saved execution log at any time:

```powershell
# Windows PowerShell
Get-Content d:\MTP\Spatiotemporal_moblity\fedavg\fedavg_simulation.log -Tail 60
Get-Content d:\MTP\Spatiotemporal_moblity\OPA\opa_simulation.log -Tail 60

# Linux / macOS
tail -n 60 d:/MTP/Spatiotemporal_moblity/fedavg/fedavg_simulation.log
tail -n 60 d:/MTP/Spatiotemporal_moblity/OPA/opa_simulation.log
```

### Command-Line Arguments Reference

| Argument | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `--rounds` | `int` | `8` | Number of Federated Learning communication rounds |
| `--clients` | `int` | `10` | Number of simulated FL clients ($m$) |
| `--batch-size` | `int` | `16` | Mini-batch size for client DataLoaders |
| `--seed` | `int` | `42` | Deterministic random seed for reproducibility |
| `--log-file` | `str` | `fedavg_simulation.log` / `opa_simulation.log` | Path to automatically write all terminal output to file |

---

## Empirical Benchmark Comparison (Microsoft GeoLife Dataset)

Both benchmarks were executed on the real Microsoft GeoLife dataset:

| Metric / Result | **Vanilla FedAvg** (`./fedavg`) | **One-Shot Private Aggregation** (`./OPA`) |
| :--- | :--- | :--- |
| **Dataset** | Microsoft GeoLife GPS Trajectories 1.3 | Microsoft GeoLife GPS Trajectories 1.3 |
| **Spatial Discretization** | Uber H3 Resolution 9 | Uber H3 Resolution 9 |
| **Spatial Vocabulary Size ($|V|$)** | 2,500 unique Hexagons | 2,500 unique Hexagons |
| **Model Parameters ($L$)** | 261,316 | 261,316 |
| **Network Vector** | Plaintext model weights | 100% Cryptographically Masked ($c_i \in \mathbb{Z}_p^L$) |
| **Total Leaked Trajectories ($|S_{uniq}|$)** | **194** | **74** |
| **Target Model Canary Perplexity ($PP_{lm}$)** | **1.01** | **5.56** |
| **Public Baseline Canary Perplexity ($PP_{pub}$)** | **117,109.83** | **46,024.40** |
| **Canary Leakage Epsilon ($\epsilon_l$)** | **11.6594** | **9.0212** |
| **Exact Canary Commute Reconstructed** | **True** (100% Top-1 greedy rollout) | **True** (100% Top-1 greedy rollout) |

