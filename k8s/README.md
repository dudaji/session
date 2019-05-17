# 실습

## Container Deploy 해보기

```shell
$ kubectl create deployment nginx --image=nginx
deployment.apps/nginx created

$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-685bd54dd7-jxxlg   0/1     ContainerCreating   0          36s

$ kubectl delete deployment nginx
deployment.extensions "nginx" deleted
```

## YAML 파일로 살펴보기

```shell
$ kubectl create deployment nginx --image=nginx --dry-run -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

### Spec -> 바라는 상태

```
$ kubectl create deployment nginx --image=nginx --dry-run -o yaml > deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

Yaml 파일 내에서 replicas: 3 으로 변경

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
        - image: nginx
          name: nginx
          resources: {}
status: {}
```

```
$ kubectl create --filename deployment.yaml
deployment.apps/nginx created

Or

$ kubectl apply --filename deployment.yaml
deployment.apps/nginx configured

$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-65f88748fd-526zp   0/1     ContainerCreating   0          5s
nginx-65f88748fd-6mq7k   0/1     ContainerCreating   0          6s
nginx-65f88748fd-b5tsp   0/1     ContainerCreating   0          6s

```

### Service 생성 (Network)

```
$ kubectl expose deployment nginx --port=8080 --target-port=80 --type=LoadBalancer
service/nginx exposed

$ kubectl get service
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP          7m50s
nginx        LoadBalancer   10.110.55.20   localhost     8080:30990/TCP   1s

$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

![nginx](https://cdn-images-1.medium.com/max/1600/1*_fKemV13IyNAB9i2yx9V2Q.png)

## Gitlab Community Edition 설치해보기

```
$ kubectl apply --filename \
 https://raw.githubusercontent.com/dudaji/session/develop/k8s/gitlab-ce.yaml
secret/gitlab-ce-postgresql created
secret/gitlab-ce-redis created
secret/gitlab-ce-gitlab-ce created
configmap/gitlab-ce-gitlab-ce created
persistentvolumeclaim/gitlab-ce-postgresql created
persistentvolumeclaim/gitlab-ce-redis created
persistentvolumeclaim/gitlab-ce-gitlab-ce-data created
persistentvolumeclaim/gitlab-ce-gitlab-ce-etc created
service/gitlab-ce-postgresql created
service/gitlab-ce-redis created
service/gitlab-ce-gitlab-ce created
deployment.extensions/gitlab-ce-postgresql created
deployment.extensions/gitlab-ce-redis created
deployment.extensions/gitlab-ce-gitlab-ce created


$ kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
gitlab-ce-gitlab-ce-7bf9759b7d-6rskq    0/1     Pending   0          2s
gitlab-ce-postgresql-764d4db6b7-b4k2h   0/1     Pending   0          2s
gitlab-ce-redis-645c89846b-dxvfp        0/1     Pending   0          2s


$ kubectl get services
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                   AGE
gitlab-ce-gitlab-ce    LoadBalancer   10.97.104.31    localhost     22:30041/TCP,80:32731/TCP,443:31068/TCP   57s
gitlab-ce-postgresql   ClusterIP      10.104.195.4    <none>        5432/TCP                                  57s
gitlab-ce-redis        ClusterIP      10.104.15.145   <none>        6379/TCP                                  57s
kubernetes             ClusterIP      10.96.0.1       <none>        443/TCP                                   16h


$ kubectl logs -f gitlab-ce-gitlab-ce-7bf9759b7d-6rskq

Thank you for using GitLab Docker Image!
Current version: gitlab-ce=9.4.1-ce.0

Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
And restart this container to reload settings.
To do it use docker exec:

  docker exec -it gitlab vim /etc/gitlab/gitlab.rb
  docker restart gitlab

For a comprehensive list of configuration options please see the Omnibus GitLab readme
https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

If this container fails to start due to permission problems try to fix it by executing:

  docker exec -it gitlab update-permissions
  docker restart gitlab

```

접속  
http://localhost

![gitlab-ce](https://computingforgeeks.com/wp-content/uploads/2018/11/install-gitlab-centos-7-fedora-29-set-root-password-1024x514.png)

```
$ kubectl deapplylete --filename \
 https://raw.githubusercontent.com/dudaji/session/develop/k8s/gitlab-ce.yaml

secret "gitlab-ce-postgresql" deleted
secret "gitlab-ce-redis" deleted
secret "gitlab-ce-gitlab-ce" deleted
configmap "gitlab-ce-gitlab-ce" deleted
persistentvolumeclaim "gitlab-ce-postgresql" deleted
persistentvolumeclaim "gitlab-ce-redis" deleted
persistentvolumeclaim "gitlab-ce-gitlab-ce-data" deleted
persistentvolumeclaim "gitlab-ce-gitlab-ce-etc" deleted
service "gitlab-ce-postgresql" deleted
service "gitlab-ce-redis" deleted
service "gitlab-ce-gitlab-ce" deleted
deployment.extensions "gitlab-ce-postgresql" deleted
deployment.extensions "gitlab-ce-redis" deleted
deployment.extensions "gitlab-ce-gitlab-ce" deleted
```
