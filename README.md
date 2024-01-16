# Sakura Internet

## Ansible install

```bash
sudo apt update
sudo apt install ansible
ansible --version
```

```bash
sudo apt-get install sshpass
cd /home/llmjp0/sakura-ansible
ansible all -m raw -a "hostname"  -k
```

以下のような出力が得られる。
```
172.16.17.231 | CHANGED | rc=0 >>
rksv-isk-2-2-e-s-21-2
Shared connection to 172.16.17.231 closed.
```

## SSH keys settings

```bash
ssh-keygen -t ed25519 -f gpu-node
ansible-playbook  playbooks/setup-ssh-keys.yml
ansible-playbook  playbooks/setup-ssh-keys.yml
```

ssh config
```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/github

Host 172.16.17.*
  User ubuntu
  IdentityFile ~/.ssh/gpu-node
  StrictHostKeyChecking no
```

## GPU node settings

```bash
ansible-playbook playbooks/setup-gpu-nodes.yml
```

## NFS settings

### NFS server (login node)

```bash
ansible-playbook playbooks/setup-nfs-server.yml
```

output
```
PLAY [nfs_server] **********************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Install NFS] *********************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create mount point] **************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create mount point] **************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create mount point] **************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Set exports] *********************************************************************************************************************************************************************************************************
changed: [localhost] => (item={'path': '/mnt/volume/shared', 'options': '*(rw,sync,no_root_squash)', 'hosts': ['172.16.17.0/24']})
changed: [localhost] => (item={'path': '/mnt/volume/home', 'options': '*(rw,sync,no_root_squash)', 'hosts': ['172.16.17.0/24']})

TASK [Restart NFS] *********************************************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************************************************
localhost                  : ok=7    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### NFS client (GPU node)

```bash
ansible-playbook playbooks/setup-nfs.yml
```

## Environment Modules settings

```bash
ansible-playbook playbooks/setup-environment-modules.yml
```

これで、`module avail` が使えるようになる。

### CUDA Toolkit install

```bash
wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda_12.1.0_530.30.02_linux.run
sudo sh cuda_12.1.0_530.30.02_linux.run
```

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CUDA Installer                                                               │
│ - [X] Driver                                                                 │
│      [X] 530.30.02                                                           │
│ + [X] CUDA Toolkit 12.1                                                      │
│   [X] CUDA Demo Suite 12.1                                                   │
│   [X] CUDA Documentation 12.1                                                │
│ - [ ] Kernel Objects                                                         │
│      [ ] nvidia-fs                                                           │
│   Options                                                                    │
│   Install                                                                    │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│ Up/Down: Move | Left/Right: Expand | 'Enter': Select | 'A': Advanced options │
└──────────────────────────────────────────────────────────────────────────────┘
```

CUDA Toolkit 12.1 以外は install しないように check を外す。
さらに A: Advanced options を選択する。すると次のようになるので、Create symbolic link from /usr/local/cuda を外す。またCreate desktop menu shortcuts も外す。

Change Toolkit Install Path を変更する。(例: `/mnt/volume/shared/nvidia/cuda/cuda-12.1`)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ CUDA Toolkit                                                                 │
│   Change Toolkit Install Path                                                │
│   [X] Create symbolic link from /usr/local/cuda                              │
│ - [X] Create desktop menu shortcuts                                          │
│      [X] All users                                                           │
│      [ ] Yes                                                                 │
│      [ ] No                                                                  │
│   [X] Install manpage documents to /usr/share/man                            │
│   Done                                                                       │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│                                                                              │
│ Up/Down: Move | Left/Right: Expand | 'Enter': Select | 'A': Advanced options │
└──────────────────────────────────────────────────────────────────────────────┘
```

(NFS上にあるので各GPU nodeから参照可能 = 各GPU nodeごとでinstallする必要はない)

### cuDNN

https://developer.nvidia.com/rdp/cudnn-download から cuDNN をダウンロードする。

Local Installer for Linux x86_64 (Tar) を download する。

```bash
tar xvf cudnn-linux-x86_64-8.9.7.29_cuda12-archive.tar.xz
```

で解答する。`/mnt/volume/shared/nvidia/cudnn/cudnn-linux-x86_64-8.9.7.29_cuda12`として置く。

### nccl

### modulefiles


## Persistent mode ON !

ref: https://llmjp.slack.com/archives/C06C75EV70W/p1705385526312259?thread_ts=1705078613.305799&cid=C06C75EV70W

```bash
nvidia-smi -q | grep "Persistence Mode"
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
    Persistence Mode                      : Disabled
```

となっていたので、これを全部ONにする。(すべてのノードで)

```bash
ansible-playbook playbooks/setup-persistence-mode.yml
```
