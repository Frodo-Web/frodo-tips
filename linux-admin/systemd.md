# Systemd usage
## Journalctl
Read logs from the end for specific service since last boot
````
journalctl -u logstash -b -e
````
## Systemd
````
Systemd конфигурация состоит из юнитов
/usr/lib/systemd/system/ - юниты из rpm пакетов
/run/systemd/system/ - юниты созданные в рантайме
/etc/systemd/system/ - юниты созданные сис админом
Формат:
[Название секции]
переменная = значение
Три секции для простого юнита: [Unit], [Service], [Install]
[Unit]
Description=MyUnit

After=syslog.target   // Запускать после сервиса или группы
After=network.target
After=nginx.service
After=mysql.service

Requires=mysql.service // Необходим запущенный сервис

Wants=redis.service // Желателен запущенный

Если сервис есть в Requires, но нет в After, то сервис будет запущен параллельно с треб. сервисом

[Service]
Type=simple // служба будет запущена незамедлительно, процесс не должен разветвляться
Type=forking // предполагает что служба запускается однократно и процесс разветвляется с завершением родительского процесса. Данный тип используется для запуска демонов
PIDFile=/work/www/myunit/shared/tmp/pids/service.pid //  чтобы системд могла отслеживать основной процесс
WorkingDirectory=/work/www/myunit/current // Рабочий каталог
User=myunit  // Пользователь и группа под которыми надо стартовать сервис
Group=myunit
Environment=RACK_ENV=production // переменные окружения
OOMScoreAdjust=-100 // Запрет на убийство сервиса вследствие нехватки памяти и срабатывание механизма OOM (-1000 - полный запрет стоит у sshd, -100 - понизить вероятность).
ExecStart=/usr/local/bin/bundle exec service -C      // Команды на старт стоп релоад
/work/www/myunit/shared/config/service.rb --daemon
ExecStop=/usr/local/bin/bundle exec service -S 
/work/www/myunit/shared/tmp/pids/service.state stop
ExecReload=/usr/local/bin/bundle exec service -S
/work/www/myunit/shared/tmp/pids/service.state restart
TimeoutSec=300 // Cколько системд ждать отработки старт стоп команд
Restart=always // Автоматически рестартовать сервис если перестанет работать

[Install]                      // Optional
// Описываем уровень запуска
WantedBy=multi-user.target

systemctl status myunit
Output:
myunit.service - MyUnit
   Loaded: loaded (/etc/systemd/system/myunit.service; disabled)
   Active: inactive (dead)

systemctl enable myunit
systemctl start myunit

При изменениях в юните перезагружать демон системд:
systemctl daemon-reload

Some features that units are able implement easily are:
socket-based activation
bus-based activation (D-Bus)
path-based activation (inotify)
device-based activation
implicit dependency mapping
instances and templates
easy security hardening (enable read only)

/etc/systemd/system/myunit.d/..conf

Type of units:
.service
.socket
.device
.mount
.automount
.swap
.target (provide sync between units on boot)
.path  (path can be use for path-based activation)
.timer (similar to cron)

[Install] нужен для функциональности enabled disabled
WantedBy - короч указывается сервис тогда в дире
/etc/systemd/system создасться дира с окончанием .wants и там будет симлинк на этот сервис
/etc/systemd/system/multi-user.target.wants
Когда делаешь disable симлинк оттуда удаляется.
Так что это штука которая видимо включает сервис на загрузке системы
RequiredBy= похожая вещь но если что то пойдёт не так то вылетят ошибки а дира будет
с окончанием .requires
Alias=  //  Можно задать другое имя

[Service]
PIDFile = // если использовался Type=forking  помогает мониторить сервис тк содержит путь к файлу с pid

ExecStart= // Если использовать -(dash) перед командой, то non-zero выходы не будут помечаться как фейл
ExecStartPre =  //  Выполнить ещё какие то команды перед основной
ExecStartPost= // тож самое только после основного процесса
ExecReload
ExecStop
ExecStopPost
RestartSec = // задаёт время перед попыткой каждого рестарта если был включен
Restart= always / on-success / on-failure / on-abnormal / on-abort / on-watchdog

