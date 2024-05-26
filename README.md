# Kubeadm Ansible Playbook

使用kubeadm和Ansible构建一个简单Kubernetes集群。

支持的操作系统：

  - Ubuntu 16.04
  - CentOS 7
  - Debian 9

环境要求:

  - Ansible `2.4.0+`

# 使用指南

将Kubernetes集群的服务器信息添加到名为 `hosts.ini` 的文件中. 例如:
```
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[kube-cluster:children]
master
node
```

在安装之前， 请根据您自己的配置修改 `group_vars/all.yml` 配置文件。

例如，选择不同的容器运行时（container_runtime）
```yaml
# Container runtimes ('containerd', 'docker')
container_runtime: docker
```

根据不同的容器运行时选择不同的配置
```yaml
# Dokcer engine version
docker_version: 24.0.7

# A list of insecure registries you might need to define
# docker_data_root: /var/lib/docker
docker_registry_mirrors: "https://docker.m.daocloud.io"
docker_dns: "8.8.8.8"
docker_insecure_registries: "https://docker.m.daocloud.io"
# docker_default_runtime:
```

**Note:** utils/certs.d保存了containerd常用的镜像仓库

例如，选择使用 `flannel`作为cni插件而不是calico:

```yaml
# Network implementation('flannel', 'calico')
network: flannel
```

**Note:** 根据服务器的环境，可能需要将 `network_interface` 修改为可用的网卡。默认情况下，`kubeadm-ansible` 使用`eth1`。

设置`kubevip`增加集群高可用功能(默认情况下禁用)。

例如，选择`172.16.10.1`作为高可用的`vip`，选择`eth0`作为高可用的网卡设备`vip_interface`：

```yaml
#High Availability and Load-Balancing
vip: "172.16.10.1"
vip_interface: "eth0"
```

完成设置后，运行 `site.yaml` playbook:

```sh
$ ansible-playbook site.yaml
...
==> master1: TASK [addon : Create Kubernetes dashboard deployment] **************************
==> master1: changed: [192.16.35.12 -> 192.16.35.12]
==> master1:
==> master1: PLAY RECAP *********************************************************************
==> master1: 192.16.35.10               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.11               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.12               : ok=34   changed=29   unreachable=0    failed=0
```

`kubeadm-ansible`会将`/etc/kubernetes/admin.conf`文件拷贝到主节点`$HOME/.kube/config`下。

如果不起作用，请从主节点自行拷贝：

```sh
$ cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

使用 kubectl 验证群集是否完全运行:

```sh

$ export KUBECONFIG=~/admin.conf
$ kubectl get node
NAME      STATUS    AGE       VERSION
master1   Ready     22m       v1.6.3
node1     Ready     20m       v1.6.3
node2     Ready     20m       v1.6.3

$ kubectl get po -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-master1                            1/1       Running   0          23m
...
```

# 重置集群

最后，可以使用`reset-site.yaml`重置已安装的Kubernetes集群:

```sh
$ ansible-playbook reset-site.yaml
```

# 附加功能
这些是可以安装到Kubernetes集群的附加功能，可以让您使用Kubernetes集群时更加轻松便捷。

可以在`group_vars/all.yml`中启用/禁用这些功能（默认情况下全部禁用）：
```
# Additional feature to install
additional_features:
  helm: false
  crictl: false
```

## Helm
在集群中安装`helm` (https://helm.sh)

## crictl
在集群中安装`crictl` (https://github.com/kubernetes-sigs/cri-tools)
