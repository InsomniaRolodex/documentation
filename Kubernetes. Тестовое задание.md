# Тестовое задание для стажёра технического писателя

## Документация Kubernetes

[Обзор кластера](#обзор-кластера)

[Работа с namespaces](#работа-с-namespaces)

[Описание деплойментов](#описание-деплойментов)

[Проверка статуса подов](#проверка-статуса-подов)

[Просмотр логов](#просмотр-логов)

[Частые проблемы и их решения](#частые-проблемы-и-их-решения)

### Обзор кластера

Управляющий узел (control plane) запущен по адресу со стандартным портом для API: 6443

CoreDNS запущен и принимает запросы. Он находится в системном пространстве имён kube-system.

Сам кластер раздёлен на несколько групп через namespaces:

* default
* kube-node-lease
* kube-public
* kube-system
* development
* production
* staging

Команда для получения дополнительной информации и диагностики возможных проблем: `kubectl cluster-info dump`

### Работа с namespaces

В данном кластере существует ряд пространств имён. Из них три базовых, созданных по умолчанию: `default`, `kube-public`, `kube-system`. Для получения полного списка введите команду `kubectl get namespace`:

```
NAME STATUS AGE
default Active 365d
kube-node-lease Active 365d
kube-public Active 365d
kube-system Active 365d
development Active 180d
production Active 180d
staging Active 180d
```

Существует два способа создания нового пространства имён:

1. Создайте файл my-namespace.yaml с содержимым:
```
apiVersion: v1
kind: Namespace
metadata:
  name: <имя нового пространства имён>
```
Запустите команду `kubectl create -f ./my-namespace.yaml`

2. `kubectl create namespace <имя нового пространства имён>`

Для **переключения между пространствами имён** существует два способа:
1. Команда `kubectl config set-context --current --namespace=<NAME>`
2. Утилита kubens. Позволяет переключаться быстрее: `kubens <NAME>`

Более подробную информацию о работе с пространствами имён вы можете прочитать в  [документации Kubernetes](https://kubernetes.io/ru/docs/concepts/overview/working-with-objects/namespaces/)

### Описание деплойментов

В неймспейсе development находятся следующие деплойменты:

* auth-service — сервис аутентификации (2 пода)
* api-gateway — входная точка для запросов (3 пода)
* user-service — работа с пользователями (2 пода)
* payment-service — платежи (1 под)
* notification-service — уведомления (1 под)

Все приложения запущены и обновлены до актуальных версий.

### Проверка статуса подов

Для проверки статуса подов введите команду `Команда: kubectl get pods -n <namespace>`

Информация о подах на текущем кластере в пространстве имён `development`:

|NAME|READY|STATUS|RESTARTS|AGE|
|-|-|-|-|-|
|auth-service-5f8d9c7b4d-x7k2m| 1/1| Running| 0| 2d|
|auth-service-5f8d9c7b4d-p9n3q| 1/1| Running| 0| 2d|
|api-gateway-6c7b8d9f5c-m4k8p| 1/1| Running| 0| 5d|
|api-gateway-6c7b8d9f5c-n2w7r| 1/1| Running| 0| 5d|
|api-gateway-6c7b8d9f5c-q1t5s| 1/1| Running| 0| 5d|
|user-service-7d8e9f0a6d-k3j6h| 1/1| Running| 0| 1d|
|user-service-7d8e9f0a6d-l5m9n| 1/1| Running| 0| 1d|
|payment-service-8e9f0a1b7e-o2p4q| 1/1| Running| 0| 12h|
|notification-service-9f0a1b2c8f-r3s5t| 1/1| Running| 0| 6h|

В данном кластере девять подов, в каждом по одному контейнеру. Все поды запущены и не перезапускались. Время работы обозначено в столбце AGE.

### Просмотр логов

Просмотр логов осуществляется командой `kubectl logs auth-service-5f8d9c7b4d-x7k2m -n <namespace> --tail=<количество логов>`

Прим. логов:

```
2024-01-15 10:23:45 INFO [main] Starting AuthService application
2024-01-15 10:23:46 INFO [main] Loading configuration from environment
2024-01-15 10:23:47 INFO [main] Connecting to database at
postgres://db:5432/auth
2024-01-15 10:23:48 INFO [main] Database connection established
technical-writer-intern-test.md 2026-03-02
4 / 5
2024-01-15 10:23:49 INFO [main] Starting HTTP server on port 8080
2024-01-15 10:23:50 INFO [main] AuthService started successfully
2024-01-15 10:25:12 INFO [http-handler] POST /api/v1/login - 200 OK - 45ms
2024-01-15 10:25:45 INFO [http-handler] POST /api/v1/login - 200 OK - 38ms
2024-01-15 10:26:01 INFO [http-handler] POST /api/v1/refresh - 200 OK -
12ms
2024-01-15 10:26:33 INFO [http-handler] POST /api/v1/login - 401
Unauthorized - 5ms
2024-01-15 10:27:15 INFO [http-handler] POST /api/v1/login - 200 OK - 42ms
2024-01-15 10:28:00 INFO [http-handler] POST /api/v1/logout - 200 OK - 3ms
2024-01-15 10:28:45 INFO [http-handler] POST /api/v1/login - 200 OK - 51ms
2024-01-15 10:29:12 INFO [http-handler] POST /api/v1/refresh - 200 OK -
15ms
2024-01-15 10:29:55 INFO [http-handler] POST /api/v1/login - 200 OK - 39ms
2024-01-15 10:30:22 INFO [http-handler] POST /api/v1/login - 401
Unauthorized - 4ms
2024-01-15 10:31:00 INFO [http-handler] POST /api/v1/login - 200 OK - 44ms
2024-01-15 10:31:30 INFO [http-handler] POST /api/v1/logout - 200 OK - 2ms
2024-01-15 10:32:15 INFO [http-handler] POST /api/v1/login - 200 OK - 47ms
```

Здесь описан процесс успешного запуска приложения, установка соединения и серия действий пользователей: login, logout, refresh.

### Частые проблемы и их решения

**Нехватка памяти**. Если контейнер превысил доступный объем предоставленной памяти, он будет остановлен системой. Чтобы решить эту проблему:
* Увеличьте лимит памяти в  деплоймента deployment-name.yaml.

* Оптимизируйте приложение, чтобы оно потребляло меньше ресурсов.

* Используйте команду `kubectl describe pod <имя пода>`, чтобы проверить текущие лимиты и реальное потребление памяти.

**Контейнер не смог запуститься**

* Используйте `kubectl logs <имя пода>` или `kubectl describe pod` для диагностики.

* Проверьте корректность команды и аргументов в манифесте yaml.

* Убедитесь в наличии нужных файлов, прав доступа и разрешений.

Более полное описание Kubernetes, его возможностей, команд и методов можно прочитать в [официальной документации Kubernetes](https://kubernetes.io/ru/docs/home/).