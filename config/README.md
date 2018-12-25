## ConfigMap

```
curl -OL https://k8s.io/examples/pods/config/redis-config
kubectl create configmap example-redis-config --from-file=redis-config
kubectl get configmap example-redis-config -o yaml

# Create Pods
kubectl create -f redis-pod.yaml
```

- 환경 변수나 설정 값들을 변수로 관리
- Pod가 생성될 때 값들을 넣어줄 수 있음 (key-value 형태)
- DEV, PROD ConfigMap을 미리 만들어놓고 배포할 때 반영
- 값을 환경변수로 넘기거나, 디스크 볼륨으로 마운트 가능
- https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/

<br>

## PersistantVolume, PersistantVolumeClaim

```
kubectl create -f pv-volume.yaml
kubectl get pv

# Create PVC
kubectl create -f pv-claim.yaml
kubectl get pvc

# Create Pods
kubectl create -f pv-pod.yaml
kubectl get pods
```

- KOPS의 경우 storage-aws addon으로 EBS StorageClass 생성되어 있음
- AWS에서 Dynamic Provisioning의 경우 계속 EBS가 생성됨(?)
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume
