# Kubernetes & Helm
Here are some usefull commands
## Выгрузка статы по неймспейсам, подам и томам в кластере
```
kubectl get pods --all-namespaces -o json > pods.json

cat pods.json | jq -r '
.items[] |
"Pod Name: \(.metadata.name)",
"Namespace: \(.metadata.namespace)",
"Volume Mounts:",
( .spec.containers[] | .volumeMounts[] | "  Mount Path: \(.mountPath) -> Volume Name: \(.name)" ),
"Host Paths (for volumes):",
( .spec.volumes[] | select(.hostPath) | "  Volume Name: \(.name) -> Host Path: \(.hostPath.path) (Type: " + (.hostPath.type // "<unset>") + ")" ),
""
' 
```
Получаем такой выхлоп примерно по каждой поде
```
Pod Name: my-pod-name
Namespace: my-namespace-name
Volume Mounts:
  Mount Path: /var/logs -> Volume Name: logs
  Mount Path: /dev/log -> Volume Name: my-logs
  Mount Path: /var/my -> Volume Name: ceph-my-domain
  Mount Path: /var/run/secrets/kubernetes.io/serviceaccount -> Volume Name: kube-api-access-cdpqz
  Mount Path: /var/run/secrets/kubernetes.io/serviceaccount -> Volume Name: kube-api-access-cdpqz
Host Paths (for volumes):
  Volume Name: logs -> Host Path: /var/logs/my-domain (Type: DirectoryOrCreate)
  Volume Name: mylog -> Host Path: /tmp/mylog/log (Type: )
  Volume Name: ceph-my-domain -> Host Path: /mnt/ceph (Type: DirectoryOrCreate)
```
Мы можем дальше собрать статистику в таком виде
```
awk '/Host Paths \(for volumes\):/{inHP=1;next}/^$/{inHP=0}/^Namespace:/{ns=$2}inHP&&/Volume Name:.*-> Host Path: \/mnt\/ceph/{match($0,/Volume Name: ([^ ]+) -> Host Path: ([^ ]+)/,a);k=ns" - "a[1]" - "a[2];c[k]++;tc++}END{for(i in c)print i" - "c[i];print"=== Total: "tc}' pod-volumes.txt

my-fo - ceph-my-domain - /mnt/ceph - 1
my-ru - ceph-my-team - /mnt/ceph - 5
=== Total: 6

```
Или с точкой монтирования внутрь контейнера
```
awk '
/^Pod Name:/ { pod = $3 }
/^Namespace:/ { ns = $2 }
/^Volume Mounts:$/ { inVM = 1; next }
/^Host Paths \(for volumes\):$/ { inHP = 1; inVM = 0; next }
/^$/ { inVM = 0; inHP = 0 }

# Capture Volume Name -> Mount Path mapping
inVM && /Mount Path:.*-> Volume Name:/ {
    match($0, /Mount Path: ([^ ]+) -> Volume Name: ([^ ]+)/, a)
    vol_to_mount[a[2]] = a[1]   # vol_to_mount["ceph-my-domain"] = "/var/my"
}

# Process Host Paths and use the mapping
inHP && /Volume Name:.*-> Host Path: \/mnt\/ceph/ {
    match($0, /Volume Name: ([^ ]+) -> Host Path: ([^ ]+)/, a)
    vol_name = a[1]
    host_path = a[2]
    mount_path = vol_to_mount[vol_name]  # Look up the mount path for this volume
    if (mount_path == "") mount_path = "N/A"  # Fallback if not found
    key = ns " - " vol_name " - " host_path " - " mount_path
    c[key]++
    tc++
}

END {
    for (i in c) print i " - " c[i]
    print "=== Total: " tc
}' pod-volumes.txt

myru - ceph-my-transfer - /mnt/ceph - /var/my - 3
mywiki - cache-config - /mnt/ceph/wiki/cache - /var/www/html/cache - 1
myru - my-bufer- /mnt/ceph/my_bufer - /var/spool/my_bufer - 2
=== Total: 6
```
## Выгрузка открытых сокетов, файлов процессов внутри контейнера
PID текущих работающих контейнеров
```
crictl ps -q | xargs -r crictl inspect | jq -r '.info.pid'
```
Открытые сокеты, файлы процессов внутри контейнера. Ошибки при этом в stderr, можно перенаправить
```
crictl ps -q | xargs -r -I {} sh -c 'pid=$(sudo crictl inspect {} | jq -r ".info.pid"); ls -l /proc/$pid/root/proc/[0-9]*/fd' > opened_sockets.txt
```
А так мы можем средствами ansible раскатить команду по всем нодам кубера
```
#!/bin/bash
crictl ps -q | xargs -r -I {} sh -c 'pid=$(sudo crictl inspect {} | jq -r ".info.pid"); ls -l /proc/$pid/root/proc/[0-9]*/fd' > /tmp/nigga_opened_sockets.txt

ANSIBLE_HOST_KEY_CHECKING=False ansible all_k8s -l список_узлов_из_группы_если_нужно -i production.ini -u username --become -m script -a "./bash_ebash.sh"
```
А потом так выкачиваем и склеиваем файлы. Остатки будут удалены с узлов
```
#!/bin/bash

# Configuration
REMOTE_FILE_PATH="/tmp/rburdin_opened_sockets.txt"
LOCAL_DOWNLOAD_DIR="./downloaded_sockets"
FINAL_CONCATENATED_FILE="all_sockets_combined.txt"
HOSTS_FILE="hosts.txt"

# Create local directory for downloads
mkdir -p "$LOCAL_DOWNLOAD_DIR"

# Check if hosts file exists
if [ ! -f "$HOSTS_FILE" ]; then
    echo "Error: $HOSTS_FILE not found!"
    echo "Please create a $HOSTS_FILE with one hostname/IP per line"
    exit 1
fi

# Counters
downloaded_count=0
failed_count=0

echo "Starting download from remote hosts..."

# Use file descriptor 3 to avoid stdin being consumed by ssh/scp
while IFS= read -r host <&3; do
    # Skip empty lines and comments
    [[ -z "$host" || "$host" =~ ^[[:space:]]*# ]] && continue
    
    echo "Processing host: $host"
    
    # Download the file
    if scp "$host:$REMOTE_FILE_PATH" "$LOCAL_DOWNLOAD_DIR/${host//[^a-zA-Z0-9._-]/_}.txt" 2>/dev/null; then
        echo "✓ Successfully downloaded from $host"
        
        # Delete the remote file after successful download
        if ssh "$host" "rm -f $REMOTE_FILE_PATH" 2>/dev/null; then
            echo "✓ Successfully deleted remote file from $host"
        else
            echo "⚠️ Warning: Could not delete remote file from $host"
        fi
        
        ((downloaded_count++))
    else
        echo "✗ File not found or failed to download from $host"
        ((failed_count++))
    fi
    echo "---"
    
done 3< "$HOSTS_FILE"  # Read from file descriptor 3

echo "Download complete!"
echo "Successfully downloaded: $downloaded_count files"
echo "Failed downloads: $failed_count files"

# Check if we have any files to concatenate
if [ $downloaded_count -eq 0 ]; then
    echo "No files were downloaded. Nothing to concatenate."
    exit 0
fi

echo "Concatenating all downloaded files..."

# Concatenate all downloaded files into a single file
if cat "$LOCAL_DOWNLOAD_DIR"/*.txt > "$FINAL_CONCATENATED_FILE" 2>/dev/null; then
    echo "✓ Successfully created $FINAL_CONCATENATED_FILE"
    
    # Delete the individual downloaded files
    rm -f "$LOCAL_DOWNLOAD_DIR"/*.txt
    echo "✓ Deleted individual downloaded files"
    
    echo "Process completed successfully!"
    echo "Final concatenated file: $FINAL_CONCATENATED_FILE"
    if [ -f "$FINAL_CONCATENATED_FILE" ]; then
        echo "Total lines in final file: $(wc -l < "$FINAL_CONCATENATED_FILE")"
    fi
else
    echo "✗ Failed to concatenate files"
    exit 1
fi

```
## Выгрузка соединений внутри контейнеров по всему кластеру
Вот такой баш скрипт
```
crictl ps -q | xargs -r -I {} sh -c 'pid=$(sudo crictl inspect {} | jq -r ".info.pid"); hostname=$(hostname); echo "host: $hostname pid: $pid"; sudo nsenter -t $pid -n netstat -an' > /tmp/rburdin_opened_connections.txt
```
Запускаем ансиблом на всём кластере
```
ANSIBLE_HOST_KEY_CHECKING=False ansible all_k8s -l all_k8s -i production.ini -u rburdin --become -m script -a "./collect_connections_opened.sh"
```
И можем выкачать с узлом и склеить этим скриптом
```
#!/bin/bash

# Configuration
REMOTE_FILE_PATH="/tmp/rburdin_opened_connections.txt"
LOCAL_DOWNLOAD_DIR="./downloaded_connections"
FINAL_CONCATENATED_FILE="all_connections_combined.txt"
HOSTS_FILE="production_hosts_connections.txt"

# Create local directory for downloads
mkdir -p "$LOCAL_DOWNLOAD_DIR"

# Check if hosts file exists
if [ ! -f "$HOSTS_FILE" ]; then
    echo "Error: $HOSTS_FILE not found!"
    echo "Please create a $HOSTS_FILE with one hostname/IP per line"
    exit 1
fi

# Counters
downloaded_count=0
failed_count=0

echo "Starting download from remote hosts..."

# Use file descriptor 3 to avoid stdin being consumed by ssh/scp
while IFS= read -r host <&3; do
    # Skip empty lines and comments
    [[ -z "$host" || "$host" =~ ^[[:space:]]*# ]] && continue

    echo "Processing host: $host"

    # Download the file
    if scp "$host:$REMOTE_FILE_PATH" "$LOCAL_DOWNLOAD_DIR/${host//[^a-zA-Z0-9._-]/_}.txt" 2>/dev/null; then
        echo "✓ Successfully downloaded from $host"

        # Delete the remote file after successful download
        if ssh "$host" "sudo rm -f $REMOTE_FILE_PATH" 2>/dev/null; then
            echo "✓ Successfully deleted remote file from $host"
        else
            echo "⚠️ Warning: Could not delete remote file from $host"
        fi

        ((downloaded_count++))
    else
        echo "✗ File not found or failed to download from $host"
        ((failed_count++))
    fi
    echo "---"

done 3< "$HOSTS_FILE"  # Read from file descriptor 3

echo "Download complete!"
echo "Successfully downloaded: $downloaded_count files"
echo "Failed downloads: $failed_count files"

# Check if we have any files to concatenate
if [ $downloaded_count -eq 0 ]; then
    echo "No files were downloaded. Nothing to concatenate."
    exit 0
fi

echo "Concatenating all downloaded files..."

# Concatenate all downloaded files into a single file
if cat "$LOCAL_DOWNLOAD_DIR"/*.txt > "$FINAL_CONCATENATED_FILE" 2>/dev/null; then
    echo "✓ Successfully created $FINAL_CONCATENATED_FILE"

    # Delete the individual downloaded files
    rm -f "$LOCAL_DOWNLOAD_DIR"/*.txt
    echo "✓ Deleted individual downloaded files"

    echo "Process completed successfully!"
    echo "Final concatenated file: $FINAL_CONCATENATED_FILE"
    if [ -f "$FINAL_CONCATENATED_FILE" ]; then
        echo "Total lines in final file: $(wc -l < "$FINAL_CONCATENATED_FILE")"
    fi
else
    echo "✗ Failed to concatenate files"
    exit 1
fi
```
Например, видим такую картину. Нужно пробить что за контейнер по pid
```
host: worker1.example.com pid: 1843899
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 10.233.93.43:58848      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:51544      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:45870      10.233.15.216:5672      ESTABLISHED
tcp        0      0 10.233.93.43:49242      10.233.15.216:5672      ESTABLISHED
tcp        0      0 10.233.93.43:48048      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:58870      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:55360      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:51506      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:49946      10.233.4.98:3306        ESTABLISHED
tcp        0      0 10.233.93.43:34672      10.233.15.216:5672      ESTABLISHED
tcp        0      0 10.233.93.43:55340      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:48114      10.233.15.216:5672      TIME_WAIT  
tcp        0      0 10.233.93.43:34628      10.233.15.216:5672      ESTABLISHED
tcp        0      0 10.233.93.43:47494      10.233.15.216:5672      ESTABLISHED
tcp        0      0 10.233.93.43:45708      10.233.15.216:5672      ESTABLISHED
```
Заходим на этот узел делаем так и вот в конце после containerd идёт id контейнера
```
cat /proc/1843899/cgroup 
12:freezer:/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pode66c6adc_a301_4c21_8828_a3b9ba47e9c9.slice/cri-containerd-d78cc36ef9160f2299c0b882a377ebfa9bf2f4fab93afa5c28bca8f45d5f812f.scope
11:memory:/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pode66c6adc_a301_4c21_8828_a3b9ba47e9c9.slice/cri-containerd-d78cc36ef9160f2299c0b882a377ebfa9bf2f4fab93afa5c28bca8f45d5f812f.scope
```
Делаем так и всё узнаём по нему
```
crictl inspect d78cc36ef9160f2299c0b882a377ebfa9bf2f4fab93afa5c28bca8f45d5f812f | less
```
## Однострочник, запуск скрипта с перехватом stdout с полным игнорированием ошибок
Любой скрипт
```
cat check_directories.sh
..
hostname 
ls -alth /var/logs/daemond /data/pprof /data/defines /tmp/superlog/ /tmp/superlog/log 2>/dev/null
```
Однострочная команда
```
ANSIBLE_LOAD_CALLBACK_PLUGINS=1 ANSIBLE_STDOUT_CALLBACK=json ANSIBLE_HOST_KEY_CHECKING=False ansible all_k8s -l group_one -i production.ini -u rburdin --become -m script -a "./check_directories.sh || true" | jq -r ".plays[].tasks[].hosts[].stdout" | less
```
## K8S
````
kubectl get secret sentry-creds -n sentry -o json | jq '.data'
echo 'dGAWGFgadafaf3Jk' | base64 --decode
kubectl get secret --namespace sentry sentry-redis -o jsonpath="{.data.redis-password}" | base64 -d
kubectl get all -n sentry
kubectl top pod --namespace=sentry
kubectl exec -it ubuntu-pod -n sentry -- /bin/bash
// Edit deployment, like exposed ports on pods etc.. There will be redeploy after that
kubectl edit deployment distribution -n development

