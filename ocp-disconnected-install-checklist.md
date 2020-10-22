# OpenShift Bare Metal Disconnected Installation Checklist

Last updated for OpenShift 4.5 - October 22, 2020

For canonical and more detailed information, please see the official docs:  https://docs.openshift.com/container-platform/4.5/installing/installing_bare_metal/installing-bare-metal.html#prerequisites

If there is a conflict between this checklist and the [official docs](https://docs.openshift.com/container-platform/4.5/installing/installing_bare_metal/installing-bare-metal.html#prerequisites), defer to the [official docs](https://docs.openshift.com/container-platform/4.5/installing/installing_bare_metal/installing-bare-metal.html#prerequisites).

## Pre-requisites

### Hardware

- [ ] Bastion machine (RHEL 8.2)
- [ ] DHCP (Can reuse bastion host)
- [ ] Container Registry (Can reuse bastion host)
- [ ] HTTP server for assets (Can reuse bastion host)
- [ ] Load balancer for API (Software or hardware)
- [ ] Load balancer for Ingress (Software or hardware)
- [ ] Bootstrap Node: 4 vCPU - 16 GB vRAM - 120 GB Storage (Will run RHCOS. After cluster install can be destroyed or re-purposed)
- [ ] Master nodes: 4 vCPU - 16 GB vRAM - 120 GB Storage
  - [ ] Master 1
  - [ ] Master 2
  - [ ] Master 3
- [ ] Worker nodes: 2 vCPU - 8 GB vRAM - 120 GB Storage
  - [ ] Worker 1
  - [ ] Worker 2
  - [ ] Worker 3
  - [ ] Worker 4
  - [ ] Worker 5
  - [ ] Worker 6
  - [ ] Worker 7
  - [ ] Worker 8

### Configured Networking

#### Load Balancers - Layer 4 (TCP) AKA TLS pass-through enabled

- [ ] API load balancer
  - [ ] Kubernetes API (Internal and External)
    - [ ] Frontend 6443/tcp
    - [ ] Backend to Bootstrap machine and Master machines (control plane)
  - [ ] Machine Config Server (Internal Only)
    - [ ] Frontend 22623/tcp
    - [ ] Backend to Bootstrap machine and Master machines (control plane)
- [ ] Routes:
  - [ ] HTTP traffic (Internal and External)
    - [ ] Frontend 80/tcp - Layer 4 (TCP) (or Layer 7 (HTTP)?)
    - [ ] Backend to worker/compute machines (data plane)
  - [ ] HTTPS traffic (Internal and External)
    - [ ] Frontend 443 - Layer 4 (TCP) mode AKA TLS pass-through
    - [ ] Backend to Bootstrap machine and Master machines (control plane)

#### DNS

- [ ] `api.<clustername>.<domain>` points to API load balancer, resolves from **outside** the cluster
- [ ] `api-int.<clustername>.<domain>` points to API Load Balancer, resolves from **inside** the cluster
- [ ] `*.apps.<clustername>.<domain>` points to Routes load balancer, resolves from outside the cluster

#### DHCP - Static IP config (delivered via DHCP) for each OpenShift node
- [ ] Bootstrap
- [ ] Master 1
- [ ] Master 2
- [ ] Master 3
- [ ] Worker 1
- [ ] Worker 2
- [ ] Worker 3
- [ ] Worker 4
- [ ] Worker 5
- [ ] Worker 6
- [ ] Worker 7
- [ ] Worker 8

#### IPAM/IP planning

- [ ] Approved subnet for Pods (default is 10.128.0.0/14 - host prefix: 23)
- [ ] Approved subnet for Services (default is 172.30.0.0/16)

#### Node Firewall Configuration (allow all below)

##### All Machines to all machines

- [ ] ICMP (Network reachability tests)
- [ ] TCP
  - [ ] 9000-9999 (Host level services, including the node exporter on ports 9100-9101 and the Cluster Version Operator on port 9099)
  - [ ] 10250-10259 (The default ports that Kubernetes reserves)
  - [ ] 10256 (openshift-sdn)
- [ ] UDP
  - [ ] 4789 (VXLAN and Geneve)
  - [ ] 6081 (VXLAN and Geneve)
  - [ ] 9000-9999 (Host level services, including the node exporter on ports 9100-9101)
- [ ] TCP/UDP
  - [ ] 30000-32767 (Kubernetes NodePort)

##### All Machines to Control Plane machines

- [ ] TCP
  - [ ] 2379-2380 (etcd server, peer, and metrics ports)
  - [ ] 6443 (Kubernetes API)


### Post-install (Day 2)

