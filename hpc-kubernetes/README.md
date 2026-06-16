# HPC - Kubernetes Integration Workflows

This section contains the HPC orchestration and Kubernetes integration workflows developed during the MSc thesis project.

The objective of this work was to investigate how containerized scientific workloads can be submitted from Kubernetes-oriented environments to Slurm-managed HPC infrastructure while using S3-compatible object storage as the shared data layer.

The workflows focused on:

* Slurm-based HPC job orchestration
* Kubernetes-to-HPC communication concepts
* SSH ProxyJump workflows
* OIDC-based temporary credential usage
* S3-compatible storage accession
* Apptainer-based execution
* automated job submission pipelines
* reproducible execution environments


---

# Architecture Overview

The following diagram summarizes the broader project architecture and the interaction between Kubernetes environments, Slurm-managed HPC infrastructure, authentication components, and S3-compatible object storage.

![Project Architecture](project_architecture.png)

The overall workflow combines:

* Kubernetes-side orchestration
* Slurm-based HPC execution
* temporary authentication credentials
* object storage communication
* containerized scientific workloads

The architecture was used as a conceptual reference throughout the project while developing the orchestration pipeline.

---

# Current Implemented Workflow

The following diagram represents the currently implemented execution workflow used during testing.

![Current Workflow](job_path.png)

The implemented pipeline follows these main steps:

1. temporary credentials are generated using an OIDC-based workflow
2. credentials are transferred through SSH-based communication
3. a bridge VM is used as an SSH jump host
4. jobs are submitted through the HPC login node
5. Slurm schedules execution on compute nodes
6. workloads execute inside Apptainer containers
7. input/output data is exchanged through S3-compatible storage

This workflow was successfully used to execute CUDA-enabled NAMD molecular dynamics workloads on GPU-enabled HPC nodes.

---

# Repository Contents

```text
hpc-kubernetes/
├── README.md
├── project_architecture.png
├── job_path.png
└── scripts/
    ├── Dockerfile
    ├── streamlit_app.py
    ├── run_namd_workflow.py
    ├── config.py
    ├── hpc_client.py
    ├── slurm_template.py
    ├── check_auth.py
    ├── create_oidc_account.py
    ├── login_sts.py
    ├── requirements.txt
    └── submit_namd_cuda_workflow.sh
```

---

# Web-Based Workflow Interface

A lightweight Streamlit application was developed as the user-facing entry point of the workflow.

The interface provides:

* HPC authentication validation
* OIDC account onboarding
* molecule dataset URL submission
* custom NAMD configuration file upload
* CPU/GPU resource selection
* live workflow logs
* MolStar-compatible visualization links

The Streamlit application serves as a lightweight orchestration layer and does not directly execute scientific workloads.

---

# Scripts

| Script                                                                   | Description                             |
| ------------------------------------------------------------------------ | --------------------------------------- |
| [`streamlit_app.py`](./scripts/streamlit_app.py)                         | Web-based workflow interface            |
| [`run_namd_workflow.py`](./scripts/run_namd_workflow.py)                 | Main orchestration workflow             |
| [`config.py`](./scripts/config.py)                                       | Runtime configuration                   |
| [`hpc_client.py`](./scripts/hpc_client.py)                               | SSH communication through Bridge VM     |
| [`slurm_template.py`](./scripts/slurm_template.py)                       | Dynamic Slurm script generation         |
| [`check_auth.py`](./scripts/check_auth.py)                               | Authentication validation               |
| [`create_oidc_account.py`](./scripts/create_oidc_account.py)             | OIDC onboarding workflow                |
| [`login_sts.py`](./scripts/login_sts.py)                                 | STS credential generation (placeholder) |
| [`submit_namd_cuda_workflow.sh`](./scripts/submit_namd_cuda_workflow.sh) | Early shell-based prototype             |

The `scripts/` directory contains the orchestration and automation components used during workflow testing between the Kubernetes-oriented environment and the Slurm-based HPC infrastructure.

The workflow was later modularized into multiple Python components in order to improve readability, maintainability, and debugging.

---

## Main Entry Point

The primary execution script is:

* [`run_namd_workflow.py`](./scripts/run_namd_workflow.py)

This is the main orchestration script intended to be executed by the workflow environment.

The script handles:

* OIDC agent initialization
* temporary credential generation
* dataset preparation
* MinIO uploads
* SSH communication setup
* remote workflow generation
* Slurm job submission
* remote job monitoring
* output collection orchestration

The remaining Python modules provide supporting functionality and are not intended to be executed directly by end users.

---

## Configuration

* [`config.py`](./scripts/config.py)

Contains centralized configuration values used across the workflow.

This includes:

* bridge VM configuration
* HPC login node configuration
* object storage configuration
* container locations
* runtime defaults
* workflow directory definitions

Runtime-specific values such as usernames, endpoints, client identifiers, and authentication parameters are provided through environment variables and Kubernetes Secrets rather than being embedded directly in the source code.

---

## Authentication Components

### `check_auth.py`

Validates that the workflow environment can successfully access all required services before job submission.

The validation process includes:

* OIDC account loading
* OIDC token validation
* SSH connectivity verification
* bridge VM communication checks

### `create_oidc_account.py`

Provides automated onboarding for users who do not already have an OIDC account.

The script guides users through:

* OIDC provider selection
* device authorization flow
* account creation
* encryption password setup

This functionality was integrated into the Streamlit interface to simplify first-time workflow access.

### `login_sts.py`

Used for temporary credential generation through an external OIDC/STS workflow.

The original implementation used in the deployment environment is not included in this repository.

A simplified placeholder version is provided to document the expected workflow interface used by the orchestration pipeline.

---

## Remote HPC Communication

### `hpc_client.py`

Handles SSH-based communication with the HPC environment.

The module provides:

* SSH ProxyJump communication
* bridge VM integration
* remote script execution
* retry handling
* connection monitoring

This component is responsible for connecting to the HPC login node through the bridge VM.

---

## Slurm Workflow Generation

### `slurm_template.py`

Dynamically generates:

* Slurm job scripts
* remote execution scripts

The generated scripts automate:

* input retrieval from object storage
* temporary credential loading
* container execution
* GPU-enabled NAMD execution
* output collection
* metadata generation
* result uploads

The generated Slurm script is submitted remotely through the HPC login node.

---

## Shell-Based Workflow

An earlier shell-based implementation was also developed during the initial stages of testing.

* [`submit_namd_cuda_workflow.sh`](./scripts/submit_namd_cuda_workflow.sh)

This version contained the complete workflow inside a single Bash script.

The later Python-based modular implementation replaced this approach in order to improve:

* readability
* maintainability
* debugging
* workflow modularity

The shell implementation is retained for historical reference and comparison purposes.

---

## Workflow Behavior

The orchestration pipeline performs the following steps automatically:

1. validate authentication
2. generate temporary credentials
3. download and prepare simulation inputs
4. upload inputs to object storage
5. establish SSH communication through the bridge VM
6. connect to the HPC login node
7. generate a Slurm job script dynamically
8. submit the job with `sbatch`
9. execute the workload inside an Apptainer container
10. upload logs, outputs, and metadata to object storage
11. provide visualization-compatible output references

The workflow was validated using CUDA-enabled NAMD molecular dynamics workloads on GPU-enabled HPC nodes.
