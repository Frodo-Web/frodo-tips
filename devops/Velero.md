# Velero
Скачать бекап
```
velero backup download 30-06-2026-backup
```

Извлечь
```
tar -xf 30-06-2026-ctv.dev.k8s.int.nic.ru-data.tar.gz
```

Зайти в нужный ресурс
```
cd resources/secrets/namespaces/dex/
```

Загрузить обратно в кластер (восстановить)
```
kubectl apply -f dex-secret.json
```
