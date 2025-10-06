# Multi-Region Landing Zone

## Overview

This repository contains an example multi-region Azure Landing Zone with activeâ€‑active failover.  It uses Terraform and Bicep to provision identical hubâ€‑spoke environments in two different Azure regions and then connects them together using Azure FrontÂ Door / Traffic Manager for global load balancing and disaster recovery.  The design demonstrates how to achieve high availability and regional resiliency while adhering to enterprise governance.

## Features

- **Activeâ€‑Active topology:** Resources (VNets, subnets, PaaS services, and AKS clusters) are deployed in two regions with identical configurations.
- **Hubâ€‑Spoke networking:** Each region uses a hub VNet for shared services (firewall, DNS, jump hosts) and multiple spoke VNets for workloads.  Azure Virtual WAN can be used for simplified hub management.
- **Global load balancing:** Azure FrontÂ Door or Traffic Manager directs traffic between regions, enabling seamless failover.
- **Infrastructure as Code:** The landing zone is defined in modular Terraform and Bicep templates to encourage reuse and compliance.
- **Policy & governance:** Builtâ€‑in policies, RBAC assignments, and tagging enforce security and cost management across both regions.
- **CI/CD pipeline:** A GitHub Actions workflow demonstrates how to lint, validate, plan, and apply Terraform/Bicep changes across regions.

## Repository Structure

```
.
├── networking/               # Hub and spoke VNet modules
├── compute/                  # AKS clusters, VM scale sets, and PaaS resources
├── terraform/                # Root Terraform configurations per region
├── bicep/                    # Alternative Bicep templates for reference
├── pipelines/                # GitHub Actions workflows and Azure DevOps YAML
├── docs/                     # Architecture diagrams and decision logs
└── README.md
```

- `networking/` – Contains reusable modules for hub VNets, spokes, route tables, and peering.
- `compute/` – Samples for deploying AKS clusters, VM scale sets, and PaaS services into spokes.
- `terraform/` – Example root modules (one per region) that call the modules and configure remote state.
- `bicep/` – Equivalent Bicep templates for comparison or for customers preferring Bicep.
- `pipelines/` – CI/CD definitions for multiâ€‑stage deployment and testing.
- `docs/` – Highâ€‑level diagrams showing regional architecture, failover flow, and network topology.

## Getting Started

1. **Prerequisites**
   - An Azure subscription with owner rights in at least two regions.
   - [Terraform](https://learn.microsoft.com/en-us/azure/developer/terraform/install)Â v1.5+ and the Azure CLI installed.
   - [Bicep CLI](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/install) if you want to deploy Bicep templates.
   - A GitHub repository with Actions enabled (for the provided workflow).

2. **Clone the repository**

   ```bash
   git clone https://github.com/OBILOTTITI22/multi-region-landing-zone.git
   cd multi-region-landing-zone
   ```

3. **Set up backend and variables**

   Configure a remote state backend (e.g., Azure Storage) and create a `terraform.tfvars` file for each region.  See `terraform/README.md` for an example.

4. **Deploy the first region**

   ```bash
   cd terraform/region1
   terraform init
   terraform plan -var-file=region1.tfvars
   terraform apply -var-file=region1.tfvars
   ```

5. **Deploy the second region**

   ```bash
   cd ../region2
   terraform init
   terraform plan -var-file=region2.tfvars
   terraform apply -var-file=region2.tfvars
   ```

6. **Configure global load balancing**

   Follow the instructions in `docs/global-load-balancing.md` to create a FrontÂ Door or Traffic Manager profile and map the regional endpoints.

## Activeâ€‑Active Architecture

Activeâ€‑active failover means both regions can serve traffic simultaneously.  You can run separate AKS clusters or PaaS services in each region and replicate data between them using geoâ€‑replicated databases (e.g., CosmosÂ DB, SQLÂ Database with active geoâ€‑replication).  Azure FrontÂ Door monitors health probes for each region and automatically directs traffic away from an unhealthy region.

## Disaster Recovery & Resiliency

In addition to activeâ€‑active frontâ€‑end, you should enable zoneâ€‑redundant services and crossâ€‑region replication for stateful components.  Use Azure Backup and Site Recovery for VMâ€‑based workloads.  Document your recovery time objectives (RTO) and recovery point objectives (RPO) in the `docs/` folder.

## CI/CD Pipeline

The provided GitHub Actions workflow (`pipelines/terraform-plan-apply.yaml`) runs on each pull request and on merges to `main`.  It performs the following steps:

1. **Lint & format** – Ensures your Terraform files follow `terraform fmt` and `tflint` rules.
2. **Validate & security scan** – Runs `terraform validate` and static analysis tools (e.g., Checkov) to catch misconfigurations.
3. **Plan** – Generates separate plans for each region and posts them as PR comments.
4. **Apply** – On merge, applies the plans to both regions with manual approval gates.

You can adapt the pipeline to Azure DevOps or other CI systems by reusing the scripts in `pipelines/`.

## Cost Management & Governance

Deploying workloads to multiple regions can increase costs.  Use Azure Policy to enforce tagging and environment naming standards, and set budgets with alerts.  The FinOps recommendations from the costâ€‑optimized landing zone project are also applicable here; see the `docs/finops-guidance.md` file for details.

## Contributing

Contributions and feedback are welcome!  Feel free to open issues or pull requests to improve the architecture, add new modules, or enhance documentation.

## License

This project is licensed under the MIT License – see the [LICENSE](LICENSE) file for details.
