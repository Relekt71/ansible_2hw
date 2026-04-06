# Ansible Playbook: Установка Clickhouse и Vector

## Описание

Данный Ansible playbook предназначен для автоматизированной установки и настройки двух систем observability:

- **Clickhouse** (v22.3.3.44) - высокопроизводительная колоночная СУБД для аналитики больших данных
- **Vector** (v0.21.1) - легковесный observability pipeline для сбора, трансформации и маршрутизации логов

Playbook обеспечивает полный цикл установки: от скачивания дистрибутивов до запуска сервисов с правильной конфигурацией.

## Что делает playbook

### Для Clickhouse (CentOS 7):
1. Устанавливает необходимые зависимости (curl, tar, which)
2. Скачивает RPM-пакеты Clickhouse указанной версии
3. Устанавливает пакеты: common-static, client, server
4. Запускает и включает сервис clickhouse-server
5. Создает базу данных `logs` (если не существует)
6. Очищает временные файлы

### Для Vector (Ubuntu 20.04):
1. Устанавливает зависимости (tar, gzip, wget, procps)
2. Создает системного пользователя `vector` для безопасного запуска
3. Создает необходимые директории (/opt/vector, /etc/vector, /var/lib/vector)
4. Скачивает официальный дистрибутив Vector с GitHub
5. Распаковывает архив и устанавливает бинарный файл
6. Создает символическую ссылку в /usr/local/bin
7. Деплоит конфигурацию через Jinja2 шаблон
8. Настраивает systemd сервис (или скрипт для Docker)
9. Автоматически перезапускает сервис при изменении конфигурации (handler)
10. Запускает и включает сервис Vector

## Параметры playbook

### Переменные Clickhouse

| Параметр | Описание | Значение по умолчанию |
|----------|----------|----------------------|
| `clickhouse_version` | Версия Clickhouse для установки | `"22.3.3.44"` |
| `clickhouse_packages` | Список устанавливаемых пакетов | `[clickhouse-client, clickhouse-server, clickhouse-common-static]` |

### Переменные Vector

| Параметр | Описание | Значение по умолчанию |
|----------|----------|----------------------|
| `vector_version` | Версия Vector | `"0.21.1"` |
| `vector_arch` | Архитектура системы | `"x86_64"` |
| `vector_install_dir` | Директория установки бинарного файла | `"/opt/vector"` |
| `vector_config_dir` | Директория для конфигурационных файлов | `"/etc/vector"` |
| `vector_data_dir` | Директория для данных Vector | `"/var/lib/vector"` |

## Теги

Playbook поддерживает теги для выборочного выполнения задач:

| Тег | Описание |
|-----|----------|
| `vector` | Все задачи по установке Vector |
| `vector/dependencies` | Установка зависимостей |
| `vector/user` | Создание пользователя vector |
| `vector/directories` | Создание директорий |
| `vector/check` | Проверка наличия установленного Vector |
| `vector/download` | Скачивание дистрибутива |
| `vector/extract` | Распаковка архива |
| `vector/install` | Установка бинарного файла |
| `vector/symlink` | Создание символической ссылки |
| `vector/config` | Деплой конфигурации |
| `vector/service` | Настройка и запуск сервиса |
| `vector/cleanup` | Очистка временных файлов |

## Требования

- **Ansible** >= 2.9
- **Python** >= 3.6 на целевых хостах
- **Docker** (для тестирования) или реальные серверы
- **Поддерживаемые ОС**:
  - Clickhouse: CentOS 7 / RHEL 7
  - Vector: Ubuntu 20.04 / 22.04

## Установка и запуск

### 1. Клонирование репозитория

    git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
    cd YOUR_REPO_NAME


### 2. Настройка inventory

Отредактируйте файл inventory/prod.yml, указав IP-адреса ваших хостов:

    ---
    all:
      children:
        clickhouse:
          hosts:
            clickhouse-01:
              ansible_host: 192.168.1.10
              ansible_user: root
        vector:
          hosts:
            vector-01:
              ansible_host: 192.168.1.11
              ansible_user: root
              
### 3. Запуск playbook

    # Проверка синтаксиса
    ansible-lint site.yml

    # Проверка без реальных изменений (dry-run)
    ansible-playbook -i inventory/prod.yml site.yml --check

    # Запуск с отображением изменений
    ansible-playbook -i inventory/prod.yml site.yml --diff

    # Запуск только Vector
    ansible-playbook -i inventory/prod.yml site.yml --tags "vector" --diff

    # Запуск только конфигурации Vector
    ansible-playbook -i inventory/prod.yml site.yml --tags "vector/config" --diff

## Результаты выполнения   

<img width="902" height="56" alt="image" src="https://github.com/user-attachments/assets/6020070d-a5a3-4016-b710-fad697a062c1" />

Задание 4: Проверка ansible-lint

<img width="975" height="74" alt="image" src="https://github.com/user-attachments/assets/14655d14-d93c-4303-915b-56f7d180e978" />

Задание 5: Запуск с флагом --check

<img width="937" height="327" alt="image" src="https://github.com/user-attachments/assets/b8e6d884-38ba-49f9-89b7-7006cb02de8c" />

Задание 6: Первый запуск с флагом --diff

<img width="946" height="330" alt="image" src="https://github.com/user-attachments/assets/e6892684-8485-4ffe-a92d-216cf35e227f" />

Задание 7: Повторный запуск с флагом --diff (идемпотентность)

<img width="946" height="330" alt="image" src="https://github.com/user-attachments/assets/c711196d-41ca-4a6e-9617-4397bf4d5a47" />

Задание 8: Запуск с тегами

<img width="946" height="330" alt="image" src="https://github.com/user-attachments/assets/dc22a07f-512f-42f5-a095-0d2a8a944dd7" />

