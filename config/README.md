## PersistantVolume, PersistantVolumeClaim

```
kubectl create -f pv-volume.yaml
kubectl get pv

# Create PVC
kubectl create -f pv-claim.yaml
kubectl get pvc

# Create Pod
kubectl create -f pv-pod.yaml
kubectl get pods
```

- KOPS의 경우 storage-aws addon으로 EBS StorageClass 생성되어 있음
- AWS에서 Dynamic Provisioning의 경우 계속 EBS가 생성됨(?)
- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume