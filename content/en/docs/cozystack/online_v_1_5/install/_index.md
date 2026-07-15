# Руководство по развертыванию Cozystack: от инфраструктуры до готового кластера

## Руководство по Cozystack

Если вы устанавливаете Cozystack впервые, рекомендуем [пройти руководство по Cozystack](https://cozystack.ru/docs/v1.5/getting-started/). В нем показан самый короткий путь к созданию proof-of-concept кластера Cozystack.

## Общий путь установки

Установка Cozystack на bare-metal серверы или виртуальные машины состоит из трех последовательных шагов. Для каждого шага есть несколько вариантов. Мы указываем рекомендуемый вариант, но также приводим альтернативы, чтобы процесс установки оставался гибким:

1. [Установите Talos Linux](https://cozystack.ru/docs/v1.5/install/talos/) на bare-metal серверы или виртуальные машины с Linux либо без операционной системы.
2. [Установите и инициализируйте кластер Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/) поверх Talos Linux.
3. [Установите и настройте Cozystack](https://cozystack.ru/docs/v1.5/install/cozystack/) в кластере Kubernetes.

## Изолированная среда

Cozystack можно установить в изолированной среде без прямого доступа к Интернету. Главное отличие такой установки — использование proxy registry для образов:

1. [Установите Talos Linux](https://cozystack.ru/docs/v1.5/install/talos/) на bare-metal серверы или виртуальные машины с Linux либо без операционной системы.
2. [Настройте узлы Talos для изолированной среды и инициализируйте кластер Kubernetes](https://cozystack.ru/docs/v1.5/install/kubernetes/air-gapped/).
3. [Установите и настройте Cozystack](https://cozystack.ru/docs/v1.5/install/cozystack/) в кластере Kubernetes.

## Автоматизированная установка с Ansible

Для типовых развертываний Linux (Ubuntu, Debian, RHEL, Rocky, openSUSE) [коллекция Ansible](https://cozystack.ru/docs/v1.5/install/ansible/) автоматизирует весь процесс: подготовку ОС, инициализацию кластера k3s и установку Cozystack.

## Установка у конкретных провайдеров

Для облачных провайдеров есть отдельные руководства, которые охватывают все шаги: от подготовки инфраструктуры до установки и настройки Cozystack. Если это ваш случай, рекомендуем использовать руководства ниже:

* [Hetzner](https://cozystack.ru/docs/v1.5/install/providers/hetzner/)
* [Oracle Cloud Infrastructure (OCI)](https://cozystack.ru/docs/v1.5/install/providers/oracle-cloud/)
* [Servers.com](https://cozystack.ru/docs/v1.5/install/providers/servers-com/)

## Обновление и настройка после развертывания

После развертывания кластера перейдите в раздел [Администрирование кластера](https://cozystack.ru/docs/v1.5/operations/), чтобы выполнить следующие действия:

* [Настройка OIDC](https://cozystack.ru/docs/v1.5/operations/oidc/)
* [Развертывание Cozystack в конфигурации с несколькими дата-центрами](https://cozystack.ru/docs/v1.5/operations/stretched/)
* [Обновление Cozystack](https://cozystack.ru/docs/v1.5/operations/cluster/upgrade/)

---

## Требования к оборудованию
Определите требования к оборудованию для вашего сценария использования Cozystack.

## Рекомендации по планированию системных ресурсов
Сколько системных ресурсов выделять на узел в зависимости от масштаба кластера.

## Установка Talos Linux на bare metal или виртуальные машины
Шаг 1: установка Talos Linux на виртуальные машины или bare metal для последующей инициализации кластера Cozystack.

## Установка и настройка кластера Kubernetes
Шаг 2: установка и настройка кластера Kubernetes, готового к установке Cozystack.

## Установка и настройка Cozystack
Шаг 3: установка Cozystack в Kubernetes-кластер — как готовой к использованию платформы или в режиме BYOP (Build Your Own Platform).

## Развертывание кластера Cozystack в облаках и у хостинг-провайдеров
Руководства по развертыванию кластеров Cozystack у конкретных облачных и хостинг-провайдеров.

## Автоматизированная установка с Ansible
Развертывание Cozystack на generic Kubernetes с помощью Ansible collection cozystack.installer

## Руководства для особых случаев развертывания Cozystack
[Руководства для особых случаев развертывания Cozystack](https://cozystack.ru/docs/v1.5/install/how-to/)