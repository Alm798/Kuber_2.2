# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes»
## `Михеев Алексей`


## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

## Ответ:

### 1. Манифесты

### ` deployment.yaml `

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-demo
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      
      # 1. Контейнер Nginx, который будет монтировать ConfigMap
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: web-content-volume
          mountPath: /usr/share/nginx/html
          readOnly: true
          
      # 2. Контейнер Multitool (Заменен на busybox для обхода ImagePullBackOff)
      - name: multitool-fixed
        image: busybox
        command: ["sh", "-c", "sleep infinity"]

      volumes:
      - name: web-content-volume
        configMap:
          name: nginx-web-content
          items:
          - key: index.html
            path: index.html
```

### ` configmap-web.yaml `

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-web-content
  namespace: default
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
        <title>ConfigMap Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; background-color: #f4f4f9; }
            .container { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
            h1 { color: #333; }
            p { color: #666; }
            .success { color: green; font-weight: bold; }
        </style>
    </head>
    <body>
        <div class="container">
            <h1>Добро пожаловать в Kubernetes!</h1>
            <p>Эта страница обслужена Nginx, конфигурация которого была предоставлена через ConfigMap.</p>
            <p class="success">Конфигурация успешно применена.</p>
            <!-- Здесь можно было бы добавить информацию, полученную из другого Pod,
                 но для простоты мы используем статический контент ConfigMap -->
        </div>
    </body>
    </html>
```
![1](https://github.com/Alm798/Kuber_2.2/blob/main/1.png)

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера

---
## **Задание 2: Настройка HTTPS с Secrets**  
### **Задача**  
Развернуть приложение с доступом по HTTPS, используя самоподписанный сертификат.

### **Шаги выполнения**  
1. **Сгенерировать SSL-сертификат**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
2. **Создать Secret**
3. **Настроить Ingress**
4. **Проверить HTTPS-доступ**

### **Что сдать на проверку**  
- Манифесты:
  - `secret-tls.yaml`
  - `ingress-tls.yaml`
- Скриншот вывода `curl -k`
![2](https://github.com/Alm798/Kuber_2.2/blob/main/2.png)

## Ответ:

## Результаты выполнения Задания 2: Настройка HTTPS с Secrets и Ingress

### 1. Манифесты

### ` vsecret-tls.yaml `

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-tls-secret
  namespace: default
type: kubernetes.io/tls
data:
  tls.crt: <CERTIFICATE_BASE64>
  tls.key: <KEY_BASE64>
```

### ` ingress-tls.yaml `

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false" 
spec:
  tls:
  - hosts:
    - myapp.example.com 
    secretName: myapp-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app-service # Сервис из Задания 1
            port:
              number: 80
---
[2](https://github.com/Alm798/Kuber_2.2/blob/main/2.png)

## **Задание 3: Настройка RBAC**  
### **Задача**  
Создать пользователя с ограниченными правами (только просмотр логов и описания подов).

### **Шаги выполнения**  
1. **Включите RBAC в microk8s**
```bash
microk8s enable rbac
```
2. **Создать SSL-сертификат для пользователя**
```bash
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN={ИМЯ ПОЛЬЗОВАТЕЛЯ}"
openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```
3. **Создать Role (только просмотр логов и описания подов) и RoleBinding**
4. **Проверить доступ**

## ## Ответ:

### 1. Команды генерации сертификатов (Согласно заданию)
Эти команды были предоставлены в задании для создания учетных данных пользователя, но в Minikube мы используем ServiceAccount для внутренней аутентификации.

```bash
# 1. Генерация приватного ключа
openssl genrsa -out developer.key 2048

# 2. Создание CSR (Запрос на подпись)
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer-user"

# 3. Подпись (Этот шаг требует доступа к CA кластера, который в Minikube закрыт)
# openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```

### 2. Манифесты RBAC

### Поскольку тестирую права для ServiceAccount (developer-sa), который я создали ранее, создал Role и RoleBinding.

### ` role-pod-reader.yaml `

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-viewer-logs
rules:
- apiGroups: [""] # Core API Group
  resources: ["pods"]
  verbs: ["get", "list", "describe"]
- apiGroups: [""]
  resources: ["pods/log"] # Право на чтение логов
  verbs: ["get"]
```

### ` rolebinding-developer.yaml `

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: developer-sa # Имя SA, который мы использовали для тестирования
  namespace: default
roleRef:
  kind: Role
  name: pod-viewer-logs
  apiGroup: rbac.authorization.k8s.io
```
### ` kubectl apply -f rbac-pod-access.yaml `

### 3. Проверка доступа## ## Ответ:

### 1. Команды генерации сертификатов (Согласно заданию)
Эти команды были предоставлены в задании для создания учетных данных пользователя, но в Minikube мы используем ServiceAccount для внутренней аутентификации.

```bash
# 1. Генерация приватного ключа
openssl genrsa -out developer.key 2048

# 2. Создание CSR (Запрос на подпись)
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer-user"

# 3. Подпись (Этот шаг требует доступа к CA кластера, который в Minikube закрыт)
# openssl x509 -req -in developer.csr -CA {CA серт вашего кластера} -CAkey {CA ключ вашего кластера} -CAcreateserial -out developer.crt -days 365
```

### 2. Манифесты RBAC

### Поскольку тестирую права для ServiceAccount (developer-sa), который я создали ранее, создал Role и RoleBinding.

### ` role-pod-reader.yaml `

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-viewer-logs
rules:
- apiGroups: [""] # Core API Group
  resources: ["pods"]
  verbs: ["get", "list", "describe"]
- apiGroups: [""]
  resources: ["pods/log"] # Право на чтение логов
  verbs: ["get"]
```

### ` rolebinding-developer.yaml `

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: developer-sa # Имя SA, который мы использовали для тестирования
  namespace: default
roleRef:
  kind: Role
  name: pod-viewer-logs
  apiGroup: rbac.authorization.k8s.io
```
### ` kubectl apply -f rbac-pod-access.yaml `

### 3. Проверка доступа
![3](https://github.com/Alm798/Kuber_2.2/blob/main/3.png)
