# ansible-role-nvidia-container-host

Container 環境で NVIDIA GPU を利用可能にする Container Host を構築する Ansible Role です。

## 実装内容

- `nvidia-container-toolkit` のインストール
- Debian/Ubuntu 向け APT リポジトリ設定
- RHEL/RHEL 系クローン向け YUM/DNF リポジトリ設定
- `nvidia-ctk runtime configure` によるコンテナランタイム設定
- `GRUB_CMDLINE_LINUX_DEFAULT` へのパラメータ追加/削除
  - 既定値: `pcie_acs_override=downstream,multifunction`
- `update-grub` / `grub2-mkconfig` 実行

> 既存の GPU ドライバ管理 (`nvidia.nvidia_driver` role など) と共存できるよう、
> 本 Role はドライバの導入・更新は行いません。

## サポート対象 (2026-04 時点)

- RHEL / Rocky Linux / AlmaLinux: 8, 9, 10
- Debian: 11 (bullseye), 12 (bookworm)
- Ubuntu LTS: 22.04 (jammy), 24.04 (noble)

## Role Variables

主な変数:

- `nvidia_container_host_manage_repository`
- `nvidia_container_host_packages`
- `nvidia_container_host_configure_runtime`
- `nvidia_container_host_runtime`
- `nvidia_container_host_set_as_default_runtime`
- `nvidia_container_host_manage_grub_cmdline_linux_default`
- `nvidia_container_host_grub_cmdline_linux_default_additional_params`
- `nvidia_container_host_grub_cmdline_linux_default_remove_params`

ディストリビューションごとの差分は `tasks/variables.yml` で `vars/{{ ansible_os_family }}-{{ ansible_distribution_major_version }}.yml` → `vars/{{ ansible_os_family }}.yml` の順に読み込む方式です。

## Example Playbook

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
