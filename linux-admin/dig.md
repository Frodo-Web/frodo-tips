# Dig usage
````
Определим днс-сервера ответственные за интересующий нас домен:

dig -t NS i-test.ru

; <<>> DiG 9.18.4-2ubuntu2.1-Ubuntu <<>> -t NS i-test.ru
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50854
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 4

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;i-test.ru.			IN	NS

;; ANSWER SECTION:
i-test.ru.		2447	IN	NS	ns2.i-test.ru.
i-test.ru.		2447	IN	NS	ns5.i-test.ru.
i-test.ru.		2447	IN	NS	ns1.i-test.ru.

;; ADDITIONAL SECTION:
ns2.i-test.ru.		2144	IN	A	89.113.19.2
ns5.i-test.ru.		2144	IN	A	18.200.123.175
ns1.i-test.ru.		2144	IN	A	89.113.17.2

;; Query time: 8 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Sat Apr 01 09:25:29 MSK 2023
;; MSG SIZE  rcvd: 140

dev@Lustre:~$ dig test-proxy-roma.i-test.ru @ns1.i-test.ru

; <<>> DiG 9.18.4-2ubuntu2.1-Ubuntu <<>> test-proxy-roma.i-test.ru @ns1.i-test.ru
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20875
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;test-proxy-roma.i-test.ru.	IN	A

;; ANSWER SECTION:
test-proxy-roma.i-test.ru. 3600	IN	CNAME	mon-web-01.master.i-test.ru.
mon-web-01.master.i-test.ru. 3600 IN	A	89.113.17.47

;; AUTHORITY SECTION:
master.i-test.ru.	3600	IN	NS	ns1.i-test.ru.
master.i-test.ru.	3600	IN	NS	ns2.i-test.ru.

;; ADDITIONAL SECTION:
ns1.i-test.ru.		3600	IN	A	89.113.17.2
ns2.i-test.ru.		3600	IN	A	89.113.19.2

;; Query time: 8 msec
;; SERVER: 89.113.17.2#53(ns1.i-test.ru) (UDP)
;; WHEN: Sat Apr 01 09:30:54 MSK 2023
;; MSG SIZE  rcvd: 170

dev@Lustre:~$ dig test-proxy-roma.i-test.ru @ns1.i-test.ru +short
mon-web-01.master.i-test.ru.
89.113.17.47

DIG_CMD = 'dig {record} @{ns_server} {_type} +short'

Опросить DNS гугла
dig @8.8.8.8 i-test.com

Опросить локальный DNS
dev@Lustre:~/Work/i-test$ dig @172.27.245.100  google.com +noall +answer
google.com.		2	IN	A	142.251.1.101
google.com.		2	IN	A	142.251.1.102
google.com.		2	IN	A	142.251.1.113
google.com.		2	IN	A	142.251.1.138
google.com.		2	IN	A	142.251.1.139
google.com.		2	IN	A	142.251.1.100

Опросить внешний DNS..
В случае обнаружения старых записей, которые просто закешировались и не обновляются
Можно залогиниться на ту машину с DNS и зафлашить запись
sudo rndc flushname google.com

Проверяем на этой же машине резолвится ли запись
dev@Lustre:~/Work/i-test$ host google.com ns1.i-test.ru
Using domain server:
Name: ns1.i-test.ru
Address: 89.113.17.2#53
Aliases: 

google.com has address 172.217.16.142
google.com has IPv6 address 2a00:1450:4001:808::200e
google.com mail is handled by 10 smtp.google.com.

То есть офис обращается к одной DNS, какая нить машину к другой и т.д. Просто ходим проверяем корректность и актуальность записей на домен

В айтоп пишешь запись с ошибкой, puppet развернёт с ошибкой. Логи daemon.log, syslog

Хостом резолвим по определенным ДНС определенный домен, dig'ом опрашиваем DNS о записях


Синтаксис команды dig

$ dig @сервер доменное.имя тип_записи флаги

Где:

@cервер — IP-адрес или доменное имя DNS-сервера (если не указано, dig будет обращаться к DNS-серверу, используемому по умолчанию);
доменное.имя — доменное имя интернет-ресурса, о котором необходимо получить информацию;
тип записи — позволяет указать, для какого типа записи необходим вывод, например A, NS, MX или TXT;
флаги — с помощью флагов утилите dig отдаются дополнительные команды; оговаривается, каким должен быть вывод команды (что в нём должно быть, а чего нет).
Опции и флаги dig
Во время работы утилиты dig могут использоваться следующие флаги:

+[no]all — отображает или скрывает все установленные по умолчанию флаги отображения;
+[no]answer — отображает только ответ на запрос;
+[no]fail — эта опция указывает, должна ли утилита переключаться на следующий DNS сервер, если текущий не отвечает (по умолчанию стоит +fail);
+short — сокращает вывод утилиты;
+[no]cmd — отключает вывод заголовка и информации об использованных опциях утилиты;
+[no]identify — используется вместе с флагом +short и отображает информацию об IP-адресе сервера;
+[no]comments — удаляет все комментарии из вывода утилиты;
+[no]trace - позволяет вывести список DNS серверов через которые прошёл запрос на получение информации о домене, по умолчанию отключено.

