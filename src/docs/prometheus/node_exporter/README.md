# 部署 Prometheus Node Exporter

```bash
conda activate ansible
```

```bash
ansible-inventory -i inventory.yaml --list
```

ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.16
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.18
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.0.218

```bash
ansible all -m ping -i inventory.yaml
```

## 依赖

```bash
ansible-galaxy install -r requirements.yml
```

## 安装

```bash
ansible-playbook -i inventory.yaml playbook.yaml
```

## 测试

```shell
curl 192.168.0.16:19100/metrics
curl 192.168.0.18:19100/metrics
```
