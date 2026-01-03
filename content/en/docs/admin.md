---
title: "Руководство Админстратора платформы"
linkTitle: "Руководство Админстратора платформы"
description: "Руководство Админстратора платформы"
weight: 42
---

## Администрирование и устранение неполадок

> Устранение неполадок в Control Plane и других сбоев в кластерах Talos Linux.

В этом руководстве мы предполагаем, что Talos настроен с включенными по умолчанию функциями, такими как [Служба обнаружения](https://docs.siderolabs.com/talos/v1.11/configure-your-talos-cluster/system-configuration/discovery) и [KubePrism](https://docs.siderolabs.com/talos/v1.11/kubernetes-guides/advanced-guides/kubeprism).
Если эти функции отключены, некоторые из шагов по устранению неполадок могут быть неприменимы или потребовать корректировки.

Данное руководство построено таким образом, что его можно выполнять шаг за шагом; пропускайте разделы, не относящиеся к вашей проблеме.

## Настройка сети

Поскольку Talos Linux — это операционная система, основанная на API, важно настроить сеть таким образом, чтобы обеспечить доступ к API.
Некоторую информацию можно получить с помощью [интерактивной панели мониторинга](https://docs.siderolabs.com/talos/v1.11/deploy-and-manage-workloads/interactive-dashboard), доступной на консоли машины.

При работе в облаке сетевые настройки должны выполняться автоматически.
В то время как при работе на физическом оборудовании может потребоваться более специфичная конфигурация, см. [руководство по настройке сети на физическом оборудовании](https://docs.siderolabs.com/talos/v1.11/platform-specific-installations/bare-metal-platforms/network-config).

## API Talos

API Talos работает на порту 50000 (../learn-more/talos-network-connectivity).
Узлы Control Plane всегда должны предоставлять доступ к API Talos, в то время как рабочие узлы нуждаются в доступе к узлам Control Plane для выдачи TLS-сертификатов для рабочих узлов.

### Проблемы с брандмауэром

Убедитесь, что брандмауэр не блокирует порт 50000 и [коммуникацию](https://docs.siderolabs.com/talos/v1.11/learn-more/talos-network-connectivity) на портах 50000/50001 внутри кластера.

### Проблемы с конфигурацией клиента

Убедитесь, что используете правильный файл конфигурации клиента `talosconfig`, соответствующий вашему кластеру.
Дополнительную информацию см. в разделе [начало работы](https://docs.siderolabs.com/talos/v1.11/getting-started/getting-started).

Наиболее распространенная проблема заключается в том, что команда `talosctl gen config` записывает `talosconfig` в файл в текущем каталоге, в то время как `talosctl` по умолчанию берет конфигурацию из стандартного расположения (`~/.talos/config`).
Путь к файлу конфигурации можно указать с помощью флага `--talosconfig` в команде `talosctl`.

### Конфликт в Kubernetes и подсетях хоста

Если `talosctl` возвращает ошибку, указывающую на то, что IP-адреса сертификатов пусты, это может быть связано с конфликтом между Kubernetes и подсетями хоста.
API Talos работает в хост-сети, но автоматически исключает подсети и сети Kubernetes из набора доступных адресов.

В конфигурации машины Talos по умолчанию указаны следующие IPv4 CIDR для подов и сервисов Kubernetes: `10.244.0.0/16` и `10.96.0.0/12`.
Если в хост-сети используется одна из этих подсетей, измените конфигурацию машины, чтобы использовать другую подсеть.

### Неверные конечные точки

Интерфейс командной строки `talosctl` подключается к API Talos через указанные конечные точки, которые должны представлять собой список адресов машин Control Plane.
В случае недоступности каких-либо конечных точек клиент автоматически повторит попытку подключения к другим точкам доступа.

Рабочие узлы не следует использовать в качестве конечной точки, поскольку они не могут перенаправлять запросы на другие узлы.

Адрес [VIP](https://docs.siderolabs.com/talos/v1.11/networking/vip) ни в коем случае нельзя использовать в качестве конечной точки API Talos.

### TCP-балансировщик нагрузки

При использовании балансировщика нагрузки TCP убедитесь, что конечная точка балансировщика нагрузки включена в список `.machine.certSANs` в конфигурации машины.

## Системные требования

Если минимальные [системные требования](https://docs.siderolabs.com/talos/v1.11/getting-started/system-requirements) не соблюдены, это может проявляться различными способами, например, в виде случайных сбоев при запуске служб или сбоев при загрузке образов из реестра контейнеров.

## Проведение проверок состояния здоровья

Talos Linux предоставляет набор базовых проверок работоспособности с помощью команды `talosctl health`, которую можно использовать для проверки состояния кластера.

В режиме по умолчанию команда `talosctl health` использует информацию из [discovery](https://docs.siderolabs.com/talos/v1.11/configure-your-talos-cluster/system-configuration/discovery) для получения сведений о членах кластера.
Это можно переопределить с помощью флагов командной строки `--control-plane-nodes` и `--worker-nodes`.

## Журналы сбора

Хотя журналы и состояние системы можно запросить через API Talos, часто бывает полезно собрать журналы со всех узлов кластера и проанализировать их в автономном режиме.
Команда `talosctl support` позволяет собирать журналы и другую информацию с узлов, указанных с помощью флага `--nodes` (поддерживается несколько узлов).

## Обнаружение и членство в кластере

Talos Linux использует [службу обнаружения](https://docs.siderolabs.com/talos/v1.11/configure-your-talos-cluster/system-configuration/discovery) для обнаружения других узлов в кластере.

Список участников на каждой машине должен быть согласованным: `talosctl -n <IP> get members`.

### Некоторые участники отсутствуют

Убедитесь в наличии подключения к службе обнаружения (по умолчанию `discovery.talos.dev:443`) и в том, что реестр обнаружения не отключен.

### Дублирующиеся участники

Не используйте одни и те же базовые секреты для генерации конфигурации машин для нескольких кластеров, поскольку некоторые секреты используются для идентификации членов одного и того же кластера.
Таким образом, если одна и та же конфигурация машины (или секреты) используется для многократного создания и уничтожения кластеров, служба обнаружения будет видеть одни и те же узлы как членов разных кластеров.

### Удаленные участники все еще присутствуют

Talos Linux удаляет себя из службы обнаружения при [перезагрузке](https://docs.siderolabs.com/talos/v1.11/configure-your-talos-cluster/lifecycle-management/resetting-a-machine).
Если устройство не было перезагружено, оно может отображаться как участник кластера в течение максимального времени жизни службы обнаружения (30 минут), после чего будет автоматически удалено.

## Проблемы с `etcd`

`etcd` — это распределенное хранилище типа «ключ-значение», используемое Kubernetes для хранения своего состояния.
Talos Linux предоставляет средства автоматизации для управления компонентами `etcd`, работающими на узлах Control Plane.
Если `etcd` неисправен, сервер API Kubernetes не сможет корректно функционировать.

Всегда рекомендуется запускать нечетное количество участников `etcd`, поскольку при наличии 3 или более участников обеспечивается отказоустойчивость в случае сбоев меньшего количества участников, чем необходимо для достижения кворума.

Типичные шаги по устранению неполадок:

* Проверьте состояние службы `etcd` с помощью команды `talosctl -n IP service etcd` для каждого узла Control Plane.
* Проверьте принадлежность `etcd` к каждому узлу Control Plane с помощью команды `talosctl -n IP etcd members`.
* Проверьте логи `etcd` с помощью команды `talosctl -n IP logs etcd`
* Проверьте наличие оповещений `etcd` с помощью команды `talosctl -n IP etcd alarm list`

### Все службы `etcd` застряли в состоянии `Pre`

Убедитесь, что хотя бы один участник был [запущен](https://docs.siderolabs.com/talos/v1.11/getting-started/getting-started#kubernetes-bootstrap).

Убедитесь, что машина может загрузить образ контейнера `etcd`, проверьте `talosctl dmesg` на наличие сообщений, начинающихся с префикса `retrying:`.

### Некоторые службы `etcd` застряли в состоянии `Pre`

Убедитесь, что трафик на порту 2380 между узлами Control Plane не заблокирован.

Убедитесь, что кворум `etcd` не потерян.

Убедитесь, что все узлы Control Plane отображаются в выводе команды `talosctl get members`.

### Отчеты и оповещения `etcd`

См. руководство по [обслуживанию etcd](https://docs.siderolabs.com/talos/v1.11/build-and-extend-talos/cluster-operations-and-maintenance/etcd-maintenance).

### Кворум `etcd` потерян

См. руководство по [аварийному восстановлению](https://docs.siderolabs.com/talos/v1.11/build-and-extend-talos/cluster-operations-and-maintenance/disaster-recovery).

### Другие вопросы

`etcd` будет запускаться только на узлах Control Plane.
Если узел назначен рабочим узлом, не следует ожидать, что на нем будет запущена команда `etcd`.

При первой загрузке узла каталог данных `etcd` (`/var/lib/etcd`) пуст и будет заполнен только при запуске `etcd`.

Если служба `etcd` аварийно завершает работу и перезапускается, проверьте ее журналы с помощью команды `talosctl -n <IP> logs etcd`.
Наиболее распространенные причины аварий:

* Неверные аргументы, переданные через `extraArgs` в конфигурации;
* При загрузке Talos с непустого диска с уже установленной Talos, `/var/lib/etcd` содержит данные из старого кластера.

## Проблемы с `kubelet` и узлами Kubernetes

Сервис `kubelet` должен быть запущен на всех узлах Talos и отвечает за запуск подов Kubernetes.
статические поды (включая компоненты Control Plane) и регистрация узла на сервере API Kubernetes.

Если `kubelet` не запущен на узле Control Plane, он заблокирует запуск компонентов Control Plane.

Узел не будет зарегистрирован в Kubernetes до тех пор, пока не будет запущен сервер API Kubernetes и не будут применены начальные манифесты Kubernetes.

### `kubelet` не запущен

Убедитесь, что образ `kubelet` доступен (`talosctl image ls --namespace system`).

Проверьте логи `kubelet` с помощью команды `talosctl -n IP logs kubelet` на наличие ошибок запуска:

* Убедитесь, что версия Kubernetes [поддерживается](https://docs.siderolabs.com/talos/v1.11/getting-started/support-matrix) в этом релизе Talos.
* Убедитесь, что дополнительные аргументы `kubelet` и дополнительная конфигурация, предоставляемая вместе с конфигурацией машины Talos, являются допустимыми.

### Talos выдает ошибку "Узел не найден"

`kubelet` еще не зарегистрировал узел на сервере API Kubernetes; это ожидаемое поведение на этапе первоначальной загрузки кластера, после чего ошибка исчезнет.
Если сообщение не исчезает, проверьте работоспособность API Kubernetes.

Менеджер контроллеров Kubernetes («kube-controller-manager») отвечает за мониторинг сертификата.
подписание запросов на подписание сертификатов (CSR) и выдача сертификатов по каждому из них.
«Кублетт» отвечает за генерацию и отправку запросов на обслуживание клиентов (CSR) для своего приложения.
связанный узел.

Состояние любого запроса на подписание сертификата (CSR) можно проверить с помощью команды `kubectl get csr`:

```
$ kubectl get csr
NAME        AGE   SIGNERNAME                                    REQUESTOR                 CONDITION
csr-jcn9j 14m kubernetes.io/kube-apiserver-client-kubelet system:bootstrap:q9pyzr Approved,Issued
csr-p6b9q 14m kubernetes.io/kube-apiserver-client-kubelet system:bootstrap:q9pyzr Approved,Issued
csr-sw6rm 14m kubernetes.io/kube-apiserver-client-kubelet system:bootstrap:q9pyzr Approved,Issued
csr-vlghg 14m kubernetes.io/kube-apiserver-client-kubelet system:bootstrap:q9pyzr Approved,Issued
```

### Команда `kubectl get nodes` выдает ошибку "Неверный внутренний IP-адрес".

Настройте правильный внутренний IP-адрес с помощью [`.machine.kubelet.nodeIP`](https://docs.siderolabs.com/talos/v1.11/reference/configuration/v1alpha1/config#Config.machine.kubelet.nodeIP)

### Команда `kubectl get nodes` выдает ошибку "Неверный внешний IP-адрес".

Talos Linux не управляет внешним IP-адресом, это делается с помощью Kubernetes Cloud Controller Manager.

### Команда `kubectl get nodes` выдает ошибку "Неверное имя узла".

По умолчанию имя узла Kubernetes определяется на основе имени хоста.
Обновите имя хоста, используя конфигурацию машины, облачную конфигурацию или DHCP-сервер.

### Узел не готов

В Kubernetes узел помечается как «готовый» только после того, как его CNI-интерфейс будет активирован.
Загрузка изображений CNI и запуск CNI занимают минуту-две.
Если узел застрял в этом состоянии слишком долго, проверьте поды CNI и логи с помощью `kubectl`.
Как правило, ресурсы, связанные с CNI, создаются в пространстве имен `kube-system`.

Например, для стандартного CNI Talos Flannel:

```
$ kubectl -n kube-system получить модули
NAME                                             READY   STATUS    RESTARTS   AGE
...
kube-flannel-25drx 1/1 Running 0 23m
kube-flannel-8lmb6 1/1 Running 0 23m
kube-flannel-gl7nx 1/1 Running 0 23m
kube-flannel-jknt9 1/1 Running 0 23m
...
```

### Дублирующиеся/Устаревшие узлы

Talos Linux не удаляет узлы Kubernetes автоматически, поэтому, если узел удален из кластера, он все равно останется в Kubernetes.
Удалите узел из Kubernetes с помощью команды `kubectl delete node <node-name>`.

### Talos жалуется на ошибки сертификатов в API `kubelet`

Эта ошибка может появиться во время начальной загрузки кластера и исчезнет после запуска сервера API Kubernetes и регистрации узла.

Пример логов Talos:

```
[talos] controller failed {"component": "controller-runtime", "controller": "k8s.KubeletStaticPodController", "error": "error refreshing pod status: error fetching pod status: Get \"https://127.0.0.1:10250/pods/?timeout=30s\": remote error: tls: internal error"}
```

По умолчанию `kubelet` выдает самоподписанный серверный сертификат, но при включении функции `rotate-server-certificates`,
`kubelet` выдает свой сертификат, используя `kube-apiserver`.
Убедитесь, что запрос на подписание сертификата `kubelet` одобрен сервером API Kubernetes.

В любом случае, эта ошибка не является критической, поскольку она влияет только на передачу информации о состоянии пода в Talos Linux.

## Control Plane Kubernetes

Control Plane Kubernetes состоит из следующих компонентов:

* `kube-apiserver` — API-сервер Kubernetes
* `kube-controller-manager` — менеджер контроллеров Kubernetes
* `kube-scheduler` — планировщик Kubernetes

При желании `kube-proxy` может запускаться как DaemonSet для обеспечения связи между подом и сервисом.

`coredns` обеспечивает разрешение имен для кластера.

CNI не является частью Control Plane, но он необходим для подов Kubernetes, использующих сетевое взаимодействие между подами.

Поиск и устранение неисправностей всегда следует начинать с `kube-apiserver`, а затем переходить к другим компонентам.

В Talos Linux `kube-apiserver` настроен на взаимодействие с `etcd`, работающим на том же узле, поэтому `etcd` должен быть работоспособен, прежде чем `kube-apiserver` сможет запуститься.
Компоненты `kube-controller-manager` и `kube-scheduler` настроены на взаимодействие с компонентом `kube-apiserver` на том же узле, поэтому они не запустятся, пока `kube-apiserver` не будет работать корректно.

### Статические модули Control Plane

Talos должен генерировать статические определения подов для Control Plane Kubernetes.
в качестве ресурсов:

```
$ talosctl -n <IP> get staticpods
NODE         NAMESPACE   TYPE        ID                        VERSION
172.20.0.2   k8s         StaticPod   kube-apiserver            1
172.20.0.2   k8s         StaticPod   kube-controller-manager   1
172.20.0.2   k8s         StaticPod   kube-scheduler            1
```

Talos должен сообщить, что статические определения подов отображаются для `kubelet`:

```
$ talosctl -n <IP> dmesg | grep 'rendered new'
172.20.0.2: user: warning: [2023-04-26T19:17:52.550527204Z]: [talos] rendered new static pod {"component": "controller-runtime", "controller": "k8s.StaticPodServerController", "id": "kube-apiserver"}
172.20.0.2: user: warning: [2023-04-26T19:17:52.552186204Z]: [talos] rendered new static pod {"component": "controller-runtime", "controller": "k8s.StaticPodServerController", "id": "kube-controller-manager"}
172.20.0.2: user: warning: [2023-04-26T19:17:52.554607204Z]: [talos] rendered new static pod {"component": "controller-runtime", "controller": "k8s.StaticPodServerController", "id": "kube-scheduler"}
```

Если статические определения подов не отображаются, проверьте состояние служб `etcd` и `kubelet` (см. выше).
а также журналы выполнения контроллера (`talosctl logs controller-runtime`).

### Состояние модуля управления

Изначально компонент `kube-apiserver` не будет запущен, и потребуется некоторое время, прежде чем он полностью заработает.
во время загрузки Bootstrap (изображение следует брать из интернета и т. д.)

Состояние компонентов Control Plane на каждом из узлов Control Plane можно проверить с помощью команды `talosctl containers -k`:

```
$ talosctl -n <IP> containers --kubernetes
  NODE         NAMESPACE   ID                                                                                            IMAGE                                               PID    STATUS
  172.20.0.2   k8s.io      kube-system/kube-apiserver-talos-default-controlplane-1                                       registry.k8s.io/pause:3.2                                2539   SANDBOX_READY
  172.20.0.2   k8s.io      └─ kube-system/kube-apiserver-talos-default-controlplane-1:kube-apiserver:51c3aad7a271        registry.k8s.io/kube-apiserver:v1.35.0 2572   CONTAINER_RUNNING
```

Журналы компонентов Control Plane можно проверить с помощью команды `talosctl logs --kubernetes` (или с помощью `-k` в качестве сокращенной записи):

```
talosctl -n <IP> logs -k kube-system/kube-apiserver-talos-default-controlplane-1:kube-apiserver:51c3aad7a271
```

Если компонент control plane сообщает об ошибке при запуске, проверьте следующее:

* Убедитесь, что версия Kubernetes [поддерживается](https://docs.siderolabs.com/talos/v1.11/getting-started/support-matrix) в этом релизе Talos.
* Убедитесь, что дополнительные аргументы и дополнительные параметры конфигурации, предоставленные вместе с конфигурацией машины Talos, являются допустимыми.

### Манифесты начальной загрузки Kubernetes

В рамках процесса начальной загрузки Talos внедряет манифесты начальной загрузки в API-сервер Kubernetes.
Существует два типа таких манифестов: системные манифесты, встроенные в Talos, и дополнительные манифесты, загружаемые извне (пользовательские CNI, дополнительные манифесты в конфигурации машины):

```
$ talosctl -n <IP> get manifests
NODE         NAMESPACE      TYPE       ID                               VERSION
172.20.0.2   controlplane   Manifest   00-kubelet-bootstrapping-token   1
172.20.0.2   controlplane   Manifest   01-csr-approver-role-binding     1
172.20.0.2   controlplane   Manifest   01-csr-node-bootstrap            1
172.20.0.2   controlplane   Manifest   01-csr-renewal-role-binding      1
172.20.0.2   controlplane   Manifest   02-kube-system-sa-role-binding   1
172.20.0.2   controlplane   Manifest   03-default-pod-security-policy   1
172.20.0.2   controlplane   Manifest   05-https://docs.projectcalico.org/manifests/calico.yaml   1
172.20.0.2   controlplane   Manifest   10-kube-proxy                    1
172.20.0.2   controlplane   Manifest   11-core-dns                      1
172.20.0.2   controlplane   Manifest   11-core-dns-svc                  1
172.20.0.2   controlplane   Manifest   11-kube-config-in-cluster        1
```

Подробную информацию о каждом манифесте можно получить, добавив параметр `-o yaml`:

```
$ talosctl -n <IP> get manifests 01-csr-approver-role-binding --namespace=controlplane -o yaml
node: 172.20.0.2
metadata:
    namespace: controlplane
    type: Manifests.kubernetes.talos.dev
    id: 01-csr-approver-role-binding
    version: 1
    phase: running
spec:
    - apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: system-bootstrap-approve-node-client-csr
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: system:certificates.k8s.io:certificatesigningrequests:nodeclient
      subjects:
        - apiGroup: rbac.authorization.k8s.io
          kind: Group
          name: system:bootstrappers
```

### Другие компоненты Control Plane

После запуска сервера API Kubernetes проблемы с другими компонентами Control Plane можно устранить с помощью команды `kubectl`:

```
kubectl get nodes -o wide
kubectl get pods -o wide --all-namespaces
kubectl describe pod -n NAMESPACE POD
kubectl logs -n NAMESPACE POD
```

## API Kubernetes

Конфигурацию клиента Kubernetes API (`kubeconfig`) можно получить с помощью Talos API, используя команду `talosctl -n <IP> kubeconfig`.
В Talos Linux кластер в основном не зависит от конечной точки API Kubernetes, но конечную точку API Kubernetes следует настроить.
корректно обеспечивается внешний доступ к кластеру.

### Конечная точка Control Plane Kubernetes

Конечная точка Control Plane Kubernetes — это единственный канонический URL-адрес, по которому осуществляется доступ к API Kubernetes.
В частности, в системах управления с высокой доступностью (HA) эта конечная точка может указывать на балансировщик нагрузки или DNS-имя, которое может
имеют несколько записей типа `A` ​​и `AAAA`.

Подобно собственному API Talos, API Kubernetes использует взаимный TLS, клиентское соединение.
сертификаты и общий центр сертификации (ЦС).
В отличие от веб-сайтов общего назначения, здесь нет необходимости в вышестоящем центре сертификации, поэтому инструменты не требуют дополнительных настроек.
например, cert-manager, Let's Encrypt или подобные продукты.
поскольку подтвержденные TLS-сертификаты не требуются.
Однако шифрование *есть*, поэтому схема URL всегда будет `https://`.

По умолчанию сервер API Kubernetes в Talos работает на порту 6443.
Таким образом, URL-адреса конечных точек Control Plane для Talos почти всегда будут иметь следующий вид:
`https://endpoint:6443`.
(Необходимо указать порт, поскольку он не соответствует порту по умолчанию `443` (https:// ...
Указанная выше `endpoint` может быть DNS-именем или IP-адресом, но она должна быть следующей:
направлено на *множество* всех узлов Control Plane, в отличие от
один.

Как уже упоминалось выше, этого можно достичь с помощью ряда стратегий, в том числе:

* внешний балансировщик нагрузки
* DNS-записи
* Встроенный в Talos общий IP-адрес ([VIP](https://docs.siderolabs.com/talos/v1.11/networking/vip))
* Установление BGP-пиринга для общего IP-адреса (например, с помощью [kube-vip](https://kube-vip.io))

Использование DNS-имени здесь — хорошая идея, поскольку это позволяет использовать любой другой вариант, одновременно предоставляя...
слой абстракции.
Это позволяет изменять базовые IP-адреса без влияния на
канонический URL.

В отличие от большинства сервисов в Kubernetes, API-сервер работает в рамках сетевой инфраструктуры хоста.
Это означает, что он использует то же сетевое пространство имен, что и хост.
Это означает, что вы можете использовать IP-адрес(а) хоста для обращения к Kubernetes.
API-сервер.

Для обеспечения доступности API важно, чтобы любой балансировщик нагрузки был осведомлен о следующем:
проверка работоспособности серверов бэкэнд-API для минимизации сбоев во время работы.
Типичные операции с узлами, такие как перезагрузка и обновление.

## Прочее

### Проверка журналов выполнения контроллера

Talos использует набор [контроллеров](https://docs.siderolabs.com/talos/v1.11/learn-more/controllers-resources), которые работают с ресурсами для построения и поддержки операций машины.

Некоторую отладочную информацию можно получить из логов контроллера с помощью команды `talosctl logs controller-runtime`:

```
talosctl -n <IP> logs controller-runtime
```

Контроллеры постоянно работают в цикле согласования, поэтому в любой момент времени они могут запускаться, выходить из строя или перезапускаться.
Это ожидаемое поведение.

Если в журнале `controller-runtime` нет новых сообщений, это означает, что контроллеры успешно завершили согласование, и текущее состояние системы соответствует желаемому состоянию системы.
