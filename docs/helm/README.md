# 🚀 Helm Cheatsheet

Полное руководство по командам Helm для управления приложениями в Kubernetes

---

## 📋 Содержание

- [Автодополнение](#🔧-автодополнение)
- [Управление репозиториями](#📦-управление-репозиториями)
- [Управление чартами](#📋-управление-чартами)
- [Установка и удаление](#🏗️-установка-и-удаление)
- [Обновление и откат](#🔄-обновление-и-откат)
- [Мониторинг релизов](#👀-мониторинг-релизов)
- [Работа со значениями](#⚙️-работа-со-значениями)
- [Получение информации](#📊-получение-информации)
- [Плагины](#🔌-плагины)
- [Отладка](#🔍-отладка)
- [Полезные ресурсы](#🛠️-полезные-ресурсы)
- [Быстрые команды](#🎯-быстрые-команды-для-ежедневной-работы)

---

## 🔧 Автодополнение

### Bash
```bash
# Настройка автодополнения в текущем сеансе
source <(helm completion bash)

# Добавить автодополнение постоянно
echo "source <(helm completion bash)" >> ~/.bashrc

# Алиас с автодополнением
alias h=helm
complete -o default -F __start_helm h
```

### Zsh
```zsh
# Настройка автодополнения в zsh
source <(helm completion zsh)

# Добавить в ~/.zshrc
echo '[[ $commands[helm] ]] && source <(helm completion zsh)' >> ~/.zshrc
```

### Fish
```fish
# Настройка автодополнения для fish
helm completion fish | source
echo 'helm completion fish | source' > ~/.config/fish/completions/helm.fish
```

**💡 Полезный трюк:** `helm list -A` = `helm list --all-namespaces`

---

## 📦 Управление репозиториями

### Основные операции
```bash
# Добавить репозиторий
helm repo add <repo-name> <url>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# Список добавленных репозиториев
helm repo list

# Обновить информацию о доступных чартах
helm repo update

# Удалить репозиторий
helm repo remove <repo-name>

# Поиск чартов в репозиториях
helm search repo <keyword>
helm search repo <chart-name> --versions

# Поиск в Artifact Hub
helm search hub <keyword>
```

### Создание индекса репозитория
```bash
# Создать index.yaml для директории с чартами
helm repo index /path/to/charts

# Объединить с существующим индексом
helm repo index /path/to/charts --merge existing-index.yaml

# Создать индекс с указанным URL
helm repo index /path/to/charts --url https://my-charts.example.com
```

---

## 📋 Управление чартами

### Создание и разработка
```bash
# Создать новый чарт
helm create <chart-name>

# Проверить синтаксис чарта
helm lint <chart-path>/

# Упаковать чарт в .tgz архив
helm package <chart-path>/

# Упаковать с подписью
helm package <chart-path>/ --sign --key '<key-name>' --keyring ~/.gnupg/secring.gpg

# Проверить зависимости
helm dependency list <chart-path>/

# Обновить зависимости
helm dependency update <chart-path>/

# Скачать зависимости
helm dependency build <chart-path>/
```

### Скачивание чартов
```bash
# Скачать чарт
helm pull <repo>/<chart>

# Скачать и распаковать
helm pull <repo>/<chart> --untar

# Скачать конкретную версию
helm pull <repo>/<chart> --version <version>

# Скачать с верификацией
helm pull <repo>/<chart> --verify

# Скачать в конкретную директорию
helm pull <repo>/<chart> --untar --untardir ./charts
```

### Просмотр информации о чарте
```bash
# Показать всю информацию о чарте
helm show all <repo>/<chart>

# Показать только values.yaml
helm show values <repo>/<chart>

# Показать только Chart.yaml
helm show chart <repo>/<chart>

# Показать README
helm show readme <repo>/<chart>

# Показать CRDs
helm show crds <repo>/<chart>
```

---

## 🏗️ Установка и удаление

### Установка приложений
```bash
# Базовая установка
helm install <release-name> <repo>/<chart>

# Установка в конкретный namespace
helm install <release-name> <repo>/<chart> --namespace <namespace> --create-namespace

# Установка с кастомными значениями
helm install <release-name> <repo>/<chart> --set key1=value1,key2=value2

# Установка с файлом значений
helm install <release-name> <repo>/<chart> --values <values-file>.yaml

# Установка с несколькими файлами значений
helm install <release-name> <repo>/<chart> -f values1.yaml -f values2.yaml

# Dry run для тестирования
helm install <release-name> <repo>/<chart> --dry-run --debug

# Установка из локального чарта
helm install <release-name> ./<chart-path>/

# Установка из URL
helm install <release-name> <chart-url>
```

### Дополнительные опции установки
```bash
# Установка с обновлением зависимостей
helm install <release-name> <repo>/<chart> --dependency-update

# Установка с верификацией
helm install <release-name> <repo>/<chart> --verify

# Установка с timeout
helm install <release-name> <repo>/<chart> --timeout 10m

# Установка с wait (ждать готовности)
helm install <release-name> <repo>/<chart> --wait --timeout 5m

# Генерация имени релиза
helm install <repo>/<chart> --generate-name

# Установка с аннотациями
helm install <release-name> <repo>/<chart> --set-annotation key1=value1,key2=value2
```

### Удаление приложений
```bash
# Удалить релиз
helm uninstall <release-name>

# Удалить с сохранением истории
helm uninstall <release-name> --keep-history

# Удалить из конкретного namespace
helm uninstall <release-name> --namespace <namespace>

# Удалить с timeout
helm uninstall <release-name> --timeout 5m

# Dry run удаления
helm uninstall <release-name> --dry-run
```

---

## 🔄 Обновление и откат

### Обновление релизов
```bash
# Обновить релиз
helm upgrade <release-name> <repo>/<chart>

# Обновление с новой версией чарта
helm upgrade <release-name> <repo>/<chart> --version <version>

# Обновление с новыми значениями
helm upgrade <release-name> <repo>/<chart> --set key1=value1,key2=value2

# Обновление с файлом значений
helm upgrade <release-name> <repo>/<chart> --values <values-file>.yaml

# Атомарное обновление (откат при неудаче)
helm upgrade <release-name> <repo>/<chart> --atomic

# Принудительное обновление
helm upgrade <release-name> <repo>/<chart> --force

# Обновление с ожиданием готовности
helm upgrade <release-name> <repo>/<chart> --wait --timeout 10m

# Сброс значений к дефолтным
helm upgrade <release-name> <repo>/<chart> --reset-values
```

### Управление откатами
```bash
# Показать историю релиза
helm history <release-name>

# Откатиться к предыдущей версии
helm rollback <release-name>

# Откатиться к конкретной ревизии
helm rollback <release-name> <revision>

# Откат с очисткой при неудаче
helm rollback <release-name> <revision> --cleanup-on-fail

# Dry run отката
helm rollback <release-name> <revision> --dry-run

# Откат с timeout
helm rollback <release-name> <revision> --timeout 5m
```

---

## 👀 Мониторинг релизов

### Список релизов
```bash
# Показать все релизы в текущем namespace
helm list

# Показать релизы во всех namespace
helm list --all-namespaces
helm list -A

# Показать все релизы (включая неуспешные)
helm list --all

# Фильтрация по статусу
helm list --deployed      # только развернутые
helm list --failed        # только неуспешные
helm list --pending       # только ожидающие
helm list --superseded    # только замещенные
helm list --uninstalled   # только удаленные

# Сортировка по дате
helm list --date

# Фильтрация по селектору
helm list --selector <key>=<value>

# Вывод в разных форматах
helm list -o yaml
helm list -o json
helm list -o table
```

### Статус релизов
```bash
# Показать статус релиза
helm status <release-name>

# Статус конкретной ревизии
helm status <release-name> --revision <revision>

# Статус в JSON формате
helm status <release-name> -o json

# Показать заметки релиза
helm get notes <release-name>

# Показать хуки релиза
helm get hooks <release-name>
```

### История и ревизии
```bash
# История изменений релиза
helm history <release-name>

# История с максимальным количеством записей
helm history <release-name> --max 10

# История в YAML формате
helm history <release-name> -o yaml
```

---

## ⚙️ Работа со значениями

### Просмотр значений
```bash
# Показать значения релиза
helm get values <release-name>

# Показать все значения (включая дефолтные)
helm get values <release-name> --all

# Показать значения конкретной ревизии
helm get values <release-name> --revision <revision>

# Вывод в разных форматах
helm get values <release-name> -o yaml
helm get values <release-name> -o json
```

### Установка значений
```bash
# Установка через --set
helm install <release-name> <repo>/<chart> \
  --set key1=value1 \
  --set key2=value2 \
  --set key3=value3

# Установка вложенных значений
helm install <release-name> <repo>/<chart> \
  --set nested.key[0].subkey=value1 \
  --set nested.key[0].subkey2=value2

# Установка из файла
helm install <release-name> <repo>/<chart> --values values.yaml

# Установка из нескольких файлов (порядок важен)
helm install <release-name> <repo>/<chart> \
  -f base-values.yaml \
  -f env-values.yaml

# Установка из URL
helm install <release-name> <repo>/<chart> \
  --values https://example.com/values.yaml

# Установка из stdin
helm install <release-name> <repo>/<chart> --values -
```

### Шаблонизация значений
```bash
# Рендеринг шаблонов
helm template <release-name> <repo>/<chart>

# Рендеринг с кастомными значениями
helm template <release-name> <repo>/<chart> --values <values-file>.yaml

# Рендеринг конкретного файла
helm template <release-name> <repo>/<chart> --show-only templates/<template-file>.yaml

# Рендеринг в конкретный namespace
helm template <release-name> <repo>/<chart> --namespace <namespace>

# Валидация шаблонов против API сервера
helm template <release-name> <repo>/<chart> --validate
```

---

## 📊 Получение информации

### Манифесты и ресурсы
```bash
# Получить все манифесты релиза
helm get manifest <release-name>

# Получить манифесты конкретной ревизии
helm get manifest <release-name> --revision <revision>

# Получить всю информацию о релизе
helm get all <release-name>

# Получить хуки релиза
helm get hooks <release-name>

# Получить заметки релиза
helm get notes <release-name>
```

### Информация о среде
```bash
# Показать переменные окружения Helm
helm env

# Проверить версию Helm
helm version

# Проверить версию с деталями
helm version --short
```

---

## 🔌 Плагины

### Управление плагинами
```bash
# Установить плагин
helm plugin install https://github.com/databus23/helm-diff

# Список установленных плагинов
helm plugin list

# Обновить плагин
helm plugin update diff

# Удалить плагин
helm plugin uninstall diff
```

### Популярные плагины
```bash
# helm-diff - сравнение изменений
helm plugin install https://github.com/databus23/helm-diff
helm diff upgrade <release-name> <repo>/<chart> --values <values-file>.yaml

# helm-secrets - работа с секретами
helm plugin install https://github.com/jkroepke/helm-secrets

# helm-unittest - тестирование чартов
helm plugin install https://github.com/quintush/helm-unittest

# helm-s3 - репозиторий в S3
helm plugin install https://github.com/hypnoglow/helm-s3
```

---

## 🔍 Отладка

### Отладочные команды
```bash
# Dry run с отладкой
helm install <release-name> <repo>/<chart> --dry-run --debug

# Шаблонизация с отладкой
helm template <release-name> <repo>/<chart> --debug

# Проверка синтаксиса
helm lint <chart-path>/

# Тестирование чарта
helm test <release-name>

# Валидация против кластера
helm template <release-name> <repo>/<chart> --validate
```

### Troubleshooting
```bash
# Просмотр логов pod'ов релиза
kubectl logs -l app.kubernetes.io/instance=<release-name>

# Проверка событий
kubectl get events --sort-by='.lastTimestamp' | grep <release-name>

# Описание ресурсов релиза
kubectl describe deployment <resource-name>

# Проверка статуса всех ресурсов
kubectl get all -l app.kubernetes.io/instance=<release-name>
```

---

## 🛠️ Полезные ресурсы

### Официальная документация
- [Helm Documentation](https://helm.sh/docs/)
- [Chart Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Chart Template Guide](https://helm.sh/docs/chart_template_guide/)

### Репозитории чартов
- [Bitnami Charts](https://github.com/bitnami/charts)
- [Prometheus Community](https://github.com/prometheus-community/helm-charts)
- [Artifact Hub](https://artifacthub.io/)

### Инструменты
- [Helmfile](https://github.com/roboll/helmfile) - декларативное управление релизами
- [Helm Operator](https://github.com/fluxcd/helm-operator) - GitOps для Helm
- [Chart Testing](https://github.com/helm/chart-testing) - CI/CD для чартов

---

## 🎯 Быстрые команды для ежедневной работы

### Алиасы для bash/zsh
```bash
# Базовые алиасы
alias h='helm'
alias hls='helm list'
alias hlsa='helm list --all-namespaces'
alias hst='helm status'
alias hget='helm get'

# Алиасы для установки и обновления
alias hi='helm install'
alias hu='helm upgrade'
alias hun='helm uninstall'
alias hr='helm rollback'

# Алиасы для работы с репозиториями
alias hra='helm repo add'
alias hru='helm repo update'
alias hrs='helm search repo'

# Функции для быстрого доступа
hgv() { helm get values "$1" --all; }
ht() { helm template "$1" "$2" --debug; }
```

### Часто используемые команды
```bash
# Быстрая настройка популярных репозиториев
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Примеры установки популярных приложений
helm install <ingress-name> bitnami/nginx-ingress-controller
helm install <monitoring-name> prometheus-community/kube-prometheus-stack
helm install <db-name> bitnami/postgresql
helm install <cache-name> bitnami/redis

# Проверка статуса всех релизов
helm list -A --output table

# Поиск и информация о чартах
helm search repo <keyword> --versions | head -10
helm show values <repo>/<chart> | head -50
```

### Полезные One-liners
```bash
# Получить все релизы со статусом failed
helm list -A --failed

# Откатить все failed релизы
helm list -A --failed -o json | jq -r '.[] | "\(.name) \(.namespace)"' | while read name ns; do helm rollback $name --namespace $ns; done

# Показать все образы в релизе
helm get manifest <release-name> | grep 'image:' | sort -u

# Проверить что будет изменено при апгрейде
helm diff upgrade <release-name> <repo>/<chart> --values <values-file>.yaml

# Создать backup значений перед обновлением
helm get values <release-name> --all > backup-values.yaml
```

**💡 Pro Tips:**
- Всегда используйте `--dry-run --debug` перед реальной установкой
- Регулярно обновляйте репозитории чартов с `helm repo update`
- Используйте версионирование чартов в продакшене
- Сохраняйте историю релизов для быстрого отката
- Тестируйте чарты с `helm test` после установки