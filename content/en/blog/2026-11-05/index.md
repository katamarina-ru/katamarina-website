---
title: "Релиз Katamarina Cozystack v1.7.0 доступен."
slug: release17
date: 2026-11-05
author: "Katamarina"
description: "Релиз Katamarina Cozystack v1.7.0 доступен"
article_types:
  - tech-article
topics:
  - platform
---

Katamarina Cozystack v1.7.0 доступен.

🚀 Появилась поддержка Proxmox VE для размещения рабочих узлов тенантных Kubernetes-кластеров.

⚙️ Пулы узлов стали отдельными ресурсами KubernetesNodes. При обновлении рабочие ВМ будут однократно пересозданы.

🛍 Добавили community Marketplace для внешних приложений, обновили дашборд и административный портал.

💾 Расширили единый интерфейс бэкапов на MongoDB, Redis, Valkey, RabbitMQ и Kafka. PostgreSQL получил восстановление на заданный момент времени — PITR.

🔐 Усилили защиту внешних подключений к базам: Redis требует TLS, PostgreSQL по умолчанию отклоняет незашифрованные соединения. Перед обновлением проверьте настройки клиентов.

Рекомендуется ознакомиться с разделом об обновлении перед применением релиза.

Ссылки:

Подробности релиза:
https://github.com/cozystack/cozystack/releases#release-v1.7.0

Документация (уже доступен русскоязычный перевод✊): 
https://cozystack.ru/docs/v1.7/
