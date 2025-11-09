# k8s-cluster-provisioning-ansible


### Directory Structure:

```text
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
