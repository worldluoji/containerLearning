### Build http-server image
```sh
docker build -t registry-local/http-server:0.0.1 .
```

### Deploy simple

```sh
kubectl create -f http-server.yaml
```

### Deploy envoy

```sh
kubectl create configmap envoy-config --from-file=envoy.yaml
kubectl create -f envoy-deploy.yaml
```
