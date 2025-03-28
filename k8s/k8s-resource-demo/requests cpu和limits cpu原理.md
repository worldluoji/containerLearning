# requests cpu和limits cpu原理
## 1. 一个限制内存和cpu的例子
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: images.my-company.example/app:v4
    resources:
      requests:
        memory: "64Mi"
        cpu: "1"
      limits:
        memory: "128Mi"
        cpu: "2"
```
Limits CPU 就是容器所在 Cgroup 控制组中的 CPU 上限值，由cpu.cfs_quota_us 和 cpu.cfs_period_us两个值决定。
即在cpu.cfs_period_us周期内分配cpu.cfs_quota_us时间给你用。
Requests CPU 的值就是控制组中的 cpu.shares 的值。语义其实就是我想要多少资源，如果没有竞争，就
分1个CPU，如果有竞争，就和竞争者按比例分配。

cpu=1 指的就是，这个 Pod 的 CPU 限额是 1 个 CPU。当然，具体“1 个 CPU”在宿主机上如何解释，
是 1 个 CPU 核心，还是 1 个 vCPU，还是 1 个 CPU 的超线程（Hyperthread），完全取决于宿主机的 CPU 实现方式。
当然，你也可以直接把这个配置写成 cpu=0.5。
但在实际使用时，我还是推荐你使用 500m 的写法，原因如下：
表达式 0.5 等价于表达式 500m， 可以看作 “500 millicpu”。有些人说成是“五百毫 cpu”，其实说的是同样的事情。 
具有小数点（如 0.5）的请求由 API 转换为 500m；最大精度是 1m。 因此，或许你应该优先考虑使用 500m 的形式。

对于内存资源来说，它的单位是 bytes。Kubernetes 支持你使用 Ei、Pi、Ti、Gi、Mi、Ki（或者 E、P、T、G、M、K）的方式来作为 bytes 的值。
比如，在我们的例子里，Memory requests 的值就是 64MiB (2 的 26 次方 bytes = 64MiB) 。
这里要注意区分 MiB（mebibyte）和 MB（megabyte）的区别。备注：1Mi=1024*1024；1M=1000*1000

在 Kubernetes 中，像 CPU 这样的资源被称作“可压缩资源”（compressible resources）。
它的典型特点是，当可压缩资源不足时，Pod 只会“饥饿”，但不会退出。
而像内存这样的资源，则被称作“不可压缩资源（incompressible resources）”。
当不可压缩资源不足时，Pod 就会因为 OOM（Out-Of-Memory）被内核杀掉。

<br>

## 2. Qos类别划分
1） Guaranteed 类别：当 Pod 里的每一个 Container 都同时设置了 requests 和 limits，
并且 requests 和 limits 值相等的时候，这个 Pod 就属于 Guaranteed 类别。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```
当这个 Pod 创建之后，它的 qosClass 字段就会被 Kubernetes 自动设置为 Guaranteed。需要注意的是，
当 Pod 仅设置了 limits 没有设置 requests 的时候，Kubernetes 会自动为它设置与 limits 相同的 requests 值，
所以，这也属于 Guaranteed 情况。

2） Burstable 类别：当 Pod 不满足 Guaranteed 的条件，但至少有一个 Container 设置了 requests。
那么这个 Pod 就会被划分到 Burstable 类别。
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

3）BestEffort： 如果一个 Pod 既没有设置 requests，也没有设置 limits，那么它的 QoS 类别就是 BestEffort。

QoS 划分的主要应用场景，是当宿主机资源紧张的时候，kubelet 对 Pod 进行 Eviction（即资源回收）时需要用到的。
具体地说，当 Kubernetes 所管理的宿主机上不可压缩资源短缺时，就有可能触发 Eviction。
比如，可用内存（memory.available）、可用的宿主机磁盘空间（nodefs.available），以及容器运行时镜像存储空间（imagefs.available）等等。

<br>

## 3. k8s默认资源回收的阈值
- memory.available<100Mi
- nodefs.available<10%
- nodefs.inodesFree<5%
- imagefs.available<15%

可以通过命令修改：
```shell
kubelet --eviction-hard=imagefs.available<10%,memory.available<500Mi,nodefs.available<5%,nodefs.inodesFree<5% --eviction-soft=imagefs.available<30%,nodefs.available<10% --eviction-soft-grace-period=imagefs.available=2m,nodefs.available=2m --eviction-max-pod-grace-period=600
```
Eviction 在 Kubernetes 里其实分为 Soft 和 Hard 两种模式。
其中，Soft Eviction 允许你为 Eviction 过程设置一段“优雅时间”，
比如上面例子里的 imagefs.available=2m，就意味着当 imagefs 不足的阈值达到 2 分钟之后，kubelet 才会开始 Eviction 的过程。
而 Hard Eviction 模式下，Eviction 过程就会在阈值达到之后立刻开始。

当 Eviction 发生的时候，kubelet 具体会挑选哪些 Pod 进行删除操作，就需要参考这些 Pod 的 QoS 类别了。
- 首当其冲的，自然是 BestEffort 类别的 Pod。
- 其次，是属于 Burstable 类别、并且发生“饥饿”的资源使用量已经超出了 requests 的 Pod。
- 最后，才是 Guaranteed 类别。并且，Kubernetes 会保证只有当 Guaranteed 类别的 Pod 的资源使用量超过了其 limits 的限制，
或者宿主机本身正处于 Memory Pressure 状态时，Guaranteed 的 Pod 才可能被选中进行 Eviction 操作。

<br>

## 4. cpuset
在使用容器的时候，你可以通过设置 cpuset 把容器绑定到某个 CPU 的核上，而不是像 cpushare 那样共享 CPU 的计算能力。
这种情况下，由于操作系统在 CPU 之间进行上下文切换的次数大大减少，容器里应用的性能会得到大幅提升。
事实上，cpuset 方式，是生产环境里部署在线应用类型的 Pod 时，非常常用的一种方式。

怎么做到呢？
首先，你的 Pod 必须是 Guaranteed 的 QoS 类型；
然后，你只需要将 Pod 的 CPU 资源的 requests 和 limits 设置为同一个相等的整数值即可。
```yaml
spec:
  containers:
  - name: nginx
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "2"
      requests:
        memory: "200Mi"
        cpu: "2"
```
这时候，该 Pod 就会被绑定在 2 个独占的 CPU 核上。当然，具体是哪两个 CPU 核，是由 kubelet 为你分配的。

<br>

## 5. LimitRange （了解）
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-min-max-demo-lr
spec:
  limits:
  - max:
      cpu: "800m"
    min:
      cpu: "200m"
    type: Container
```
如果设置了上面的limit range， 但是pod的yaml中没有设置resource对cpu进行显示设置，
就会用上面LimitRange的参数进行限制。
但是，这个对象不常用，因为它会对POD中的每个容器都进行相同的限制，有时候我们的POD里不止1个容器。