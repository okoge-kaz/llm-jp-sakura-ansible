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

## CUDA Toolkit install

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

## cuDNN

https://developer.nvidia.com/rdp/cudnn-download から cuDNN をダウンロードする。

Local Installer for Linux x86_64 (Tar) を download する。

```bash
tar xvf cudnn-linux-x86_64-8.9.7.29_cuda12-archive.tar.xz
```

で解凍する。`/mnt/volume/shared/nvidia/cudnn/cudnn-linux-x86_64-8.9.7.29_cuda12`として置く。

## nccl

https://developer.nvidia.com/nccl/nccl-download に行き, Emailやら名前を入れて, cuda driver 12.1 に対応する最新版として, 2.18.3 を選び, NCCLをダウンロード (nccl_2.18.3-1+cuda12.1_x86_64.txz)

```
/mnt/nfs/shared/nvidia/nccl
```

下に nccl_2.18.3-1+cuda12.1_x86_64.txz をおいて展開するだけ

## nvidia fabricmanager

```
https://docs.nvidia.com/datacenter/tesla/pdf/fabric-manager-user-guide.pdf
```
のインストール手順に従う.
NVIDIA のrepositoryの設定は, driver インストールの途中で行われているので, 基本は apt install するだけ.
driver のバージョンと正確に合わせるために,

```
sudo apt install nvidia-fabricmanager-535=535.129.03-1
```

とする (nvidia の driver は apt でインストールされていないため, バージョン番号は, /var/log/nvidia-install.log をまさぐって取得した)

インストール後,

```
nvidia-smi -r
systemctl restart nvidia-fabricmanager
```

以上の操作は ansible に記述した

```
~llmjp0/sakura-ansible/roles/nvidia/tasks/fabricmanager.yml
```

```
nvidia-smi -q 
```

で 

```
   ...
  Fabric
        State                             : Completed
        Status                            : Success
   ...
```

のような出力が 8 GPU 分出ていればOK

なお, nvidia-fabricmanager-535 を apt install すると, nvidia-driver も依存関係でインストールされるのではないか, i.e., aptを使わずに(https://jp.download.nvidia.com/XFree86/Linux-x86_64/535.129.03/NVIDIA-Linux-x86_64-535.129.03.run で) 入れた driver を上書きしてしまうのではないかと恐れていたが, どうやらそんなことはなかった模様 

# Persistent mode ON !

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

## modulefiles

```
/mnt/nfs/shared/modulefiles
```

下に適宜配置していく. llmjp0 の .bashrc に以下を記述

```
module use --append /mnt/nfs/shared/modulefiles
```

(ここに足すのが一番手っ取り早いと思ったが, 全ユーザ共通の設定とはならないので正しいやり方かどうかは確信がない. /mnt/nfs/home/llmjp0/sakura-ansible/roles/env/tasks/main.yml で /etc/environment で MODULEPATH を指定しようとしているかのような記述があったが, NFSサーバに一回設定するだけのほうが簡単そうなので使わずにいる).

```
/mnt/nfs/shared/modulefiles/{cuda,nccl,ucx,openmpi} を作成
```

## ucx

UCX, OpenMPI に関しては最新版で攻めても良かったが UCXバージョンとOpenMPIバージョンの相性とかが怖かったので, mdx で実証済みの, openmpi-4.1.5, ucx-1.14.1 の組み合わせとした

```
/mnt/nfs/shared/ucx
```

下に,

```
https://github.com/openucx/ucx/releases/download/v1.14.1/ucx-1.14.1.tar.gz
```

をダウンロード

```
module load cuda
tar ucx-1.14.1.tar.gz
./contrib/configure-release-mt --prefix=/mnt/nfs/shared/ucx/ucx-1.14.1 --with-cuda=${CUDA_HOME} --enable-optimizations
make -j 40
make -j 40 install
```

## OpenMPI

```
/mnt/nfs/shared/mpi
```

下に

```
https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.5.tar.bz2
```

をダウンロード

```
module load cuda
module load udx
../configure --prefix=/mnt/nfs/shared/mpi/openmpi-4.1.5 --with-ucx=${UCX_HOME} --with-cuda=${CUDA_HOME} --with-platform=../contrib/platform/mellanox/optimized
make -j 40
make -j 40 install
```

## 付記(2023/1/17)

* NFSサーバ遅い (tar の展開や, configureなどで感じる. メタデータ関係が遅い感じ)
* `/etc/exports` ファイルが間違っていた模様 (以下の *(rw, ..) の * が余分)

```
- hosts: nfs_server
  become: yes
  vars:
    nfs_exports:
      - path: "/mnt/volume/shared"
        options: "*(rw,sync,no_root_squash)"
        hosts:
          - "172.16.17.0/24"
      - path: "/mnt/volume/home"
        options: "*(rw,sync,no_root_squash)"
        hosts:
          - "172.16.17.0/24"
```

## python

```
ansible-playbook playbooks/install-venv.yml
```

pyenvをlogin nodeにインストール

```
curl https://pyenv.run | bash
```

~/.bashrc に 以下の内容を書き込み

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

```
pyenv install 3.10
```