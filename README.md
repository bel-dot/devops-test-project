## Розгортання в Kubernetes

### Конфігураційні параметри

#### ConfigMap (`cartservice-config`)



Використовується для передачі загальних параметрів конфігурації у вигляді змінних оточення:

| Ключ | Значення | Опис |
| --- | --- | --- |
| `PORT` | `8080` | Мережевий порт gRPC-сервера застосунку |
| `LISTEN_ADDR` | `0.0.0.0` | Адреса прив'язки сокета сервера |

#### Secret (`cartservice-secret`)



Використовується для передачі адреси підключення до сховища:

| Ключ | Значення | Опис |
| --- | --- | --- |
| `REDIS_ADDR` | `redis-cart:6379` | Внутрішня DNS-адреса сервісу Redis у кластері |

---

### Порядок розгортання

Крок 1. Запуск кластера Minikube та активація аддонів:

```bash
minikube start --driver=docker
minikube addons enable metrics-server
minikube addons enable ingress

```

Крок 2. Завантаження локально зібраних образів у Minikube:

```bash
minikube image load ghcr.io/bel-dot/devops-test-project/cartservice:1.0.0
minikube image load ghcr.io/bel-dot/devops-test-project/cartservice:1.1.0

```

Крок 3. Застосування всіх маніфестів однією командою:

```bash
kubectl apply -f k8s/

```

Крок 4. Перевірка стану об'єктів у просторі імен devops-biliakov:

```bash
kubectl get pods,deployments,replicasets,services,pvc -n devops-biliakov

```

Крок 5. Перевірка доступності сервісу через HTTP/2:

```bash
curl -I --http2-prior-knowledge http://$(minikube ip):30080

```

## Пакування застосунку в Helm-чарт

Для автоматизації розгортання та керування конфігураціями між різними середовищами (`dev` та `prod`) створено власний Helm-чарт у каталозі `chart/`, який описує зв'язку мікросервісу `cartservice` та сховища `redis`.

### Версіонування в `Chart.yaml`

* **`version` (`0.1.0`)** — версія самого Helm-чарта (шаблонів, структури маніфестів та значень за замовчуванням). Збільшується при внесенні змін у конфігурацію деплою.


* **`appVersion` (`1.0.0`)** — версія самого застосунку (`cartservice`), який розгортається в контейнерах. Має інформаційний характер і підставляється за замовчуванням, якщо тег образу явно не перевизначений.



### Параметри `values.yaml`

| Параметр | Опис | Значення за замовчуванням |
| --- | --- | --- |
| `replicaCount` | Кількість реплік `cartservice` | `1` |
| `image.repository` | Адреса реєстру та назва образу застосунку | `ghcr.io/bel-dot/devops-test-project/cartservice` |
| `image.pullPolicy` | Політика завантаження Docker-образу | `IfNotPresent` |
| `image.tag` | Тег версії образу застосунку | `"1.0.0"` |
| `service.type` | Тип Kubernetes Service (`NodePort`, `ClusterIP`) | `NodePort` |
| `service.port` | Порт сервісу та контейнера gRPC | `8080` |
| `service.nodePort` | Зовнішній порт вузла (для типу `NodePort`) | `30080` |
| `config.port` | Значення `PORT` у ConfigMap | `"8080"` |
| `config.listenAddr` | Значення `LISTEN_ADDR` у ConfigMap | `"0.0.0.0"` |
| `secret.redisAddr` | Адреса підключення до Redis у Secret | `"redis-cart:6379"` |
| `resources` | Запити та ліміти CPU і RAM для `cartservice` | `50m/64Mi` – `150m/128Mi` |
| `ingress.enabled` | Увімкнення створення об'єкта Ingress | `false` |
| `ingress.className` | Клас Ingress-контролера | `"nginx"` |
| `redis.enabled` | Увімкнення розгортання бази даних Redis | `true` |
| `redis.image.repository` | Образ бази даних Redis | `redis` |
| `redis.image.tag` | Тег образу Redis | `"alpine"` |
| `redis.persistence.enabled` | Увімкнення PersistentVolumeClaim для Redis | `true` |
| `redis.persistence.size` | Розмір виділеного тому PVC | `256Mi` |

### Пріоритет перевизначення значень

Під час рендеру або встановлення чарта Helm застосовує значення у такому порядку (від найнижчого пріоритету до найвищого):

1. Базовий файл `chart/values.yaml` (значення за замовчуванням).


2. Файли середовищ, передані через прапорець `-f` (наприклад, `-f chart/values-prod.yaml` перезаписує базові поля).


3. Окремі параметри, передані через прапорець `--set` (мають найвищий пріоритет і перевизначають будь-які файли).



### Команди встановлення та керування релізом

Перевірка синтаксису та рендер шаблонів без встановлення в кластер:

```bash
helm lint chart/
helm template cartservice-release chart/ -f chart/values-prod.yaml

```

Встановлення у середовище розробки (**Dev**):

```bash
helm install cartservice-release chart/ -f chart/values-dev.yaml -n devops-biliakov --create-namespace

```

Встановлення або оновлення для продакшн-середовища (**Prod**):

```bash
helm upgrade --install cartservice-release chart/ -f chart/values-prod.yaml -n devops-biliakov --create-namespace

```

Перегляд історії, відкат та видалення релізу:

```bash
helm history cartservice-release -n devops-biliakov
helm rollback cartservice-release 1 -n devops-biliakov
helm uninstall cartservice-release -n devops-biliakov

```

