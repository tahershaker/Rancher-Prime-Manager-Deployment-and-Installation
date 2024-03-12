# Rancher Prime Manager - Deployment and Installation

This repo is created to provide the reader all the required information on deploying and installing _**Rancher Prime**_ manager on your kubernetes environment. The information provided in this repo will include, but not limited to, the kubernetes requirements, software requirements, network requirements and more. This repo will also provide the reader with a step-by-step guide for deploying _**Rancher Prime**_ Manager with the different supported methods of deployments.

---

<p align="center">
    <img src="Images/IntroPic.png">
</p>

---

## Overview

Rancher is a container management platform built for organizations that deploy containers in production. Rancher makes it easy to run Kubernetes everywhere, meet IT requirements, and empower DevOps teams. Rancher Manager provides multi-cluster management and lCM [lifecycle Management] across multiple infrastructure and multi-cloud. It also provides the DevOps teams with integrated tools for running containerized workload. With Rancher, you can deploy and manage not only kubernetes clusters but also containerized applications.

Rancher is a Kubernetes management tool to deploy and run clusters anywhere and on any provider.
Rancher can provision Kubernetes from a hosted provider, provision compute nodes and then install Kubernetes onto them, or import existing Kubernetes clusters running anywhere.

Rancher adds significant value on top of Kubernetes, first by centralizing authentication and role-based access control (RBAC) for all of the clusters, giving global admins the ability to control cluster access from one location.

It then enables detailed monitoring and alerting for clusters and their resources, ships logs to external providers, and integrates directly with Helm via the Application Catalog. If you have an external CI/CD system, you can plug it into Rancher, but if you don't, Rancher even includes Fleet to help you automatically deploy and upgrade workloads.

