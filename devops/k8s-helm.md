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