[Socket]
ListenStream= // address for a stream socket, TCP
ListenDatagram= // For UDP using sockets
ListenSequentialPacket // Unix Socket
ListenFIFO

[Mount]
[AutoMount]
[Swap]
[Path]

[Timer]
Used to schedule tasks to operate at a certain time or after a certain delay
OnBootSec= // After system is booted
OnStartupSec= // when systemd started
Линкуется с .timer юнитом и там команды спит он или активничает и что то выполнять каждое
н время

Template unit file look like this:
example@.service
Instance will have a name between those
example@instance1.service

systemctl is the central managment tool for controlling the init system
systemctl start/stop/restart/reload/reload-or-restart application.service (.service for services)

systemctl enable/disable application.service

enable/disable created/deletes symlink 
/lib/systemd/system /etc/systemd/system -> /etc/systemd/system/some_target.target.wants - здесь системд ищет файлы на автостарт

systemctl status application.service

systemctl is-active app.service // возвращает active или inactive а также код выхода 0 если
активный, это удобно использовать в скриптах
systemctl is-enabled app.service
systemctl is-failed app.service

systemctl list-units
systemctl list-units --all
systemctl list-units --all --state=inactive
systemctl list-units --type=service
systemctl list-unit-files
Output
UNIT FILE                                  STATE
proc-sys-fs-binfmt_misc.automount          static
dev-hugepages.mount                        static
dev-mqueue.mount                           static
proc-fs-nfsd.mount                         static
proc-sys-fs-binfmt_misc.mount              static
sys-fs-fuse-connections.mount              static
sys-kernel-config.mount                    static
sys-kernel-debug.mount                     static
tmp.mount                                  static
var-lib-nfs-rpc_pipefs.mount               static
org.cups.cupsd.path                        enabled

systemctl cat atd.service
Output
[Unit]
Description=ATD daemon
[Service]
Type=forking
ExecStart=/usr/bin/atd
[Install]
WantedBy=multi-user.target

systemctl list-dependencies sshd.service
Output
sshd.service
├─system.slice
└─basic.target
  ├─microcode.service
  ├─rhel-autorelabel-mark.service
  ├─rhel-autorelabel.service
  ├─rhel-configure.service
  ├─rhel-dmesg.service
  ├─rhel-loadmodules.service
  ├─paths.target
  ├─slices.target
. . .

systemctl show sshd.service
Output
Id=sshd.service
Names=sshd.service
Requires=basic.target
Wants=system.slice
WantedBy=multi-user.target
Conflicts=shutdown.target
Before=shutdown.target multi-user.target
After=syslog.target network.target auditd.service systemd-journald.socket basic.target system.slice
Description=OpenSSH server daemon
. . .

systemctl show sshd.service -p Conflicts
Output
Conflicts=shutdown.target

sudo systemctl mask nginx.service
systemctl list-unit-files
Output
nginx.service                          masked

sudo systemctl start nginx.service
Output
Failed to start nginx.service: Unit nginx.service is masked.

sudo systemctl unmask nginx.service

Targets are special unit files that describe a system state or synchronization point. 
Targets do not do much themselves, but are instead used to group other units together.
For instance, there is a swap.target that is used to indicate that swap is ready for use. Units that are part of this process can sync with this target by indicating in their configuration that they are WantedBy= or RequiredBy= the swap.target. Units that require swap to be available can specify this condition using the Wants=, Requires=, and After= specifications to indicate the nature of their relationship.

systemd имеет дефолтный .target который используется на буте
systemctl get-default
Output
multi-user.target

systemctl set-default graphical.target
systemctl list-unit-files --type=target

systemctl list-dependencies multi-user.target
sudo systemctl isolate multi-user.target // Будут запущены только эти службы, graphical.target будут выключены. Из графики в консольку. Важно что графика имеет зависимости от multi-user но не наоборот.

Есть таргеты как шорткаты для ивентов
например power off и reboot

sudo systemctl rescue // В сингл юзер мод, заместо isolate rescue.target
sudo systemctl halt
sudo systemctl poweroff / reboot  // sudo reboot делает как раз это тк линк
````
