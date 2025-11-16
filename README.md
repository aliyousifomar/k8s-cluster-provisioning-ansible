# k8s-cluster-provisioning-ansible


![Screencast from 2025-11-13 18-50-03](https://github.com/user-attachments/assets/ffa75625-4bf1-45e2-8075-6007f7ef1c02)




### Directory Structure:

```text
├── ansible
│   ├── group_vars
│   │   ├── control_plane.yml
│   │   └── worker.yml
│   ├── inventory
│   │   └── hosts
│   ├── roles
│   │   ├── common
│   │   │   ├── handlers
│   │   │   │   └── main.yml
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   ├── kubeadm-control
│   │   │   └── tasks
│   │   │       └── main.yml
│   │   └── kubeadm-worker
│   │       └── tasks
│   │           └── main.yml
│   ├── site.yml
│   ├── vars
│   │   └── main.yml
│   └── worker2.yml
├── kubernetes
│   └── monitoring
│       ├── monitoring.coreos.com_prometheuses.yaml
│       ├── role-rbinding-read-secret.yaml
│       └── values.yaml
├── LICENSE
└── README.md
```

## Prerequisites
- SSH access to all hosts with sudo privileges.
    To setup this part:
      - make sure you have network connectivity from controller/jumpbox to both VMs.
      - configure passwordless ssh with a user with sudo rights to both VMs.
- Ansible 2.9+ installed on the management machine (I used core 2.16.3).
- Basic knowledge of YAML and Kubernetes components.

## Usage Instructions

1. **Edit hosts inventory:**
   - Modify `ansible/inventory/hosts` with IPs or hostnames for control plane and workers.

2. **Set group variables:**
   - Adjust values in `group_vars/control_plane.yml` and `group_vars/worker.yml` to match your environment.

3. **Run the provisioning playbook:**

`ansible-playbook -i ansible/inventory/hosts ansible/site.yml -K  # where -K is to supply the sudo/become password`

*note:* The following screenshot shows the playbook running:

![Screencast from 2025-11-09 09-37-37](https://github.com/user-attachments/assets/6cd02583-25aa-4f75-9e71-d2ec5b87a02a)


4. **Validate deployment:**
- Use `kubectl` on the control plane node to verify nodes are healthy:

`kubectl get nodes -o wide`

![Screencast from 2025-11-09 09-39-06](https://github.com/user-attachments/assets/6547ee88-377a-43aa-81ac-a930199f07a6)


## Customization
- Adjust or add roles as needed for cloud-specific steps, monitoring setup, or advanced networking integrations.
- Templates and variables can be extended to support different OS families or Kubernetes distributions.

## Known Limitations
- Assumes passwordless SSH connectivity.
- Packages and repositories used require outbound internet access. This can be avoided by implementing a local repository in a secured environment.

## Future Enhancements

- High-availability control plane support.
- Automated certificate rotation.
- Built-in monitoring and alerting integrations.
- CI/CD integration.
- Automated upgrade.
- Automated Security scanning for the cluster on Infrastructure level.

## Additions:
- Added another worker node by creating a playbook and modifying the hosts file to add the details of the targeted VM.

# Adding monitoring solution to the cluster  📈 

## 1. Overview
A brief overview of the monitoring stack setup, including components like Prometheus, Grafana, Alertmanager, etc.

## 2. Prerequisites
- Kubernetes cluster ready and accessible
- Helm installed and configured
- Namespace created: `monitoring`

## 3. Prepare Environment

```bash
# Create the monitoring namespace
kubectl create namespace monitoring

# Add Prometheus Helm repository
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus stack
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

## 4. Configure Helm Values
Create or update your `values.yaml` with the following key settings to expose Grafana on a NodePort:

```yaml
grafana:
adminPassword: ""
  extraSecretMounts:
    - name: grafana-admin-password
      secretName: grafana-admin-password
      mountPath: /etc/secrets/grafana
      defaultMode: 0440
  adminPasswordFile: /etc/secrets/grafana/grafana-admin-password
 # The following will make the service available on a node-IP (worker node) on port 30432
 service:
   type: NodePort
   nodePort: 30432
image:
repository: docker.io/grafana/grafana
tag: 9.3.8
```

## 5. Install/Upgrade the Stack

```bash
helm upgrade --install monitoring-stack prometheus-community/kube-prometheus-stack -n monitoring -f values.yaml --timeout 10m0s

```


## 6. Check Deployment Status

```bash
helm status monitoring-stack -n monitoring
kubectl -n monitoring get pods
```

## Recent Progress and Improvements ✅

### bootstrapping a cluster with multiple control planes for high availability

- ** Load balancer: According to kubernetes guidelines, a load balancer should be deployed to provide a single point of entrance for the API server, I have used an external load balancer in a VM (used haproxy). Load balancer deployment can be viewed at `load-balancer.yml`. 
- **External Load Balancer Support:** The first control plane node is initialized with `--control-plane-endpoint` flag pointing to your VIP or DNS, allowing all control plane nodes to share a consistent API endpoint.
- **Automated Certificate Handling:** Using the `--upload-certs` flag, certificates are securely shared among control plane nodes during joining.
- **Dynamic Join Command Distribution:** Join commands for control plane and worker nodes are generated and fetched dynamically, allowing seamless node addition.
- **Pre-flight etcd Health Checks:** Before adding a control plane node, etcd cluster health is validated to prevent premature join attempts that cause learner sync errors.
- **Retry Logic:** Joining control plane and worker nodes implement retry and delay logic to handle temporary issues with etcd sync or network delays gracefully.
- **Node Cleanup Automation:** Kubelet service is automatically stopped and Kubernetes config files removed on nodes before attempting join, eliminating preflight errors caused by leftover state—even on freshly deployed OS images.
- **Idempotency and Error Handling:** Tasks are designed to allow safe re-runs with explicit failure detection for distinct errors (such as “already joined”) to improve automation resilience.

## Usage

1. **Prepare Inventory and Variables:**
   - Define your control plane and worker hosts in the Ansible inventory.
   - Configure group variables for Kubernetes network CIDR, VIP address, tokens, etc.

2. **Run site Setup Playbook:**
`ansible-playbook -i inventory/hosts site.yml --limit control_plane -K`

# Insert the playbook run here

4. **Verify Cluster:**

# Insert verification screencast here
---

## Recommendations and Notes

- Ensure the external load balancer is properly configured to route port 6443 to all control plane nodes.
- Verify networking connectivity and firewall rules permit etcd ports (2379, 2380) and Kubernetes API traffic.
- Use idempotent Ansible running techniques and check logs/debugs during provisioning.
- This automation assumes passwordless SSH access with a user that has sudo privileges from the Ansible control node.