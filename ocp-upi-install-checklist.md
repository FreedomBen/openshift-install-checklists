# OpenShift Bare Metal/UPI Installation Checklist

Last updated for OpenShift 4.6 - November 2, 2020

For canonical and more detailed information, please see the official docs:  https://docs.openshift.com/container-platform/4.6/welcome/index.html

For UPI on vSphere:  https://docs.openshift.com/container-platform/4.6/installing/installing_vsphere/installing-vsphere.html
Also useful (for SRV records for example): https://www.openshift.com/blog/installing-ocp-4.3-on-vmware-with-upi

This blog post can also help explain things in more detail but do note it is OCP 4.3 (which is close to out of date as I write this on US Election Day, November of 2020):  https://www.openshift.com/blog/installing-ocp-4.3-on-vmware-with-upi

If there is a conflict between this checklist and the [official docs](https://docs.openshift.com/container-platform/4.6/welcome/index.html), defer to the [official docs](https://docs.openshift.com/container-platform/4.6/welcome/index.html).

## Pre-requisites

### Hardware

- [ ] Bastion machine (RHEL 8.2)
- [ ] DHCP (Can reuse bastion host)
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

- [ ] Forward DNS records - (A records)
  - [ ] Load Balancer DNS
    - [ ] API: `api.<clustername>.<domain>` points to API load balancer, resolves from **outside** the cluster
    - [ ] API: `api-int.<clustername>.<domain>` points to API Load Balancer, resolves from **inside** the cluster
    - [ ] Routes/Ingress: `*.apps.<clustername>.<domain>` points to Routes load balancer, resolves from outside the cluster
  - [ ] Node DNS
    - [ ] Bootstrap - Example: `ocp-<clustername>-bootstrap.<domain>` (Remove after control is plane is up)
    - [ ] Master 1 - Example: `ocp-<clustername>-master01.<domain>`
    - [ ] Master 2 - Example: `ocp-<clustername>-master02.<domain>`
    - [ ] Master 3 - Example: `ocp-<clustername>-master03.<domain>`
    - [ ] Worker 1 - Example: `ocp-<clustername>-worker01.<domain>`
    - [ ] Worker 2 - Example: `ocp-<clustername>-worker02.<domain>`
    - [ ] Worker 3 - Example: `ocp-<clustername>-worker03.<domain>`
    - [ ] Worker 4 - Example: `ocp-<clustername>-worker04.<domain>`
    - [ ] Worker 5 - Example: `ocp-<clustername>-worker05.<domain>`
    - [ ] Worker 6 - Example: `ocp-<clustername>-worker06.<domain>`
    - [ ] Worker 7 - Example: `ocp-<clustername>-worker07.<domain>`
    - [ ] Worker 8 - Example: `ocp-<clustername>-worker08.<domain>`
  - [ ] etcd (`etcd-<index>.<clustername>.<domain>`) - (Can be CNAME)
    - [ ] `etcd-0.<clustername>.<domain>`
    - [ ] `etcd-1.<clustername>.<domain>`
    - [ ] `etcd-2.<clustername>.<domain>`
- [ ] Reverse DNS records - (PTR records) - Used by RHCOS to set hostnames
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
- [ ] Service Discovery (SRV records for Hostname: `_etcd-server-ssl._tcp.<clustername>.<domain>` priority: 0, weight: 10 and port: 2380)
  - [ ] Server: `etcd-0.<clustername>.<domain>`
  - [ ] Server: `etcd-1.<clustername>.<domain>`
  - [ ] Server: `etcd-2.<clustername>.<domain>`


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


### Post-install

#### Authentication
- [ ] [Internal oauth](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/authentication_and_authorization/configuring-internal-oauth)
- [ ] [Configure HTPasswd IDP](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/authentication_and_authorization/configuring-identity-providers#configuring-htpasswd-identity-provider)
- [ ] [Configure Main IDP (LDAP, OIDC, etc)](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/authentication_and_authorization/configuring-identity-providers)
- [ ] [Create "break glass" user using HTPasswd](#)
- [ ] [Remove kubeadmin user](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/authentication_and_authorization/removing-kubeadmin) ([alt link](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/preparing-for-users.html#understanding-kubeadmin_post-install-preparing-for-users))
- [ ] [LDAP Group Sync](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/authentication_and_authorization/ldap-syncing)

#### Machine and Node Config
- [ ] Create infra nodes ([Add label to worker](https://access.redhat.com/solutions/4287111)) ([Create machineset](https://docs.openshift.com/container-platform/4.6/machine_management/creating-infrastructure-machinesets.html)) ([Move resources to infra nodes](https://docs.openshift.com/container-platform/4.6/machine_management/creating-infrastructure-machinesets.html#moving-resources-to-infrastructure-machinesets))
- [ ] [Machine Health Checks](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/node-tasks.html#post-installation-config-deploying-machine-health-checks)
- [ ] [Cluster Auto-scaling](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/cluster-tasks.html#cluster-autoscaler-about_post-install-cluster-tasks)
- [ ] [Node Tuning Operator](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/node-tasks.html#post-using-node-tuning-operator) ([Blog Post](https://www.openshift.com/blog/node-tuning-operator-and-friends-in-openshift-4.5))
- [ ] [Max Pods per Node](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/node-tasks.html#nodes-nodes-managing-max-pods-about_post-install-node-tasks)
- [ ] [Taints and Tolerations](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/node-tasks.html#post-install-taints-tolerations)
- [ ] [Configure Huge Pages](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/node-tasks.html#configuring-huge-pages_post-install-node-tasks)
- [ ] [Device Plugins](https://docs.openshift.com/container-platform/3.11/dev_guide/device_plugins.html)
- [ ] [Real Time Kernel](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/machine-configuration-tasks.html#nodes-nodes-rtkernel-arguments_post-install-machine-configuration-tasks)
- [ ] [Journald settings](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/machine-configuration-tasks.html#machineconfig-modify-journald_post-install-machine-configuration-tasks)

#### Persistent Storage
- [ ] [Setup Persistent Storage](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/storage-configuration.html) - [NFS](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-nfs.html) - [OCS](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-ocs.html) - [vSphere](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-vsphere.html) - [local](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-local.html) - [EFS](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-efs.html) - [EBS](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-aws.html) - [Azure Disk](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-azure.html) - [Azure File](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-azure-file.html) - [hostPath](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-hostpath.html) - [iSCSI](https://docs.openshift.com/container-platform/4.6/storage/persistent_storage/persistent-storage-iscsi.html)
- [ ] [Dynamic Provisioning](https://docs.openshift.com/container-platform/4.6/storage/dynamic-provisioning.html)

#### Registry
- [ ] Enable Registry:  ([Bare Metal and vSPhere](https://docs.openshift.com/container-platform/4.6/registry/configuring-registry-operator.html#registry-removed_configuring-registry-operator))
- [ ] Configure Registry:  [AWS](https://docs.openshift.com/container-platform/4.6/registry/configuring_registry_storage/configuring-registry-storage-aws-user-infrastructure.html) - [GCP](https://docs.openshift.com/container-platform/4.6/registry/configuring_registry_storage/configuring-registry-storage-gcp-user-infrastructure.html) - [Azure](https://docs.openshift.com/container-platform/4.6/registry/configuring_registry_storage/configuring-registry-storage-azure-user-infrastructure.html) - [Bare Metal](https://docs.openshift.com/container-platform/4.6/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html) - [vSphere](https://docs.openshift.com/container-platform/4.6/registry/configuring_registry_storage/configuring-registry-storage-vsphere.html)
- [ ] [Expose the registry](https://docs.openshift.com/container-platform/4.6/registry/securing-exposing-registry.html)

#### Security and Compliance
- [ ] [Compliance Operator](https://docs.openshift.com/container-platform/4.6/security/compliance_operator/compliance-operator-understanding.html)
- [ ] [File Integrity Operator](https://docs.openshift.com/container-platform/4.6/security/file_integrity_operator/file-integrity-operator-understanding.html)
- [ ] [Replace Default Ingress Certificate](https://docs.openshift.com/container-platform/4.6/security/certificates/replacing-default-ingress-certificate.html)
- [ ] [Adding API Server Certificates](https://docs.openshift.com/container-platform/4.6/security/certificates/api-server.html)
- [ ] [Service Certificates](https://docs.openshift.com/container-platform/4.6/security/certificates/service-serving-certificate.html)
- [ ] [Enable etcd Encryption](https://docs.openshift.com/container-platform/4.6/security/encrypting-etcd.html) ([alternate link](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/post-installation_configuration/post-install-cluster-tasks#post-install-etcd-tasks))
- [ ] [Container Security Operator](https://docs.openshift.com/container-platform/4.6/security/pod-vulnerability-scan.html)
- [ ] [Red Hat CoreOS (RHCOS) Hardening Guide](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html/security_and_compliance/container-security#security-hardening)
- [ ] [USB Guard](https://docs.openshift.com/container-platform/4.6/post_installation_configuration/machine-configuration-tasks.html#rhcos-add-extensions_post-install-machine-configuration-tasks) ([RHEL docs](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/security_hardening/index#usbguard_protecting-systems-against-intrusive-usb-devices))

#### Logging
- [ ] [Install logging stack](https://docs.openshift.com/container-platform/4.6/logging/cluster-logging-deploying.html)
- [ ] [Cluster logging configuration](https://docs.openshift.com/container-platform/4.6/logging/config/cluster-logging-configuring-cr.html)
- [ ] [Setup log forwarding (external log aggregators)](https://docs.openshift.com/container-platform/4.6/logging/cluster-logging-external.html)

#### Monitoring
- [ ] [Configure monitoring stack](https://docs.openshift.com/container-platform/4.6/monitoring/configuring-the-monitoring-stack.html#configuring-the-monitoring-stack)
- [ ] [Enable monitoring for user-defined projects](https://docs.openshift.com/container-platform/4.6/monitoring/enabling-monitoring-for-user-defined-projects.html)
- [ ] [Setup Alerts](https://docs.openshift.com/container-platform/4.6/monitoring/managing-alerts.html)
- [ ] [Expose custom metrics for autoscaling](https://docs.openshift.com/container-platform/4.6/monitoring/exposing-custom-application-metrics-for-autoscaling.html)

#### Service Mesh


#### Metering

