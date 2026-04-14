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

| 変数名 | 概要 | デフォルト値 | 指定可能な値 |
| --- | --- | --- | --- |
| `nvidia_container_host_manage_repository` | NVIDIA のパッケージリポジトリと鍵の管理を本 Role で行うかどうか。 | `true` | `true`, `false` |
| `nvidia_container_host_packages` | NVIDIA コンテナ対応としてインストールするパッケージ一覧。 | `['nvidia-container-toolkit']` | パッケージ名のリスト |
| `nvidia_container_host_configure_runtime` | `nvidia-ctk runtime configure` を実行するかどうか。 | `true` | `true`, `false` |
| `nvidia_container_host_runtime` | `nvidia-ctk runtime configure` の対象ランタイム。 | `docker` | ランタイム名文字列（例: `docker`, `containerd`） |
| `nvidia_container_host_set_as_default_runtime` | `nvidia-ctk` 実行時に `--set-as-default` を付けるかどうか。 | `false` | `true`, `false` |
| `nvidia_container_host_runtime_service_name` | ランタイム設定変更後に再起動するサービス名。 | `{{ nvidia_container_host_runtime }}` | 対象ホストで有効なサービス名 |
| `nvidia_container_host_restart_runtime_service` | 設定変更時にランタイムサービスを再起動するかどうか。 | `true` | `true`, `false` |
| `nvidia_container_host_manage_grub_cmdline_linux_default` | `GRUB_CMDLINE_LINUX_DEFAULT` を管理するかどうか。 | `true` | `true`, `false` |
| `nvidia_container_host_grub_cmdline_linux_default_additional_params` | `GRUB_CMDLINE_LINUX_DEFAULT` に追加するカーネルパラメータ。 | `['pcie_acs_override=downstream,multifunction']` | カーネルパラメータ文字列のリスト |
| `nvidia_container_host_grub_cmdline_linux_default_remove_params` | `GRUB_CMDLINE_LINUX_DEFAULT` から削除するカーネルパラメータ。 | `[]` | カーネルパラメータ文字列のリスト |
| `nvidia_container_host_reboot_after_grub_update` | GRUB 更新後にホストを再起動するかどうか。 | `false` | `true`, `false` |
| `nvidia_container_host_reboot_timeout` | 再起動有効時の完了待機タイムアウト（秒）。 | `600` | 正の整数 |

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
