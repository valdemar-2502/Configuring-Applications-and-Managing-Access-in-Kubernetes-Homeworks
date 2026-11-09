# Домашнее задание к занятию «Настройка приложений и управление доступом в Kubernetes» - Kadancev Vladimir

### Примерное время выполнения задания

120 минут

### Цель задания

Научиться:
- Настраивать конфигурацию приложений с помощью **ConfigMaps** и **Secrets**
- Управлять доступом пользователей через **RBAC**

Это задание поможет вам освоить ключевые механизмы Kubernetes для работы с конфигурацией и безопасностью. Эти навыки необходимы для уверенного администрирования кластеров в реальных проектах. На практике навыки используются для:
- Хранения чувствительных данных (Secrets)
- Гибкого управления настройками приложений (ConfigMaps) 
- Контроля доступа пользователей и сервисов (RBAC)

------

## **Подготовка**
### **Чеклист готовности**
- Установлен Kubernetes (MicroK8S, Minikube или другой)
- Установлен `kubectl`
- Редактор для YAML-файлов (VS Code, Vim и др.)
- Утилита `openssl` для генерации сертификатов

------

### Инструменты, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S
2. [Инструкция](https://minikube.sigs.k8s.io/docs/start/) по установке Minikube
3. [Инструкция](https://kubernetes.io/docs/tasks/tools/) по установке kubectl
4. [Инструкция](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools) по установке VS Code

### Дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.
4. [Описание](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) RBAC.
5. [Пользователи и авторизация RBAC в Kubernetes](https://habr.com/ru/company/flant/blog/470503/).
6. [RBAC with Kubernetes in Minikube](https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b).

------

## **Задание 1: Работа с ConfigMaps**
### **Задача**
Развернуть приложение (nginx + multitool), решить проблему конфигурации через ConfigMap и подключить веб-страницу.

### **Шаги выполнения**
1. **Создать Deployment** с двумя контейнерами
   - `nginx`
   - `multitool`
3. **Подключить веб-страницу** через ConfigMap
4. **Проверить доступность**

### **Что сдать на проверку**
- Манифесты:
  - `deployment.yaml`
  - `configmap-web.yaml`
- Скриншот вывода `curl` или браузера
## Решение:
Манифест: [deployment.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/deployment.yaml)

Манифест: [configmap-web.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/configmap-web.yaml)

```
 kubectl apply -f configmap-web.yaml 
 kubectl apply -f deployment.yaml
 kubectl get pods -l app=web-app                 
 kubectl get configmap web-content               
 kubectl exec deploy/web-app -c multitool -- curl -s localhost:80   
```
 Поды в статусе `Running` (2/2):
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/01.png)

 Вывод ConfigMap:
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/02.png)

 Проверка доступности страницы через curl из контейнера multitool (возвращает HTML-код страницы из ConfigMap):
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/03.png)


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

## Решение:
Манифест: [secret-tls.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/secret-tls.yaml)

Манифест: [ingress-tls.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/ingress-tls.yaml)

Генерация сертификата:

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=myapp.example.com"
```
Создание Secret и Service:

```
kubectl create secret tls tls-secret --cert=tls.crt --key=tls.key --dry-run=client -o yaml > secret-tls.yaml
kubectl apply -f secret-tls.yaml
kubectl apply -f service-web.yaml
kubectl get secret tls-secret

```
Создание Ingress:

```
kubectl apply -f ingress-tls.yaml
kubectl get ingress
```

Проверка HTTPS-доступа:

curl -k --resolve myapp.example.com:443:127.0.0.1 https://myapp.example.com
curl -kv --resolve myapp.example.com:443:127.0.0.1 https://myapp.example.com 2>&1 | grep -E "subject|issuer"

Secret типа kubernetes.io/tls:

![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/04.png)

Ingress с привязанным хостом и секретом:

![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/05.png)

Проверка через curl (вывод HTML и информация о сертификате CN=myapp.example.com):

![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/06.png)

---
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

### **Что сдать на проверку**  
- Манифесты:
  - `role-pod-reader.yaml`
  - `rolebinding-developer.yaml`
- Команды генерации сертификатов
- Скриншот проверки прав (`kubectl get pods --as=developer`)

## Решение:
Манифест: [role-pod-reader.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/role-pod-reader.yaml)

Манифест: [rolebinding-developer.yaml](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/rolebinding-developer.yaml)
```
openssl genrsa -out developer.key 2048
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
sudo openssl x509 -req -in developer.csr \
  -CA /var/snap/microk8s/current/certs/ca.crt \
  -CAkey /var/snap/microk8s/current/certs/ca.key \
  -CAserial ./ca.srl -CAcreateserial \
  -out developer.crt -days 365
