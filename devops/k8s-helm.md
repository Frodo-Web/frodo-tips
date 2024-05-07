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
