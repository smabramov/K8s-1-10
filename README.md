# «Helm» - Абрамов Сергей

### Цель задания

В тестовой среде Kubernetes необходимо установить и обновить приложения с помощью Helm.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение, например, MicroK8S.
2. Установленный локальный kubectl.
3. Установленный локальный Helm.
4. Редактор YAML-файлов с подключенным репозиторием GitHub.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://helm.sh/docs/intro/install/) по установке Helm. [Helm completion](https://helm.sh/docs/helm/helm_completion/).

------

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

------

### Решение 

```
serg@k8snode:~$ helm version
version.BuildInfo{Version:"v3.16.3", GitCommit:"cfd07493f46efc9debd9cc1b02a0961186df7fdf", GitTreeState:"clean", GoVersion:"go1.22.7"}

```
Подготовим фаЙлы:

[deployment.yaml](https://github.com/smabramov/K8s-1-10/blob/741823fc2babba0a05cdb72fa65d63b9d2261f87/code/nginx_chart/templates/deployment.yaml)

[service.yaml](https://github.com/smabramov/K8s-1-10/blob/741823fc2babba0a05cdb72fa65d63b9d2261f87/code/nginx_chart/templates/service.yaml)

[Chart.yaml](https://github.com/smabramov/K8s-1-10/blob/741823fc2babba0a05cdb72fa65d63b9d2261f87/code/nginx_chart/Chart.yaml)

[values.yaml](https://github.com/smabramov/K8s-1-10/blob/741823fc2babba0a05cdb72fa65d63b9d2261f87/code/nginx_chart/values.yaml)

Создадим шаблон на основе этих файлов:

```

serg@k8snode:~/git/K8s-1-10/code$ helm template nginx_chart
---
# Source: netology/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: netology/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: netology
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```

Поменяем номер версии приложения в файле Chart.yaml:

```
  apiVersion: v2
  name: netology

  type: application

  version: 0.1.2
  appVersion: "1.19.0"

```
```
serg@k8snode:~/git/K8s-1-10/code$ helm template nginx_chart
---
# Source: netology/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: netology/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: netology
          image: "nginx:1.19.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```

Видим, что версия образа поменялась на 1.19.0 Внесем версию в файл values.yaml:

```

replicaCount: 1

image:
  repository: nginx
  tag: "1.20.0"

appPort: 80

```

```

serg@k8snode:~/git/K8s-1-10/code$ helm template nginx_chart
---
# Source: netology/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: netology/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: netology
          image: "nginx:1.20.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP

```

Видим, что версия образа встала 1.20.0, это произошло, потому что при задании условий подстановки переменных было указано, что в первую очередь данные брать из файла с переменными, если в них не будет указано данных, тогда подставлять данные из файла Chart.yaml

-----


### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

-----

### Решение

Запустим установку:


```
serg@k8snode:~/git/K8s-1-10/code$ helm install netology1  nginx_chart
NAME: netology1
LAST DEPLOYED: Fri Nov 29 14:57:41 2024
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/main-9cc98f77c-q5vgp   1/1     Running   0          31s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   22d
service/main         ClusterIP   10.152.183.136   <none>        80/TCP    31s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/main   1/1     1            1           31s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/main-9cc98f77c   1         1         1       31s
serg@k8snode:~/git/K8s-1-10/code$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netology1       default         1               2024-11-29 14:57:41.804407264 +0300 MSK deployed        netology-0.1.2  1.20.0  

```

Запустим две реплики:


```
serg@k8snode:~/git/K8s-1-10/code$ helm upgrade --install netology1 --set replicaCount=2 nginx_chart
Release "netology1" has been upgraded. Happy Helming!
NAME: netology1
LAST DEPLOYED: Fri Nov 29 14:59:26 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/main-9cc98f77c-bsmmk   1/1     Running   0          5s
pod/main-9cc98f77c-q5vgp   1/1     Running   0          110s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1     <none>        443/TCP   22d
service/main         ClusterIP   10.152.183.136   <none>        80/TCP    110s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/main   2/2     2            2           110s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/main-9cc98f77c   2         2         2       110s
serg@k8snode:~/git/K8s-1-10/code$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netology1       default         2               2024-11-29 14:59:26.936590668 +0300 MSK deployed        netology-0.1.2  1.20.0

```

Удалим netology1:


```
serg@k8snode:~/git/K8s-1-10/code$ helm uninstall netology1
release "netology1" uninstalled
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.152.183.1   <none>        443/TCP   22d

```

Создаем версию в namespace=app1:


```
serg@k8snode:~/git/K8s-1-10/code$ helm install netology1 --namespace app1 --create-namespace nginx_chart
NAME: netology1
LAST DEPLOYED: Fri Nov 29 15:12:33 2024
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all -n app1
NAME                       READY   STATUS    RESTARTS   AGE
pod/main-9cc98f77c-gsl52   1/1     Running   0          19s

NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/main   ClusterIP   10.152.183.100   <none>        80/TCP    19s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/main   1/1     1            1           19s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/main-9cc98f77c   1         1         1       19s
serg@k8snode:~/git/K8s-1-10/code$ helm list --namespace=app1
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netology1       app1            1               2024-11-29 15:12:33.705336302 +0300 MSK deployed        netology-0.1.2  1.20.0 

```
Создаем вторую версию в namespace=app1:

```

serg@k8snode:~/git/K8s-1-10/code$ helm upgrade --install netology2 --namespace app1 --create-namespace nginx_chart
Release "netology2" does not exist. Installing it now.
Error: Unable to continue with install: Service "main" in namespace "app1" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key "meta.helm.sh/release-name" must equal "netology2": current value is "netology1"

```
Ошибка, т.к. мы пытаемся запустить несколько приложений в одном пространстве имен, с разными параметрами, но одинаковыми именами. Настраиваем переменные таким образом, чтобы имена ресурсов приложения так же менялись, тогда возможен запуск дополнительного приложения в этом пространстве имен. 

Поправим deployment.yaml и service.yaml изменив имя на main1 и запустим повторно:

```
serg@k8snode:~/git/K8s-1-10/code$ helm upgrade --install netology2 --namespace app1 --create-namespace nginx_chart
Release "netology2" does not exist. Installing it now.
NAME: netology2
LAST DEPLOYED: Fri Nov 29 15:43:00 2024
NAMESPACE: app1
STATUS: deployed
REVISION: 1
TEST SUITE: None
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all -n app1
NAME                         READY   STATUS    RESTARTS   AGE
pod/main-9cc98f77c-gsl52     1/1     Running   0          31m
pod/main1-846cf6c79d-std4g   1/1     Running   0          42s

NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/main    ClusterIP   10.152.183.100   <none>        80/TCP    31m
service/main1   ClusterIP   10.152.183.161   <none>        80/TCP    42s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/main    1/1     1            1           31m
deployment.apps/main1   1/1     1            1           42s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/main-9cc98f77c     1         1         1       31m
replicaset.apps/main1-846cf6c79d   1         1         1       42s
serg@k8snode:~/git/K8s-1-10/code$ helm list --namespace=app1
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netology1       app1            1               2024-11-29 15:12:33.705336302 +0300 MSK deployed        netology-0.1.2  1.20.0     
netology2       app1            1               2024-11-29 15:43:00.098055306 +0300 MSK deployed        netology-0.1.2  1.20.0

```
Запускаем третью версию в namespace=app2:

```
serg@k8snode:~/git/K8s-1-10/code$ helm upgrade --install netology3 --namespace app2 --create-namespace nginx_chart
Release "netology3" does not exist. Installing it now.
NAME: netology3
LAST DEPLOYED: Fri Nov 29 15:46:35 2024
NAMESPACE: app2
STATUS: deployed
REVISION: 1
TEST SUITE: None
serg@k8snode:~/git/K8s-1-10/code$ helm list --namespace=app2
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
netology3       app2            1               2024-11-29 15:46:35.902214021 +0300 MSK deployed        netology-0.1.2  1.20.0     
serg@k8snode:~/git/K8s-1-10/code$ kubectl get all -n app2
NAME                       READY   STATUS    RESTARTS   AGE
pod/main-9cc98f77c-gz28j   1/1     Running   0          39s

NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/main   ClusterIP   10.152.183.169   <none>        80/TCP    39s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/main   1/1     1            1           39s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/main-9cc98f77c   1         1         1       39s

```

![k1](https://github.com/smabramov/K8s-1-10/blob/741823fc2babba0a05cdb72fa65d63b9d2261f87/png/k1.png)


### Правила приёма работы

1. Домашняя работа оформляется в своём Git репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, `helm`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

