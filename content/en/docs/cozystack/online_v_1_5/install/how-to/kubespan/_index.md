---
title: "KubeSpan"
linkTitle: "KubeSpan"
description: "KubeSpan"
weight: 1
---

# Как включить KubeSpan

Talos Linux предоставляет full-mesh сеть WireGuard для вашего кластера ([Cozystack Documentation](https://cozystack.ru/docs/v1.5/install/how-to/kubespan/)).

Чтобы включить эту функциональность, настройте [KubeSpan](https://www.talos.dev/v1.13/talos-guides/network/kubespan/) и [Cluster Discovery](https://www.talos.dev/v1.13/kubernetes-guides/configuration/discovery/) в конфигурации Talos Linux ([Cozystack Documentation](https://cozystack.ru/docs/v1.5/install/how-to/kubespan/)):

```yaml
machine:
  network:
    kubespan:
      enabled: true
cluster:
  discovery:
    enabled: true
```

Поскольку KubeSpan инкапсулирует трафик в туннель WireGuard, для Kube-OVN также нужно настроить более низкое значение MTU ([Cozystack Documentation](https://cozystack.ru/docs/v1.5/install/how-to/kubespan/)).

Для этого добавьте следующее в компонент `networking` вашего Platform Package ([Cozystack Documentation](https://cozystack.ru/docs/v1.5/install/how-to/kubespan/)):

```yaml
apiVersion: cozystack.io/v1alpha1
kind: Package
metadata:
  name: cozystack.cozystack-platform
spec:
  # ...
  components:
    networking:
      values:
        kube-ovn:
          mtu: 1222
```