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

## Role Variables

| Variable | Description | Default | Allowed values |
| --- | --- | --- | --- |
| `nvidia_container_host_manage_repository` | Whether this role manages NVIDIA package repositories and repository keys. | `true` | `true`, `false` |
| `nvidia_container_host_packages` | Package list installed for NVIDIA container support. | `['nvidia-container-toolkit']` | List of package names |
| `nvidia_container_host_configure_runtime` | Whether to run `nvidia-ctk runtime configure`. | `true` | `true`, `false` |
| `nvidia_container_host_runtime` | Target container runtime for `nvidia-ctk runtime configure`. | `docker` | Runtime name string (e.g. `docker`, `containerd`) |
| `nvidia_container_host_set_as_default_runtime` | Whether to pass `--set-as-default` to `nvidia-ctk`. | `false` | `true`, `false` |
| `nvidia_container_host_runtime_service_name` | Service name restarted after runtime config changes. | `{{ nvidia_container_host_runtime }}` | Valid service name on the target host |
| `nvidia_container_host_restart_runtime_service` | Whether to restart the runtime service when config changes. | `true` | `true`, `false` |
| `nvidia_container_host_manage_grub_cmdline_linux_default` | Whether to manage `GRUB_CMDLINE_LINUX_DEFAULT`. | `true` | `true`, `false` |
| `nvidia_container_host_grub_cmdline_linux_default_additional_params` | Kernel parameters to add to `GRUB_CMDLINE_LINUX_DEFAULT`. | `['pcie_acs_override=downstream,multifunction']` | List of kernel parameter strings |
| `nvidia_container_host_grub_cmdline_linux_default_remove_params` | Kernel parameters to remove from `GRUB_CMDLINE_LINUX_DEFAULT`. | `[]` | List of kernel parameter strings |
| `nvidia_container_host_reboot_after_grub_update` | Whether to reboot the host after GRUB settings are updated. | `false` | `true`, `false` |
| `nvidia_container_host_reboot_timeout` | Timeout (seconds) for reboot completion when reboot is enabled. | `600` | Positive integer |

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
