# Ambassador Pattern

## Using an Ambassador to Shard a Service

```
# create shared redis containers
kubectl create -f redis-shards.yaml

# create DNS names using kubernetes service
kubectl create -f redis-service.yaml

# https://github.com/twitter/twemproxy
kubectl create configmap --from-file=nutcracker.yaml
```

## Using an Ambassador to Do Experimentation or Request Splitting

```
```