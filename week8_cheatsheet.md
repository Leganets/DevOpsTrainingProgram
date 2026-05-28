# Шпаргалка: Неделя 8 — Kubernetes Basics

## Урок 31: Kubernetes архитектура и основы

### Что такое Kubernetes?

**Kubernetes (K8s)** — система оркестрации контейнеров. Автоматизирует развёртывание, масштабирование и управление контейнеризированными приложениями.

**Зачем нужен:**
- ✅ Масштабирование (запуск нескольких копий приложения)
- ✅ Self-healing (автоматический перезапуск упавших контейнеров)
- ✅ Rolling updates (обновление без downtime)
- ✅ Load balancing (распределение нагрузки)
- ✅ Работа на кластере серверов (не только один сервер)

---

### Архитектура Kubernetes

**Control Plane (Master Node)** — управляющий узел:
- **API Server** — точка входа в кластер (все команды идут сюда)
- **etcd** — база данных кластера (хранит состояние)
- **Scheduler** — решает, на каком узле запустить Pod
- **Controller Manager** — следит, чтобы желаемое состояние = реальному

**Worker Nodes** — рабочие узлы (где запускаются контейнеры):
- **kubelet** — агент на узле, запускает контейнеры
- **kube-proxy** — управляет сетью
- **Container Runtime** — Docker/containerd (запускает контейнеры)

---

### Основные объекты Kubernetes

#### 1. Pod

**Pod** — минимальная единица в Kubernetes. Обёртка вокруг одного или нескольких контейнеров.

**Особенности:**
- Контейнеры в одном Pod делят сеть (localhost работает между ними)
- Контейнеры в одном Pod делят storage (volumes)
- Обычно 1 Pod = 1 контейнер (best practice)

**Пример:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
```

---

#### 2. Deployment

**Deployment** — управляет несколькими копиями Pod (ReplicaSet).

**Зачем:**
- Запустить N копий приложения
- Автоматический перезапуск при падении
- Rolling updates (обновление без downtime)
- Rollback (откат к предыдущей версии)

**Пример:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3    # 3 копии Pod
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

---

#### 3. Service

**Service** — точка доступа к Pod. Балансирует нагрузку между копиями.

**Зачем:**
- Pod имеют динамические IP (меняются при перезапуске)
- Service даёт постоянный IP и DNS имя
- Балансирует запросы между Pod

**Типы Service:**
- **ClusterIP** — доступен только внутри кластера (по умолчанию)
- **NodePort** — доступен снаружи через порт узла (30000-32767)
- **LoadBalancer** — создаёт внешний балансировщик (облако)

**Пример:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx    # Выбирает Pod с label app=nginx
  ports:
  - port: 80           # Порт Service
    targetPort: 80     # Порт контейнера
    nodePort: 30080    # Порт на узле (30000-32767)
```

---

### kubectl — CLI для Kubernetes

**kubectl** — главный инструмент для работы с Kubernetes.

#### Информация о кластере
```bash
kubectl cluster-info
kubectl get nodes
```

#### Работа с Pod
```bash
kubectl get pods                    # Список Pod
kubectl describe pod <pod-name>     # Детальная информация
kubectl logs <pod-name>             # Логи
kubectl exec -it <pod-name> -- sh   # Зайти в контейнер
```

#### Работа с Deployment
```bash
kubectl get deployments                           # Список Deployment
kubectl describe deployment <name>                # Детали
kubectl scale deployment <name> --replicas=5      # Масштабирование
```

#### Работа с Service
```bash
kubectl get services                # Список Service
kubectl describe service <name>     # Детали
```

#### Применить конфигурацию
```bash
kubectl apply -f deployment.yaml    # Создать/обновить
kubectl delete -f deployment.yaml   # Удалить
```

---

### k3s — облегчённый Kubernetes

**k3s** — облегчённая версия Kubernetes от Rancher.

**Преимущества:**
- ✅ Требует всего 512MB RAM и 1 CPU
- ✅ Быстрая установка (одна команда)
- ✅ Полноценный Kubernetes (не эмулятор)
- ✅ Использует те же kubectl команды
- ✅ Идеален для обучения и production на слабых серверах

**Установка k3s:**
```bash
# Установить k3s
curl -sfL https://get.k3s.io | sh -

# Проверить статус
systemctl status k3s

# Настроить kubectl
mkdir -p ~/.kube
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
chown $(id -u):$(id -g) ~/.kube/config
chmod 600 ~/.kube/config

# Проверить узлы
kubectl get nodes
```

---

### Docker Compose vs Kubernetes

| Критерий | Docker Compose | Kubernetes |
|----------|----------------|------------|
| Масштаб | Один сервер | Кластер серверов |
| Масштабирование | Ручное | Автоматическое |
| Self-healing | Restart policy | Автоматический перезапуск |
| Load balancing | Nginx вручную | Встроенный (Service) |
| Rolling updates | Нет | Да |
| Сложность | Простой | Сложный |
| Use case | Dev/Test | Production |

---

### Минимальные требования для k3s

**Официальные требования:**
- **CPU:** 1 ядро (минимум), 2 ядра (рекомендуется)
- **RAM:** 512MB (минимум), 1GB (рекомендуется)
- **Диск:** 10GB (минимум), 20GB (рекомендуется)

**Рекомендуемая конфигурация для обучения:**
- **CPU:** 2 ядра
- **RAM:** 2GB
- **Диск:** 20GB SSD
- **OS:** Ubuntu 22.04 LTS

---

### Основные команды (краткая справка)

```bash
# Кластер
kubectl get nodes
kubectl cluster-info

# Pod
kubectl get pods
kubectl describe pod <name>
kubectl logs <name>
kubectl exec -it <name> -- sh

# Deployment
kubectl get deployments
kubectl scale deployment <name> --replicas=N
kubectl describe deployment <name>

# Service
kubectl get services
kubectl describe service <name>

# Применить конфигурацию
kubectl apply -f file.yaml
kubectl delete -f file.yaml

# k3s
systemctl status k3s
systemctl restart k3s
```

---

### Типичные ошибки (не повторяй!)

1. ❌ Забыть `labels` в Pod template (selector не найдёт Pod)
2. ❌ Неправильный `selector` в Service (не совпадает с labels Pod)
3. ❌ NodePort вне диапазона 30000-32767
4. ❌ Забыть `containerPort` в Pod spec
5. ❌ Использовать GET вместо POST для Flask API (405 Method Not Allowed)

---

### Best Practices

1. ✅ Используй Deployment вместо голых Pod (автоматический перезапуск)
2. ✅ Всегда указывай labels для Pod (для selector)
3. ✅ Используй NodePort для тестирования, LoadBalancer для production
4. ✅ Масштабируй через `kubectl scale`, а не редактируй YAML вручную
5. ✅ Проверяй статус через `kubectl get pods` перед тестированием
6. ✅ Используй `kubectl describe` для диагностики проблем
7. ✅ Логи через `kubectl logs`, а не заходи в контейнер

---

**Повтори эту шпаргалку перед следующим уроком!**
