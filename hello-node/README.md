# Tutorial

## Create images

```
$ docker build -t hello-node:v1 .
```

## Create a Deployment

```
$ kubectl run hello-node --image=hello-node:v1 --port=8080 --image-pull-policy=Never
$ kubectl get deployments
$ kubectl get pods
$ kubectl expose deployment hello-node --type=LoadBalancer --name=hello-node
```

## Rolling Update

예전 버전에서는 `ReplicationController`를 사용. 이 경우 `rolling-update command`
최근 버전에서는 `Deployment`를 사용. 이 경우 `kubectl set image` 또는 `kubectl patch`

```
$ docker build -t hello-node:v2 .
$ kubectl set image deployment/hello-node hello-node=hello-node:v2
$ kubectl rolling-update NAME [NEW_NAME] --image=IMAGE:TAG
```

## Clean up

```
$ kubectl delete service hello-node
$ kubectl delete deployment hello-node
```

## Helm - Kubernetes package manager

```
$ helm ls
$ helm search jenkins
$ helm install stable/jenkins --name jenkins
$ helm status jenkins
$ helm delete jenkins
```
