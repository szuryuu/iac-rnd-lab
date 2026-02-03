# Infrastructure R&D Lab

A comprehensive Infrastructure-as-Code (IaC) playground designed for experimenting with modern DevOps tools, security patterns, and zero-trust networking. This repository serves as a testing ground for advanced concepts like HCP Boundary, infrastructure testing, and automated drift detection.

---

## Experimental Highlights

This project goes beyond standard provisioning by implementing experimental workflows and cutting-edge tools:

-   **Zero Trust Access (HCP Boundary)**
    Implementation of HashiCorp Boundary to manage secure remote access to infrastructure without traditional VPNs or exposing SSH ports directly to the public internet.

-   **Infrastructure Testing (Terratest)**
    Validates infrastructure code reliability using Go-based unit tests (terratest) before deployment, ensuring that resources like Virtual Networks and Compute instances behave as expected.

-   **Automated Drift Detection**
    A proactive GitHub Actions workflow that runs on a schedule to detect and report any manual changes (configuration drift) that deviate from the defined Terraform state.

## Architecture Overview

The infrastructure is organized into a modular, multi-environment architecture:

### 1. Environment Strategy
-   **Shared**: Core networking and centralized services (HCP Boundary, Observability stacks).
-   **Dev / Staging / Prod**: Isolated environments for application workloads, managed via Terraform workspaces/directories.

### 2. Configuration Management (Ansible)
Once Terraform provisions the compute layer, Ansible takes over to configure the OS and software stack:
-   **Security**: SSH hardening, Firewall rules (UFW), and user management.
-   **Containerization**: Automated Docker engine installation and setup.
-   **Web Serving**: Nginx configuration with dynamic templates.
-   **Monitoring**: Node Exporter and observability agents.

## CI/CD Pipelines

Automated workflows powered by GitHub Actions handle the entire lifecycle:

| Workflow | Description |
| :--- | :--- |
| **Terraform Plan & Apply** | Validates HCL syntax, runs terraform plan, and applies changes on merge. |
| **Infrastructure Tests** | Executes Go tests (go test -v) to verify deployment logic. |
| **Drift Detection** | Scheduled check to alert on configuration drift. |
| **Ansible Configuration** | Triggers playbook execution to configure instances post-provisioning. |
| **Security Scanning** | Scans IaC code for misconfigurations and vulnerabilities. |

## Project Structure

```bash
.
├── .github/workflows/    # CI/CD definitions (Drift, Plan/Apply, Testing)
├── ansible/              # Configuration Management
│   ├── roles/            # Modular roles (docker, nginx, security)
│   └── playbooks/        # Main site.yml entrypoint
├── terraform/            # Infrastructure Provisioning
│   ├── environments/     # Environment isolation (dev, prod, shared)
│   │   ├── dev/test/     # Go/Terratest files
│   │   └── shared/       # Boundary & Observability scripts
│   └── modules/          # Reusable modules (compute, network)
└── docs/
    ├── rnd/     
    │   ├── boundary.md   
    │   └── terratest.md       
    └── README.md       
```

##  Tech Stack

-   Provisioning: Terraform (Azure Provider)
-   Config Management: Ansible
-   Access Management: HashiCorp Boundary (HCP)
-   Testing: Go, Terratest
-   CI/CD: GitHub Actions
-   Cloud: Microsoft Azure

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

