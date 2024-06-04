# Kubernetes & Helm
Here are some usefull commands
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
