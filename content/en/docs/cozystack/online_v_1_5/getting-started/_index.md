---
title: "Начало работы с Cozystack"
linkTitle: "Начало работы с Cozystack"
description: "Начало работы с Cozystack"
weight: 1
---



# Начало работы с Cozystack: развёртывание частного облака с нуля

Это руководство поможет вам выполнить первое развёртывание кластера Cozystack. По ходу работы вы познакомитесь с ключевыми концепциями, научитесь пользоваться Cozystack через дашборд и CLI и получите рабочий proof-of-concept.

Руководство разделено на несколько шагов. Завершайте каждый шаг перед переходом к следующему:

| Шаг | Описание |
| --- | --- |
| [Требования: подготовьте инфраструктуру и инструменты](https://cozystack.ru/docs/v1.5/getting-started/requirements/) | Подготовьте инфраструктуру и установите необходимые CLI-инструменты на свою машину перед началом этого руководства. |
| 1. [Установите Talos Linux](https://cozystack.ru/docs/v1.5/getting-started/install-talos/) | Установите дистрибутив Talos Linux, подготовленный для Cozystack, с помощью `boot-to-talos` — это, вероятно, самый простой способ установки. |
| 2. [Установите и инициализируйте кластер Kubernetes](https://cozystack.ru/docs/v1.5/getting-started/install-kubernetes/) | Выполните bootstrap кластера Kubernetes с помощью Talm — инструмента управления конфигурацией Talos, созданного для Cozystack. |
| 3. [Установите и настройте Cozystack](https://cozystack.ru/docs/v1.5/getting-started/install-cozystack/) | Установите Cozystack, получите административный доступ, выполните базовую настройку и откройте дашборд Cozystack. |
| 4. [Создайте tenant для пользователей и команд](https://cozystack.ru/docs/v1.5/getting-started/create-tenant/) | Создайте пользовательский tenant — основу RBAC в Cozystack — и получите доступ к нему через дашборд и Cozystack API. |
| 5. [Разверните управляемые приложения](https://cozystack.ru/docs/v1.5/getting-started/deploy-app/) | Начните работать с Cozystack: разверните виртуальную машину, управляемое приложение и Kubernetes-кластер tenant'а. |

---

## Требования и набор инструментов

Подготовьте инфраструктуру и установите необходимые инструменты.

## 1. Установка Talos Linux

Установка Talos Linux на любую машину с помощью cozystack/boot-to-talos.

## 2. Установка и bootstrap кластера Kubernetes

Используйте Talm CLI, чтобы выполнить bootstrap кластера Kubernetes, готового к установке Cozystack.

## 3. Установка и настройка Cozystack

Установите Cozystack, получите административный доступ, выполните базовую настройку и включите UI-дашборд.

## 4. Создание пользовательского tenant'а и настройка доступа

Создайте пользовательский tenant — основу RBAC в Cozystack — и получите доступ к нему через дашборд и Cozystack API.

## 5. Развёртывание управляемых приложений, ВМ и Kubernetes-кластера tenant'а

Начните использовать Cozystack: разверните виртуальную машину, управляемое приложение и Kubernetes-кластер tenant’а.

[Cozystack Documentation](https://cozystack.ru/docs/v1.5/getting-started/)