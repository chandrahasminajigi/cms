# Write-up

## Analyze, choose, and justify the appropriate resource option for deploying the app

The CMS app is a single Flask web application. It stores article data in Azure SQL Database, stores images in Azure Blob Storage, and uses Microsoft Entra ID for sign-in. I compared two ways of hosting it: an Azure Virtual Machine (VM) and Azure App Service.

### Comparison

| Criteria | Virtual Machine | App Service (Web App) |
|---|---|---|
| **Cost** | You pay for the VM whenever it is allocated, plus the OS disk, public IP, and networking. You also spend time on patching and maintenance. | One App Service plan covers compute, OS, runtime, and the built-in features. There are low-cost tiers for small apps, and the plan can be scaled down or stopped. |
| **Scalability** | Scaling up means resizing the VM. Scaling out needs a VM Scale Set, a load balancer, and a custom image or configuration. | Scale up by changing the plan tier. Scale out by adding instances, manually or with autoscale rules. The platform load-balances automatically. |
| **Availability** | You are responsible for OS updates, reboots, web server configuration, and failover. High availability needs extra VMs and a load balancer. | Azure manages the OS and runtime. The platform provides health checks, automatic restarts, and an SLA on the paid tiers. Deployment slots allow zero-downtime releases. |
| **Workflow** | You must provision the VM, install Python, configure a web server (for example Gunicorn with Nginx), open ports, set up TLS certificates, and manage security patches. Deployment is manual or scripted. | You deploy code directly from GitHub, from GitHub Actions, or with `az webapp up`. HTTPS, a default domain, app settings (environment variables), and log streaming are built in. |
| **Security and maintenance** | You manage the OS, firewall rules, and SSH or RDP access. | Azure manages the OS and runtime patching. App settings keep secrets out of the code. |

### Choice: Azure App Service

I chose **Azure App Service** to deploy the CMS app.

### Justification

- **It fits the app.** The CMS is one Flask web app with no special OS requirements, custom drivers, or background services. It does not need full control of a server, so a VM would be more than the app needs.
- **Lower cost.** A small App Service plan is enough for this workload, and I do not pay for the extra management work that a VM requires. The data lives in managed services (Azure SQL Database and Blob Storage), so the web tier can stay small.
- **Less operational work.** Azure patches the OS and runtime, so I can spend my time on the application.
- **Simple deployment.** The code is deployed from the GitHub repository. App settings hold the SQL connection string, the Blob Storage credentials, and the Entra ID client details, so none of them are committed to the code.
- **Built-in features I need.** HTTPS is provided by default, which the Entra ID redirect URI requires. Log streaming helped me confirm successful logins and debug issues.
- **Easy to scale.** If traffic grows, I can scale up or out from the portal without changing the application.
- **Good availability.** The platform restarts unhealthy instances automatically, and the paid tiers have an availability SLA.

## Assess app changes that would change your decision

I would reconsider and move to a VM if the app or its requirements changed in these ways:

- **Custom OS or software.** The app needs a specific operating system configuration, a system-level library, or software that the App Service runtime does not support.
- **Background services.** The app needs long-running daemons, custom network services, or non-HTTP protocols that App Service does not support well.
- **Full server control.** There are compliance or security requirements for custom firewall rules, OS-level hardening, or specific agents installed on the host.
- **Heavy or specialised compute.** The workload needs GPUs, large amounts of memory, or other specialised hardware that an App Service plan does not offer.
- **Multiple tightly coupled components.** Several services must run on the same machine, for example a database, a message queue, and the web app together.
- **Cost at large scale.** For very large and steady workloads, a reserved VM or a container-based platform such as Azure Kubernetes Service or Container Apps could be cheaper or more flexible than App Service.

If the app needed containers or microservices rather than raw server control, I would choose App Service for Containers or Azure Container Apps rather than a VM.

As long as the CMS stays a single website backed by managed Azure services, App Service remains the better choice.
