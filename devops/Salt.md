# Salt
See all grains (system info + custom grains):
```
salt-call grains.items
```
Отображает пути до питонов и их версии, версию ядра, дистр и всё такое. Айпишники, ДНСы, поддерживаемые технологии цпу. Много инфы по системе
<br>

Отобразить конкретный grain
```
salt-call grains.item role
..
local:
    ----------
    role:
        k8s-ca
```

Пиллары, параметры которые подтягиваются с мастеров
```
salt-call pillar.items
..
    server:
        ----------
        RedHat:
            ----------
            packages:
                - lsof
                - vim-enhanced
                - sysstat
                - nano
                - wget
                - telnet
                - bash-completion
                - tcpdump
```

Получить специфическое значение пиллар
```
salt-call pillar.get role
..
local:
```

Dry-run что будет применено к хосту
```
salt-call state.apply test=True
```

State execution cache, показывает что было применено последние разы, даже если код ролей поменялся уже
```
salt-call state.show_highstate | less
```

А вот посмотреть список всех ролей, что применялись
```
salt-call state.show_top
..
[INFO    ] Loading fresh modules for state activity
local:
    ----------
    base:
        - RedHat
        - network
        - chrony
        - aliases
        - openldap.client
        - sudo
        - users
        - sshd
        - exim
        - login_access
        - resolv
        - server
        - ssl
        - sysctl
        - salt.minion
        - k8s-ca
        - zabbix-agent
        - prometheus
        - ocsinventory
```
