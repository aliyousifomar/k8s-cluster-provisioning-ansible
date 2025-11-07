# k8s-cluster-provisioning-ansible


### Directory Structure:

ansible/
├── group_vars/
│   ├── control_plane.yml
│   └── worker.yml
├── inventory/
│   └── hosts
├── roles/
│   ├── common/              # General setups
│   ├── kubeadm-control/     # Control plane node tasks
│   └── kubeadm-worker/      # Worker node tasks
├── site.yml                 # Entrypoint playbook
└── vars/
    └── main.yml             # Global variables
kubernetes/

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


4. **Validate deployment:**
- Use `kubectl` on the control plane node to verify nodes are healthy:

`kubectl get nodes -o wide`

## Customization
- Adjust or add roles as needed for cloud-specific steps, monitoring setup, or advanced networking integrations.
- Templates and variables can be extended to support different OS families or Kubernetes distributions.

## Known Limitations
- Tested primarily with Ubuntu 24.04.3 LTS.
- VMs running on two x KVM/QEMU hosts.
- Assumes passwordless SSH connectivity.
- Network plugin default requires outbound internet access.

## Future Enhancements

- High-availability control plane support.
- Automated certificate rotation.
- Built-in monitoring and alerting integrations.
- CI/CD integration.
- Automated upgrade.
- Automated Security scanning for the cluster on Infrastructure level.