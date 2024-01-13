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