sudo chown $USER:$USER developer.crt ca.srl
openssl x509 -in developer.crt -noout -subject -dates
```
Генерация сертификата для пользователя developer:
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/07.png)

Применение Role и RoleBinding:
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/08.png)

 Проверка прав:
![kubernetes](https://github.com/valdemar-2502/Configuring-Applications-and-Managing-Access-in-Kubernetes-Homeworks/blob/main/screenshots/09.png)

---
## Шаблоны манифестов с учебными комментариями
### **1. Deployment с ConfigMap (nginx + multitool)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
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
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config # ПОДКЛЮЧЕНИЕ ConfigMap
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config # УКАЖИТЕ имя созданного ConfigMap
```
### **2. ConfigMap для веб-страницы**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-content # ИЗМЕНИТЕ: Укажите имя ConfigMap
  namespace: default # ОПЦИОНАЛЬНО: Укажите namespace, если не default
data:
  # КЛЮЧЕВОЙ МОМЕНТ: index.html будет подключен как файл
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
      <title>Страница из ConfigMap</title> # ИЗМЕНИТЕ: Заголовок страницы
    </head>
    <body>
      <h1>Привет от Kubernetes!</h1> # ДОБАВЬТЕ: Свой контент страницы
    </body>
    </html>
```

### **3. Secret для TLS-сертификата**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret # ИЗМЕНИТЕ при необходимости
type: kubernetes.io/tls
data:
  tls.crt: # ЗАМЕНИТЕ на base64-код сертификата (cat tls.crt | base64 -w 0)
  tls.key: # ЗАМЕНИТЕ на base64-код ключа (cat tls.key | base64 -w 0)
```
### **4. Role для просмотра подов**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-viewer # ИЗМЕНИТЕ: Название роли
  namespace: default # ВАЖНО: Role работает только в указанном namespace
rules:
- apiGroups: [""] # КЛЮЧЕВОЙ МОМЕНТ: "" означает core API group
  resources: # РАЗРЕШЕННЫЕ РЕСУРСЫ:
    - pods # Доступ к просмотру подов
    - pods/log # Доступ к логам подов
  verbs: # РАЗРЕШЕННЫЕ ДЕЙСТВИЯ:
    - get # Просмотр отдельных подов
    - list # Список всех подов
    - watch # Мониторинг изменений
    - describe # Просмотр деталей
# ДОПОЛНИТЕЛЬНО: Можно добавить больше правил для других ресурсов
```
---

## **Правила приёма работы**
1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать:
   - Скриншоты вывода команд `kubectl`
   - Скриншоты результатов выполнения
   - Тексты манифестов или ссылки на них
3. Для заданий с TLS приложите команды генерации сертификатов

## **Критерии оценивания задания**
1. Зачёт: Все задачи выполнены, манифесты корректны, есть доказательства работы (скриншоты).
2. Доработка (на доработку задание направляется 1 раз): основные задачи выполнены, при этом есть ошибки в манифестах или отсутствуют проверочные скриншоты.
3. Незачёт: работа выполнена не в полном объёме, есть ошибки в манифестах, отсутствуют проверочные скриншоты. Все попытки доработки израсходованы (на доработку работа направляется 1 раз). Этот вид оценки используется крайне редко.

## **Срок выполнения задания**  
1. 5 дней на выполнение задания.
2. 5 дней на доработку задания (в случае направления задания на доработку).
