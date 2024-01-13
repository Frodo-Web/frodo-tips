# k3s
- Если сравнивать с традиционным кластером Kubernetes, то в k3s нет четкого различия между мастер и воркер нодами. Управление подами происходит на любой ноде, независимо от роли, которую они играют. Таким образом, понятия мастер и воркер нод не применимы к кластеру k3s.
- Rancher Labs избавились от множества необязательных компонентов Kubernetes, например, от плагинов объема хранилища и API-интерфейсов облачных провайдеров.
- Затем были добавлены некоторые важные элементы, включая containerd, Flannel, CoreDNS, CNI, ingress controller Traefik, локальное хранилище данных, встроенный сервис Load Balancer. Все эти элементы упакованы в один бинарный файл и выполняются в рамках одного процесса. Помимо этого, дистрибутив также поддерживает Helm-charts из коробки. K3s не имеет внешних зависимостей.
- Другое ключевое отличие k3s от k8s заключается в способе управления состоянием кластера. Kubernetes полагается на распределенную базу данных «ключ-значение» etcd для хранения всего состояния кластера. Изначально разработчики k3s заменили etcd облегченной базой данных под названием SQLite. Однако позже команда создала (они называют его революционным) проект под названием Kine, что означает «Kine is not etcd». Kine предоставляет прокладку API, которая позволяет k3s поддерживать различные серверные части баз данных, включая MySQL, Postgres и SQLite. Он принимает запросы etcd v3 от Kubernetes, преобразовывает их в SQL-запросы и отправляет в серверную часть вашей базы данных.
- Также выделяется еще одна отличительная особенность от старшего брата — автоматическое развертывание. Оно позволяет развертывать манифесты Kubernetes и Helm-чарты, помещая их в определенный каталог. То есть k3s отслеживает изменения и заботится об их применении без дальнейшего взаимодействия. Просто создайте/обновите свою конфигурацию, и k3s обеспечит актуальность ваших deployments .


