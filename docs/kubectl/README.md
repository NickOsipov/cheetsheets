# 🚀 kubectl Cheatsheet

Полное руководство по командам kubectl для управления кластерами Kubernetes

---

## 📋 Содержание

- [Автодополнение](#🔧-автодополнение)
- [Контекст и конфигурация](#🎯-контекст-и-конфигурация)
- [Создание ресурсов](#🏗️-создание-ресурсов)
- [Просмотр ресурсов](#👀-просмотр-ресурсов)
- [Обновление ресурсов](#🔄-обновление-ресурсов)
- [Патчинг ресурсов](#🩹-патчинг-ресурсов)
- [Масштабирование](#📏-масштабирование)
- [Удаление ресурсов](#🗑️-удаление-ресурсов)
- [Работа с подами](#🏃‍♂️-работа-с-подами)
- [Копирование файлов](#📂-копирование-файлов)
- [Работа с нодами](#🖥️-работа-с-нодами)
- [Форматирование вывода](#📊-форматирование-вывода)
- [Отладка](#🔍-отладка)
- [Полезные ресурсы](#🛠️-полезные-ресурсы)
- [Быстрые команды](#🎯-быстрые-команды-для-ежедневной-работы)

---

## 🔧 Автодополнение

### Bash
```bash
# Настройка автодополнения в текущем сеансе
source <(kubectl completion bash)

# Добавить автодополнение постоянно
echo "source <(kubectl completion bash)" >> ~/.bashrc

# Алиас с автодополнением
alias k=kubectl
complete -o default -F __start_kubectl k
```

### Zsh
```zsh
# Настройка автодополнения в zsh
source <(kubectl completion zsh)

# Добавить в ~/.zshrc
echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc
```

### Fish
```fish
# Для kubectl версии 1.23+
echo 'kubectl completion fish | source' > ~/.config/fish/completions/kubectl.fish
source ~/.config/fish/completions/kubectl.fish
```

**💡 Полезный трюк:** `kubectl -A` = `kubectl --all-namespaces`

---

## 🎯 Контекст и конфигурация

### Просмотр конфигурации
```bash
# Показать объединенную конфигурацию kubeconfig
kubectl config view

# Показать сырую конфигурацию с сертификатами
kubectl config view --raw

# Получить пароль пользователя e2e
kubectl config view -o jsonpath='{.users[?(@.name == "e2e")].user.password}'

# Список всех пользователей
kubectl config view -o jsonpath='{.users[*].name}'
```

### Управление контекстами
```bash
# Показать список контекстов
kubectl config get-contexts

# Показать только имена контекстов
kubectl config get-contexts -o name

# Показать текущий контекст
kubectl config current-context

# Переключить контекст
kubectl config use-context my-cluster-name

# Установить namespace по умолчанию для текущего контекста
kubectl config set-context --current --namespace=my-namespace
```

### Управление кластерами и пользователями
```bash
# Добавить кластер
kubectl config set-cluster my-cluster-name

# Добавить пользователя с базовой аутентификацией
kubectl config set-credentials kubeuser --username=user --password=pass

# Создать новый контекст
kubectl config set-context my-context --user=cluster-admin --namespace=foo

# Удалить пользователя
kubectl config unset users.foo
```

### Полезные алиасы
```bash
# Быстрое переключение контекста
alias kx='f() { [ "$1" ] && kubectl config use-context $1 || kubectl config current-context ; } ; f'

# Быстрое переключение namespace
alias kn='f() { [ "$1" ] && kubectl config set-context --current --namespace $1 || kubectl config view --minify | grep namespace | cut -d" " -f6 ; } ; f'
```

---

## 🏗️ Создание ресурсов

### Применение манифестов
```bash
# Создать ресурс из файла
kubectl apply -f ./manifest.yaml

# Создать из нескольких файлов
kubectl apply -f ./file1.yaml -f ./file2.yaml

# Создать все ресурсы в директории
kubectl apply -f ./dir/

# Создать ресурс из URL
kubectl apply -f https://example.com/manifest.yaml
```

### Быстрое создание ресурсов
```bash
# Создать Deployment с nginx
kubectl create deployment nginx --image=nginx

# Создать Job
kubectl create job hello --image=busybox:1.28 -- echo "Hello World"

# Создать CronJob
kubectl create cronjob hello --image=busybox:1.28 --schedule="*/1 * * * *" -- echo "Hello World"

# Получить документацию по Pod манифестам
kubectl explain pods
```

### Создание из stdin
```bash
# Создать несколько объектов из stdin
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args: [sleep, "1000000"]
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox:1.28
    args: [sleep, "1000"]
EOF
```

### Создание Secret
```bash
# Создать Secret с несколькими ключами
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF
```

---

## 👀 Просмотр ресурсов

### Базовые команды просмотра
```bash
# Список сервисов в текущем namespace
kubectl get services

# Список всех подов во всех namespace
kubectl get pods --all-namespaces
kubectl get pods -A  # сокращенная форма

# Подробная информация о подах
kubectl get pods -o wide

# Получить конкретный Deployment
kubectl get deployment my-dep

# Получить YAML пода
kubectl get pod my-pod -o yaml
```

### Подробное описание
```bash
# Описать ноду
kubectl describe nodes my-node

# Описать под
kubectl describe pods my-pod
```

### Сортировка и фильтрация
```bash
# Сортировать сервисы по имени
kubectl get services --sort-by=.metadata.name

# Сортировать поды по количеству перезапусков
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# Сортировать PersistentVolumes по размеру
kubectl get pv --sort-by=.spec.capacity.storage

# Получить поды с определенным лейблом
kubectl get pods --selector=app=cassandra

# Получить только запущенные поды
kubectl get pods --field-selector=status.phase=Running

# Показать лейблы для всех подов
kubectl get pods --show-labels
```

### JSONPath запросы
```bash
# Получить версии подов с лейблом app=cassandra
kubectl get pods --selector=app=cassandra -o jsonpath='{.items[*].metadata.labels.version}'

# Получить значение ключа с точками
kubectl get configmap myconfig -o jsonpath='{.data.ca\.crt}'

# Получить base64 значение с дефисами
kubectl get secret my-secret --template='{{index .data "key-name-with-dashes"}}'

# Получить внешние IP всех нод
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# Показать готовые ноды
kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' | grep "Ready=True"
```

### Полезные команды
```bash
# Декодировать секреты без внешних инструментов  
kubectl get secret my-secret -o go-template='{{range $k,$v := .data}}{{"### "}}{{$k}}{{"\n"}}{{$v|base64decode}}{{"\n\n"}}{{end}}'

# Список событий, отсортированных по времени
kubectl get events --sort-by=.metadata.creationTimestamp

# Только предупреждения
kubectl events --types=Warning

# Сравнить текущее состояние с манифестом
kubectl diff -f ./my-manifest.yaml

# Получить статус субресурса Deployment
kubectl get deployment nginx-deployment --subresource=status
```

---

## 🔄 Обновление ресурсов

### Rolling Updates
```bash
# Обновить образ в Deployment
kubectl set image deployment/frontend www=image:v2

# История развертываний
kubectl rollout history deployment/frontend

# Откатить к предыдущей версии
kubectl rollout undo deployment/frontend

# Откатить к конкретной ревизии
kubectl rollout undo deployment/frontend --to-revision=2

# Следить за статусом обновления
kubectl rollout status -w deployment/frontend

# Перезапустить Deployment
kubectl rollout restart deployment/frontend
```

### Замена ресурсов
```bash
# Заменить под на основе JSON из stdin
cat pod.json | kubectl replace -f -

# Принудительная замена (с простоем сервиса)
kubectl replace --force -f ./pod.json

# Обновить версию образа пода
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -
```

### Сервисы и экспозиция
```bash
# Создать сервис для nginx на порту 80
kubectl expose rc nginx --port=80 --target-port=8000

# Автомасштабирование
kubectl autoscale deployment foo --min=2 --max=10
```

### Лейблы и аннотации
```bash
# Добавить лейбл
kubectl label pods my-pod new-label=awesome

# Удалить лейбл
kubectl label pods my-pod new-label-

# Перезаписать лейбл
kubectl label pods my-pod new-label=new-value --overwrite

# Добавить аннотацию
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq

# Удалить аннотацию
kubectl annotate pods my-pod icon-url-
```

---

## 🩹 Патчинг ресурсов

### Strategic Merge Patch
```bash
# Пометить ноду как неразмещаемую
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'

# Обновить образ контейнера
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'
```

### JSON Patch
```bash
# Обновить образ используя JSON patch с позиционными массивами
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# Отключить livenessProbe
kubectl patch deployment valid-deployment --type json -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'

# Добавить элемент в позиционный массив
kubectl patch sa default --type='json' -p='[{"op": "add", "path": "/secrets/1", "value": {"name": "whatever"}}]'

# Обновить количество реплик через subresource
kubectl patch deployment nginx-deployment --subresource='scale' --type='merge' -p '{"spec":{"replicas":2}}'
```

---

## 📏 Масштабирование

```bash
# Масштабировать ReplicaSet до 3 реплик
kubectl scale --replicas=3 rs/foo

# Масштабировать ресурс из файла
kubectl scale --replicas=3 -f foo.yaml

# Условное масштабирование
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql

# Масштабировать несколько контроллеров
kubectl scale --replicas=5 rc/foo rc/bar rc/baz
```

---

## 🗑️ Удаление ресурсов

```bash
# Удалить под используя тип и имя из файла
kubectl delete -f ./pod.json

# Удалить под без grace period
kubectl delete pod unwanted --now

# Удалить поды и сервисы с одинаковыми именами
kubectl delete pod,service baz foo

# Удалить по лейблу
kubectl delete pods,services -l name=myLabel

# Удалить все в namespace
kubectl -n my-ns delete pod,svc --all

# Удалить поды по паттерну
kubectl get pods -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}' | xargs kubectl delete -n mynamespace pod
```

---

## 🏃‍♂️ Работа с подами

### Логи
```bash
# Показать логи пода
kubectl logs my-pod

# Логи по лейблу
kubectl logs -l name=myLabel

# Предыдущие логи
kubectl logs my-pod --previous

# Логи конкретного контейнера (multi-container)
kubectl logs my-pod -c my-container

# Следить за логами в реальном времени
kubectl logs -f my-pod

# Логи всех контейнеров с лейблом
kubectl logs -f -l name=myLabel --all-containers
```

### Выполнение команд
```bash
# Запустить интерактивную оболочку
kubectl run -i --tty busybox --image=busybox:1.28 -- sh

# Запустить nginx в определенном namespace
kubectl run nginx --image=nginx -n mynamespace

# Сгенерировать YAML без создания
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# Подключиться к работающему контейнеру
kubectl attach my-pod -i

# Проброс портов
kubectl port-forward my-pod 5000:6000

# Выполнить команду в поде
kubectl exec my-pod -- ls /

# Интерактивная оболочка
kubectl exec --stdin --tty my-pod -- /bin/sh

# Multi-container случай
kubectl exec my-pod -c my-container -- ls /
```

### Отладка
```bash
# Создать сеанс отладки в существующем поде
kubectl debug my-pod -it --image=busybox:1.28

# Отладка ноды
kubectl debug node/my-node -it --image=busybox:1.28
```

### Метрики
```bash
# Показать метрики всех подов
kubectl top pod

# Метрики конкретного пода с контейнерами
kubectl top pod POD_NAME --containers

# Сортировать по CPU
kubectl top pod POD_NAME --sort-by=cpu
```

---

## 📂 Копирование файлов

```bash
# Копировать локальную директорию в под
kubectl cp /tmp/foo_dir my-pod:/tmp/bar_dir

# Копировать в конкретный контейнер
kubectl cp /tmp/foo my-pod:/tmp/bar -c my-container

# Копировать в pod в другом namespace
kubectl cp /tmp/foo my-namespace/my-pod:/tmp/bar

# Копировать из пода локально
kubectl cp my-namespace/my-pod:/tmp/foo /tmp/bar
```

### Альтернативный способ с tar
```bash
# Копировать в под используя tar
tar cf - /tmp/foo | kubectl exec -i -n my-namespace my-pod -- tar xf - -C /tmp/bar

# Копировать из пода используя tar
kubectl exec -n my-namespace my-pod -- tar cf - /tmp/foo | tar xf - -C /tmp/bar
```

---

## 🖥️ Работа с нодами

```bash
# Пометить ноду как неразмещаемую
kubectl cordon my-node

# Осушить ноду для обслуживания
kubectl drain my-node

# Вернуть ноду в работу
kubectl uncordon my-node

# Метрики всех нод
kubectl top node

# Метрики конкретной ноды
kubectl top node my-node

# Информация о кластере
kubectl cluster-info

# Дамп состояния кластера
kubectl cluster-info dump

# Дамп в директорию
kubectl cluster-info dump --output-directory=/path/to/cluster-state
```

### Управление Taints
```bash
# Посмотреть существующие taints
kubectl get nodes -o='custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect'

# Добавить taint
kubectl taint nodes foo dedicated=special-user:NoSchedule
```

---

## 📊 Форматирование вывода

| Формат | Описание |
|--------|----------|
| `-o=custom-columns=<spec>` | Таблица с пользовательскими колонками |
| `-o=custom-columns-file=<filename>` | Таблица из файла шаблона |
| `-o=go-template=<template>` | Go шаблон |
| `-o=go-template-file=<filename>` | Go шаблон из файла |
| `-o=json` | JSON формат |
| `-o=jsonpath=<template>` | JSONPath выражение |
| `-o=jsonpath-file=<filename>` | JSONPath из файла |
| `-o=name` | Только имя ресурса |
| `-o=wide` | Дополнительная информация |
| `-o=yaml` | YAML формат |

### Примеры custom-columns
```bash
# Все образы в кластере
kubectl get pods -A -o=custom-columns='DATA:spec.containers[*].image'

# Образы в namespace default
kubectl get pods --namespace default --output=custom-columns="NAME:.metadata.name,IMAGE:.spec.containers[*].image"

# Исключить определенный образ
kubectl get pods -A -o=custom-columns='DATA:spec.containers[?(@.image!="registry.k8s.io/coredns:1.6.2")].image'

# Все поля метаданных
kubectl get pods -A -o=custom-columns='DATA:metadata.*'
```

---

## 🔍 Отладка

### Уровни детализации
| Уровень | Описание |
|---------|----------|
| `--v=0` | Только важная информация |
| `--v=1` | Разумный уровень по умолчанию |
| `--v=2` | Полезная информация о состоянии сервиса |
| `--v=3` | Расширенная информация об изменениях |
| `--v=4` | Уровень отладки |
| `--v=5` | Уровень трассировки |
| `--v=6` | Отображение запрашиваемых ресурсов |
| `--v=7` | Заголовки HTTP запросов |
| `--v=8` | Содержимое HTTP запросов |
| `--v=9` | Полное содержимое HTTP запросов |

### Примеры отладки
```bash
# Отладка на уровне 4
kubectl get pods --v=4

# Показать HTTP заголовки
kubectl get pods --v=7

# Полная трассировка
kubectl get pods --v=9
```

---

## 🛠️ Полезные ресурсы

### API ресурсы
```bash
# Список всех поддерживаемых типов ресурсов
kubectl api-resources

# Только ресурсы в namespace
kubectl api-resources --namespaced=true

# Ресурсы не в namespace
kubectl api-resources --namespaced=false

# Только имена ресурсов
kubectl api-resources -o name

# Расширенный вывод
kubectl api-resources -o wide

# Ресурсы поддерживающие list и get
kubectl api-resources --verbs=list,get

# Ресурсы в конкретной API группе
kubectl api-resources --api-group=extensions
```

### Работа с Deployments и Services
```bash
# Логи Deployment
kubectl logs deploy/my-deployment

# Логи конкретного контейнера в Deployment
kubectl logs deploy/my-deployment -c my-container

# Проброс портов к Service
kubectl port-forward svc/my-service 5000

# Проброс с именованным портом
kubectl port-forward svc/my-service 5000:my-service-port

# Проброс к Deployment
kubectl port-forward deploy/my-deployment 5000:6000

# Выполнить команду в первом поде Deployment
kubectl exec deploy/my-deployment -- ls
```

---

## 🎯 Быстрые команды для ежедневной работы

```bash
# Общий алиас для kubectl
alias k='kubectl'

# Быстрый доступ к подам
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kgn='kubectl get nodes'

# Описание ресурсов
alias kdp='kubectl describe pod'
alias kds='kubectl describe service'
alias kdd='kubectl describe deployment'

# Логи и выполнение
alias kl='kubectl logs'
alias ke='kubectl exec -it'

# Применение манифестов
alias ka='kubectl apply -f'
alias kd='kubectl delete -f'
```

---

**📚 Документация:** [kubernetes.io/docs/reference/kubectl](https://kubernetes.io/docs/reference/kubectl/)

**🎯 JSONPath:** [kubernetes.io/docs/reference/kubectl/jsonpath](https://kubernetes.io/docs/reference/kubectl/jsonpath/)

**⚙️ Опции kubectl:** [kubernetes.io/docs/reference/kubectl/kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/)

---

*Этот cheatsheet охватывает основные команды kubectl для ежедневной работы с Kubernetes. Сохраните его для быстрого доступа к нужным командам!*