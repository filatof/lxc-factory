# Proxmox LXC Cluster

Ansible-плейбук для автоматического создания кластера LXC-контейнеров в Proxmox.

## Описание

Плейбук клонирует базовый LXC-контейнер и создает на его основе заданное количество контейнеров с автоматической настройкой:
- Уникальные VMID
- Последовательные IP-адреса
- Именование по шаблону
- Разные размеры дисков (первый контейнер — 20GB, остальные — 70GB)

## Структура проекта

```
.
├── group_vars/
│   ├── prod.yml          # Переменные для production
│   └── stage.yml         # Переменные для staging
├── inventory.yml         # Инвентарь Proxmox-хостов
├── playbooks/
│   ├── create_cluster.yml
│   └── tasks/
│       └── create_one_lxc.yml
└── README.md
```

## Требования

- Ansible с коллекцией `community.general`
- Доступ к Proxmox API
- Базовый LXC-контейнер для клонирования

## Настройка переменных

Основные параметры в `group_vars/{env}.yml`:

| Параметр | Описание |
|----------|----------|
| `count` | Количество создаваемых контейнеров |
| `vmid_start` | Начальный VMID (например, 7710) |
| `ip_start` | Последний октет начального IP (например, 201 → 192.168.1.201) |
| `hostname_prefix` | Префикс имени хоста (например, k8s-prod) |
| `lxc_id` | VMID базового контейнера для клонирования |
| `gateway` | Шлюз сети |
| `bridge` | Сетевой мост Proxmox |
| `storage` | Хранилище для дисков |
| `proxmox_node` | Имя узла Proxmox |

## Запуск

### Stage окружение
```bash
ansible-playbook -i inventory.yml playbooks/create_cluster.yml -e "env=stage" --ask-vault-pass
```

### Production окружение
```bash
ansible-playbook -i inventory.yml playbooks/create_cluster.yml -e "env=prod" --ask-vault-pass
```

## Пример результата

При запуске с `count: 4` и `ip_start: 201` будут созданы:
- k8s-prod-01 (VMID: 7710, IP: 192.168.1.201, Disk: 20GB)
- k8s-prod-02 (VMID: 7711, IP: 192.168.1.202, Disk: 70GB)
- k8s-prod-03 (VMID: 7712, IP: 192.168.1.203, Disk: 70GB)
- k8s-prod-04 (VMID: 7713, IP: 192.168.1.204, Disk: 70GB)

## Безопасность

Пароли хранятся в зашифрованном виде с помощью Ansible Vault. Для расшифровки необходим vault-пароль.