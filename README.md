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
