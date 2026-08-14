# Server Diagnostics Jenkins Pipeline

Jenkins Pipeline для автоматической диагностики удалённого Linux-сервера.

Pipeline проверяет сервер с двух сторон:

1. С Jenkins Agent — сетевую доступность.
2. Через SSH — состояние сервера и запущенного сервиса.

После выполнения создаётся отчёт `diagnostic-report.txt`, который автоматически архивируется в Jenkins.

---

## Что проверяет Pipeline

### С Jenkins Agent

- `ping` — ICMP-доступность сервера;
- TCP-порт — доступность указанного порта;
- HTTP — доступность HTTP-сервиса;
- `traceroute` / `tracepath` — маршрут до сервера;
- `mtr` — маршрут и потери пакетов.

### На целевом сервере

- hostname;
- uptime;
- состояние systemd-сервиса;
- включён ли сервис в автозагрузке;
- `systemctl status`;
- последние 50 записей `journalctl`;
- Main PID сервиса;
- дочерние процессы;
- процессы с наибольшей загрузкой CPU;
- прослушиваемые порты;
- использование RAM;
- использование диска;
- OOM-события;
- локальный HTTP-запрос через `127.0.0.1`.

---

## Параметры

Pipeline принимает следующие параметры:

| Параметр | Описание | Пример |
|---|---|---|
| `SERVER` | IP-адрес или hostname сервера | `192.168.56.10` |
| `SSH_USER` | Пользователь для SSH | `artem` |
| `SERVICE` | Имя systemd-сервиса без `.service` | `nginx` |
| `PORT` | TCP-порт сервиса | `80` |

Пример:

    SERVER   = 192.168.56.10
    SSH_USER = artem
    SERVICE  = nginx
    PORT     = 80

---

## SSH

**Перед запуском Pipeline необходимо предварительно настроить SSH-доступ между Jenkins Agent и целевым сервером.**

Pipeline сам SSH-доступ не настраивает.

Jenkins Agent должен иметь возможность подключиться к серверу без интерактивного ввода пароля.

Например:

    ssh artem@192.168.56.10

Рекомендуется использовать SSH-ключ.

Необходимо убедиться, что:

- SSH-сервер запущен на целевой машине;
- Jenkins Agent может подключиться к SSH-порту;
- публичный ключ Jenkins Agent добавлен в `~/.ssh/authorized_keys`;
- пользователь `SSH_USER` существует на сервере;
- пользователь имеет необходимые права для выполнения диагностических команд;
- подключение работает без запроса пароля.

Проверка:

    ssh -o BatchMode=yes artem@192.168.56.10

Pipeline использует non-interactive SSH-подключение.

---

## Требования

### Jenkins Agent

Желательно наличие:

    ping
    nc
    telnet
    curl
    traceroute
    tracepath
    mtr
    ssh

Если необходимая утилита отсутствует, соответствующая проверка получает статус `UNKNOWN` или `NOT AVAILABLE`.

### Target Server

Необходимы:

    systemctl
    journalctl
    ps
    ss
    free
    df
    dmesg
    curl

---

## Структура Pipeline

    Initialize
        ↓
    Network Diagnostics
        ↓
    Server Diagnostics
        ↓
    Create Summary
        ↓
    Archive Report

### Initialize

Создаёт отчёт и записывает основные параметры проверки.

### Network Diagnostics

Проверяет доступность сервера с Jenkins Agent.

### Server Diagnostics

Подключается к серверу по SSH и собирает информацию о системе и сервисе.

### Create Summary

Формирует краткий итог диагностики.

### Archive Report

Архивирует `diagnostic-report.txt` в Jenkins.

---

## Пример отчёта

    ========================================
    SERVER DIAGNOSTIC REPORT
    ========================================

    Host    : 192.168.56.10
    Port    : 80
    Service : nginx
    SSH User: artem

    ========================================
    NETWORK DIAGNOSTICS
    ========================================

    PING
    TCP PORT CHECK
    HTTP CHECK
    TRACEROUTE
    MTR

    ========================================
    SERVER DIAGNOSTICS
    ========================================

    HOST
    UPTIME
    SERVICE STATE
    SERVICE STATUS
    SERVICE LOGS
    PROCESS
    TOP PROCESSES BY CPU
    LISTENING PORT
    MEMORY
    DISK
    OOM
    LOCAL HTTP CHECK

    ========================================
    DIAGNOSTIC SUMMARY
    ========================================

---

## Что позволяет определить

Pipeline помогает быстро определить уровень проблемы:

    Jenkins Agent
         │
         ├── Ping
         ├── TCP Port
         ├── HTTP
         ├── Traceroute / Tracepath
         └── MTR
                 │
                 ▼
           Target Server
                 │
                 ├── SSH
                 ├── systemd
                 ├── Processes
                 ├── Listening Ports
                 ├── Memory
                 ├── Disk
                 ├── OOM
                 └── Local HTTP

Таким образом можно определить:

- сервер недоступен по сети;
- TCP-порт недоступен;
- HTTP-сервис не отвечает;
- есть проблемы с маршрутом;
- SSH недоступен;
- systemd-сервис не запущен;
- сервис не слушает необходимый порт;
- высокая загрузка CPU;
- нехватка памяти;
- нехватка дискового пространства;
- произошёл OOM;
- сервис работает локально, но недоступен извне.

> `traceroute` или `mtr` со статусом `TIMEOUT` не обязательно означают проблему с сервером. ICMP/TTL-трафик может фильтроваться firewall или сетевым оборудованием, в то время как TCP/HTTP продолжает работать нормально.

---

## Результат

После выполнения Pipeline создаётся:

    diagnostic-report.txt

Файл автоматически архивируется в Jenkins и может использоваться для анализа проблем или прикладываться к инциденту.


Ссылка на chatGPT: https://chatgpt.com/c/6a7d050e-e000-83ed-853c-5f63fe0426e4
