# FaaS

```shell
# install kubeless, cli
kubectl create ns kubeless
kubectl create -f https://github.com/kubeless/kubeless/releases/download/v1.0.3/kubeless-v1.0.3.yaml

export OS=$(uname -s| tr '[:upper:]' '[:lower:]')
curl -OL https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless_$OS-amd64.zip
unzip kubeless_$OS-amd64.zip
sudo mv bundles/kubeless_$OS-amd64/kubeless /usr/local/bin/

# install kubeless UI
kubectl create -f https://raw.githubusercontent.com/kubeless/kubeless-ui/master/k8s.yaml
kubectl get svc ui -n kubeless
kubectl port-forward svc/ui -n kubeless 3000:3000
```

## Deploy Funcions

```
# Deploy functions
$ kubeless function deploy hello --runtime python2.7 \
    --from-file test.py \
    --handler test.hello
INFO[0000] Deploying function...
INFO[0000] Function hello submitted for deployment
INFO[0000] Check the deployment status executing 'kubeless function ls hello'

# Call the function
$ kubectl get functions
$ kubeless function ls
$ kubeless function call hello --data 'Hello world!'

# Clean up
$ kubeless function delete hello
$ kubectl delete -f https://github.com/kubeless/kubeless/releases/download/$RELEASE/kubeless-$RELEASE.yaml
```
