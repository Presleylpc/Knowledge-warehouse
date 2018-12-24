# [CroeDNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)

> Kubernetes 1.6或更高版本。使用CoreDNS，版本1.9或更高版本。
> 适当的附加组件：kube-dns或CoreDNS

## 介绍
DNS是使用插件管理器集群插件自动启动的内置Kubernetes服务 。

从Kubernetes v1.12开始，CoreDNS是推荐的DNS服务器，
DNS服务器支持正向查找（A记录），端口查找（SRV记录），反向IP地址查找（PTR记录）等。



## CoreDNS
CoreDNS是一个通用的权威DNS服务器，可以作为集群DNS，符合dns规范。

### CoreDNS ConfigMap选项
CoreDNS是一个模块化和可插拔的DNS服务器，每个插件都为CoreDNS添加了新功能。这可以通过维护Corefile来配置，Corefile是CoreDNS配置文件。集群管理员可以修改CoreDNS Corefile的ConfigMap以更改服务发现的工作方式。

在Kubernetes中，CoreDNS安装了以下默认的Corefile配置。

apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
Corefile配置包括以下CoreDNS 插件：

- errors：错误记录到stdout。
- health：CoreDNS的运行状况报告为http://localhost：8080/health。
- kubernetes：CoreDNS将根据Kubernetes服务和pod的IP回复DNS查询。
- `pods insecure`: 用于向后兼容kube-dns。您可以使用该`pods verified`选项，该选项仅在相同名称空间中存在具有匹配IP的窗格时才返回A记录。`pods disabled`如果您不使用pod记录，则可以使用该选项。

- Upstream：用于解析指向外部主机的服务（外部服务）。

- prometheus：CoreDNS的度量标准可以在http://localhost:9153/Prometheus格式的指标中找到。
- proxy：任何不在Kubernetes集群域内的查询都将转发到预定义的解析器（/etc/resolv.conf）。
- cache：这将启用前端缓存。
- loop：检测简单的转发循环，如果找到循环则停止CoreDNS进程。
- reload：允许自动重新加载已更改的Corefile。
- loadbalance：这是一个循环DNS负载均衡器，通过在答案中随机化A，AAAA和MX记录的顺序。
我们可以通过修改此configmap来修改默认行为。

## 使用CoreDNS配置Stub域和上游名称服务器
CoreDNS能够使用代理插件配置存根域和上游名称服务器。

**例:**
如果集群运营商的Consul域服务器位于10.150.0.1，并且所有Consul名称都具有后缀.consul.local。要在CoreDNS中配置它，集群管理员在CoreDNS ConfigMap中创建以下节。
```
consul.local:53 {
        errors
        cache 30
        proxy . 10.150.0.1
    }
```

要明确强制所有非群集DNS查找在172.16.0.1要经过特定的名称服务器，指定proxy和upstream域名服务器，而不使用/etc/resolv.conf
```
proxy .  172.16.0.1
```
```
upstream 172.16.0.1
```
因此，最终的ConfigMap以及默认Corefile配置将如下所示：
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream 172.16.0.1
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . 172.16.0.1
        cache 30
        loop
        reload
        loadbalance
    }
    consul.local:53 {
        errors
        cache 30
        proxy . 10.150.0.1
    }
```


