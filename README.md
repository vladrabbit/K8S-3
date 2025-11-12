# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

------

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

### Решение 1

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netology-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: netology-web
  template:
    metadata:
      labels:
        app: netology-web
    spec:
      containers:
        - name: nginx
          image: nginx:1.25-alpine
          ports:
            - name: http-nginx
              containerPort: 80
        - name: multitool
          image: wbitt/network-multitool:alpine-minimal
          env:
            - name: HTTP_PORT
              value: "8080"
          ports:
            - name: http-mt
              containerPort: 8080

```

![SCR1](https://github.com/vladrabbit/K8S-3/blob/main/SCR/1.1.png)

2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: netology-svc
spec:
  selector:
    app: netology-web
  ports:
    - name: nginx-port
      port: 9001
      targetPort: http-nginx
      protocol: TCP
    - name: multitool-port
      port: 9002
      targetPort: http-mt
      protocol: TCP
  type: ClusterIP

```

![SCR2](https://github.com/vladrabbit/K8S-3/blob/main/SCR/1.2.png)

3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mt-client
spec:
  containers:
    - name: mt
      image: wbitt/network-multitool:alpine-minimal
      command: ["sleep","infinity"]
  restartPolicy: Never

```

![SCR3](https://github.com/vladrabbit/K8S-3/blob/main/SCR/1.3.png)

![SCR4](https://github.com/vladrabbit/K8S-3/blob/main/SCR/1.4.png)


### Решение 2


1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.

```yaml

apiVersion: v1
kind: Service
metadata:
  name: netology-svc-nodeport
spec:
  selector:
    app: netology-web
  ports:
    - name: nginx-port
      port: 9001
      targetPort: http-nginx
      protocol: TCP
      nodePort: 30001
    - name: multitool-port
      port: 9002
      targetPort: http-mt
      protocol: TCP
  type: NodePort

```

![SCR5](https://github.com/vladrabbit/K8S-3/blob/main/SCR/2.1.png)

2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

![SCR6](https://github.com/vladrabbit/K8S-3/blob/main/SCR/2.3.png)
![SCR7](https://github.com/vladrabbit/K8S-3/blob/main/SCR/2.2.png)



