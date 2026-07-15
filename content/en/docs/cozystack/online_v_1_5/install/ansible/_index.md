---
title: "Автоматизированная установка с Ansible"
linkTitle: "Автоматизированная установка с Ansible"
description: "Автоматизированная установка с Ansible"
weight: 1
---


# Автоматизированная установка с Ansible

Ansible collection [cozystack.installer](https://github.com/cozystack/ansible-cozystack) автоматизирует весь pipeline развертывания: подготовку ОС, bootstrap кластера k3s и установку Cozystack. Она подходит для развертывания Cozystack на bare-metal серверах или ВМ со стандартным дистрибутивом Linux.

## Когда использовать Ansible

Рассмотрите этот подход, если:
* Вам нужно полностью автоматизированное, повторяемое развертывание от чистой ОС до работающего Cozystack
* Вы развертываете платформу на generic Linux (Ubuntu, Debian, RHEL, Rocky, openSUSE), а не на Talos Linux
* Вы хотите управлять несколькими узлами через один inventory-файл

Шаги ручной установки без Ansible см. в руководстве [Generic Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/generic/).

## Предварительные требования

### Управляющая машина

* Python >= 3.9
* Ansible >= 2.15

### Целевые узлы

* **Операционная система**: Ubuntu/Debian, RHEL 8+/CentOS Stream 8+/Rocky/Alma или openSUSE/SLE
* **Архитектура**: amd64 или arm64
* **SSH-доступ** с passwordless sudo
* Требования к CPU, RAM и дискам см. в [требованиях к оборудованию](https://cozystack.ru/docs/v1.5/install/hardware-requirements/)

## Установка

### 1. Установка Ansible collection

```bash
ansible-galaxy collection install git+https://github.com/cozystack/ansible-cozystack.git
```

Установите необходимые dependency collections. Файл `requirements.yml` не включен в packaged collection, поэтому скачайте его из репозитория:

```bash
curl --silent --location --output /tmp/requirements.yml \
  https://raw.githubusercontent.com/cozystack/ansible-cozystack/main/requirements.yml
ansible-galaxy collection install --requirements-file /tmp/requirements.yml
```

Это установит следующие зависимости:
* `ansible.posix`, `community.general`, `kubernetes.core` — из Ansible Galaxy
* [k3s.orchestration](https://github.com/k3s-io/k3s-ansible) — collection для развертывания k3s, устанавливается из Git

### 2. Создание inventory

Создайте файл `inventory.yml`. Внутренний (private) IP каждого узла должен использоваться как host key, потому что KubeOVN проверяет IP хостов через `NODE_IPS`. Public IP, если он отличается, указывается в `ansible_host`.

```yaml
cluster:
  children:
    server:
      hosts:
        10.0.0.10:
          ansible_host: 203.0.113.10
    agent:
      hosts:
        10.0.0.11:
          ansible_host: 203.0.113.11
        10.0.0.12:
          ansible_host: 203.0.113.12
  vars:
    ansible_port: 22
    ansible_user: ubuntu

    # настройки k3s — доступные версии см. на https://github.com/k3s-io/k3s/releases
    k3s_version: v1.35.0+k3s3
    token: # ЗАМЕНИТЕ на надежный случайный secret
    api_endpoint: 
    cluster_context: my-cluster

    # настройки Cozystack
    cozystack_api_server_host: 
    cozystack_root_host: 
    cozystack_platform_variant: 
    # cozystack_k3s_extra_args:
```

Замените `token` на надежный случайный secret. Этот token используется для подключения узлов k3s и дает полный доступ к кластеру. Сгенерируйте его командой `openssl rand -hex 32`.

Всегда явно закрепляйте `cozystack_chart_version`. Collection поставляется с версией по умолчанию, которая может не совпадать с релизом, который вы хотите развернуть. Укажите ее в inventory, чтобы избежать неожиданных обновлений:

```yaml
cozystack_chart_version: "1.5.0"
```

Доступные версии см. в [релизах Cozystack](https://github.com/cozystack/cozystack/releases).

### 3. Создание playbook

Создайте файл `site.yml`, который последовательно запускает подготовку ОС, развертывание k3s и установку Cozystack.

Репозиторий collection содержит примеры prepare playbooks для каждого поддерживаемого семейства ОС в каталоге [examples/](https://github.com/cozystack/ansible-cozystack/tree/main/examples). Скопируйте подходящий для вашей целевой ОС файл в каталог проекта, затем подключите его как локальный playbook.

Скопируйте `prepare-ubuntu.yml` из [examples/ubuntu/](https://github.com/cozystack/ansible-cozystack/tree/main/examples/ubuntu), затем создайте `site.yml`:

```yaml
- name: Prepare nodes
  ansible.builtin.import_playbook: prepare-ubuntu.yml

- name: Deploy k3s cluster
  ansible.builtin.import_playbook: k3s.orchestration.site

- name: Install Cozystack
  ansible.builtin.import_playbook: cozystack.installer.site
```

### 4. Запуск playbook

```bash
ansible-playbook --inventory inventory.yml site.yml
```

Playbook автоматически выполняет следующие шаги:

1. **Prepare nodes** — устанавливает необходимые пакеты (`nfs-common`, `open-iscsi`, `multipath-tools`), настраивает sysctl, включает storage services
2. **Deploy k3s** — выполняет bootstrap кластера k3s с настройками, совместимыми с Cozystack (отключает встроенные Traefik, ServiceLB, kube-proxy, Flannel; задает `cluster-domain=cozy.local`)
3. **Install Cozystack** — устанавливает Helm и plugin helm-diff (используется для идемпотентных обновлений), развертывает chart `cozy-installer`, ожидает operator и CRD, затем создает Platform Package

## Справочник конфигурации

### Основные переменные

| Переменная | По умолчанию | Описание |
| --- | --- | --- |
| cozystack_api_server_host | (обязательно) | Внутренний IP узла control plane. |
| cozystack_chart_version | 1.5.0 | Версия Helm chart Cozystack. **Закрепляйте явно.** |
| cozystack_platform_variant | isp-full-generic | Platform variant: `default`, `isp-full`, `isp-hosted`, `isp-full-generic`. |
| cozystack_root_host | "" | Домен для сервисов Cozystack. Оставьте пустым, чтобы пропустить publishing configuration. |

### Сеть

| Переменная | По умолчанию | Описание |
| --- | --- | --- |
| cozystack_pod_cidr | 10.42.0.0/16 | Диапазон Pod CIDR. |
| cozystack_pod_gateway | 10.42.0.1 | Gateway pod-сети. |
| cozystack_svc_cidr | 10.43.0.0/16 | Диапазон Service CIDR. |
| cozystack_join_cidr | 100.64.0.0/16 | Join CIDR для межузлового взаимодействия. |
| cozystack_api_server_port | 6443 | Порт Kubernetes API server. |

### Расширенные настройки

| Переменная | По умолчанию | Описание |
| --- | --- | --- |
| cozystack_chart_ref | oci://ghcr.io/cozystack/cozystack/cozy-installer | OCI reference для Helm chart. |
| cozystack_operator_variant | generic | Operator variant: `generic`, `talos`, `hosted`. |
| cozystack_namespace | cozy-system | Namespace для Cozystack |
| cozystack_release_name | cozy-installer | |
| cozystack_release_namespace | kube-system | |
| cozystack_kubeconfig | /etc/rancher/k3s/k3s.yaml | |
| cozystack_create_platform_package | true | |
| cozystack_helm_version | 3.17.3 | |
| cozystack_helm_binary | /usr/local/bin/helm | |
| cozystack_helm_diff_version | 3.12.5 | |
| cozystack_operator_wait_timeout | 300 | |

### Переменные prepare playbook

Примерные prepare playbooks (скопированные из каталога `examples/`) поддерживают дополнительные переменные:

| Переменная | По умолчанию | Описание |
| --- | --- | --- |
| cozystack_flush_iptables | false | |
| cozystack_k3s_extra_args | | Дополнительные аргументы для k3s server (например, `--tls-san=<PUBLIC_IP>` для узлов за NAT). |

## Проверка

После завершения playbook проверьте развертывание с первого server-узла:

```bash
# Проверить operator
kubectl get deployment cozystack-operator --namespace cozy-system

# Проверить Platform Package
kubectl get packages.cozystack.io cozystack.cozystack-platform

# Проверить все pods
kubectl get pods --all-namespaces
```

## Идемпотентность

Playbook идемпотентен: повторный запуск не будет повторно применять ресурсы, которые не изменились. Cozystack operator использует Helm для управления ресурсами, и Ansible collection использует `helm-diff`, чтобы видеть изменения перед их применением, аналогично `kubectl diff`.
