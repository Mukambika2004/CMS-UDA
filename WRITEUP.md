# Write-up

### Analyze, choose, and justify the appropriate resource option for deploying the app.

The CMS is a Flask application that relies on Azure SQL Database, Azure Blob Storage, and Microsoft identity (MSAL). It benefits from managed platform features, continuous deployment, and built-in scaling, so I compared Azure Virtual Machines (VMs) and Azure App Service against those needs.

| Criteria | Azure VM | Azure App Service |
| --- | --- | --- |
| **Cost** | Pay for VM size 24/7 plus separate load balancer, patching effort, and scaling headroom. Operational overhead increases total cost of ownership. | Pay for an App Service Plan tier; scaling can be right-sized to workloads. Managed runtime reduces ops overhead and bundled features (TLS, deployment slots) offset price. |
| **Scalability** | Manual scale up/out. Requires configuring Virtual Machine Scale Sets or custom scripts; scaling windows are slower. | Built-in automatic scale rules (CPU, schedule) and instant scale-out with additional instances. Easy to integrate with Azure Monitor autoscale. |
| **Availability** | Need to manage OS patching, health probes, and redundancy. High availability needs multiple VMs + Load Balancer. | Platform handles OS patching and offers SLA-backed availability per plan. Deployment slots enable zero-downtime swaps. |
| **Workflow & Deployment** | Must configure Python, drivers (ODBC), gunicorn/nginx manually. CI/CD means custom scripts or agents; secrets managed via VM settings or Key Vault integration. | App Service supports GitHub Actions + Oryx build, integrates with Key Vault, includes logging/monitoring. Environment variables align with existing `Config` class patterns. |

**Choice: Azure App Service**

I selected Azure App Service because it minimizes management tasks for this Python 3.10 web workload, aligns with the project’s GitHub-driven deployment, and natively supports the dependencies (Flask, pyodbc, MSAL, Blob SDK). The platform also simplifies SSL, scaling, and application monitoring without requiring VM-level administration. For a student project where rapid deployments and reliability matter more than low-level control, App Service delivers the best balance of cost, scalability, and operational simplicity.

### Assess app changes that would change your decision.

- **Heavy lift-and-shift or custom OS requirements:** If the app required a nonstandard runtime, custom drivers, or background services that App Service cannot host, a VM (or container-based Azure Kubernetes Service) would be more appropriate.
- **Persistent background workers:** Long-running background jobs or daemons that must run alongside the web process might necessitate VM or Azure Container Apps where process management is under our control.
- **Hybrid networking constraints:** If strict network appliances or self-managed VPN agents were required on the host, a VM deployed inside an isolated subnet would become preferable.
- **Cost optimization for static workloads:** For very predictable, low-traffic workloads where a single small VM can be reserved, a VM with Azure Spot/Reserved pricing might reduce costs enough to justify the extra management effort.

*Detail how the app and any other needs would have to change for you to change your decision in the last section.* 
