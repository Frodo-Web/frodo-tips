# Jira queries
````
Пример отсортированного запроса по тайтлу
project = INFRA AND summary ~ "Просим увеличить объём свободного места на" ORDER BY updated DESC
Поиск по описанию
project = INFRA AND description  ~ "8.8.8.8" ORDER BY updated DESC

Пример поиска по комментам в проекте WORK
project = WORK  AND comment  ~ "No such file or directory:" ORDER BY updated DESC

Все задачи, с которых watch не убирал
project = INFRA AND watcher = currentUser() ORDER BY updated DESC
````