![](https://k3s.io/img/how-it-works-k3s-revised.svg)

### Основные понятия (k8s)
````
Nodes (node.md) — машины в кластере K8s.
Pods (pods.md) — группа контейнеров с общими разделами, которые запускаются как одно приложение.
Replication Controllers (replication-controller.md) способ репликации, который гарантирует, что определенное количество «реплик» будут запущены в любой момент времени.
Services (services.md) — набор логически объединенных подов и политик доступа.
Volumes (volumes.md) — доступная в контейнере директория с данными или без них.
Labels (labels.md) — пары ключ/значение, которые прикрепляются к объектам. Например, к подам. Labels могут применяться для создания и выбора наборов объектов.
Операторы (operators.md) — программное обеспечения Kubernetes, необходимое для включения в кластер сервисов, сохраняющих состояние между выполнениями (stateful). Например, СУБД.
Kubectl Command Line Interface (kubectl.md) — интерфейс командной строки для управления Kubernetes.

Кластер. Развертывание Kubernetes — это, в первую очередь, поднятие кластера. В кластере обязательно есть хотя бы одна нода. Она состоит из набора машин, которые запускают контейнерные приложения. Обычно на одной ноде кластер не ограничивается, поскольку дополнительные ноды решают проблему отказустойчивости.
Nods (узлы). Это физические или виртуальные машины, на которых развертываются и запускаются контейнеры с приложениями. Каждый узел содержит сервисы, необходимые для запуска подов.

Master (мастер-нода) — узел, управляющий всем кластером. Он следит за остальными нодами и распределяет между ними нагрузку с помощью менеджера контроллеров (controller manager) и планировщика (scheduler). Как правило, мастер-нода занимается только управлением и не берет на себя рабочие нагрузки. Для повышения отказоустойчивости существует несколько мастер-нод.
Worker (рабочие ноды) — узлы, на которых работают контейнеры. В зависимости от параметров ноды (объема памяти и центрального процессора) на одном узле может работать множество контейнеров. Чем больше рабочих узлов, тем больше приложений можно запустить. Также количество влияет на отказоустойчивость кластера, потому что при выходе из строя одной ноды нагрузка распределяется по оставшимся.

Pods (отсеки). Pod определяется представлением запроса на запуск (execute) одного или более контейнеров на одном узле. Они разделяют доступ к таким ресурсам, как: тома хранилища и сетевой стек.

Kube-apiserver. С помощью сервера API обеспечивается работа API кластера, обрабатываются REST-операции и предоставляется интерфейс, через который остальные компоненты взаимодействуют друг с другом. Кроме этого, через него проходят запросы на изменение состояния или чтение кластера. Работает на master-нодах.
Kube-scheduler. Компонент-планировщик, который определяет на каких узлах разворачивать pods. Он учитывает такие факторы, как ограничения, требования к ресурсам, местонахождение данных и пр. Работает на master-нодах.
Etcd. Распределенное хранилище в формате «ключ-значение». В нем хранится состояние всего кластера. Главная задача etcd — обеспечить отказоустойчивость кластера и консистентность данных. Etcd — самостоятельный проект. Он развивается отдельно от Kubernetes и применяется в разных продуктах. Работает на master-нодах.
Kube-proxy. Служба, которая управляет правилами балансировки нагрузки. Она конфигурирует правила IPVS или iptables, через которые выполняются проксирование и роутинг. Работает на worker-нодах.
Kube-controller-manager. Компонент запускает работу контроллеров. Работает на master-нодах.
Kubelet. Cлужба управляет состоянием ноды: запуском, остановкой и поддержанием работы контейнеров и подов. Работает на worker-нодах.

Исполняемые среды контейнеров. Исполняемая среда контейнера — это программа, предназначенная для запуска контейнера в Kubernetes. Оркестратор поддерживает различные среды для запуска контейнеров: Docker, containerd, CRI-O, и любые реализации Kubernetes CRI (Container Runtime Interface).
Развертывание. При разворачивании приложения в Kubernetes, мы сообщаем мастеру, что нужно запустить контейнеры приложений. Мастер планирует запуск контейнеров на узлах кластера. Узлы связываются с мастером с помощью Kubernetes API, который предоставляет мастер.

Набор реплик. Набор реплик гарантирует, сколько реплик модуля должно быть запущено. Это можно рассматривать как замену контроллера репликации. Основное различие между набором реплик и контроллером репликации состоит в том, что контроллер репликации поддерживает только селектор на основе равенства, тогда как набор реплик поддерживает селектор на основе набора.
Сервис. Сервис в Kubernetes — абстракция, определяющая логический набор подов и политику доступа к ним (иногда такой набор подов еще называют микросервисом). Как правило, этот набор подов определяется на основе меток (присваиваются в момент создания подов) и селекторов.

Persistent Volumes (постоянные тома)
Подсистема позволяет создавать ресурс хранения, который не зависит от модульности. Использование PV гарантирует, что хранилище будет постоянным.

Namespaces (пространства имен)
Способ организации кластеров в виртуальные подкластеры, которые обычно применяют, когда к кластеру K8s организован доступ нескольких команд со своими задачами. В кластере поддерживается любое количество пространств имен, изолированных друг от друга, но с возможностью взаимодействия.

Deployment (развертывание)
Развертывание можно определить для создания новых наборов реплик или для удаления существующих развертываний и принятия всех их ресурсов новыми развертываниями. А также процесс реконфигурации кластера с его текущего состояния на желаемое состояние.

StatefulSet (набор состояния)
Это контроллер Kubernetes, который используют для эксплуатации сохраняющих состояние приложений в виде контейнеров (подов) в кластере Kubernetes. StatefulSet присваивают каждому поду идентификатор-липучку (sticky identity) — порядковый номер начиная с нуля — а не случайные ID каждой реплике пода.


kind: Service:

Service: This directs the traffic to a pod.
Port: The port of this service
TargetPort: This is the actual port on which your application is running inside the container. The target port on the pod(s) to forward traffic to.
nodePort: The port on the node where external traffic will come in on
ClusterIP: Request comes through ingress and points to service name and port.

So the traffic flows Ingress-->Service-->Endpoint(Basically has POD IP)-->POD

Service types:

ClusterIP (default): Internal clients send requests to a stable internal IP address.
NodePort: Clients send requests to the IP address of a node on one or more nodePort values that are specified by the Service.
LoadBalancer: Clients send requests to the IP address of a network load balancer.
ExternalName: Internal clients use the DNS name of a Service as an alias for an external DNS name.
Headless: You can use a headless service when you want a Pod grouping, but don't need a stable IP address.
````
### Service types
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/ClusterIP%20with%20proxy.png?raw=true)
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/NodePort%20service.png?raw=true)
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/LoadBalancer%20service.png?raw=true)
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/Ingress%20controller.png?raw=true)

