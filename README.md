## Задание 1

**Выполните действия:**

1. Запустите Kubernetes локально, используя k3s или minikube на свой выбор.
1. Добейтесь стабильной работы всех системных контейнеров.
2. В качестве ответа пришлите скриншот результата выполнения команды kubectl get po -n kube-system.

### Решение 1
![task1](https://github.com/jinnonn/k8s-p1-hw/blob/main/изображение_2023-11-02_013327538.png)
------
## Задание 2

Есть файл с деплоем:

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis
        env:
         - name: REDIS_PASSWORD
           value: password123
        ports:
        - containerPort: 6379
```

------
**Выполните действия:**

1. Измените файл с учётом условий:

 * redis должен запускаться без пароля;
 * создайте Service, который будет направлять трафик на этот Deployment;
 * версия образа redis должна быть зафиксирована на 6.0.13.

2. Запустите Deployment в своём кластере и добейтесь его стабильной работы.
3. В качестве решения пришлите получившийся файл.
### Решение 2

#### Deployment объект

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: redis
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: master
        image: bitnami/redis:6.0.13
        env:
         - name: ALLOW_EMPTY_PASSWORD
           value: "yes"
        ports:
         - containerPort: 6379
```

#### Service объект

```
apiVersion: v1
kind: Service
metadata:
  name: redis-service
spec:
  selector:
    app: redis
  ports:
    - protocol: TCP
      port: 6379
      targetPort: 6379
```
#### Проверка работы Redis:
![task2](https://github.com/jinnonn/k8s-p1-hw/blob/main/изображение_2023-11-02_022659922.png)
![task2](https://github.com/jinnonn/k8s-p1-hw/blob/main/изображение_2023-11-02_022743940.png)
------
## Задание 3

**Выполните действия:**

1. Напишите команды kubectl для контейнера из предыдущего задания:

 - выполнения команды ps aux внутри контейнера;
 - просмотра логов контейнера за последние 5 минут;
 - удаления контейнера;
 - проброса порта локальной машины в контейнер для отладки.

2. В качестве решения пришлите получившиеся команды.

### Решение 3

1. kubectl exec redis-7bfccd74cd-fttjj -- ps aux
2. kubectl logs --since=5m redis-7bfccd74cd-fttjj
3. kubectl delete pod redis-7bfccd74cd-fttjj
4. kubectl port-forward redis-7bfccd74cd-fttjj 6379:6379 (данная команда создает сессию порт форвардинга, т.е. она непостоянна и подходит для отладки, хотя можно также настроить Service с типом NodePort, если указал неверную команду, то просьба "направить" меня в нужную сторону :) )

---

## Задание 4*

Есть конфигурация nginx:

```
location / {
    add_header Content-Type text/plain;
    return 200 'Hello from k8s';
}
```

**Выполните действия:**

1. Напишите yaml-файлы для развёртки nginx, в которых будут присутствовать:

 - ConfigMap с конфигом nginx;
 - Deployment, который бы подключал этот configmap;
 - Ingress, который будет направлять запросы по префиксу /test на наш сервис.

2. В качестве решения пришлите получившийся файл.

### Решение 4*

#### ConfigMap
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    }
    http {
      server {
        listen 80;
        server_name localhost;
        location / {
          add_header Content-Type text/plain;
          return 200 'Hello from k8s';
        }
      }
    }
```
#### Deployment
```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: master
        image: nginx:latest
        env:
         - name: NGINX_CONF
           valueFrom:
             configMapKeyRef:
               name: nginx-config
               key: nginx.conf
        ports:
         - containerPort: 80
```
#### Ingress
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
    - host: jinon.com
      http:
        paths:
        - path: /test
          pathType: Exact
          backend:
            service:
              name: nginx
              port:
                number: 80
```
#### Service
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```



