---
title: "Установка и настройка Cozystack"
linkTitle: "Установка и настройка Cozystack"
description: "Установка и настройка Cozystack"
weight: 3
---

# 3. Установка и настройка Cozystack

## Цели

Это руководство описывает установку Cozystack как **готовой к использованию платформы**. Если вы хотите собрать собственную платформу, устанавливая только нужные компоненты, см. [руководство BYOP (Build Your Own Platform)](https://cozystack.ru/docs/v1.5/install/cozystack/kubernetes-distribution/).

На этом шаге мы установим Cozystack поверх [Kubernetes-кластера, подготовленного на предыдущем шаге](https://cozystack.ru/docs/v1.5/getting-started/install-kubernetes/).

Руководство проведёт вас через следующие этапы:
1. Установка оператора Cozystack
2. Подготовка файла конфигурации Cozystack и его применение
3. Настройка хранилища
4. Настройка сети
5. Развёртывание etcd, ingress и стека мониторинга в корневом tenant’е
6. Завершение развёртывания и вход в дашборд Cozystack

## 1. Установка оператора Cozystack

Установите оператор Cozystack с помощью Helm chart из OCI-реестра. Оператор управляет всеми компонентами платформы.

```bash
helm upgrade --install cozystack oci://ghcr.io/cozystack/cozystack/cozy-installer \
  --version X.Y.Z \
  --namespace cozy-system
```

Замените `X.Y.Z` на нужную версию Cozystack. Доступные версии перечислены на [странице релизов Cozystack](https://github.com/cozystack/cozystack/releases).

Если установка прерывается из-за того, что `cozy-system` уже существует:
Helm отказывается забирать себе namespace, который создал не он, и выводит ошибку `invalid ownership metadata` (или `namespaces` в зависимости от версии Helm), если `cozy-system` остался после ранее прерванной установки или был создан вручную для этой цели.

Если namespace **не** управляется другим инструментом (Terraform, Argo CD, другим Helm release и т. д.), повторите команду с флагом `--take-ownership` (требуется Helm 3.17+), чтобы Helm смог принять его под управление:

```bash
helm upgrade --install cozystack oci://ghcr.io/cozystack/cozystack/cozy-installer \
  --version X.Y.Z \
  --namespace cozy-system \
  --create-namespace \
  --take-ownership
```

> [!CAUTION]
> Не используйте `--take-ownership`, если `cozy-system` уже принадлежит другой системе — Helm незаметно станет новым владельцем, а последующие обновления или удаление релиза Cozystack могут изменить или удалить namespace (и всё, что было принято под управление этим флагом) вопреки ожиданиям этой системы.

## 2. Подготовка и применение Platform Package

### 2.1. Подготовьте файл конфигурации

Теперь, когда оператор запущен, подготовим для него файл конфигурации. Возьмите пример ниже и сохраните его в файл `cozystack-platform.yaml`:

```yaml
apiVersion: cozystack.io/v1alpha1
kind: Package
metadata:
  name: cozystack.cozystack-platform
spec:
  variant: isp-full
  components:
    platform:
      values:
        publishing:
          host: example.org
          apiServerEndpoint: api.example.org
          exposedServices:
          - dashboard
          - api
        networking:
          podCIDR: 10.244.0.0/16
          podGateway: 10.244.0.1
          serviceCIDR: 10.96.0.0/12
          joinCIDR: 100.64.0.0/16
```

**Что нужно сделать:**
1. Замените `example.org` в `publishing.host` и `publishing.apiServerEndpoint` на маршрутизируемое полное доменное имя (FQDN), которым вы управляете. Если у вас есть только публичные IP-адреса, вы можете использовать сервис [nip.io](https://nip.io/) с dash-нотацией.
2. Используйте для `networking.*` те же значения, что и на предыдущем шаге, где вы выполняли bootstrap Kubernetes-кластера через Talos. Чтобы узнать эти значения, вы можете проверить конфигурационные файлы Talos или выполнить команды `talosctl`. Значения из примера — это разумные значения по умолчанию, которые подходят для большинства случаев.

В этой конфигурации есть и другие значения, которые в рамках руководства менять не требуется. Тем не менее, вот что они означают:
- `metadata.name` должен быть равен `cozystack.cozystack-platform`, чтобы совпасть с PackageSource, созданным установщиком.
- `publishing.host` используется как основной домен для всех сервисов, создаваемых в Cozystack, например дашборда, Grafana и т. д.
- `publishing.apiServerEndpoint` — это endpoint Cluster API. Он используется для генерации kubeconfig-файлов для пользователей. Рекомендуется использовать поддомен от `publishing.host`.
- `spec.variant: isp-full` означает, что используется наиболее полный набор компонентов Cozystack. Подробнее о вариантах см. в [справочнике по вариантам Cozystack](https://cozystack.ru/docs/v1.5/operations/configuration/variants/).
- `publishing.exposedServices` перечисляет сервисы, которые должны быть доступны пользователям — здесь это дашборд (UI) и API.
- `networking.*` — это внутренняя сетевая конфигурация базового Kubernetes-кластера:
  - `networking.podCIDR` — диапазон CIDR, из которого Kube-OVN выделяет IP-адреса pod’ам. Он не должен пересекаться ни с одной физической сетью.
  - `networking.podGateway` — адрес шлюза, который Kube-OVN назначает подсети pod’ов по умолчанию. Используйте адрес `.1` сети `podCIDR` (например, `10.244.0.1` для `10.244.0.0/16`).
  - `networking.serviceCIDR` — диапазон CIDR для сервисов `ClusterIP`. Он **обязательно** должен совпадать со значением `cluster.network.serviceSubnets`, которое вы использовали при bootstrap Kubernetes-кластера: это значение вшивается в kube-apiserver и не может быть легко изменено позже.
  - `networking.joinCIDR` — диапазон CIDR для `join` подсети Kube-OVN, внутренней сети, которая переносит трафик между узлами кластера и pod’ами. Значение по умолчанию `100.64.0.0/16` входит в общее адресное пространство [RFC 6598](https://datatracker.ietf.org/doc/html/rfc6598) (`100.64.0.0/10`), зарезервированное для такого внутреннего использования. Меняйте его только если оно пересекается с вашей физической сетью. Подробнее см. в [справочнике по Kube-OVN join subnet](https://kubeovn.github.io/docs/stable/en/guide/subnet/#join-subnet).

Подробнее об этом файле конфигурации см. в [справочнике Platform Package](https://cozystack.ru/docs/v1.5/operations/configuration/platform-package/).

По умолчанию Cozystack собирает анонимную статистику использования. Подробнее о том, какие данные собираются и как это отключить, см. в [документации по телеметрии](https://cozystack.ru/docs/v1.5/operations/configuration/telemetry/).

### 2.2. Примените Platform Package

Примените файл конфигурации:
```bash
kubectl apply -f cozystack-platform.yaml
```

Во время установки можно следить за логами оператора:
```bash
kubectl logs -n cozy-system -l app.kubernetes.io/name=cozy-installer -f
```

### 2.3. Проверьте статус установки

Подождите немного, затем проверьте статус установки:
```bash
kubectl get packages.cozystack.io cozystack.cozystack-platform
```

Повторяйте проверку, пока во всех строках не появится `True`, как в этом примере:
```bash
kubectl get cozystack -A
```

Пример вывода:
```text
cozy-cert-manager                cert-manager                4m1s   True    Release reconciliation succeeded
cozy-cert-manager                cert-manager-issuers        4m1s   True    Release reconciliation succeeded
cozy-cilium                      cilium                      4m1s   True    Release reconciliation succeeded
cozy-cluster-api                 capi-operator               4m1s   True    Release reconciliation succeeded
cozy-cluster-api                 capi-providers              4m1s   True    Release reconciliation succeeded
cozy-dashboard                   dashboard                   4m1s   True    Release reconciliation succeeded
cozy-grafana-operator            grafana-operator            4m1s   True    Release reconciliation succeeded
cozy-kamaji                      kamaji                      4m1s   True    Release reconciliation succeeded
cozy-kubeovn                     kubeovn                     4m1s   True    Release reconciliation succeeded
cozy-kubevirt-cdi                kubevirt-cdi                4m1s   True    Release reconciliation succeeded
cozy-kubevirt-cdi                kubevirt-cdi-operator       4m1s   True    Release reconciliation succeeded
cozy-kubevirt                    kubevirt                    4m1s   True    Release reconciliation succeeded
cozy-kubevirt                    kubevirt-operator           4m1s   True    Release reconciliation succeeded
cozy-linstor                     linstor                     4m1s   True    Release reconciliation succeeded
cozy-linstor                     piraeus-operator            4m1s   True    Release reconciliation succeeded
cozy-mariadb-operator            mariadb-operator            4m1s   True    Release reconciliation succeeded
cozy-metallb                     metallb                     4m1s   True    Release reconciliation succeeded
cozy-monitoring                  monitoring                  4m1s   True    Release reconciliation succeeded
cozy-postgres-operator           postgres-operator           4m1s   True    Release reconciliation succeeded
cozy-rabbitmq-operator           rabbitmq-operator           4m1s   True    Release reconciliation succeeded
cozy-redis-operator              redis-operator              4m1s   True    Release reconciliation succeeded
cozy-telepresence                telepresence                4m1s   True    Release reconciliation succeeded
cozy-victoria-metrics-operator   victoria-metrics-operator   4m1s   True    Release reconciliation succeeded
tenant-root                      tenant-root                 4m1s   True    Release reconciliation succeeded
```

Список компонентов в вашей установке может отличаться от приведённого выше, так как он зависит от выбранного варианта и версии Cozystack.
Когда у всех компонентов появится `READY: True`, можно переходить к настройке подсистем.

## 3. Настройка хранилища

Kubernetes нужен слой хранилища для предоставления persistent volumes приложениям, но собственного такого механизма у него нет. В качестве подсистемы хранения Cozystack использует [LINSTOR](https://github.com/LINBIT/linstor-server).
Далее мы получим доступ к интерфейсу LINSTOR, создадим storage pool’ы и определим storage class’ы.

### 3.1. Проверьте устройства хранения

Настройте alias для доступа к LINSTOR:
```bash
alias linstor='kubectl exec -n cozy-linstor deploy/linstor-controller -- linstor'
```

Выведите список узлов и проверьте их готовность:
```bash
linstor node list
```

Выведите список доступных пустых устройств:
```bash
linstor physical-disk list
```

### 3.2. Создайте storage pool’ы

Создайте storage pool’ы на основе ZFS:
```bash
linstor storage-pool create zfs srv1 data data
linstor storage-pool create zfs srv2 data data
linstor storage-pool create zfs srv3 data data
```

[Рекомендуется](https://github.com/LINBIT/linstor-server/issues/463#issuecomment-3401472020) установить для ZFS storage pool’ов `failmode=continue`, чтобы обработкой отказов диска занимался DRBD, а не ZFS.

```bash
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv1 -- zpool set failmode=continue data
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv2 -- zpool set failmode=continue data
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv3 -- zpool set failmode=continue data
```

Проверьте результат, выведя список storage pool’ов:
```bash
linstor storage-pool list
```

Пример вывода:
| StoragePool | Node | Driver | PoolName | FreeCapacity | TotalCapacity | CanSnapshots |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| DfltDisklessStorPool | srv1 | DISKLESS | | | | False |
| DfltDisklessStorPool | srv2 | DISKLESS | | | | False |
| DfltDisklessStorPool | srv3 | DISKLESS | | | | False |
| data | srv1 | ZFS | data | 96.41 GiB | 99.50 GiB | True |
| data | srv2 | ZFS | data | 96.41 GiB | 99.50 GiB | True |
| data | srv3 | ZFS | data | 96.41 GiB | 99.50 GiB | True |

### 3.3. Создайте storage class’ы

Наконец, можно создать несколько storage class’ов, один из которых будет классом по умолчанию.
Создайте файл с описанием storage class’ов. Ниже приведён разумный пример по умолчанию с двумя классами: `local` (по умолчанию) и `replicated`.

`storageclasses.yaml`:
```yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: linstor.csi.linbit.com
parameters:
  linstor.csi.linbit.com/storagePool: data
  linstor.csi.linbit.com/layerList: storage
  linstor.csi.linbit.com/allowRemoteVolumeAccess: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: replicated
provisioner: linstor.csi.linbit.com
parameters:
  linstor.csi.linbit.com/storagePool: data
  linstor.csi.linbit.com/autoPlace: "3"
  linstor.csi.linbit.com/layerList: drbd storage
  linstor.csi.linbit.com/allowRemoteVolumeAccess: "true"
  property.linstor.csi.linbit.com/DrbdOptions/auto-quorum: suspend-io
  property.linstor.csi.linbit.com/DrbdOptions/Resource/on-no-data-accessible: suspend-io
  property.linstor.csi.linbit.com/DrbdOptions/Resource/on-suspended-primary-outdated: force-secondary
  property.linstor.csi.linbit.com/DrbdOptions/Net/rr-conflict: retry-connect
volumeBindingMode: Immediate
allowVolumeExpansion: true
```

Примените конфигурацию storage class’ов:
```bash
kubectl apply -f storageclasses.yaml
```

Убедитесь, что storage class’ы были успешно созданы:
```bash
kubectl get storageclasses
```

Пример вывода:
| NAME | PROVISIONER | RECLAIMPOLICY | VOLUMEBINDINGMODE | ALLOWVOLUMEEXPANSION |
| :--- | :--- | :--- | :--- | :--- |
| local (default) | linstor.csi.linbit.com | Delete | WaitForFirstConsumer | true |
| replicated | linstor.csi.linbit.com | Delete | Immediate | true |

## 4. Настройка сети

Далее мы настроим способ доступа к кластеру Cozystack. На этом шаге есть два варианта в зависимости от доступной инфраструктуры:
- Для собственного bare metal или self-hosted ВМ выбирайте вариант с MetalLB. MetalLB — это балансировщик нагрузки по умолчанию в Cozystack.
- Для ВМ и выделенных серверов у облачных провайдеров выбирайте настройку через публичные IP. [Большинство облачных провайдеров не поддерживают MetalLB](https://metallb.universe.tf/installation/clouds/).

Загляните в раздел [provider-specific installation](https://cozystack.ru/docs/v1.5/install/providers/). Возможно, там уже есть инструкции для вашего провайдера, подходящие для развёртывания production-ready кластера.

### 4.a Настройка MetalLB

В Cozystack используются три типа IP-адресов:
1. IP-адреса узлов: постоянные и действуют только внутри кластера.
2. Виртуальный floating IP: используется для доступа к одному из узлов кластера и также действует только внутри кластера.
3. IP-адреса внешнего доступа: используются LoadBalancer’ами для публикации сервисов за пределами кластера.

Сервисы с внешними IP могут публиковаться в двух режимах: L2 и BGP. Режим L2 проще, но требует, чтобы все узлы находились в одном L2-сегменте. Режим BGP более надёжен и масштабируем, но требует поддержки BGP со стороны сетевого оборудования.

Выберите диапазон неиспользуемых IP-адресов для сервисов; в примере используется диапазон `192.168.100.200-192.168.100.250`. Если вы используете режим L2, эти IP должны либо принадлежать той же сети, что и узлы, либо до них должен быть настроен проброс трафика (маршрутизация).

Для режима BGP также потребуются IP-адреса BGP peer’ов и локальные и удалённые номера AS. В примере ниже используется `192.168.20.254` как IP peer’а и номера AS 65000 и 65001 как локальный и удалённый соответственно.

Создайте и примените файл с описанием пула адресов `metallb-ip-address-pool.yml`:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: cozystack
  namespace: cozy-metallb
spec:
  addresses:
  # используется для публикации сервисов за пределами кластера
  - 192.168.100.200-192.168.100.250
  autoAssign: true
  avoidBuggyIPs: false
```

```bash
kubectl apply -f metallb-ip-address-pool.yml
```

Создайте и примените ресурсы, необходимые для L2- или BGP-анонса.
L2Advertisement использует имя ресурса IPAddressPool, который мы создали на предыдущем шаге.

`metallb-l2-advertisement.yml`:
```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: cozystack
  namespace: cozy-metallb
spec:
  ipAddressPools:
  - cozystack
```

Примените изменения:
```bash
kubectl apply -f metallb-l2-advertisement.yml
```

Теперь, когда MetalLB настроен, включите `ingress` в `tenant-root`:
```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"components":{"ingress":{"enabled":true}}}}'
```

Чтобы убедиться, что всё настроено корректно, проверьте HelmRelease’ы `ingress` и `ingress-nginx-system`:
```bash
kubectl get hr -n tenant-root
```

Пример корректного вывода:
```text
ingress                47m   True    Helm upgrade succeeded for release tenant-root/ingress.v3 with...
ingress-nginx-system   47m   True    Helm upgrade succeeded for release tenant-root/ingress-nginx-s...
```

Затем проверьте состояние сервиса `root-ingress-controller`:
```bash
kubectl get svc -n tenant-root root-ingress-controller
```

Сервис должен быть развёрнут как `TYPE: LoadBalancer` и иметь корректный внешний IP:
| NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) |
| :--- | :--- | :--- | :--- | :--- |
| root-ingress-controller | LoadBalancer | 10.96.16.141 | 192.168.100.200 | 80:31632/TCP, 443:30113/TCP |

### 4.b. Настройка публичных IP узлов

Если ваш облачный провайдер не поддерживает MetalLB, вы можете опубликовать ingress controller через `externalIPs` напрямую.
Если публичные IP привязаны непосредственно к узлам, укажите их. Если публичные IP предоставляются облаком как виртуальные (Elastic IP), укажите IP-адреса **внешних** сетевых интерфейсов.
В примере используются `192.168.100.11`, `192.168.100.12` и `192.168.100.13`.

Сначала добавьте внешние IP в Platform Package:
```bash
kubectl patch packages.cozystack.io cozystack.cozystack-platform --type=merge -p '{"spec":{"components":{"platform":{"values":{"publishing":{"externalIPs":["192.168.100.11","192.168.100.12","192.168.100.13"]}}}}}}'
```

Затем включите `ingress` для корневого tenant’а:
```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"components":{"ingress":{"enabled":true}}}}'
```

Наконец, добавьте внешние IP-адреса в список `externalIPs` в конфигурации Ingress:
```bash
kubectl patch -n tenant-root ingresses.apps.cozystack.io ingress --type=merge -p '{"spec":{"externalIPs":["192.168.100.11","192.168.100.12","192.168.100.13"]}}'
```

После этого ваш Ingress будет доступен по указанным IP-адресам. Проверьте это так:
```bash
kubectl get svc -n tenant-root root-ingress-controller
```

Сервис должен быть развёрнут как `TYPE: ClusterIP` и содержать полный список внешних IP:
| NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) |
| :--- | :--- | :--- | :--- | :--- |
| root-ingress-controller | ClusterIP | 10.96.91.83 | 192.168.100.11, 192.168.100.12, 192.168.100.13 | 80/TCP, 443/TCP |

## 5. Завершение установки

### 5.1. Настройте сервисы корневого tenant’а

Включите `etcd` и `monitoring` для корневого tenant’а:
```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"components":{"monitoring":{"enabled":true},"etcd":{"enabled":true}}}}'
```

### 5.2. Проверьте состояние и состав кластера

Проверьте созданные persistent volume’ы:
```bash
kubectl get pvc -A
```

Пример вывода:
| NAME | STATUS | VOLUME | CAPACITY | ... |
| :--- | :--- | :--- | :--- | :--- |
| data-etcd-0 | Bound | pvc-4cbd29cc-a29f-453d-b412-451647cd04bf | 10Gi | ... |
| data-etcd-1 | Bound | pvc-1579f95a-a69d-4a26-bcc2-b15ccdbede0d | 10Gi | ... |
| data-etcd-2 | Bound | pvc-907009e5-88bf-4d18-91e7-b56b0dbfb97e | 10Gi | ... |
| grafana-db-1 | Bound | pvc-7b3f4e23-228a-46fd-b820-d033ef4679af | 10Gi | ... |
| grafana-db-2 | Bound | pvc-ac9b72a4-f40e-47e8-ad24-f50d843b55e4 | 10Gi | ... |
| vmselect-cachedir-vmselect-longterm-0 | Bound | pvc-622fa398-2104-459f-8744-565eee0a13f1 | 2Gi | ... |
| vmselect-cachedir-vmselect-longterm-1 | Bound | pvc-fc9349f5-02b2-4e25-8bef-6cbc5cc6d690 | 2Gi | ... |
| vmselect-cachedir-vmselect-shortterm-0 | Bound | pvc-7acc7ff6-6b9b-4676-bd1f-6867ea7165e2 | 2Gi | ... |
| vmselect-cachedir-vmselect-shortterm-1 | Bound | pvc-e514f12b-f1f6-40ff-9838-a6bda3580eb7 | 2Gi | ... |
| vmstorage-db-vmstorage-longterm-0 | Bound | pvc-e8ac7fc3-df0d-4692-aebf-9f66f72f9fef | 10Gi | ... |
| vmstorage-db-vmstorage-longterm-1 | Bound | pvc-68b5ceaf-3ed1-4e5a-9568-6b95911c7c3a | 10Gi | ... |
| vmstorage-db-vmstorage-shortterm-0 | Bound | pvc-cee3a2a4-5680-4880-bc2a-85c14dba9380 | 10Gi | ... |
| vmstorage-db-vmstorage-shortterm-1 | Bound | pvc-d55c235d-cada-4c4a-8299-e5fc3f161789 | 10Gi | ... |

Убедитесь, что все pod’ы запущены:
```bash
kubectl get pod -n tenant-root
```

пример вывода:
| NAME | READY | STATUS | RESTARTS | AGE |
| :--- | :--- | :--- | :--- | :--- |
| etcd-0 | 1/1 | Running | 0 | 2m1s |
| etcd-1 | 1/1 | Running | 0 | 106s |
| etcd-2 | 1/1 | Running | 0 | 82s |
| grafana-db-1 | 1/1 | Running | 0 | 119s |
| grafana-db-2 | 1/1 | Running | 0 | 13s |
| grafana-deployment-74b5656d6-5dcvn | 1/1 | Running | 0 | 90s |
| grafana-deployment-74b5656d6-q5589 | 1/1 | Running | 1 (105s ago) | 111s |

Получите публичный IP ingress controller:
```bash
kubectl get svc -n tenant-root root-ingress-controller
```

Пример вывода:
| NAME | TYPE | CLUSTER-IP | EXTERNAL-IP | PORT(S) |
| :--- | :--- | :--- | :--- | :--- |
| root-ingress-controller | LoadBalancer | 10.96.16.141 | 192.168.100.200 | 80:31632/TCP, 443:30113/TCP |

### 5.3 Доступ к дашборду Cozystack

Если вы включили `dashboard` в список `publishing.exposedServices` вашего Platform Package (как показано на шаге 2), дашборд Cozystack уже доступен.
Если в исходной конфигурации его не было, обновите Platform Package:

```bash
kubectl patch packages.cozystack.io cozystack.cozystack-platform --type=json -p '[{"op":"add","path":"/spec/components/platform/values/publishing/exposedServices/-","value":"dashboard"}]'
```

Откройте `dashboard.example.org`, чтобы перейти в системный дашборд, где `example.org` — домен, указанный вами для `tenant-root`. Там вы увидите окно входа, ожидающее токен аутентификации.

Получите токен аутентификации для `tenant-root`:
```bash
kubectl get secret -n tenant-root tenant-root -o go-template='{{ printf "%s\n" .data.token | base64decode }}'
```

Войдите с этим токеном. Теперь вы можете пользоваться дашбордом как администратор.
Дальше вы сможете:
- Настроить OIDC и использовать его вместо токенов для аутентификации.
- Создавать пользовательские tenant’ы и выдавать пользователям доступ через токены или OIDC.

### 5.4 Доступ к метрикам в Grafana

Используйте `grafana.example.org` для доступа к системному мониторингу, где `example.org` — домен, указанный для `tenant-root`. В этом примере `grafana.example.org` доступен по адресу 192.168.100.200.

логин: `admin`
получите пароль:
```bash
kubectl get secret -n tenant-root grafana-admin-password -o go-template='{{ printf "%s\n" .data.password | base64decode }}'
```

## Следующий шаг

Продолжите руководство по Cozystack и [создайте пользовательский tenant](https://cozystack.ru/docs/v1.5/getting-started/create-tenant/).