### Варианты развёртывания

![](https://habrastorage.org/r/w1560/webt/u7/i8/7t/u7i87ter4pxy33dukkm4aua0bca.png)

![](https://habrastorage.org/r/w1560/getpro/habr/post_images/40b/cec/c4e/40bcecc4e8067697191666361c0a8b47.png)

![](https://habrastorage.org/r/w1560/getpro/habr/post_images/938/6f1/fec/9386f1fecb3ab3d8baf893203b9392cc.png)

### Install process
#### Install server node
````
curl -sfL https://get.k3s.io |  INSTALL_K3S_EXEC="server --disable=traefik --write-kubeconfig-mode=644" sh -
````
#### Get server token
````
cat /var/lib/rancher/k3s/server/node-token
..
K10bee7230b21ba8d0b83d0bf8175a699b7764051c9b60dc3804519aee4b8e91a64::server:82abb84fed35c7c37db4ad97caef5e51
````
#### Install agent node
````
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.122.205:6443 K3S_TOKEN=K10bee7230b21ba8d0b83d0bf8175a699b7764051c9b60dc3804519aee4b8e91a64::server:82abb84fed35c7c37db4ad97caef5e51 sh -
````
### Command examples

````
Delete everything from current (default) namespace:
kubectl delete all --all
Delete everything from specific namespace:
kubectl delete all --all -n {namespace}
Or you can re-create that namespace:
kubectl delete namespace {namespace}
kubectl create namespace {namespace}


Check health status:
{ echo -e "\n==== Kubernetes Status ====\n" && \
  kubectl get --raw '/healthz?verbose' && \
  kubectl version && \
  kubectl get nodes && \
  kubectl cluster-info;
} | grep -z 'Ready\| ok\|passed\|running'

Also usefull comand for cheking config:
k3s check-config


kubectl get events
..
LAST SEEN   TYPE      REASON                    OBJECT                       MESSAGE
39m         Normal    Starting                  node/debian12-k3s-agent01    Starting kubelet.
39m         Warning   InvalidDiskCapacity       node/debian12-k3s-agent01    invalid capacity 0 on image filesystem
39m         Normal    NodeAllocatableEnforced   node/debian12-k3s-agent01    Updated Node Allocatable limit across pods
39m         Normal    NodeHasSufficientMemory   node/debian12-k3s-agent01    Node debian12-k3s-agent01 status is now: NodeHasSufficientMemory
39m         Normal    Starting                  node/debian12-k3s-agent01    
39m         Normal    NodeHasNoDiskPressure     node/debian12-k3s-agent01    Node debian12-k3s-agent01 status is now: NodeHasNoDiskPressure
39m         Normal    NodeHasSufficientPID      node/debian12-k3s-agent01    Node debian12-k3s-agent01 status is now: NodeHasSufficientPID
39m         Normal    NodeReady                 node/debian12-k3s-agent01    Node debian12-k3s-agent01 status is now: NodeReady
39m         Normal    Synced                    node/debian12-k3s-agent01    Node synced successfully
39m         Normal    RegisteredNode            node/debian12-k3s-agent01    Node debian12-k3s-agent01 event: Registered Node debian12-k3s-agent01 in Controller
3m52s       Warning   FreeDiskSpaceFailed       node/debian12-k3s-server01   Failed to garbage collect required amount of images. Attempted to free 388231987 bytes, but only found 0 bytes eligible to free.

kubectl get pods --all-namespaces
OR
k3s kubectl get pods -n kube-system 
..
NAMESPACE     NAME                                      READY   STATUS    RESTARTS        AGE
kube-system   local-path-provisioner-84db5d44d9-npr7c   1/1     Running   0               9m26s
kube-system   coredns-6799fbcd5-g4wch                   1/1     Running   0               9m26s
kube-system   metrics-server-67c658944b-9cc5h           1/1     Running   0               9m26s

k3s kubectl get node
..
NAME                    STATUS   ROLES                  AGE     VERSION
debian12-k3s-server01   Ready    control-plane,master   9m45s   v1.28.5+k3s1

kubectl get no -o wide
..
NAME                    STATUS   ROLES                  AGE   VERSION        INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
debian12-k3s-server01   Ready    control-plane,master   50m   v1.28.5+k3s1   192.168.122.205   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-12-amd64   containerd://1.7.11-k3s2
debian12-k3s-agent01    Ready    <none>                 50s   v1.28.5+k3s1   192.168.122.204   <none>        Debian GNU/Linux 12 (bookworm)   6.1.0-12-amd64   containerd://1.7.11-k3s2

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

kubectl get pods,services --all-namespaces

k3s kubectl get endpoints -n kube-system
..
NAME             ENDPOINTS                                  AGE
kube-dns         10.42.0.4:53,10.42.0.4:53,10.42.0.4:9153   28m
metrics-server   10.42.0.2:10250                            28m

k3s kubectl cluster-info
..
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:https/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

kubectl top nodes
..
NAME                    CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
debian12-k3s-agent01    15m          1%     303Mi           31%       
debian12-k3s-server01   83m          8%     602Mi           62%

kubectl top pods -n kube-system
..
NAME                                      CPU(cores)   MEMORY(bytes)   
coredns-6799fbcd5-2xrlx                   4m           23Mi            
local-path-provisioner-84db5d44d9-47ms7   1m           15Mi            
metrics-server-67c658944b-5kqvt           7m           30Mi

kubectl api-resources
kubectl describe pods --all-namespaces
kubectl describe pod <pod-name>

You can list all Persistent Volumes sorted by capacity
kubectl get pv --sort-by=.spec.capacity.storage --all-namespaces

top -o %MEM -b -n1 | head -n 24
..
top - 16:23:54 up  4:25,  2 users,  load average: 0.17, 0.11, 0.09
Tasks: 111 total,   1 running, 110 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st 
MiB Mem :    960.6 total,     90.4 free,    766.1 used,    244.2 buff/cache     
MiB Swap:      0.0 total,      0.0 free,      0.0 used.    194.5 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  16754 root      20   0 5535804 371128  64420 S   0.0  37.7   8:50.79 k3s-ser+
  17214 root      20   0 5060100  62276  18952 S   0.0   6.3   0:28.64 contain+
  17879 frodo     20   0  758772  36176  14920 S   0.0   3.7   0:38.99 metrics+
  17806 root      20   0  764400  30292  10100 S   0.0   3.1   0:19.22 coredns
  17944 root      20   0  733140  20480   7304 S   0.0   2.1   0:02.82 local-p+

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

kubectl create deployment my-nginx --image=nginx
..
deployment.apps/my-nginx created

kubectl expose deployment my-nginx --type=NodePort --port=80
..
service/my-nginx exposed

kubectl get pods,services
..
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-7fbf685c4d-mr7gb   1/1     Running   0          36s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        43h
service/my-nginx     NodePort    10.43.72.142   <none>        80:31839/TCP   13s

kubectl get service my-nginx  -o jsonpath="{.spec}"
..
{"clusterIP":"10.43.72.142","clusterIPs":["10.43.72.142"],"externalTrafficPolicy":"Cluster","internalTrafficPolicy":"Cluster","ipFamilies":["IPv4"],"ipFamilyPolicy":"SingleStack","ports":[{"nodePort":31839,"port":80,"protocol":"TCP","targetPort":80}],"selector":{"app":"my-nginx"},"sessionAffinity":"None","type":"NodePort"}

kubectl get service my-nginx  -o jsonpath="{.spec.ports[0]}"
..
{"nodePort":31839,"port":80,"protocol":"TCP","targetPort":80}

PORT=$(kubectl get service my-nginx  -o jsonpath="{.spec.ports[0].nodePort}") && echo $PORT
..
31839

kubecl describe deployments
..
Name:                   my-nginx
Namespace:              default
CreationTimestamp:      Sun, 07 Jan 2024 10:38:33 +0000
Labels:                 app=my-nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=my-nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=my-nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   my-nginx-7fbf685c4d (1/1 replicas created)
Events:          <none>

kubectl get deployments
kubectl delete deployment my-nginx
..
deployment.apps "my-nginx" deleted

k3s kubectl get ingress,svc,pods -n retail-project-dev
k3s kubectl get svc
..
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP        46h
my-nginx     NodePort    10.43.72.142   <none>        80:31839/TCP   136m

k3s kubectl describe svc my-nginx
..
Name:                     my-nginx
Namespace:                default
Labels:                   app=my-nginx
Annotations:              <none>
Selector:                 app=my-nginx
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.72.142
IPs:                      10.43.72.142
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31839/TCP
Endpoints:                <none>
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

kubectl delete services my-nginx
..
service "my-nginx" deleted

kubectl describe pvc --all-namespaces

kubectl create namespace testing
kubectl config set-context --current --namespace=testing

kubectl config view
kubectl config view --minify -o jsonpath='{..namespace}'
..
testing

k3s crictl images
..
IMAGE                                        TAG                 IMAGE ID            SIZE
docker.io/rancher/local-path-provisioner     v0.0.24             b29384aeb4b13       14.9MB
docker.io/rancher/mirrored-coredns-coredns   1.10.1              ead0a4a53df89       16.2MB
docker.io/rancher/mirrored-metrics-server    v0.6.3              817bbe3f2e517       29.9MB
docker.io/rancher/mirrored-pause             3.6                 6270bb605e12e       301kB

k3s crictl rmi // prune to delete any images no currently used by a running container
crictl rmi IMAGE-ID [IMAGE-ID...]

kubectl scale --help
kubectl scale --replicas=0 deployment/<your-deployment>
# Scale a resource identified by type and name specified in "foo.yaml" to 3.
kubectl scale --replicas=3 -f foo.yaml
# If the deployment named mysql's current size is 2, scale mysql to 3.
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql
# Scale multiple replication controllers.
kubectl scale --replicas=5 rc/foo rc/bar rc/baz
Scale statefulset named 'web' to 3.
kubectl scale --replicas=3 statefulset/web

We can even execute commands inside containers
kubectl exec $POD_NAME -c $CONTAINER_NAME -- /bin/cat /usr/share/nginx/html/index.html
kubectl exec nginx-deployment-86c794df95-lj7c6 -c nginx -- /bin/cat /usr/share/nginx/html/index.html
..
Mon Jan  8 09:42:38 UTC 2024

kubectl exec nginx-deployment-86c794df95-lj7c6 -c debian -- apt install curl -y
// Meaning containers inside pod are using the same net stack
kubectl exec nginx-deployment-86c794df95-lj7c6 -c debian -- curl -vvv -s -o - telnet://localhost:80

Get inside containers of a pod

kubectl exec --stdin --tty -c debian nginx-deployment-86c794df95-lj7c6 -- /bin/sh
# cat /etc/*-release
..
PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
NAME="Debian GNU/Linux"
VERSION_ID="12"
...
We can see our neighboor container with nginx running
# ss -ntulp
Netid          State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port          Process          
tcp            LISTEN          0               511                            0.0.0.0:80                          0.0.0.0:*                              
tcp            LISTEN          0               511                               [::]:80                             [::]:*

kubectl get pods --selector=app=nginx -o jsonpath='{.items[*].status.podIP}'
..
10.42.1.6 10.42.0.20
````
## Kubernetes Deployment YAML File with Examples

- apiVersion: Specifies the Kubernetes API version, such as “apps/v1” for Deployments.
- kind: Specifies the type of Kubernetes resource, in this case, “Deployment.”
- metadata: Provides metadata for the Deployment, including the name, labels, and annotations.
- spec: Defines the desired state of the Deployment, including the number of replicas, the pod template, and any other related specifications. It includes:
- replicas: Specifies the desired number of identical pod replicas to run.
- selector: Specifies the labels that the Replica Set uses to select the pods it should manage.
- template: Contains the pod template used for creating new pods, including container specifications, image names, and container ports.

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: example-image
        ports:
        - containerPort: 8080
````
- apiVersion: Specifies the Kubernetes API version. In this case, it’s using the “apps/v1” API version, which is appropriate for Deployments.
- kind: Specifies the type of Kubernetes resource. Here, it’s “Deployment,” indicating that this configuration file is defining a Deployment.
- spec: This section defines the desired state of the Deployment.
- replicas: 3: Specifies that you want to run three replicas of your application.
- selector: Describes the selector to match pods managed by this Deployment.
- matchLabels: Specifies the labels that the Replica Set created by the Deployment should use to select the pods it manages. In this case, pods with the label app: example are selected.
- template: Defines the pod template used for creating new pods.
- metadata: Contains the labels to apply to the pods created from this template. In this case, the pods will have the label app: example.
- spec: Describes the specification of the pods.
- containers: This section specifies the containers to run in the pod.
- name: example-container: Assigns a name to the container.
- image: example-image: Specifies the Docker image to use for this container.
- ports: Defines the ports to open in the container.
- containerPort: 8080: Indicates that the container will listen on port 80.

### Some examples
nginx-deployment.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
````
Pass environment variables in Kubernetes deployment YAML (and why not to do that!)
env-deployment.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: my-app-container
        image: my-app-image
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: db.example.com
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: api-key
````
Environment variables can be viewed by anyone with access to the pod, which poses a security risk. Instead, use Kubernetes Secrets to store and securely mount sensitive data into containers.

Kubernetes deployment YAML with resource limits
nginx-deployment-with-resource-limits.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "256Mi"  # Maximum memory allowed
            cpu: "200m"       # Maximum CPU allowed (200 milliCPU)
          requests:
            memory: "128Mi"  # Initial memory request
            cpu: "100m"       # Initial CPU request
````

Kubernetes deployment YAML with health checks
nginx-deployment-with-health-checks.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /                # The path to check for the liveness probe
            port: 80               # The port to check on
          initialDelaySeconds: 15  # Wait this many seconds before starting the probe
          periodSeconds: 10        # Check the probe every 10 seconds
        readinessProbe:
          httpGet:
            path: /                # The path to check for the readiness probe
            port: 80               # The port to check on
          initialDelaySeconds: 5   # Wait this many seconds before starting the probe
          periodSeconds: 5         # Check the probe every 5 seconds
````
- livenessProbe: The liveness probe checks whether the container is still alive. It uses an HTTP GET request to the / path on port 80 of the container. If the probe fails, K8s will restart the container.
- readinessProbe: The readiness probe checks whether the container is ready to serve traffic. It also uses an HTTP GET request to the / path on port 80 of the container. If the probe fails, the container is marked as not ready, and K8s won’t send traffic to it.

Kubernetes deployment YAML with persistent volumes
The PVC defines the storage requirements, and a Persistent Volume (PV) is dynamically provisioned or statically assigned to meet those requirements.
nginx-deployment-with-pvc.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
           —mountPath: "/usr/share/nginx/html"
           name: my-pvc
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: my-pvc  # Name of the Persistent Volume Claim
````
A PVC is defined using persistentVolumeClaim. In this case, it’s referenced by the name my-pvc.

The my-pvc PVC must be defined separately in another YAML file. The Deployment’s container can then mount the volume specified by the name: data in the volumes section.

mypvc.yaml
````
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi  # Request 1 Gigabyte of storage
````

- accessModes specifies the access mode for the volume. It’s set to ReadWriteOnce, indicating that it can be mounted in read-write mode by a single node at a time.
- resources.requests.storage specifies the amount of storage requested. In this example, it requests 1 gigabyte of storage.

Don’t forget to apply both files in this case using the kubctl -f apply commands.

````
kubectl apply -f mypvc.yaml
kubectl apply -f nginx-deployment-with-pvc.yaml
````

Kubernetes deployment YAML with affinity settings
nginx-deployment-with-affinity.yaml
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-type
                operator: In
                values:
                - nginx-node
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
````
With these affinity settings, Kubernetes will attempt to schedule the NGINX Pods on nodes labeled as nginx-node, and it will make sure that no two NGINX Pods with the label app: nginx run on the same node, promoting fault tolerance and reliability in your NGINX deployment.

- nodeAffinity is used to ensure that the Pods are scheduled only on nodes with a specific label. Pods will be required to be scheduled on nodes with the label node-type: nginx-node.
- podAntiAffinity is used to ensure that no two NGINX Pods with the label app: nginx are scheduled on the same node. The topologyKey specifies that the scheduling is based on the hostname of the nodes.

simple-rest-golang.yaml
````
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: simple-rest-golang-deployment
  namespace: retail-project-dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: simple-rest-golang
  template:
    metadata:
      labels:
        app: simple-rest-golang
    spec:
      containers:
        - name: simple-rest-golang
          image: fransafu/simple-rest-golang:1.0.0
          resources:
            requests:
              memory: "64Mi"
              cpu: "100m"
            limits:
              memory: "128Mi"
              cpu: "500m"
          ports:
          - containerPort: 8080
          imagePullPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  name: simple-rest-golang-service
  namespace: retail-project-dev
spec:
  ports:
  - port: 80
    targetPort: 8080
    name: tcp
  selector:
    app: simple-rest-golang
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: simple-rest-golang-ingress
  namespace: retail-project-dev
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: simple-rest-golang-service
          servicePort: 80
````
Deploying 2 pods with 2 containers in them
````
apiVersion: apps/v1   
kind: Deployment    
metadata:            
  name: nginx-deployment    
  labels:        
    app: nginx    
spec:            
  replicas: 2    
  selector:       
    matchLabels: 
      app: nginx
  template:        
    metadata:    
      labels:    
        app: nginx
    spec: 
      volumes:
      - name: html
        emptyDir: {}
      containers:    
      - name: nginx
        image: nginx:alpine    
        ports:
          - containerPort: 80 
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      - name: debian
        image: debian
        volumeMounts:
        - name: html
          mountPath: /html
        command: ["/bin/sh", "-c"]
        args:
          - while true; do
              date > /html/index.html;
              sleep 120;
            done
````
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/k8s_multicloud_diagram-01.png)
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/k8s_multicloud_diagram-03.png)
Now we can see 2 different pods with 2 containers in them
````
kubectl get pods
..
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-86b7598c75-g5b8l   2/2     Running   0          50m
nginx-deployment-86b7598c75-6jr4g   2/2     Running   0          50m
````

In this case, Kubernetes runs init-myservice until it is successful, then init-mydb, and then the myapp-container application.
````
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
  initContainers:
  - name: init-myservice
    image: busybox:1.28
  - name: init-mydb
    image: busybox:1.28
````

Inter-process communications (IPC)
Containers in a Pod share the same IPC namespace, which means they can also communicate with each other using standard inter-process communications such as SystemV semaphores or POSIX shared memory.

The first container, producer, creates a standard Linux message queue, writes a number of random messages, and then writes a special exit message. The second container,  consumer, opens that same message queue for reading and reads messages until it receives the exit message. We also set the restart policy to 'Never', so the Pod stops after termination of both containers.
````
apiVersion: v1
kind: Pod
metadata:
  name: mc2
spec:
  containers:
  - name: producer
    image: allingeek/ch6_ipc
    command: ["./ipc", "-producer"]
  - name: consumer
    image: allingeek/ch6_ipc
    command: ["./ipc", "-consumer"]
  restartPolicy: Never
````
![](https://github.com/Frodo-Web/frodo-tips/blob/main/devops/images/k8s_multicloud_diagram-02.png)
 Now you can check logs for each container and verify that the 2nd container received all messages from the 1st container, including the exit message: 
````
$ kubectl logs mc2 -c producer
...
Produced: f4
Produced: 1d
Produced: 9e
Produced: 27
$ kubectl logs mc2 -c consumer
...
Consumed: f4
Consumed: 1d
Consumed: 9e
Consumed: 27
Consumed: done
````

#### Working with service
````
nginx_test_deployment.yml
..
apiVersion: apps/v1   
kind: Deployment    
metadata:            
  name: nginx-test-deployment
  labels:        
    app: nginx-test
spec:            
  replicas: 2    
  selector:       
    matchLabels: 
      app: nginx-test-pod
  template:        
    metadata:    
      labels:    
        app: nginx-test-pod
    spec: 
      containers:    
      - name: nginx
        image: nginx:alpine    
        ports:
          - containerPort: 80


nginx_test_service.yml
..
apiVersion: v1
kind: Service
metadata:
  name: nginx-test-service
spec:
  selector:
    app: nginx-test-pod
  ports:
  - name: http
    port: 8080
    targetPort: 80
#   nodePort: 3333 // nodePort: Forbidden: may not be used when `type` is 'ClusterIP'
    protocol: TCP


cat nginx_test_pod.yml
..
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test-client
spec:
  restartPolicy: Never
  containers:
  - name: nginx-client
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "echo -en 'GET / HTTP/1.0\r\n\r\n' | nc nginx-test-service 8080"]

kubectl logs nginx-test-client
..
HTTP/1.1 200 OK
...

````
#### Alternatives to the Deployment Object

- DaemonSet—deploys a pod on all cluster nodes or a certain subset of nodes
- StatefulSet—used for stateful applications. Similar to a Deployment, but each pod is unique and has a persistent identifier.

A DaemonSet runs copies of a pod on all cluster nodes, or a selection of nodes within a cluster. Whenever a node is added to the cluster, the DaemonSet controller checks if it is eligible, and if so, runs the pod on it. When a node is removed from the cluster, the pods are moved to garbage collection. Deleting a DaemonSet also results in removal of the pods it created.

````
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on 
      # master nodes
     —key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
     —name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
       —name: varlog
          mountPath: /var/log
       —name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
     —name: varlog
        hostPath:
          path: /var/log
     —name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
````

Kubernetes Statefulset Example YAML

A StatefulSet manages a group of pods while maintaining a sticky identity for each pod, with a persistent identifier that remains even if the pod is shut down and restarted. Pods also have PersistentVolumes that can store data that outlines the lifecycle of each individual pod. 

The following example shows a YAML configuration for a headless Service that controls the network domain, and a StatefulSet that runs 3 instances of an NGINX web server.
````
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
 —port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
     —name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
       —containerPort: 80
          name: web
        volumeMounts:
       —name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
 —metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
````
- metadata.name — must be a valid DNS subdomain name.
- .spec.selector.matchLabels and .spec.template.metadata.labels —both of these must match and are referenced by the headless Service to route requests to the application.
- spec.selector.replicas —specifies that the StatefulSet should run three replicas of the container, each with a unique persistent identifier.
- spec.template.spec.containers —specifies what NGINX image to run and how it should mount the PersistentVolumes.
- volumeClaimTemplates —provides persistent storage using the my-storage-class storage class. In a real environment, your cluster will have one or more storage classes defined by the cluster administrator, which provide different types of persistent storage.

## How to move k3s data to another location
The standard data location used for k3s is /run/k3s, /var/lib/kubelet/pods, /var/lib/rancher. Because this directory contains all containers/images/volumes, it can be large. So you no need to store this in OS Volume when you can use separate data volume.
````
du -d 1 -h /run/k3s/
..
162M	/run/k3s/containerd
162M	/run/k3s/

du -d 1 -h /var/lib/kubelet/pods
..
88K	/var/lib/kubelet/pods/9ee1bfb5-5e82-4ab8-9b2a-14c72abd171a
68K	/var/lib/kubelet/pods/990d1b97-53fa-4ff4-a1d2-665c9d25342c
80K	/var/lib/kubelet/pods/23f39cee-7287-4b79-acf5-6109785c1036
240K	/var/lib/kubelet/pods

du -d 1 -h /var/lib/rancher/
..
407M	/var/lib/rancher/k3s
407M	/var/lib/rancher/
````
### 1. Stop daemon
````
# systemctl stop k3s
# systemctl stop k3s-agent
# /usr/local/bin/k3s-killall.sh
````
### 2. Copy files to new location
````
# mv /run/k3s/ /Toshiba/k3s/
# mv /var/lib/kubelet/pods/ /Toshiba/k3s-pods/
# mv /var/lib/rancher/ /Toshiba/k3s-rancher/
````
### 3. Create symbolic link
````
# ln -s /Toshiba/k3s/ /run/k3s
# ln -s /Toshiba/k3s-pods/ /var/lib/kubelet/pods
# ln -s /Toshiba/k3s-rancher/ /var/lib/rancher
````
### 4. Start daemon
````
# systemctl start k3s
# systemctl start k3s-agent
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
