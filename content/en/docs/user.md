---
title: "Руководство Пользователя платформы"
linkTitle: "Руководство Пользователя платформы"
description: "Руководство Пользователя платформы"
weight: 43
---

# Интерактивная панель управления

> Инструмент для проверки состояния работающей системы

Интерактивная панель мониторинга включена для всех платформ, кроме образов SBC. Панель мониторинга можно отключить с помощью параметра ядра talos.dashboard.disabled=1

Панель управления работает только на физической видеоконсоли (не последовательной консоли) на втором виртуальном терминале (TTY). 
Первый виртуальный терминал отображает журналы ядра. Переключение между виртуальными терминалами осуществляется с помощью клавиш `<Alt+F1>`и `<Alt+F2>`

Клавиши `<F1>`— `<Fn>`используются для переключения между различными экранами панели управления.

Панель управления использует либо фреймбуфер UEFI, либо фреймбуфер VGA/VESA (для загрузки через устаревший BIOS).

## Управление разрешением панели управления


В системах со старым BIOS разрешение экрана можно настроить с помощью [`vga=` kernel parameter](https://docs.kernel.org/fb/vesafb.html).

В современных ядрах и платформах этот параметр часто игнорируется. Для получения надежных результатов рекомендуется загрузка с UEFI .

При работе в режиме UEFI разрешение экрана можно установить через настройки гипервизора или микропрограммы UEFI.

## Сводная информация (`F1`)

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=d35339535e51eb7b15456a5af52b8404" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=e14103d301b3ef32aa72c7e86c35336e 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=bb8736e7d213e21f1626c415ab79ac78 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=7f46f23ed09dd6514cd2142e96ed4442 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0a95e859089462874314eed0f0fbf3c3 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=c441116ab6ca1b037ef5756d725e5f53 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-1.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=974f517194643fca7d4a6a2443fadc26 2500w" />

В заголовке представлена ​​краткая информация об узле:

* имя хоста
* версия ОС
* время безотказной работы
* информация об аппаратном обеспечении процессора и памяти
* загрузка ЦП и памяти, количество процессов

В табличном представлении отображается сводная информация о машине:

* UUID (из данных SMBIOS)
* имя кластера (если доступна конфигурация машины)
* этап работы машины: Installing, Upgrading, Booting, Maintenance, Running, Rebooting, Shutting down, и т. д.
* проверка готовности этапа работы машины: проверяет состояние сервиса Talos, состояние статического модуля и т. д. (для Runningэтапа).
* тип машины: плоскость управления/рабочий узел
* количество обнаруженных членов в кластере
* версия Kubernetes
* состояние компонентов Kubernetes kubeletи компонентов плоскости управления Kubernetes (только на controlplaneмашинах)
* информация о сети: имя хоста, адреса, шлюз, подключение, DNS и NTP-серверы.

В нижней части экрана отображаются журналы ядра, как и на виртуальном терминале TTY 1.

## Экран монитора (`F2`)

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=151073a1b4f39370e311c3967a01966d" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=03aebccdc1abf22cf841796dfe29652a 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=5e07a80a15f329b389cbc7e2a1ebf1e4 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=20e466fed281c7aa0c48ec45b1b6789d 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=8a98b536f7a84ca01b17dbd9d5ceff94 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=124a8b59ee91508ee094b422831ff8e0 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-2.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=719bb7c8178068437e57b222c842ab23 2500w" />

На экране монитора отображается информация об использовании ресурсов машины в режиме реального времени: ЦП, память, дисковое пространство, сеть и процессы.

## Экран настройки сети (`F3`)

>  Примечание: экран настройки сети доступен только для `metal` платформ.

<img src="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=2aed8497fba28c5c79bb8bcce95187b8" alt="Interactive Dashboard Summary Screen" width="920" data-og-width="1556" data-og-height="1234" data-path="talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=280&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0c79880a78f985fca04ef3c2471e015e 280w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=560&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=fb97a0c9c2f4b08c1719534504155e55 560w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=840&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=c03944da93c166b50f9085d26f7ba858 840w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=1100&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=8d346f280ac480635fea29e156e4f41c 1100w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=1650&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=0cf7393ee7d9963fe48ac2d9504254b7 1650w, https://mintcdn.com/siderolabs-fe86397c/ggZ02hqyfoEX1q-x/talos/v1.9/deploy-and-manage-workloads/images/interactive-dashboard-3.png?w=2500&fit=max&auto=format&n=ggZ02hqyfoEX1q-x&q=85&s=cb99976afffc0984cf300f89ae389cf8 2500w" />

Экран настройки сети предоставляет возможности редактирования `metal` [конфигурации сети платформы](https://docs.siderolabs.com/talos/v1.11/platform-specific-installations/bare-metal-platforms/network-config).

Экран разделён на три секции:

* В самом левом разделе можно ввести параметры сетевой конфигурации: имя хоста, DNS и NTP-серверы, настроить сетевой интерфейс (через DHCP или статический IP-адрес) и т.д.
* В средней части отображается текущая конфигурация сети.
* В самой правой части отображается конфигурация сети, которая будет применена после нажатия кнопки «Сохранить».

После сохранения конфигурации сетевой конфигурации платформы она немедленно применяется к машине.