// Ingress logs
kubectl -n kube-system logs daemonsets/rke2-ingress-nginx-controller

// Watch all logs on the deployment
kubectl logs -n development --follow deployments/application-admin

// Copy file to a pod
kubectl cp myfile.txt my-namespace/my-pod:/tmp/myfile.txt

// Forcefully delete stucked in terminating state pods
kubectl delete pod milvus-testing-kafka-0 --grace-period=0 --force -n milvus

But pay attention to this warning:
Warning: Immediate deletion does not wait for confirmation that the running resource has been terminated. The resource may continue to run on the cluster indefinitely.

// Rollback to the same revision to restore resources which were deleted after release
helm rollback <name> <revision_number> -n namespace

// Redeploy statefull set with new metadata, without deleteing anything else (For example, in case of PV expansion)
kubectl delete statefulset victoria-metrics-storage -n victoria-metrics --cascade=orphan
````
## K8S Inspecting Flows and Outputs
````
// Inspecting Flows and Outputs
kubectl get ClusterOutput -n cattle-logging-system
kubectl describe ClusterOutput kafka-broker-logs-DC -n cattle-logging-system
..
Name:         kafka-broker-logs-DC
Namespace:    cattle-logging-system
Labels:       app.kubernetes.io/managed-by=Helm
              objectset.rio.cattle.io/hash=eb144977a6e6a2bacddb574707e5ea19af81c705
