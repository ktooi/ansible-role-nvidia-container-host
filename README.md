# ansible-role-nvidia-container-host

An Ansible Role to prepare a container host for NVIDIA GPU workloads.

## What this role does

- Installs `nvidia-container-toolkit`
- Configures NVIDIA package repositories:
  - APT for Debian/Ubuntu
  - YUM/DNF for RHEL-compatible distributions
- Runs `nvidia-ctk runtime configure` for container runtime integration
- Manages `GRUB_CMDLINE_LINUX_DEFAULT` parameters
  - Default additional parameter: `pcie_acs_override=downstream,multifunction`
- Runs `update-grub` / `grub2-mkconfig` when GRUB settings change

> This role is designed to coexist with external NVIDIA driver management roles
> (e.g. `nvidia.nvidia_driver`). It does **not** install or update kernel drivers.

## Supported distributions (as of April 2026)

- RHEL / Rocky Linux / AlmaLinux: 8, 9, 10
- Debian: 11 (bullseye), 12 (bookworm)
- Ubuntu LTS: 22.04 (jammy), 24.04 (noble)

## Variables

Key role variables:

- `nvidia_container_host_manage_repository`
- `nvidia_container_host_packages`
- `nvidia_container_host_configure_runtime`
- `nvidia_container_host_runtime`
- `nvidia_container_host_set_as_default_runtime`
- `nvidia_container_host_manage_grub_cmdline_linux_default`
- `nvidia_container_host_grub_cmdline_linux_default_additional_params`
- `nvidia_container_host_grub_cmdline_linux_default_remove_params`

Distribution-specific values are loaded via `tasks/variables.yml` in this order:

1. `vars/{{ ansible_os_family }}-{{ ansible_distribution_major_version }}.yml`
2. `vars/{{ ansible_os_family }}.yml`

## Example playbook

```yaml
- hosts: gpu_container_hosts
  become: true
  roles:
    - role: ktooi.nvidia_container_host
      vars:
        nvidia_container_host_runtime: docker
        nvidia_container_host_set_as_default_runtime: true
        nvidia_container_host_grub_cmdline_linux_default_additional_params:
          - pcie_acs_override=downstream,multifunction
```

## Japanese README

- See [README_ja.md](README_ja.md) for Japanese documentation.
