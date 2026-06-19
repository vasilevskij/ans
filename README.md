# Реализация плейбука Ansible для настройки межсетевых экранов UFW и Firewalld

ФИО: Василевский Артём Валентинович

Специальность:  Компьютерная безопасность (математические методы и программные системы) 

Группа: СДП-КБ-221


Автоматизированная настройка файерволов на Linux:
- **UFW** — Debian / Ubuntu
- **Firewalld** — RHEL / Rocky Linux / AlmaLinux / fedora

ОС определяется автоматически по `ansible_os_family`.

---

## Структура

```
vasilevskij-ans/
├── playbook.yml
├── requirements.yml
├── Vagrantfile
├── inventory/
│   ├── production.yml
│   └── staging.yml
├── group_vars/
│   └── all.yml
├── host_vars/
│   └── example-server.yml
└── roles/
    ├── firewall_common/   ← определение ОС, валидация, бэкап
    ├── ufw/               ← настройка UFW
    ├── firewalld/         ← настройка Firewalld
    └── firewall_security/ ← sysctl + SSH hardening
```

---

## Быстрый старт

```bash
# 1. Установить зависимости
ansible-galaxy collection install -r requirements.yml

# 2. Проверить синтаксис
ansible-playbook playbook.yml --syntax-check

# 3. Dry-run на staging
ansible-playbook playbook.yml -i inventory/staging.yml --check -v

# 4. Применить
ansible-playbook playbook.yml -i inventory/production.yml -v
```

---

## Ключевые переменные (`group_vars/all.yml`)

| Переменная | По умолчанию | Описание |
|-----------|-------------|---------|
| `firewall_default_in` | `deny` | Политика входящего трафика |
| `firewall_default_out` | `allow` | Политика исходящего трафика |
| `firewall_default_zone` | `public` | Зона Firewalld по умолчанию |
| `firewall_rate_limit_ssh` | `true` | Rate limit для SSH |
| `firewall_backup` | `true` | Создавать бэкапы |
| `debug_mode` | `false` | Подробный вывод |
| `firewall_force` | — | Принудительно задать `ufw` или `firewalld` |

---

## Теги

```bash
ansible-playbook playbook.yml --tags backup    # только бэкап
ansible-playbook playbook.yml --tags ufw       # только UFW
ansible-playbook playbook.yml --tags firewalld # только Firewalld
ansible-playbook playbook.yml --tags security  # только sysctl + SSH
```

---

## Тестирование через Vagrant

```bash
vagrant up        # запустить Ubuntu + Rocky
vagrant provision # перезапустить Ansible
vagrant destroy   # удалить тестовые машины
```

---

## Проверка после применения

```bash
# На Debian/Ubuntu
ufw status verbose

# На RHEL/Rocky
firewall-cmd --list-all

# Логи
ls /var/log/ansible-firewall/
```
