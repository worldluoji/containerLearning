# k8s中ngnix403解决方法
进入对应的pod:
```shell
kubectl exec -it nginx-deployment-758578bd74-88rc5 bash
```
在mountPath: /usr/share/nginx/html对应的目录下，加入index.html
```
echo "hello world" >> index.html
```
有了它才能正常。