Annotations:  meta.helm.sh/release-name: logging-resources
              meta.helm.sh/release-namespace: cattle-logging-system
              objectset.rio.cattle.io/id: default-rancher-core-logging-logging-resources
API Version:  logging.banzaicloud.io/v1beta1
Kind:         ClusterOutput
Metadata:
  Creation Timestamp:  2023-12-27T20:58:16Z
  Generation:          1
  Resource Version:    658456668
  UID:                 6ea7629c-69d7-4525-8be4-4f0e95601d5b
Spec:
  Kafka:
    Brokers:  10.88.88.88:9092,10.89.89.89:9092,10.68.68.68:9092
    Buffer:
      chunk_limit_size:  512KB
      retry_forever:     true
    default_topic:       k8s-DC
    Format:
      Type:  json
Status:
  Active:  true
Events:    <none>

kubectl get Output -n development
kubectl get Flows -n development
kubectl get ClusterFlows -n cattle-logging-system
kubectl describe Flows application-backend -n development
...
Name:         application-backend
Namespace:    development
Labels:       app.kubernetes.io/managed-by=Helm
Annotations:  meta.helm.sh/release-name: application-backend
              meta.helm.sh/release-namespace: development
API Version:  logging.banzaicloud.io/v1beta1
Kind:         Flow
Metadata:
  Creation Timestamp:  2024-01-10T13:45:34Z
  Generation:          2
  Resource Version:    725358702
  UID:                 d99fb765-db24-4559-9098-ff7570f0c334