Rancher comes in 3 different flavours - Rancher (Community), _**Rancher Prime**_, and **Rancher Hosted**. Rancher v2.7 introduces _**Rancher Prime**_, an evolution of the Rancher enterprise offering. _**Rancher Prime**_ is a new edition of the commercial, enterprise offering built on the the same source code. Rancher’s product will therefore continue to be 100% open source with additional value coming in from security assurances, extended lifecycle, access to focused architectures and Kubernetes advisories. _**Rancher Prime**_ will also offer options to get production support for innovative Rancher projects. With _**Rancher Prime**_, installation assets are hosted on a trusted registry owned and managed by Rancher. For a detailed comparison, please refer to [this link](https://www.rancher.com/products/rancher-platform)

---

<p align="center">
    <img src="Images/Comparison.png">
</p>

---

Rancher Manager is an enterprise ready, production-level Kubernetes management platform that is deployed as a container running on a kubernetes cluster and is installed using Helm Charts. Rancher can be installed on any Kubernetes cluster. This cluster can use upstream Kubernetes, or it can use one of Rancher's Kubernetes distributions, or it can be a managed Kubernetes cluster from a provider such as Amazon EKS. For more info, please refer to [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster#kubernetes-cluster).

Rancher is installed using the Helm package manager for Kubernetes. Helm charts provide template syntax for Kubernetes YAML manifest documents. With Helm, we can create configurable deployments instead of just using static files.

Rancher can be installed in any of the below options:
- Single Node Kubernetes Cluster [Not Recommended]
- High-Availability Kubernetes cluster [Recommended]
- Rancher on EKS Install with the AWS Marketplace
- Docker Install [For Testing and Demonstration ONLY]

In each of the mentioned options above, it is expected to have a direct internet access to the Rancher Manager, however, Rancher also support an AirGapped Environment as well as being deployed behind an HTTP proxy. Please note that there are additional consideration that need to be taken care of before the installation.

---

## Rancher Manager High-Level Architecture Overview

Rancher Manager is deployed as a container on top of a kubernetes cluster. For the best performance and security, it is recommended to have a dedicated Kubernetes cluster for the Rancher management server. Running user workloads on this cluster is not advised. Also, a high-availability Kubernetes cluster (3 Master Nodes, 3 Worker Nodes) is recommended for production.

The majority of Rancher 2.x software runs on the Rancher Server/Manager. Rancher Server/Manager includes all the software components used to manage the entire Rancher deployment. The Rancher manager will be deployed on a dedicated kubernetes cluster (upstream cluster) and then other kubernetes cluster can be provisioned from or imported to Rancher Manager. Each downstream kubernetes cluster (Cluster provisioned by or imported to Rancher) will have a cluster agent running inside the cluster which will allow the communication between the Rancher Manager and the downstream cluster. 

The figure below illustrates the high-level architecture of Rancher 2.x. The figure depicts a Rancher Server installation that manages two downstream Kubernetes clusters: one created by RKE and another created by Amazon EKS (Elastic Kubernetes Service).

---

<p align="center">
    <img src="Images/RancherArch.png">
</p>

---

## Installation Requirements

To deploy and install Rancher Manager on a kubernetes cluster, several requirements must be available for a successful deployment. Since there are a high number of influencing factors that may vary over time, the requirements listed here should be understood as reasonable starting points that work well for most use cases. 

---

### Underlying Linux Operating System - Upstream Cluster - Management Cluster

All supported operating systems are 64-bit x86. Rancher should work with any modern Linux distribution. For ARM, installing Rancher on ARN is still under experimental, for more info, please refer to [this link](https://ranchermanager.docs.rancher.com/how-to-guides/advanced-user-guides/enable-experimental-features/rancher-on-arm64) 

Deploying Rancher is supported on more of the operating system available with their respective kubernetes distributions, the Rancher support matrix lists all the OS and Docker versions were tested for each Rancher version. For the support matrix, please refer to [this link](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions?_gl=1*15eexfp*_ga*MjAyNzQzNTA0NC4xNzA5MTkxMTM2*_ga_Y7SFXF9L00*MTcxMDI1NTY2MC4xNS4xLjE3MTAyNjQyODUuNTMuMC4w)

Please Note:
- Docker is required for nodes that will run RKE clusters. It is not required for RKE2 or K3s clusters.
- The ntp (Network Time Protocol) package should be installed. This prevents errors with certificate validation that can occur when the time is not synchronized between the client and server.
- Some distributions of Linux may have default firewall rules that block communication within the Kubernetes cluster. Since Kubernetes v1.19, firewalld must be turned off, because it conflicts with the Kubernetes networking plugins.

---

### Kubernetes Cluster - Upstream Cluster - Management Cluster

Rancher can be installed on any type of kubernetes cluster, however, it is recommended to install it on the tested and supported kubernetes distribution type with respected to the version. Rancher is tested and supported on RKE, RKE2, K3S, EKS, AKS, GKE. For the supported version, please refer to the [support matrix](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-2/).

Each kubernetes distribution may need a some different consideration to be taken care of. For example; Rancher requires an Ingress to be installed and in some cases require to have specific configuration on the ingress depending on the ingress type, however, RKE2 and K3S come with an ingress already deployed so no need to do any thing else for it. For a reference to all the supported and tested kubernetes distribution, please refer to [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster#kubernetes-cluster)

Please Note: If you install Rancher on a hardened Kubernetes cluster, check the [Exempting Required Rancher Namespaces section](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/authentication-permissions-and-global-configuration/psa-config-templates#exempting-required-rancher-namespaces) for detailed requirements.

---

### Hardware Requirements (Sizing) - Upstream Cluster - Management Cluster

To deploy Rancher on a kubernetes cluster, The following table lists a general minimum CPU and memory requirements for each node in the upstream cluster. This table is based on utilizing RKE, RKE2, K3S, and Hosted Kubernetes (EKS, AKS, GKS). For more info, please refer to [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#hardware-requirements)

| Managed Infrastructure Size | Maximum Number of Clusters | Maximum Number of Nodes | vCPUs            | RAM              |
|:--------------------------: | :------------------------: | :---------------------: | :---:            | :-:              |
| <img width=400/>            | <img width=400/>           | <img width=400/>        | <img width=400/> | <img width=600/> |
| Small	| 150 | 1500 | 4 | 16 GB |
| Medium | 300 | 3000 | 8 | 32 GB |
| Large | 500 | 5000 | 16 | 64 GB |

Please Note: for the large deployment, it is required to follow the best practices found in [this link](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/tuning-and-best-practices-for-rancher-at-scale)

---

### Networking - Upstream Cluster - Management Cluster

Each node used should have a static IP configured, regardless of whether you are installing Rancher on a single node or on an HA cluster. In case of DHCP, each node should have a DHCP reservation to make sure the node gets the same IP allocated.

To operate properly, Rancher requires a number of ports to be open on Rancher nodes and on downstream Kubernetes cluster nodes. for the list of required ports on the upstream and downstream clusters depending on the kubernetes distribution used, please refer to [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements/port-requirements)

---

### Disk - Upstream Cluster - Management Cluster

Rancher performance depends on etcd in the cluster performance. To ensure optimal speed, we recommend always using SSD disks to back your Rancher management Kubernetes cluster. On cloud providers, you will also want to use the minimum size that allows the maximum IOPS. In larger clusters, consider using dedicated storage devices for etcd data and wal directories.

---

### Ingress - Upstream Cluster - Management Cluster

Each node in the Kubernetes cluster that Rancher is installed on should run an Ingress. The Ingress should be deployed as DaemonSet to ensure your load balancer can successfully route traffic to all nodes. For RKE, RKE2 and K3s installations, you don't have to install the Ingress manually because it is installed by default.

For hosted Kubernetes clusters (EKS, GKE, AKS), you will need to set up the ingress.
- Amazon EKS: For details on how to install Rancher on Amazon EKS, including how to install an ingress so that the Rancher server can be accessed, refer to [this page](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster/rancher-on-amazon-eks).
- AKS: For details on how to install Rancher with Azure Kubernetes Service, including how to install an ingress so that the Rancher server can be accessed, refer to [this page](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster/rancher-on-aks).
- GKE: For details on how to install Rancher with Google Kubernetes Engine, including how to install an ingress so that the Rancher server can be accessed, refer to [this page](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster/rancher-on-gke).

---

### Cert Manager - Upstream Cluster - Management Cluster

Rancher uses cert-manager to automatically generate and renew TLS certificates for HA deployments of Rancher. When used to deploy Rancher, it gives the option to automatically generate and rotate self-signed or Let’s Encrypt signed certificates. It is recommended to deploy cert manager using helm charts. Cert Manager requires a CDR (CustomResourceDefinition) to be installed before installing Cert manager. This CRD allows users to interact with cert manager directly with kubectl and allows a user to create custom resources. It is possible to force the installation of the CDR while installing Cert Manager using Helm by specifying the option `--set installCRDs=true`. To deploy cert manager using Helm, use the supported Helm repository for cert manager - jetstack Helm repository

### Certificate for Rancher Server

Rancher has four different options for working with SSL certificates:
- Use self-signed certificates generated by cert-manager.
- Use certificates from Let’s Encrypt.
- Use certificates from a public CA.
- Use certificates from a private CA.

---

### Helm Options for deploying Rancher

When using Helm to deploy Rancher, there are sever option to be set to deploy Rancher as required. For a list of Helm Options, please refer to [this link](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-references/helm-chart-options) Some of the common options are 

| Option           | Default Value	  | Description       | 
|:---------------: | :--------------: | :---------------: |
| <img width=400/> | <img width=400/> | <img width=1400/> |
| `bootstrapPassword` | " " | `string` - Set the bootstrap password for the first admin user. After logging in, the admin will need to reset their password. A randomly generated bootstrap password is used if this value is not set.|
| `hostname` | " " | `string` - the Fully Qualified Domain Name for your Rancher Server |
| `ingress.tls.source` | "rancher" | `string` - Where to get the cert for the ingress. - "rancher, letsEncrypt, secret" |
| `letsEncrypt.email` | " " | `string` - Your email address |
| `letsEncrypt.environment` | "production" | `string` - Valid options: "staging, production" |
| `privateCA` | false | `bool` - Set to true if your cert is signed by a private CA |
| `global.cattle.psp.enabled` | true | `bool` - select 'false' to disable PSPs for Kubernetes v1.25 and above when using Rancher v2.7.2-v2.7.4. When using Rancher v2.7.5 and above, Rancher attempts to detect if a cluster is running a Kubernetes version where PSPs are not supported, and will default it's usage of PSPs to false if it can determine that PSPs are not supported in the cluster. |

---

## Deployment Best Practices - Tips and Tricks

- Install Rancher on a High-Available Kubernetes Cluster
  - A high-availability Kubernetes installation, defined as an installation of Rancher on a Kubernetes cluster with at least three nodes, should be used in any production installation of Rancher, as well as any installation deemed "important." Multiple Rancher instances running on multiple nodes ensure high availability that cannot be accomplished with a single node environment.
- Deploy Rancher on a Dedicated Kubernetes Cluster
  - For the best performance and security, it is recommended to have a dedicated Kubernetes cluster for the Rancher management server. Running user workloads on this cluster is not advised. Other kubernetes cluster can be provisioned from or imported to Rancher Manager.
- Deploy Rancher on a Properly Configured Kubernetes Cluster
  - It's important to follow K8s and etcd best practices when deploying your nodes, including disabling swap, double checking you have full network connectivity between all machines in the cluster, using unique hostname, MAC addresses, and product_uuids for every node, checking that all correct ports are opened, and deploying with ssd backed etcd. More details can be found in the kubernetes docs and etcd's performance op guide.
- Consideration When Deploying Rancher on RKE Cluster
  - RKE keeps record of the cluster state in a file called cluster.rkestate. This file is important for the recovery of a cluster and/or the continued maintenance of the cluster through RKE. Because this file contains certificate material, we strongly recommend encrypting this file before backing up. After each run of rke up you should backup the state file.
- Consideration When Deploying Rancher vSphere
  - If you are installing Rancher in a vSphere environment, refer to the best practices documented in [this link](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/on-premises-rancher-in-vsphere).
- Minimize Network Latency
  - keep the nodes in the dedicated Rancher Manager cluster as close as possible to avoid issues with etcd.
- Consider Using Node Taints
  - Node taints keep Rancher Manager-specific workloads from running on control plane nodes. If there is high CPU or memory usage on control plane nodes, node taints make sure control plane nodes focus on only running the control plane and etcd.
- Run All Nodes in the Cluster in the Same Datacenter
  - For best performance, run all three of your nodes in the same geographic datacenter. If you are running nodes in the cloud, such as AWS, run each node in a separate Availability Zone. For example, launch node 1 in us-west-2a, node 2 in us-west-2b, and node 3 in us-west-2c.
- Development and Production Environments Should be Similar
  - It's strongly recommended to have a "staging" or "pre-production" environment of the Kubernetes cluster that Rancher runs on. This environment should mirror your production environment as closely as possible in terms of software and hardware configuration.

---

## Deploy Rancher

As explained in this page, Rancher support several different deployment options that best fit your needs. These deployment options can be summarizes as:
- Single Node Kubernetes Cluster [Not Recommended]
- High-Availability Kubernetes cluster [Recommended]
- Rancher on EKS Install with the AWS Marketplace
- Docker Install [For Testing and Demonstration ONLY]

All the above listed deployments can be in an environment with direct internet access, behind an HTTP proxy or in an Air Gapped environment

With respect to certificate, Rancher support the below options:
- Rancher Deployment with Self-Signed Certificate [Cert Manager is Required]
- Rancher Deployment with Let's Encrypt Signed Certificate [Cert Manager is Required]
- Rancher Deployment with Public Certificate Authority [Cert Manager is NOT Required - Cert rotation is managed by user]
- Rancher Deployment with Private Certificate Authority [Cert Manager is NOT Required - Cert rotation is managed by user]

In this repo we will install Rancher on a two node kubernetes cluster (not recommended - only for demo) that are based on RKE2 and SUSE Linux Enterprise Server 15 SP 5. We will be deploying Rancher in 5 different deployment options. For a step-by-step guide and to follow the deployment process, choose the required option from the list below and click on the dedicated link of the option to be redirected to the option dedicated step-by-step guide page.
- [Rancher with Self-Signed Certificate](/01-%20Rancher-With-Self-Signed-Cert/) - Direct Internet Access
- [Rancher with Let's Encrypt Signed Certificate](/02-%20Rancher-with-Lets-Encrypt-Signed-Cert/) - Direct Internet Access
- [Rancher with Certificate Authority Signed Certificate](/03-%20Rancher-with-CA-Signed-Cert/) - Direct Internet Access + Public Certificate Authority (AWS ACM)
- [Rancher with Self-Signed Certificate In An AirGapped Environment](/04-%20Rancher-AirGapped/) - No Direct Internet Access
- [Rancher With Self-Signed Certificate Behind AN HTTP Proxy](/05_%20Rancher-With-HTTP-Proxy/) - No Direct Internet Access



---

## References:
- [Rancher Prime Platform](https://www.rancher.com/products/rancher-platform)
- [Rancher Documentation - Main Page](https://ranchermanager.docs.rancher.com/)
- [Rancher Architecture](https://ranchermanager.docs.rancher.com/reference-guides/rancher-manager-architecture)
- [Hardware Requirements](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements#hardware-requirements)
- [Support Matrix](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-2/)
- [Tips For Running Rancher](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/tips-for-running-rancher)
- [Tunning and Best Practices for Rancher At Scale](https://ranchermanager.docs.rancher.com/reference-guides/best-practices/rancher-server/tuning-and-best-practices-for-rancher-at-scale)
- [Port Requirements](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-requirements/port-requirements)
- [Rancher Helm Options](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/installation-references/helm-chart-options)




---

**Enjoy** :blush:

