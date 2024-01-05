# k3s
- Если сравнивать с традиционным кластером Kubernetes, то в k3s нет четкого различия между мастер и воркер нодами. Управление подами происходит на любой ноде, независимо от роли, которую они играют. Таким образом, понятия мастер и воркер нод не применимы к кластеру k3s.
- Rancher Labs избавились от множества необязательных компонентов Kubernetes, например, от плагинов объема хранилища и API-интерфейсов облачных провайдеров.
- Затем были добавлены некоторые важные элементы, включая containerd, Flannel, CoreDNS, CNI, ingress controller Traefik, локальное хранилище данных, встроенный сервис Load Balancer. Все эти элементы упакованы в один бинарный файл и выполняются в рамках одного процесса. Помимо этого, дистрибутив также поддерживает Helm-charts из коробки. K3s не имеет внешних зависимостей.
- Другое ключевое отличие k3s от k8s заключается в способе управления состоянием кластера. Kubernetes полагается на распределенную базу данных «ключ-значение» etcd для хранения всего состояния кластера. Изначально разработчики k3s заменили etcd облегченной базой данных под названием SQLite. Однако позже команда создала (они называют его революционным) проект под названием Kine, что означает «Kine is not etcd». Kine предоставляет прокладку API, которая позволяет k3s поддерживать различные серверные части баз данных, включая MySQL, Postgres и SQLite. Он принимает запросы etcd v3 от Kubernetes, преобразовывает их в SQL-запросы и отправляет в серверную часть вашей базы данных.
- Также выделяется еще одна отличительная особенность от старшего брата — автоматическое развертывание. Оно позволяет развертывать манифесты Kubernetes и Helm-чарты, помещая их в определенный каталог. То есть k3s отслеживает изменения и заботится об их применении без дальнейшего взаимодействия. Просто создайте/обновите свою конфигурацию, и k3s обеспечит актуальность ваших deployments .


![](https://k3s.io/img/how-it-works-k3s-revised.svg)

### Варианты развёртывания

![](https://habrastorage.org/r/w1560/webt/u7/i8/7t/u7i87ter4pxy33dukkm4aua0bca.png)

![](https://habrastorage.org/r/w1560/getpro/habr/post_images/40b/cec/c4e/40bcecc4e8067697191666361c0a8b47.png)

![](https://habrastorage.org/r/w1560/getpro/habr/post_images/938/6f1/fec/9386f1fecb3ab3d8baf893203b9392cc.png)

### Install process
#### Install server node
````
curl -sfL https://get.k3s.io |  INSTALL_K3S_EXEC="server --disable=traefik --write-kubeconfig-mode=644" sh -
````

### Command examples

````
kubectl get pods --all-namespaces
..
NAMESPACE     NAME                                      READY   STATUS    RESTARTS        AGE
kube-system   local-path-provisioner-84db5d44d9-npr7c   1/1     Running   0               9m26s
kube-system   coredns-6799fbcd5-g4wch                   1/1     Running   0               9m26s
kube-system   metrics-server-67c658944b-9cc5h           1/1     Running   0               9m26s
kube-system   helm-install-traefik-hl7t6                0/1     Error     0               9m27s
kube-system   helm-install-traefik-crd-sd9vw            0/1     Error     1 (8m45s ago)   9m27s
kube-system   helm-install-traefik-s8nbg                0/1     Error     0               8m35s
kube-system   helm-install-traefik-crd-5rr4l            0/1     Error     1 (64s ago)     8m34s
kube-system   helm-install-traefik-xzqvv                0/1     Pending   0               44s
kube-system   helm-install-traefik-crd-h2n8c            0/1     Pending   0               43s


k3s kubectl get node
..
NAME                    STATUS   ROLES                  AGE     VERSION
debian12-k3s-server01   Ready    control-plane,master   9m45s   v1.28.5+k3s1

kubectl get no -o wide
..
NAME                    STATUS   ROLES                  AGE   VERSION        INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
debian12-k3s-server01   Ready    control-plane,master   18m   v1.28.5+k3s1   192.168.122.205   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-12-amd64   containerd://1.7.11-k3s2

kubectl get namespaces
..
NAME              STATUS   AGE
kube-system       Active   21m
kube-public       Active   21m
kube-node-lease   Active   21m
default           Active   21m

kubectl -n kube-system get svc
..
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns         ClusterIP   10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP   23m
metrics-server   ClusterIP   10.43.164.147   <none>        443/TCP                  23m

kubectl config view
..
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:6443
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

kubectl delete -n kube-system pod helm-install-traefik-s8nbg

````
## Uninstall k3s
Here is two ways, the first is reset the data, the second is full uninstall method
### Method 1. Reset the data
````
rm -f /etc/rancher/k3s/*
rm -f /var/lib/rancher/k3s/*
````
### Method 2. Full uninstall script
#### Uninstall from agent node (k3s-agent-uninstall.sh)
````
/usr/local/bin/k3s-agent-uninstall.sh
````
#### Uninstall from server node (k3s-uninstall.sh)
````
/usr/local/bin/k3s-uninstall.sh
..
+ id -u
+ [ 0 -eq 0 ]
+ /usr/local/bin/k3s-killall.sh
+ [ -s /etc/systemd/system/k3s.service ]
+ basename /etc/systemd/system/k3s.service
+ systemctl stop k3s.service
+ [ -x /etc/init.d/k3s* ]
+ killtree 1326 1469 1503
+ kill -9 1326 1424 1668 1469 1559 1675 1503 1566 1792
+ do_unmount_and_remove /run/k3s
+ set +x
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/d6d1b82c75f7f251b5a2b11a925a05a7645b2f1db97e019eb3fb2113a94954c9/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/cade7cff97e9ddd6f69ed4c73dd372eae795fa10cb51096099f522b0e6bab3d0/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/b6db1ca266c3277350a3e3d53aa04d2c705a164929e645e6759ab6645366afd9/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/abcb820934aa0e6538dabe37a8a704b1f0be82d15d2b61802829f6dae1698a85/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/80ae18134229b5edd6ee51640283b5157dffbe08e8cc8f0b9c3c60062366ecdf/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.runtime.v2.task/k8s.io/6d6a7e25fb61c8e60c1b16cbd38dceb2ea04dd53b2b2204d3901b7cb188ebf78/rootfs
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/b6db1ca266c3277350a3e3d53aa04d2c705a164929e645e6759ab6645366afd9/shm
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/80ae18134229b5edd6ee51640283b5157dffbe08e8cc8f0b9c3c60062366ecdf/shm
sh -c 'umount -f "$0" && rm -rf "$0"' /run/k3s/containerd/io.containerd.grpc.v1.cri/sandboxes/6d6a7e25fb61c8e60c1b16cbd38dceb2ea04dd53b2b2204d3901b7cb188ebf78/shm
+ do_unmount_and_remove /var/lib/rancher/k3s
+ set +x
+ do_unmount_and_remove /var/lib/kubelet/pods
+ set +x
sh -c 'umount -f "$0" && rm -rf "$0"' /var/lib/kubelet/pods/f2dcfe33-6c9a-4a54-9ef8-6f5e572d5fa1/volumes/kubernetes.io~projected/kube-api-access-n22f4
sh -c 'umount -f "$0" && rm -rf "$0"' /var/lib/kubelet/pods/f0f796e2-65f0-4642-a97f-8861d0a2113b/volumes/kubernetes.io~projected/kube-api-access-cb449
sh -c 'umount -f "$0" && rm -rf "$0"' /var/lib/kubelet/pods/5b788f62-7771-4d75-8bcb-ecd72a58ef6f/volumes/kubernetes.io~projected/kube-api-access-tmzks
+ do_unmount_and_remove /var/lib/kubelet/plugins
+ set +x
+ do_unmount_and_remove /run/netns/cni-
+ set +x
sh -c 'umount -f "$0" && rm -rf "$0"' /run/netns/cni-21eb0922-1b23-065e-394d-7819d106d3bc
sh -c 'umount -f "$0" && rm -rf "$0"' /run/netns/cni-0b4079b1-c038-2929-1395-8bdcdf23a1b1
sh -c 'umount -f "$0" && rm -rf "$0"' /run/netns/cni-07acb30f-0871-5465-901e-41d56e5e5664
+ xargs -r -t -n 1 ip netns delete
+ grep cni-
+ ip netns show
+ remove_interfaces
+ read ignore iface ignore
+ grep master cni0
+ ip link show
+ iface=veth0bf53f4c
+ [ -z veth0bf53f4c ]
+ ip link delete veth0bf53f4c
+ read ignore iface ignore
+ iface=veth87bc9b6c
+ [ -z veth87bc9b6c ]
+ ip link delete veth87bc9b6c
+ read ignore iface ignore
+ ip link delete cni0
+ ip link delete flannel.1
+ ip link delete flannel-v6.1
Cannot find device "flannel-v6.1"
+ ip link delete kube-ipvs0
Cannot find device "kube-ipvs0"
+ ip link delete flannel-wg
Cannot find device "flannel-wg"
+ ip link delete flannel-wg-v6
Cannot find device "flannel-wg-v6"
+ command -v tailscale
+ [ -n  ]
+ rm -rf /var/lib/cni/
+ iptables-restore
+ grep -iv flannel
+ grep -v CNI-
+ grep -v KUBE-
+ iptables-save
+ ip6tables-restore
+ grep -iv flannel
+ grep -v CNI-
+ grep -v KUBE-
+ ip6tables-save
+ command -v systemctl
/usr/bin/systemctl
+ systemctl disable k3s
Removed "/etc/systemd/system/multi-user.target.wants/k3s.service".
+ systemctl reset-failed k3s
+ systemctl daemon-reload
+ command -v rc-update
+ rm -f /etc/systemd/system/k3s.service
+ rm -f /etc/systemd/system/k3s.service.env
+ trap remove_uninstall EXIT
+ [ -L /usr/local/bin/kubectl ]
+ rm -f /usr/local/bin/kubectl
+ [ -L /usr/local/bin/crictl ]
+ rm -f /usr/local/bin/crictl
+ [ -L /usr/local/bin/ctr ]
+ rm -f /usr/local/bin/ctr
+ rm -rf /etc/rancher/k3s
+ rm -rf /run/k3s
+ rm -rf /run/flannel
+ rm -rf /var/lib/rancher/k3s
+ rm -rf /var/lib/kubelet
+ rm -f /usr/local/bin/k3s
+ rm -f /usr/local/bin/k3s-killall.sh
+ type yum
+ type rpm-ostree
+ type zypper
+ remove_uninstall
+ rm -f /usr/local/bin/k3s-uninstall.sh
````
## Доп инфа
https://habr.com/ru/articles/508636/ - устанавливается по умолчанию traefik (обратный прокси сервер для микросервисов и контейнеров)

![](https://raw.githubusercontent.com/Frodo-Web/frodo-tips/main/devops/images/traefik-architecture.webp)

Пример конфигурации:
docker-compose.yml:
````
version: '3'
services:
  traefik:
    image: traefik:v2.2
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - 80:80
      - 443:443 
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yml:/traefik.yml:ro
````
traefik.yml
````
entryPoints:
  http:
    address: ":80"
  https:
    address: ":443"

http:
  routers:
    http-catchall:
      rule: hostregexp(`{host:.+}`)
      entrypoints:
      - http
      middlewares:
      - redirect-to-https
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: false

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
````
