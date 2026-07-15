---
title: "Установка Cozystack как платформы"
linkTitle: "Установка Cozystack как платформы"
description: "Установка Cozystack как платформы"
weight: 1
---


# Установка Cozystack как платформы

Третий шаг развертывания кластера Cozystack — установка Cozystack в кластер Kubernetes, ранее установленный и настроенный на узлах Talos Linux. Предварительное условие для этого шага — [установленный кластер Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/).

Если вы устанавливаете Cozystack впервые, рекомендуем начать с [руководства по Cozystack](https://cozystack.ru/docs/v1.5/getting-started/).

Чтобы спланировать production-ready установку, следуйте руководству ниже. По структуре оно повторяет tutorial, но содержит гораздо больше деталей и объясняет разные варианты установки.

## 1. Установка Cozystack Operator

Установите Cozystack operator с помощью Helm из OCI registry:

```bash
helm upgrade --install cozystack oci://ghcr.io/cozystack/cozystack/cozy-installer \
  --version X.Y.Z \
  --namespace cozy-system \
  --create-namespace
```

Замените `X.Y.Z` на нужную версию Cozystack. Доступные версии можно найти на [странице релизов Cozystack](https://github.com/cozystack/cozystack/releases).

Эта команда устанавливает operator, CRD и создает ресурс `PackageSource`.

### Установка на ОС без Talos

По умолчанию Cozystack operator настроен на использование функции [KubePrism](https://www.talos.dev/v1.13/kubernetes-guides/configuration/kubeprism/) из Talos Linux, которая позволяет обращаться к Kubernetes API через локальный адрес на узле.

Если вы устанавливаете Cozystack не на Talos Linux, укажите variant operator во время установки:

```bash
helm upgrade --install cozystack oci://ghcr.io/cozystack/cozystack/cozy-installer \
  --version X.Y.Z \
  --namespace cozy-system \
  --create-namespace \
  --set cozystackOperator.variant=generic \
  --set cozystack.apiServerHost=<YOUR_API_SERVER_IP> \
  --set cozystack.apiServerPort=6443
```

Замените `<YOUR_API_SERVER_IP>` на внутренний IP-адрес вашего Kubernetes API server (только IP, без протокола и порта).

Полное руководство по развертыванию Cozystack на generic-дистрибутивах Kubernetes см. в разделе [Развертывание Cozystack на Generic Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/generic/).

## 2. Описание и применение Platform Package

После запуска operator следующий шаг — описать Platform Package и применить его. Platform Package — это ресурс `Package`, который определяет [variant Cozystack](https://cozystack.ru/docs/v1.5/operations/configuration/variants/), [настройки компонентов](https://cozystack.ru/docs/v1.5/operations/configuration/components/), ключевые сетевые настройки, опубликованные сервисы и другие параметры.

Конфигурацию Cozystack можно обновлять после установки. Однако некоторые значения, показанные ниже, обязательны для установки.

Ниже минимальный пример `cozystack-platform.yaml`:

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
          podCIDR: "10.244.0.0/16"
          podGateway: "10.244.0.1"
          serviceCIDR: "10.96.0.0/16"
          joinCIDR: "100.64.0.0/16"
```

Имя `Package` должно быть `cozystack.cozystack-platform`, чтобы совпадать с PackageSource, созданным installer. Проверить доступные PackageSources можно командой `kubectl get packagesource`.

### 2.1. Выбор variant

Состав Cozystack определяется variant. Variant `isp-full` — самый полный: он охватывает все уровни от оборудования до managed applications. Выбирайте его, если планируете строить публичное или приватное облако.

Если вы развертываете Cozystack в предоставленном Kubernetes-кластере или хотите развернуть только часть компонентов, выберите другой variant. См. [обзор и сравнение variants](https://cozystack.ru/docs/v1.5/operations/configuration/variants/).

### 2.2. Тонкая настройка компонентов

Можно добавить дополнительные компоненты или убрать компоненты, включенные по умолчанию. См. [справочник components](https://cozystack.ru/docs/v1.5/operations/configuration/components/).

Если вы развертываете платформу на ВМ или выделенных серверах облачного провайдера, скорее всего, нужно изменить конфигурацию некоторых компонентов. См. раздел [установка у конкретных провайдеров](https://cozystack.ru/docs/v1.5/install/providers/). В нем может быть полное руководство для вашего провайдера, которое можно использовать для развертывания production-ready кластера.

### 2.3. Описание сетевой конфигурации

Замените `example.org` в `publishing.host` и `publishing.apiServerEndpoint` на маршрутизируемое fully-qualified domain name (FQDN), которым вы управляете. Если у вас есть только IP-адреса, можно использовать сервис [nip.io](https://nip.io/) с записью через дефис.

В следующем разделе приведены разумные значения по умолчанию. Проверьте, что они совпадают с настройками узлов Talos, которые вы использовали на предыдущих шагах. Если вы использовали Talm для установки Kubernetes, значения должны совпадать.

```yaml
        networking:
          podCIDR: "10.244.0.0/16"
          podGateway: "10.244.0.1"
          serviceCIDR: "10.96.0.0/16"
          joinCIDR: "100.64.0.0/16"
```

Cozystack по умолчанию собирает анонимную статистику использования. Подробнее о том, какие данные собираются и как отказаться от сбора, см. в [документации по Telemetry](https://cozystack.ru/docs/v1.5/operations/configuration/telemetry/).

### 2.4. Применение Platform Package

Когда конфигурационный файл будет готов, примените его:

```bash
kubectl apply -f cozystack-platform.yaml
```

Во время установки можно отслеживать логи operator:

```bash
kubectl logs -n cozy-system deploy/cozystack-operator -f
```

Подождите некоторое время, затем проверьте состояние установки:

```bash
kubectl get hr -A
```

Дождитесь, пока все releases перейдут в состояние `Ready`:

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

## Разделение узлов Control Plane и Worker

Обычно Cozystack требует как минимум три worker-узла для запуска workload’ов в HA mode. В компонентах Cozystack нет tolerations, которые позволили бы им запускаться на узлах control plane.

Однако для тестирования часто используется всего три узла. Либо у вас могут быть только крупные аппаратные узлы, и вы хотите использовать их одновременно для control-plane и worker workload’ов. В этом случае нужно удалить control-plane taint с узлов.

Пример удаления control-plane taint с узлов:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

## 3. Настройка хранилища

Kubernetes нужна подсистема хранения, чтобы предоставлять persistent volumes приложениям, но собственной такой подсистемы в Kubernetes нет. Cozystack предоставляет [LINSTOR](https://github.com/LINBIT/linstor-server) как подсистему хранения.

На следующих шагах мы получим доступ к интерфейсу LINSTOR, создадим storage pools и определим storage classes.

### 3.1. Проверка устройств хранения

Настройте alias для доступа к LINSTOR:

```bash
alias linstor='kubectl exec -n cozy-linstor deploy/linstor-controller -- linstor'
```

Выведите список узлов и проверьте их готовность:

```bash
linstor node list
```

Пример вывода показывает имена узлов и состояние:

```text
+--------------------------------------------------------+
| Node | NodeType  | Addresses                | State    |
|========================================================|
| srv1 | SATELLITE | 100.64.0.1:3366 (PLAIN)  | Online   |
| srv2 | SATELLITE | 100.64.0.2:3366 (PLAIN)  | Online   |
| srv3 | SATELLITE | 100.64.0.3:3366 (PLAIN)  | Online   |
+--------------------------------------------------------+
```

Выведите список доступных пустых устройств:

```bash
linstor physical-storage list
```

Пример вывода показывает те же имена узлов:

```text
+----------------------------------------------------------------------------------------------------------------------------------+
| Size          | Rotational | Node | DeviceName | PhysicalStoragePool |
|==================================================================================================================================|
| 100.00 GiB    | False      | srv1 | /dev/nvme0n1 |                     |
| 100.00 GiB    | False      | srv2 | /dev/nvme0n1 |                     |
| 100.00 GiB    | False      | srv3 | /dev/nvme0n1 |                     |
+----------------------------------------------------------------------------------------------------------------------------------+
```

### 3.2. Создание Storage Pools

Создайте storage pools с помощью ZFS или LVM. Также можно восстановить ранее созданные storage pools после reset узла.

```bash
linstor physical-storage create-device-pool --pool-name data ZFS srv1 /dev/nvme0n1
linstor physical-storage create-device-pool --pool-name data ZFS srv2 /dev/nvme0n1
linstor physical-storage create-device-pool --pool-name data ZFS srv3 /dev/nvme0n1
```

[Рекомендуется](https://github.com/LINBIT/linstor-server/issues/463#issuecomment-3401472020) выставить `failmode=continue` для ZFS storage pools чтобы позволить DRBD обрабатывать ошибки диска вместо ZFS.

```bash
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv1 -- zpool set failmode=continue data
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv2 -- zpool set failmode=continue data
kubectl exec -ti -n cozy-linstor ds/linstor-satellite.srv3 -- zpool set failmode=continue data
```

Проверьте результат, выведя список storage pools:

```bash
linstor storage-pool list
```

Пример вывода:

```text
+--------------------------------------------------------------------------------------------------+
| StoragePool          | Node | Driver   | PoolName | FreeCapacity | TotalCapacity | CanSnapshots |
|==================================================================================================|
| DfltDisklessStorPool | srv1 | DISKLESS |          |              |               | False        |
| DfltDisklessStorPool | srv2 | DISKLESS |          |              |               | False        |
| DfltDisklessStorPool | srv3 | DISKLESS |          |              |               | False        |
| data                 | srv1 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         |
| data                 | srv2 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         |
| data                 | srv3 | ZFS      | data     |    96.41 GiB |     99.50 GiB | True         |
+--------------------------------------------------------------------------------------------------+
```

### 3.3. Создание Storage Classes

Создайте storage classes, один из которых должен быть default class.

Создайте файл с определениями storage classes. Ниже приведен разумный пример по умолчанию с двумя классами: `local` (default) и `replicated`.

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

Примените конфигурацию storage class:

```bash
kubectl apply -f storageclasses.yaml
```

Проверьте, что storage classes успешно созданы:

```bash
kubectl get storageclasses
```

Пример вывода:

```text
NAME              PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local (default)   linstor.csi.linbit.com   Delete          WaitForFirstConsumer   true                   1m
replicated        linstor.csi.linbit.com   Delete          Immediate              true                   1m
```

## 4. Настройка сети

Далее настроим доступ к кластеру Cozystack. На этом шаге есть два варианта в зависимости от доступной инфраструктуры:

* Для собственного bare metal или self-hosted ВМ выберите вариант MetalLB. MetalLB — load balancer Cozystack по умолчанию.
* Для ВМ и выделенных серверов у облачных провайдеров выберите настройку public IP. [Большинство облачных провайдеров не поддерживают MetalLB](https://metallb.universe.tf/installation/clouds/).

См. раздел [установка у конкретных провайдеров](https://cozystack.ru/docs/v1.5/install/providers/). Там могут быть инструкции для вашего провайдера, которые можно использовать для развертывания production-ready кластера.

### 4.a Настройка MetalLB

Cozystack использует три типа IP-адресов:

1. **Node IPs**: постоянные адреса, действительные только внутри кластера.
2. **Virtual floating IP**: используется для доступа к одному из узлов кластера и действителен только внутри кластера.
3. **External access IPs**: используются LoadBalancers для публикации сервисов за пределами кластера.

Сервисы с external IP можно публиковать в двух режимах: L2 и BGP. Режим L2 прост, но требует, чтобы все IP находились в одной L2-сети. BGP более надежен и масштабируем, но требует поддержки со стороны сетевого оборудования.

Выберите диапазон неиспользуемых IP для сервисов; здесь используется диапазон `192.168.100.200-192.168.100.250`. Если используется режим L2, эти IP должны быть либо из той же сети, что и узлы, либо к ним должны быть прописаны статические маршруты.

Для режима BGP также потребуются IP-адреса BGP peers и локальный и удаленный AS numbers. Здесь мы используем `192.168.20.254` как peer IP, а AS numbers 65000 и 65001 — как local и remote.

Создайте и примените файл с описанием address pool.

`metallb-ip-address-pool.yml`:

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

Создайте и примените ресурсы, необходимые для L2 или BGP advertisement. L2Advertisement использует имя ресурса IPAddressPool, который мы создали на предыдущем шаге.

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

После настройки MetalLB включите ingress в tenant-root:

```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"ingress":true}}'
```

Чтобы подтвердить успешную настройку, проверьте HelmReleases `ingress` и `ingress-nginx-system`:

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

Сервис должен быть развернут как `TYPE: LoadBalancer` и иметь корректный external IP:

```text
NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)                      AGE
root-ingress-controller   LoadBalancer   10.96.16.141   192.168.100.200   80:31632/TCP,443:30113/TCP   1m
```

### 4.b. Настройка Node Public IP

Если ваш облачный провайдер не поддерживает MetalLB, ingress controller можно опубликовать через external IPs узлов.

Если public IP подключены напрямую к узлам, укажите их. Если public IP предоставляются через 1:1 NAT, укажите внутренние адреса внешних сетевых интерфейсов.

Здесь используются `192.168.100.11`, `192.168.100.12` и `192.168.100.13`.

Сначала примените patch к Platform Package, указав IP для публикации:

```bash
kubectl patch packages.cozystack.io cozystack.cozystack-platform --type=merge -p '{"spec":{"components":{"platform":{"values":{"publishing":{"externalIPs":["192.168.100.11","192.168.100.12","192.168.100.13"]}}}}}}'
```

Затем включите ingress для root tenant:

```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"ingress":true}}'
```

После этого Ingress будет доступен на указанных IP. Проверьте это следующим образом:

```bash
kubectl get svc -n tenant-root root-ingress-controller
```

Сервис должен быть развернут как `TYPE: ClusterIP` и иметь полный диапазон external IP:

```text
NAME                     TYPE       CLUSTER-IP   EXTERNAL-IP                                   PORT(S)                      AGE
root-ingress-controller  ClusterIP  10.96.91.83  192.168.100.11,192.168.100.12,192.168.100.13  80/TCP,443/TCP               1m
```

## 5. Завершение установки

### 5.1. Настройка сервисов Root Tenant

Включите `etcd` и `monitoring` для root tenant:

```bash
kubectl patch -n tenant-root tenants.apps.cozystack.io root --type=merge -p '{"spec":{"etcd":true,"monitoring":true}}'
```

### 5.2. Проверка состояния и состава кластера

Проверьте созданные persistent volumes:

```bash
kubectl get pvc -A
```

Пример вывода:

```text
NAME                                     STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-etcd-0                              Bound    pvc-4cbd29cc-a29f-453d-b412-451647cd04bf   10Gi       RWO            local          1m
data-etcd-1                              Bound    pvc-1579f95a-a69d-4a26-bcc2-b15ccdbede0d   10Gi       RWO            local          1m
data-etcd-2                              Bound    pvc-907009e5-88bf-4d18-91e7-b56b0dbfb97e   10Gi       RWO            local          1m
grafana-db-1                             Bound    pvc-7b3f4e23-228a-46fd-b820-d033ef4679af   10Gi       RWO            local          1m
grafana-db-2                             Bound    pvc-ac9b72a4-f40e-47e8-ad24-f50d843b55e4   10Gi       RWO            local          1m
vmselect-cachedir-vmselect-longterm-0    Bound    pvc-622fa398-2104-459f-8744-565eee0a13f1   2Gi        RWO            local          1m
vmselect-cachedir-vmselect-longterm-1    Bound    pvc-fc9349f5-02b2-4e25-8bef-6cbc5cc6d690   2Gi        RWO            local          1m
vmselect-cachedir-vmselect-shortterm-0   Bound    pvc-7acc7ff6-6b9b-4676-bd1f-6867ea7165e2   2Gi        RWO            local          1m
vmselect-cachedir-vmselect-shortterm-1   Bound    pvc-e514f12b-f1f6-40ff-9838-a6bda3580eb7   2Gi        RWO            local          1m
vmstorage-db-vmstorage-longterm-0        Bound    pvc-e8ac7fc3-df0d-4692-aebf-9f66f72f9fef   10Gi       RWO            local          1m
vmstorage-db-vmstorage-longterm-1        Bound    pvc-68b5ceaf-3ed1-4e5a-9568-6b95911c7c3a   10Gi       RWO            local          1m
vmstorage-db-vmstorage-shortterm-0       Bound    pvc-cee3a2a4-5680-4880-bc2a-85c14dba9380   10Gi       RWO            local          1m
vmstorage-db-vmstorage-shortterm-1       Bound    pvc-d55c235d-cada-4c4a-8299-e5fc3f161789   10Gi       RWO            local          1m
```

Проверьте, что все pods запущены:

```bash
kubectl get pods -A
```

Теперь можно получить public IP ingress controller:

```bash
kubectl get svc -n tenant-root root-ingress-controller
```

Пример вывода:

```text
NAME                      TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)                      AGE
root-ingress-controller   LoadBalancer   10.96.16.141   192.168.100.200   80:31632/TCP,443:30113/TCP   1m
```

### 5.3 Доступ к Cozystack Dashboard

Если вы включили `dashboard` в `publishing.exposedServices` вашего Platform Package, Cozystack Dashboard уже должен быть доступен.

Если исходный Package не включал его, примените patch к Platform Package:

```bash
kubectl patch packages.cozystack.io cozystack.cozystack-platform --type=json -p '[{"op":"add","path":"/spec/components/platform/values/publishing/exposedServices/-","value":"dashboard"}]'
```

Откройте `dashboard.example.org`, чтобы получить доступ к системному dashboard, где `example.org` — ваш домен, указанный для `tenant-root`. Там вы увидите окно входа, которое ожидает authentication token.

Получите authentication token для `tenant-root`:

```bash
kubectl get secret -n tenant-root tenant-root -o go-template='{{ printf "%s\n" (index .data "password" | base64decode) }}'
```

Войдите с помощью token. Теперь dashboard доступен вам как администратору.

Далее вы сможете:
* Настроить OIDC для аутентификации вместо tokens.
* Создавать user tenants и предоставлять пользователям доступ к ним через tokens или OIDC.

### 5.4 Доступ к метрикам в Grafana

Используйте `grafana.example.org` для доступа к системному мониторингу, где `example.org` — ваш домен, указанный для `tenant-root`. В этом примере `grafana.example.org` находится по адресу 192.168.100.200.

login: `admin`
запросите password:

```bash
kubectl get secret -n tenant-root grafana-admin-password -o go-template='{{ printf "%s\n" (index .data "password" | base64decode) }}'
```

## Следующие шаги

* [Настройте OIDC](https://cozystack.ru/docs/v1.5/operations/oidc/).
* [Создайте user tenant](https://cozystack.ru/docs/v1.5/getting-started/create-tenant/).
