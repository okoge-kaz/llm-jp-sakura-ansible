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