Spec:
  Global Output Refs:
    kafka-broker-app
  Match:
    Select:
      container_names:
        backend
      Labels:
        app.role:  application-backend
Status:
  Active:  true
Events:    <none>

kubectl describe Flow ingress -n kube-system
..
Name:         ingress
Namespace:    kube-system
Labels:       app.kubernetes.io/managed-by=Helm
              objectset.rio.cattle.io/hash=eb144977a6e6a2bacddb574707e5ea19af81c705
Annotations:  meta.helm.sh/release-name: logging-resources
              meta.helm.sh/release-namespace: cattle-logging-system
              objectset.rio.cattle.io/id: default-rancher-core-logging-logging-resources
API Version:  logging.banzaicloud.io/v1beta1
Kind:         Flow
Metadata:
  Creation Timestamp:  2023-12-27T20:58:16Z
  Generation:          1
  Resource Version:    1411151
  UID:                 79fadb5c-666-47cb-9ab2-f2f3ae35c314
Spec:
  Global Output Refs:
    kafka-broker-ingress
  Match:
    Select:
      container_names:
        rke2-ingress-nginx-controller
      Labels:
        app.kubernetes.io/name:  rke2-ingress-nginx
Status:
  Active:  true
Events:    <none>
````

## Ceph
````
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/toolbox.yaml
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash

ceph status
..
  cluster:
    id:     46a19703-4531-44fb-bc7a-77777777777
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,b,c (age 3w)
    mgr: a(active, since 7w), standbys: b
    mds: 1/1 daemons up, 1 hot standby
    osd: 3 osds: 3 up (since 7w), 3 in (since 7w)
 
  data:
    volumes: 1/1 healthy
    pools:   3 pools, 49 pgs
    objects: 14.89k objects, 58 GiB
    usage:   175 GiB used, 425 GiB / 600 GiB avail
    pgs:     49 active+clean
 
  io:
    client:   1.8 KiB/s rd, 682 B/s wr, 2 op/s rd, 0 op/s wr


ceph df
..
--- RAW STORAGE ---
CLASS     SIZE    AVAIL     USED  RAW USED  %RAW USED
hdd    600 GiB  425 GiB  175 GiB   175 GiB      29.21
TOTAL  600 GiB  425 GiB  175 GiB   175 GiB      29.21
 
--- POOLS ---
POOL             ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
myfs-metadata     1   16  497 MiB      197  1.5 GiB   0.37    132 GiB
myfs-replicated   2   32   57 GiB   14.70k  171 GiB  30.23    132 GiB
.mgr              3    1  449 KiB        2  1.3 MiB      0    132 GiB
````
## Helm
````
helm ls --all-namespaces
helm list -n sentry
helm repo list
helm search repo redis
helm show chart bitnami/redis
helm install sentry-redis bitnami/redis --namespace sentry -f redis-values.yaml
helm upgrade sentry-kafka bitnami/kafka --namespace sentry  -f kafka-values.yaml
helm get values sentry-kafka -n sentry
helm get manifest sentry -n sentry
helm template sentry -n sentry

