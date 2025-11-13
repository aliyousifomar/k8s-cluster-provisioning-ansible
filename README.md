# k8s-cluster-provisioning-ansible


![Screencast from 2025-11-13 14-51-05 (2)](https://github.com/user-attachments/assets/7193bd84-3c5c-4eee-b513-0f838d729375)


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




