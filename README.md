# DeathstarBench Deployment

This repository provides scripts to deploy the **Social Network** application from the [DeathStarBench](https://github.com/delimitrou/DeathStarBench) benchmark suite on a **Kubernetes cluster**, along with optional **monitoring** tools and **HPA** (Horizontal Pod Autoscaler) configuration.

## Repository Structure

```
.
├── install_k8s.sh           # Installs Kubernetes with containerd and Flannel
├── deploy_monitoring.sh     # Deploys Prometheus & Grafana
├── fetch_ports.sh           # Gets the exposed ports of services
├── deploy_socialnetowrk.sh  # Deploys the Social Network app using Helm
├── hpa_values.yaml          # Values for HPA configuration
├── service_values.yaml      # Values for service deployment
├── README.md                # This file
```

## Quickstart

### 1. Install Kubernetes

#### Option 1: Bare-metal Kubernetes on Ubuntu 24.04

```bash
chmod +x install_k8s.sh
./install_k8s.sh
```

This will:
- Install `kubeadm`, `kubelet`, and `kubectl`
- Configure `containerd` as the container runtime
- Apply the Flannel CNI
- Initialize a single-node cluster

#### Option 2: Kind 

Requirements: Docker

```bash
brew install kind 
kind create cluster --name socialnetwork
kubectl cluster-info --context ocialnetwork
```

### 2. (Optional) Deploy Monitoring Stack

```bash
chmod +x deploy_monitoring.sh
./deploy_monitoring.sh
```

Installs:
- Prometheus
- Grafana

### 3. Deploy the Social Network App

Requirements:
```bash
sudo apt install libssl-dev (previous requirement)
sudo apt install zlib1g-dev
sudo apt-get install luarocks
sudo luarocks install luasocket
```

Once ready:

```bash
chmod +x deploy_socialnetowrk.sh
./deploy_socialnetowrk.sh
```

This script:
- Clones DeathStarBench (with PR #352)
- Initializes submodules
- Installs the app via Helm 
- Compiles wrk loader

### 4. Fetch Service Ports

```bash
chmod +x fetch_ports.sh
./fetch_ports.sh
```

Lists exposed NodePorts for UI and services.


## Custom values for HPA and main services

You can customize HPA values via `hpa_values.yaml`.
You can customize service resources via `service_values.yaml`.

To apply HPA:
```bash
kubectl apply -f hpa_values.yaml -n socialnetwork
kubectl apply -f service_values.yaml -n socialnetwork
```


## Load Testing:

You can use the `wrk` tool with the custom loader available in the fork:
```
https://github.com/bserracanta/DeathStarBench/tree/master/socialNetwork/loader
```

Change url to nginx NodePort address.
The default output file might have a different name, to check it run the following query inside the loader directory:

```bash
 ../../wrk2/wrk -D exp -t 1 -c 1 -d 50s -L -s ./../wrk2/scripts/social-network/compose-post.lua http://<nginxNodePort:Port>/wrk2-api/post/compose -T 1s -R 5
```


Requirements: python3, wrk compiled.

```bash
cd DeathStarBench/socialNetwork/loader
python3 loader.py --query_rate 40
```

