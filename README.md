## Домашнее задание к занятию 5 «Тестирование roles»
=========

### Требования
------------

Для запуска использовались следующие версии:

- Ansible [core 2.15.13]
- Molecule 6.0.3
- Tox 4.30.3
- Python 3.9.23
- Docker version 29.1.3


### Создание виртуального окружения в Python

```bash
mkdir venv
python3 -m venv venv/
source venv/bin/activate
pip3 install molecule molecule_docker molecule_podman
```

Также можно указать конкретные версии, но все зависит от версии python3 в системе. Например, при версии python3.12 не смог установить "molecule==3.5.2"

Чтобы отключиться от окружения используется команда ```deactivate```

### Molecule

**Примечание1**: Возника роблема с неймингом. Директория с ролью называется "vector-role", но тест запускался с ошибкой, пока не убрал "-" из названия. Переименовал в "vector_role"

**Примечание2**: В роли используется перезапуск systemd службы, а тест происходит внутри docker контейнера. Поэтому нужны соответствующие образы. Вот тут их можно найти: [docker-systemd](https://github.com/antmelekhin/docker-systemd/tree/main)

Запуск теста:
В директории с роль запускаем команду:
```bash
molecule test -s default
```

### Tox

1. Запускаем контейнер
```bash
docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash
```
где path_to_repo — путь до корня репозитория с vector-role на файловой системе. В моем случае это было ```/home/username/.ansible/roles/vector_role:/opt/vector-role```

2. Внутри контейнера выполняем команду 
```bash 
tox
```

В моем случае тест закончился так:

```bash
py37-ansible210: commands succeeded
py37-ansible30: commands succeeded
ERROR:   py39-ansible210: commands failed
ERROR:   py39-ansible30: commands failed
```
**Причина ошибки:**
Несовместимость между:

- Molecule (требует определенную версию Ansible)
- Ansible 3.0 (которая не имеет should_retry_error в API)

## О роли

Ansible-роль для установки и настройки **Vector – Observability Data Pipeline** на серверах.
Vector — это высокопроизводительный инструмент для сбора, обработки и маршрутизации логов и метрик в централизованные системы наблюдения

**Эта роль:**

- Устанавливает Vector
- Настраивает конфигурацию Vector
- Управляет его службой (systemd)
- Обеспечивает запуск и перезапуск при изменениях конфигурации

### Требования
------------

Перед использованием роли убедитесь, что:

- Ansible >= 2.9
- Целевые хосты доступны по SSH
- На хостах есть sudo‑доступ для пользователя Ansible

### Переменные роли
--------------
Все переменные роли находятся в `defaults/main.yml`.

| Переменная        | По умолчанию | Описание |
|-------------------|--------------|----------|
| `vector_version`  | `"0.51.1-1"` | Версия Vector, которая будет установлена |

### Пример использования
----------------

Вот пример простого playbook, который устанавливает Vector на группу хостов:

```yml
- name: Install Vector
  hosts: vector
  vars:
    clickhouse_host: "{{ hostvars['clickhouse-01'].ansible_host }}"
  roles:
    - vector-role
```

### Лицензия
-------

MIT License