Вместе с dig можно применять следующие опции:

-4 — позволяет использовать только IPv4;
-6 — позволяет использовать только IPv6;
-x — предназначена для получения домена по IP;
-f — используется для чтения списка доменов из файла;
-b IP-адрес — позволяет указать исходящий IP-адрес, с которого отправлен запрос к DNS-серверу, полезно, если к компьютеру подключено несколько сетевых карт;
-r — предотвращает чтение настроек из файла ~/.digrc;
-t — позволяет указать тип записи, которую надо получить;
-p — позволяет указать номер порта DNS сервера;
-u — отображает время в секундах вместо миллисекунд.

HEADER — отображает информацию о версии утилиты, ID запроса, полученных ошибках и использованных флагах вывода. Выводится и другая важная информация о количестве запросов, обращений к DNS-серверу и т. д.;
QUESTION SECTION — секция, которая отображает текущий запрос;
ANSWER SECTION — секция, в которой отображается результат обработки созданного запроса (в данном случае это IP-адрес домена).

Для того чтобы получить информацию о перечисленных в файле sites.txt доменах, используйте команду:
dig -f sites.txt +noall +answer

Получить домен по ip
dev@Lustre:~/Work/i-test$ dig -x 89.223.47.132 +short
bill2.nevalink.net.

trace игнорит dns кеш
dig +trace @192.168.0.1 google.com

; <<>> DiG 9.18.4-2ubuntu2.1-Ubuntu <<>> +trace @192.168.0.1 google.com
; (1 server found)
;; global options: +cmd
.			497709	IN	NS	l.root-servers.net.
.			497709	IN	NS	a.root-servers.net.
.			497709	IN	NS	g.root-servers.net.
.			497709	IN	NS	j.root-servers.net.
.			497709	IN	NS	f.root-servers.net.
.			497709	IN	NS	b.root-servers.net.
.			497709	IN	NS	i.root-servers.net.
.			497709	IN	NS	e.root-servers.net.
.			497709	IN	NS	d.root-servers.net.
.			497709	IN	NS	h.root-servers.net.
.			497709	IN	NS	c.root-servers.net.
.			497709	IN	NS	k.root-servers.net.
.			497709	IN	NS	m.root-servers.net.
.			497709	IN	RRSIG	NS 8 0 518400 20230416170000 20230403160000 60955 . hj9gK83RWBmQTmc+RymB2R8ku81+TA9iKY680btiUlh7EjXa/yLH3eH7 b6EbLqvAM83He8+9YEa+4+6ymLYCAn1CpwwyjD8h52zTRB++zUmUAxhU OqqQbJ0eYPpjpqEJFboXi/Ccw/qYlhqfkt+qNfFYmXkVbafgJkyM9sfu vHPsWWvSd7fDNd+9Xb4qJk583s8EET1zaySZ6OO+qSwCy/LoiAmp4Pjd W+tld7726MBu+qwox7AFD+D5+T/ZIq3zj8jEuwHzf+x5pq25TVfBoLhe +JDdgv7PKlImXhlZc/goShg1GyGaixjV3rmfowisoBm8QIbhVHHUbvFa db0hFg==
;; Received 1137 bytes from 192.168.0.1#53(192.168.0.1) in 7 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.			86400	IN	RRSIG	DS 8 1 86400 20230417050000 20230404040000 60955 . YK8GB0+pwNsrXGYh0G9gGpjg1j9vkfHZo00muQgAm8JFPfj7+7S8hGpg xfzRcVnujAAWIslvu8UlYGfr1TN1e2SrzJ65j3rrHp+X3Qj1moD6v94U x9tvt47x1h5BxIz6WNCtvQAnt2uCvEseYBAD3aigULCmYyQ5FFlp1W0B CFeqjU/0/ShwpIqsTfuSpKYR5Y3l0TVk78OkGGg5rm5Rl4jSg/Umd136 dUCPhgOa3pR7Qudw24SJaWRWOwhFHjW/PjBsywvowyAaBdyHYNA67Adi wfWPRpNcqQIY0d7DmjXkmxUwS8TGcNH9J3VAQqSNtbJetpHVoQ4A0zVW g7zIgw==
;; Received 1170 bytes from 198.97.190.53#53(h.root-servers.net) in 51 ms



You won't find this hint file on every computer. Usually this is only needed when you run a nameserver like BIND, Unbound etc. Those nameservers need the information about the root servers.
When you do a DNS query your computer usually asks your /etc/hosts file first. If it doesn't find an answer there, it will ask the nameserver (see contents of /etc/resolv.conf) for an answer. The nameserver will then either have the answer in its cache or ask the root servers and later other nameservers until the right answer arrives. So there is no need for a hint file on your local computer.

Вот этот root hint файл можно посмотреть так
dig . ns
Рутовые серваки. Их на самом деле там сотни по многим странам.

Yes, the "x.gtld-servers.net" are the authoritative servers for the "com" top level domain, so they have all the "pointers" for the .com domains. You can see the nameservers for the TLD by running

