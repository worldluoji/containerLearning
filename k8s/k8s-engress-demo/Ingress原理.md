# Ingress原理
Ingress 的功能其实很容易理解：所谓 Ingress，就是 Service 的“Service”。

举个例子，假如我现在有这样一个站点：`https://cafe.example.com`。
其中，`https://cafe.example.com/coffee`，对应的是“咖啡点餐系统”。而`https://cafe.example.com/tea`，对应的则是“茶水点餐系统”。
这两个系统，分别由名叫 coffee 和 tea 这样两个 Deployment 来提供服务。
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
spec:
  tls:
  - hosts:
    - cafe.example.com    #当用户访问 cafe.example.com 的时候，实际上访问到的是这个Ingress
    secretName: cafe-secret
  rules:
  - host: cafe.example.com
    http:
      paths:
      - path: /tea  #访问cafe.example.com/tea时路由到tea-svc
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```
所谓 Ingress 对象，其实就是 Kubernetes 项目对“反向代理”的一种抽象。
一个 Ingress 对象的实现的主要内容，实际上就是一个“反向代理”服务（比如：Nginx这种具体实现，也可以是其它的，比如Envoy等等）的配置文件的描述。
而这个代理服务对应的转发规则，就是 IngressRule。

例子：
Ingress使用nginx, 首先需要安装社区提供的nginx-ingress-controller
```
$ kubectl apply -f https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/mandatory.yaml 不行就下载下来
```
当一个新的 Ingress 对象由用户创建后，nginx-ingress-controller 就会根据 Ingress 对象里定义的内容，
生成一份对应的 Nginx 配置文件（/etc/nginx/nginx.conf），并使用这个配置文件启动一个 Nginx 服务。
 
此外，nginx-ingress-controller 还允许你通过 Kubernetes 的 ConfigMap 对象来对上述 Nginx 配置文件进行定制。
这个 ConfigMap 的名字，需要以参数的方式传递给 nginx-ingress-controller。
而你在这个 ConfigMap 里添加的字段，将会被合并到最后生成的 Nginx 配置文件当中。

安装好nginx-ingress-controller后，将nginx服务暴露出去即可访问
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```
源码：https://github.com/resouer/kubernetes-ingress/blob/master/examples/complete-example/README.md

一个 Nginx Ingress Controller 为你提供的服务，其实是一个可以根据 Ingress 对象和被代理后端 Service 的变化，
来自动进行更新的 Nginx 负载均衡器。

目前，业界常用的各种反向代理项目，比如 Nginx、HAProxy、Envoy、Traefik 等，
都已经为 Kubernetes 专门维护了对应的 Ingress Controller。


不过，你可能会有一个疑问，如果我的请求没有匹配到任何一条 IngressRule，那么会发生什么呢？
首先，既然 Nginx Ingress Controller 是用 Nginx 实现的，那么它当然会为你返回一个 Nginx 的 404 页面。

不过，Ingress Controller 也允许你通过 Pod 启动命令里的–default-backend-service 参数，设置一条默认规则，
比如：–default-backend-service=nginx-default-backend。

这样，任何匹配失败的请求，就都会被转发到这个名叫 nginx-default-backend 的 Service。
所以，你就可以通过部署一个专门的 Pod，来为用户返回自定义的 404 页面了。