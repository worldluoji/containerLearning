### Install istio


## precheck
```sh
istioctl x precheck
```


```sh
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.12.0
cp bin/istioctl /usr/local/bin
istioctl install --set profile=demo -y
```

### Istio monitoring

- Grafana dashboard `7639`


### istio安装，如出现错误，可以查看 istio-system namepace下的pod日志信息 (logs and describe)
### https://istio.io/latest/zh/docs/setup/install/istioctl/