dig -t ns com
dig -t ns us
dig -t ns dk
dig -t ns aero

dev@Lustre:~$ dig -t ns dk

; <<>> DiG 9.18.4-2ubuntu2.1-Ubuntu <<>> -t ns dk
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39223
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 13

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;dk.				IN	NS

;; ANSWER SECTION:
dk.			86400	IN	NS	l.nic.dk.
dk.			86400	IN	NS	a.nic.dk.
dk.			86400	IN	NS	d.nic.dk.
dk.			86400	IN	NS	s.nic.dk.
dk.			86400	IN	NS	b.nic.dk.
dk.			86400	IN	NS	c.nic.dk.

;; ADDITIONAL SECTION:
l.nic.dk.		86400	IN	A	130.226.213.138
l.nic.dk.		86400	IN	AAAA	2001:878:0:e000:82:e2:d5:8a
a.nic.dk.		86400	IN	A	212.88.78.122
a.nic.dk.		86400	IN	AAAA	2001:1580:0:180d::122
d.nic.dk.		86400	IN	A	185.159.198.45
d.nic.dk.		86400	IN	AAAA	2620:10a:80ab::45
s.nic.dk.		86400	IN	A	193.176.144.15
s.nic.dk.		86400	IN	AAAA	2a00:d78:0:102:193:176:144:15
b.nic.dk.		86400	IN	A	193.163.102.222
b.nic.dk.		86400	IN	AAAA	2a01:630:0:80::53
c.nic.dk.		86400	IN	A	194.0.46.53
c.nic.dk.		86400	IN	AAAA	2001:678:74::53

;; Query time: 644 msec
;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
;; WHEN: Tue Apr 04 10:14:48 MSK 2023
;; MSG SIZE  rcvd: 395


Корневые серверы DNS — DNS-серверы, обеспечивающие работу корневой зоны DNS в сети Интернет. Корневые серверы DNS отвечают на запросы других DNS-серверов в ходе трансляции доменных имён в IP-адреса и позволяют получить список DNS-серверов для любого домена верхнего уровня (TLD): RU, COM, NET, MUSEUM и др.

Короч, CNAME это тупа привязка, Alias Name. То есть он на том же NS серваке посмотрит A-запись, никаких резолвов
dev@Lustre:~$ dig test-proxy-roma.i-test.ru +noall +answer
test-proxy-roma.i-test.ru. 3486	IN	CNAME	mon-web-01.master.i-test.ru.
mon-web-01.master.i-test.ru. 3486 IN	A	89.113.17.47

Короч почтовая программа такой вводишь google.com она по MX записи выясняет где находится почтовый сервер и уже с ним будет работать например
для доставки почты. Если MX не будет почта будет доставлятся по А записи. То есть первым делом походу почтовые программы смотрят на
MX запись а потом на A.
dev@Lustre:~$ dig -t MX google.com +noall +answer +additional

google.com.		275	IN	MX	10 smtp.google.com.
smtp.google.com.	275	IN	A	74.125.133.27
smtp.google.com.	275	IN	AAAA	2a00:1450:400c:c00::1a
smtp.google.com.	275	IN	AAAA	2a00:1450:400c:c0c::1a
smtp.google.com.	275	IN	AAAA	2a00:1450:400c:c0c::1b
smtp.google.com.	275	IN	A	108.177.15.26
smtp.google.com.	275	IN	A	173.194.76.27
smtp.google.com.	275	IN	AAAA	2a00:1450:400c:c07::1a
smtp.google.com.	275	IN	A	108.177.15.27
smtp.google.com.	275	IN	A	173.194.76.26

Каждая MX запись имеет приоритет, что позволяет разгрузить нагрузку между несколькими почтовыми серверами. Соответственно, приоритет сообщает почтовым программам(доставщикам), к какому из серверов обратится первому. И далее по убывающей, например если сервер с высоким приоритетом не отвечает.
Примечание: Высокий приоритет в данном контексте означает не самое большое число, а самое маленькое, т.е. 10 выше чем 50.

Во у gmail лучше видно про приоритеты

dev@Lustre:~$ dig -t MX gmail.com +noall +answer +additional
gmail.com.		33	IN	MX	5 gmail-smtp-in.l.google.com.
gmail.com.		33	IN	MX	10 alt1.gmail-smtp-in.l.google.com.
gmail.com.		33	IN	MX	20 alt2.gmail-smtp-in.l.google.com.
gmail.com.		33	IN	MX	30 alt3.gmail-smtp-in.l.google.com.
gmail.com.		33	IN	MX	40 alt4.gmail-smtp-in.l.google.com.
gmail-smtp-in.l.google.com. 300	IN	A	66.102.1.27
alt3.gmail-smtp-in.l.google.com. 54 IN	AAAA	2a00:1450:4010:c1c::1b
alt4.gmail-smtp-in.l.google.com. 54 IN	A	74.125.200.26
alt4.gmail-smtp-in.l.google.com. 54 IN	AAAA	2404:6800:4003:c00::1a

````