// Render helm chart ignoring the yaml errors, so you can see what's parsed wrong
helm --debug template . -f values.yaml

// if you use this annotation on the PVC, it will skip deleting the resource on uninstall.
helm.sh/resource-policy: "keep"
````
## Fleet
How to deploy manually in fleet
1. Create YAML
```
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: #string
  annotations:
    {}
    #  key: string
  labels:
    {}
    #  key: string
  namespace: fleet-default
spec:
  branch: master
  insecureSkipTLSVerify: false
  paths:
#    - string
  repo: ''
  targets:
    - clusterSelector:
        matchExpressions:
          - key: provider.cattle.io
            operator: NotIn
            values:
              - harvester
#    - clusterGroup: string
#      clusterGroupSelector:
#        matchExpressions:
#          - key: string
#            operator: string
#            values:
#              - string
#        matchLabels:  key: string
#      clusterName: string
#      clusterSelector:
#        matchExpressions:
#          - key: string
#            operator: string
#            values:
#              - string
#        matchLabels:  key: string
#      name: string
#  caBundle: string
#  clientSecretName: string
#  forceSyncGeneration: int
#  helmRepoURLRegex: string
#  helmSecretName: string
#  imageScanCommit:
#    authorEmail: string
#    authorName: string
#    messageTemplate: string
#  imageScanInterval: string
#  keepResources: boolean
#  paused: boolean
#  pollingInterval: string
#  revision: string
#  serviceAccount: string
#  targetNamespace: string
__clone: true
```
Edit it to something like this
```
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: #string
  annotations:
    {}
    #  key: string
  labels:
    {}
    #  key: string
  name: milvus-testing
  namespace: fleet-default
spec:
  branch: TASK-8338
  clientSecretName: gitrepo-auth-hehehexbxbbs
  helmSecretName: helm-secret
  insecureSkipTLSVerify: false
  paths:
    - /milvus
  paused: true
  repo: https://gitlab.company.com/infrastructure/kubernetes/fleet-core.git
  targets:
    - clusterName: my-cluster-prod
#    - clusterGroup: string
#      clusterGroupSelector:
#        matchExpressions:
#          - key: string
#            operator: string
#            values:
#              - string
#        matchLabels:  key: string
#      clusterName: string
#      clusterSelector:
#        matchExpressions:
#          - key: string
#            operator: string
#            values:
#              - string
#        matchLabels:  key: string
#      name: string
#  caBundle: string
#  clientSecretName: string
#  forceSyncGeneration: int
#  helmRepoURLRegex: string
#  helmSecretName: string
#  imageScanCommit:
#    authorEmail: string
#    authorName: string
#    messageTemplate: string
#  imageScanInterval: string
#  keepResources: boolean
#  paused: boolean
#  pollingInterval: string
#  revision: string
#  serviceAccount: string
#  targetNamespace: string
__clone: true
```
