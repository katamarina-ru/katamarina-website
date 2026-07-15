# Требования и набор инструментов

## Набор инструментов

На вашей рабочей станции должны быть установлены следующие инструменты:

* [talosctl](https://www.talos.dev/v1.13/talos-guides/install/talosctl/), клиент командной строки для Talos Linux (используйте серию v1.13.x, соответствующую Cozystack 1.5.0).
* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl), клиент командной строки для Kubernetes.
* [Talm](https://github.com/cozystack/talm?tab=readme-ov-file#installation), собственный менеджер конфигурации Talos Linux от Cozystack:

```bash
curl -sSL https://github.com/cozystack/talm/raw/refs/heads/main/hack/install.sh | sh -s
```

## Требования к оборудованию

Для прохождения этого руководства вам потребуется следующая конфигурация:

**Узлы кластера:** три bare-metal сервера или виртуальные машины. Требования к оборудованию зависят от вашего сценария использования:

### Минимальная

Ниже приведены базовые требования для запуска небольшой инсталляции. Минимальная рекомендуемая конфигурация для каждого узла:

| Компонент | Требование |
| --- | --- |
| Хосты | 3x физических хоста (или ВМ с host CPU passthrough) |
| Architecture | x86_64 |
| CPU | 8 ядер |
| RAM | 24 ГБ |
| Основной диск | 50 ГБ SSD (или RAW для ВМ) |
| Дополнительный диск | 256 ГБ SSD (raw) |

**Подходит для:**
* Dev/Test-сред
* Небольших демонстрационных конфигураций
* 1-2 tenants
* До 3 кластеров Kubernetes
* Небольшого числа ВМ или баз данных

### Рекомендуемая

Для небольших production-сред рекомендуемая конфигурация каждого узла выглядит так:

| Компонент | Требование |
| --- | --- |
| Хосты | 3x физических хоста |
| Architecture | x86_64 |
| CPU | 16-32 ядра |
| RAM | 64 ГБ |
| Основной диск | 100 ГБ SSD или NVMe |
| Дополнительный диск | 1-2 ТБ SSD или NVMe |

**Подходит для:**
* Небольших и средних production-сред
* 5-10 tenants
* 5+ кластеров Kubernetes
* Десятков виртуальных машин или баз данных
* S3-совместимого хранилища

### Оптимальная

Для средних и крупных production-сред оптимальная конфигурация каждого узла выглядит так:

| Компонент | Требование |
| --- | --- |
| Хосты | 6x+ физических хостов |
| Architecture | x86_64 |
| CPU | 32-64 ядра |
| RAM | 128-256 ГБ |
| Основной диск | 200 ГБ SSD или NVMe |
| Дополнительный диск | 4-10 ТБ NVMe |

**Подходит для:**
* Крупных production-сред
* 20+ tenants
* Десятков кластеров Kubernetes
* Сотен виртуальных машин и баз данных
* S3-совместимого хранилища

**Хранилище:**
* **Основной диск**: используется для Talos Linux, хранилища etcd и загруженных образов. Требуется низкая задержка.
* **Дополнительный диск**: используется для данных пользовательских приложений (ZFS pool).

**ОС:**
* Любой дистрибутив Linux, например Ubuntu.
* Есть и [другие способы установки](https://cozystack.ru/docs/v1.5/install/talos/), для которых на старте требуется либо любой Linux, либо вообще не требуется установленная ОС.

**BIOS/UEFI Settings:**
* **Secure Boot.**
Talos Linux ships pre-signed kernel modules and works with Secure Boot enabled. On non-Talos Ubuntu hosts, the default piraeus-operator flow compiles DRBD in-cluster; the resulting unsigned modules are rejected by kernel lockdown when Secure Boot is enforced. The simplest path is to disable Secure Boot in BIOS/UEFI; alternatively, follow [Ubuntu + Secure Boot](https://cozystack.ru/docs/v1.5/install/kubernetes/ubuntu-secure-boot/) to pre-install dkms-signed DRBD on the host.

**Сеть:**
* Маршрутизируемый домен с FQDN. Если его нет, можно использовать [nip.io](https://nip.io/) с dash-нотацией.
* Узлы должны находиться в одном L2-сегменте сети.
* Anti-spoofing должен быть отключён. Это необходимо для MetalLB — балансировщика нагрузки, используемого в Cozystack.

**Виртуальные машины:**
* В настройках гипервизора должен быть включён CPU passthrough, а модель CPU должна быть установлена в `host`.
* Должна быть включена nested virtualization. Это требуется для виртуальных машин и Kubernetes-кластеров tenant’ов.

Более подробное описание требований к оборудованию для разных сценариев см. в разделе [Требования к оборудованию](https://cozystack.ru/docs/v1.5/install/hardware-requirements/)