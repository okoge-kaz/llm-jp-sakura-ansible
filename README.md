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
