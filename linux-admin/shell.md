# Shell
Run script as another user
````
sudo -u zabbix bash -c "export external_logging=console;/etc/zabbix/externalscripts/database_query.py database_host.ru db_name table_name"
````
