---

layout: post

title:  dashboard安装

tag: CICD

---
# dashboard安装

有了k8s集群，安装一个dashboard是很有必要的，对于资源的查看管理很有帮助。

dashboard的release信息地址：<https://github.com/kubernetes/dashboard/releases>

1、安装方式很简单：

![](images/https://github.com/superhxf/superhxf.github.io/blob/master/_posts/images/20190807164409.png)

执行命令即可；

注意，官方提供的svc只能集群内部方访问，如果外部访问需要配置成loadbalance，或者nodeport，或者Ingress。

在浏览器中输入`https://集群地址：分配的端口`就可以访问了。

2、获取登录的token

​	执行命令即可。

```
$ kubectl get secret -n kube-system $(kubectl get serviceaccount -n kube-system kubernetes-dashboard-admin -o jsonpath='{.secrets[].name}') -o jsonpath='{.data.token}' | base64 --decode
```



