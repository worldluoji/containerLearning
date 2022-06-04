### Deploy simple

```sh
kubectl create ns simple
kubectl create -f http-server.yaml
kubectl create -f istio-specs.yaml
```

### Check ingress ip

```sh
kubectl get svc -n istio-system

可以看到我的机器上EXTERNAL-IP为localhost

export INGRESS_IP=localhost

如果EXTERNAL-IP值是<none>（或永久<pending>），则环境不为入口网关提供外部负载平衡器。
在这种情况下，您可以使用服务的节点端口来访问网关:

使用服务的节点端口来访问:

    设置入口端口： 

        export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

        export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

    设置入口IP取决于群集提供者：

        export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')

```

### Access the simple via ingress

```sh
curl -H "Host: simple.http.io" $INGRESS_IP/health -v
```
