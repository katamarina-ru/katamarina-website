---
title: "Установка и настройка Cozystack"
linkTitle: "Установка и настройка Cozystack"
description: "Установка и настройка Cozystack"
weight: 1
---

# Установка и настройка Cozystack

**Третий шаг** развертывания кластера Cozystack — установка Cozystack в ранее [установленный и настроенный кластер Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/). Предварительное условие для этого шага — установленный кластер Kubernetes.

Cozystack можно установить в двух режимах, в зависимости от того, насколько детально вы хотите управлять устанавливаемыми компонентами:

## Как платформа

Установите Cozystack как готовую к использованию платформу, где все компоненты управляются автоматически. Вы выбираете [variant](https://cozystack.ru/docs/v1.5/operations/configuration/variants/) (например, `isp-full`), а Cozystack устанавливает и настраивает все необходимые компоненты — сеть, хранилище, мониторинг, dashboard, operators и managed applications.

Это рекомендуемый подход для большинства пользователей, которым нужна полностью функциональная платформа из коробки.

**[Установить Cozystack как платформу](https://cozystack.ru/docs/v1.5/install/cozystack/platform/)**

## Build Your Own Platform (BYOP)

Используйте Cozystack, чтобы собрать собственную платформу, устанавливая только нужные компоненты. Вы устанавливаете operator с variant `default`, который предоставляет только package registry (PackageSources). Затем с помощью CLI-инструмента `cozypkg` выборочно устанавливаете отдельные packages — networking, storage, ingress, operators и любые другие компоненты, доступные в репозитории Cozystack.

Этот подход подходит, если у вас уже есть существующий кластер Kubernetes с частью инфраструктуры или если вам нужны только отдельные компоненты из экосистемы Cozystack.

**[Собрать собственную платформу с Cozystack](https://cozystack.ru/docs/v1.5/install/cozystack/kubernetes-distribution/)**

---

## Установка Cozystack как платформы

Установите Cozystack как готовую к использованию платформу, где все компоненты управляются автоматически.

## Собственная платформа (BYOP)

Соберите собственную платформу с Cozystack, устанавливая только нужные компоненты с помощью CLI-инструмента `cozypkg`.