# Clickhouse (CH) tips
Connect to CH remotely
```
clickhouse-client --host=192.168.1.100 --port=9000 --user=user1 --password=mysecretpassword --database=mydatabase
```

## Grafana + Clickhouse queries examples
```
WHERE (LENGTH('${dc:text}') > 3 OR '${dc:text}' = 'All' OR hostname LIKE concat('%', $dc, '%'))
WHERE ('${dc:raw}' = '(dc1|dc2|dc3)' OR hostname LIKE concat('%', '${dc:raw}', '%'))

```
