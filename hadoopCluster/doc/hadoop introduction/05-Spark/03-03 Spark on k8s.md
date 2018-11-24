# 00-03 在Kubernetes上运行Spark[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#running-spark-on-kubernetes)

Spark可以在[Kubernetes](https://kubernetes.io/)管理的集群上运行。此功能使用已添加到Spark的本机Kubernetes调度程序。

**Kubernetes调度程序目前是实验性的。在未来的版本中，可能会出现围绕配置，容器映像和入口点的行为变化。**

# 先决条件[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#prerequisites)

- Spark 2.3或更高版本的可运行分发版。
- 运行版本> = 1.6的Kubernetes集群，并使用[kubectl为其](https://kubernetes.io/docs/user-guide/prereqs/)配置访问权限。如果您还没有可用的Kubernetes群集，则可以使用[minikube](https://kubernetes.io/docs/getting-started-guides/minikube/)在本地计算机上设置测试群集。
  - 我们建议使用最新版本的minikube并启用DNS插件。
  - 请注意，默认的minikube配置不足以运行Spark应用程序。我们建议使用3个CPU和4g内存，以便能够使用单个执行程序启动简单的Spark应用程序。
- 您必须具有相应的权限才能列出，创建，编辑和删除群集中的[pod](https://kubernetes.io/docs/user-guide/pods/)。您可以通过运行来验证是否可以列出这些资源`kubectl auth can-i <list|create|edit|delete> pods`。
  - 必须允许驱动程序窗格使用的服务帐户凭据创建窗格，服务和配置映射。
- 您必须在群集中配置[Kubernetes DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)。

# 运行方式[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#how-it-works)

![Spark集群组件](http://spark.apache.org/docs/latest/img/k8s-cluster-mode.png "Spark集群组件")

`spark-submit`可以直接用于将Spark应用程序提交到Kubernetes集群。提交机制的工作原理如下：

- Spark创建一个在[Kubernetes pod中](https://kubernetes.io/docs/concepts/workloads/pods/pod/)运行的Spark驱动程序。
- 驱动程序创建执行程序，这些执行程序也在Kubernetes pod中运行并连接到它们，并执行应用程序代码。
- 当应用程序完成时，执行程序窗格会终止并清理，但驱动程序窗格会保留日志并在Kubernetes API中保持“已完成”状态，直到它最终被垃圾收集或手动清理。

请注意，在完成状态，驾驶员舱并*没有*使用任何的计算或存储资源。

驱动程序和执行程序pod调度由Kubernetes处理。可以通过[节点选择器](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector)使用其配置属性在可用节点的子集上调度驱动程序和执行程序窗格。在将来的版本中，可以使用更高级的调度提示，如[节点/ pod关联](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)。

# 向Kubernetes提交申请[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#submitting-applications-to-kubernetes)

## Docker image[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#docker-images)

Kubernetes要求用户提供可以部署到pod中的容器中的镜像。这些镜像构建为在Kubernetes支持的容器运行时环境中运行。Docker是一个容器运行时环境，经常与Kubernetes一起使用。Spark（从2.​​3版本开始）附带了一个可用于此目的的Dockerfile，或者可以自定义以满足各个应用程序的需求。它可以在`kubernetes/dockerfiles/`目录中找到。

Spark还附带了一个`bin/docker-image-tool.sh`脚本，可用于构建和发布Docker镜像以与Kubernetes后端一起使用。

示例用法是：

```
$ ./bin/docker-image-tool.sh -r <repo> -t my-tag build
$ ./bin/docker-image-tool.sh -r <repo> -t my-tag push
```

## 群集模式[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#cluster-mode)

要在群集模式下启动Spark Pi，

```
$ bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=<spark-image> \
    local:///path/to/examples.jar
```

通过将`--master`命令行参数传递给应用程序配置`spark-submit`或通过设置`spark.master`在应用程序配置中指定的Spark主服务器必须是具有该格式的URL`k8s://<api_server_url>`。对主字符串进行前缀`k8s://`将导致Spark应用程序在Kubernetes集群上启动，并在其中联系API服务器`api_server_url`。如果URL中未指定HTTP协议，则默认为`https`。例如，将master设置`k8s://example.com:443`为等同于将其设置为`k8s://https://example.com:443`，但要在不同端口上没有TLS的情况下连接，则master将设置为`k8s://http://example.com:8080`。

在Kubernetes模式下，默认情况下使用由其指定的Spark应用程序名称`spark.app.name`或`--name`参数to`spark-submit`来命名创建的Kubernetes资源，如驱动程序和执行程序。因此，应用程序名称必须由小写字母数字字符，`-`以及`.`必须重新开始，并以字母数字字符结束。

如果您有Kubernetes群集设置，则通过执行来发现apiserver URL的一种方法`kubectl cluster-info`。

```
$ kubectl cluster-info
Kubernetes master is running at http://127.0.0.1:6443
```

在上面的示例中，可以`spark-submit`通过指定`--master k8s://http://127.0.0.1:6443`spark-submit的参数来使用特定的Kubernetes集群。此外，还可以使用身份验证代理`kubectl proxy`与Kubernetes API进行通信。

本地代理可以通过以下方式启动：

```
$ kubectl proxy
```

如果本地代理在localhost：8001上运行，`--master k8s://http://127.0.0.1:8001`则可以用作spark-submit的参数。最后，请注意，在上面的示例中，我们指定了一个具有特定URI的jar，其方案为`local://`。此URI是Docker镜像中已有的示例jar的位置。

## 依赖管理[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#dependency-management)

如果您的应用程序的依赖项都托管在远程位置（如HDFS或HTTP服务器）中，则可以通过相应的远程URI引用它们。此外，应用程序依赖项可以预先安装到自定义构建的Docker镜像中。可以通过使用`local://`URI引用它们和/或`SPARK_EXTRA_CLASSPATH`在Dockerfiles中设置环境变量，将这些依赖项添加到类路径中。`local://`在引用定制的Docker镜像中的依赖关系时，也需要该方案`spark-submit`。请注意，目前尚不支持使用提交客户端的本地文件系统中的应用程序依赖项。

### 使用远程依赖项[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#using-remote-dependencies)

当存在远程位置（如HDFS或HTTP服务器）中存在应用程序依赖项时，驱动程序和执行程序窗格需要Kubernetes[init容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)来下载依赖项，以便驱动程序和执行程序容器可以在本地使用它们。

init-container处理在`spark.jars`（或`--jars`选项`spark-submit`）和`spark.files`（或`--files`选项`spark-submit`）中指定的远程依赖项。它还处理远程托管的主应用程序资源，例如主应用程序jar。以下显示了使用`spark-submit`命令使用远程依赖项的示例：

```
$ bin/spark-submit \
    --master k8s://https://<k8s-apiserver-host>:<k8s-apiserver-port> \
    --deploy-mode cluster \
    --name spark-pi \
    --class org.apache.spark.examples.SparkPi \
    --jars https://path/to/dependency1.jar,https://path/to/dependency2.jar
    --files hdfs://host:port/path/to/file1,hdfs://host:port/path/to/file2
    --conf spark.executor.instances=5 \
    --conf spark.kubernetes.container.image=<spark-image> \
    https://path/to/examples.jar
```

## 秘密管理[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#secret-management)

Kubernetes[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)可用于为Spark应用程序提供访问安全服务的凭据。要将用户指定的秘密安装到驱动程序容器中，用户可以使用表单的配置属性`spark.kubernetes.driver.secrets.[SecretName]=<mount path>`。类似地，表单的配置属性`spark.kubernetes.executor.secrets.[SecretName]=<mount path>`可用于将用户指定的秘密安装到执行程序容器中。请注意，假设要安装的秘密与驱动程序和执行程序窗格的名称相同。例如，要在驱动程序和执行程序容器中将命名秘密`spark-secret`装入路径`/etc/secrets`，请在`spark-submit`命令中添加以下选项：

```
--conf spark.kubernetes.driver.secrets.spark-secret=/etc/secrets
--conf spark.kubernetes.executor.secrets.spark-secret=/etc/secrets
```

请注意，如果使用init-container，则安装到驱动程序容器中的任何秘密也将安装到驱动程序的init-container中。类似地，安装到执行程序容器中的任何秘密也将被安装到执行程序的init容器中。

## 内省和调试[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#introspection-and-debugging)

这些是您可以通过不同方式调查正在运行/已完成的Spark应用程序，监视进度和执行操作的方法。

### 访问日志[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#accessing-logs)

可以使用Kubernetes API和`kubectl`CLI访问日志。当Spark应用程序运行时，可以使用以下方法从应用程序流式传输日志：

```
$ kubectl -n=<namespace> logs -f <driver-pod-name>
```

如果安装在群集上，也可以通过[Kubernetes仪表板](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)访问相同的日志。

### 访问驱动程序UI[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#accessing-driver-ui)

可以使用本地访问与任何应用程序关联的UI[`kubectl port-forward`](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/#forward-a-local-port-to-a-port-on-the-pod)。

```
$ kubectl port-forward <driver-pod-name> 4040:4040
```

然后，可以访问Spark驱动程序UI`http://localhost:4040`。

### 调试[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#debugging)

可能有几种失败。如果Kubernetes API服务器拒绝来自spark-submit的请求，或者由于其他原因拒绝连接，则提交逻辑应指示遇到的错误。但是，如果在运行应用程序期间出现错误，通常最好的方法是通过Kubernetes CLI进行调查。

要获取有关围绕驱动程序窗格做出的调度决策的一些基本信息，您可以运行：

```
$ kubectl describe pod <spark-driver-pod>
```

如果pod遇到运行时错误，则可以使用以下命令进一步探测状态：

```
$ kubectl logs <spark-driver-pod>
```

可以用类似的方式检查失败的执行程序窗格的状态和日志。最后，删除驱动程序窗格将清理整个spark应用程序，包括所有执行程序，相关服务等。驱动程序窗格可以被认为是Spark应用程序的Kubernetes表示。

## Kubernetes特色[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#kubernetes-features)

### 命名空间[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#namespaces)

Kubernetes具有[命名空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)的概念。命名空间是在多个用户之间划分群集资源的方法（通过资源配额）。Spark on Kubernetes可以使用命名空间来启动Spark应用程序。这可以通过`spark.kubernetes.namespace`配置来使用。

Kubernetes允许使用[ResourceQuota](https://kubernetes.io/docs/concepts/policy/resource-quotas/)在各个名称空间上设置资源限制，对象数量等。管理员可以组合使用命名空间和ResourceQuota来控制运行Spark应用程序的Kubernetes集群中的共享和资源分配。

### RBAC[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#rbac)

在启用了[RBAC的](https://kubernetes.io/docs/admin/authorization/rbac/)Kubernetes集群中，用户可以配置各种Spark on Kubernetes组件使用的Kubernetes RBAC角色和服务帐户来访问Kubernetes API服务器。

Spark驱动程序窗格使用Kubernetes服务帐户访问Kubernetes API服务器以创建和监视执行程序窗格。驱动程序窗格使用的服务帐户必须具有相应的权限，以便驱动程序能够执行其工作。具体而言，至少必须授予服务帐户一个[`Role`或`ClusterRole`](https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole)允许驱动程序容器创建容器和服务的服务帐户。默认情况下，如果在创建pod时未指定服务帐户，则会`default`在指定的命名空间中自动为驱动程序pod分配服务帐户`spark.kubernetes.namespace`。

根据部署的Kubernetes的版本和设置，此`default`服务帐户可能具有或不具有允许驱动程序pod在默认Kubernetes[RBAC](https://kubernetes.io/docs/admin/authorization/rbac/)策略下创建pod和服务的[角色](https://kubernetes.io/docs/admin/authorization/rbac/)。有时，用户可能需要指定具有正确角色的自定义服务帐户。Spark on Kubernetes支持通过配置属性指定驱动程序pod使用的自定义服务帐户`spark.kubernetes.authenticate.driver.serviceAccountName=<service account name>`。例如，要使驱动程序pod使用`spark`服务帐户，用户只需将以下选项添加到`spark-submit`命令：

```
--conf spark.kubernetes.authenticate.driver.serviceAccountName=spark
```

要创建自定义服务帐户，用户可以使用该`kubectl create serviceaccount`命令。例如，以下命令创建名为的服务帐户`spark`：

```
$ kubectl create serviceaccount spark
```

授予服务帐户a`Role`或`ClusterRole`，a`RoleBinding`或`ClusterRoleBinding`需要。要创建一个`RoleBinding`或`ClusterRoleBinding`，用户可以使用`kubectl create rolebinding`（或`clusterrolebinding`for`ClusterRoleBinding`）命令。例如，以下命令`edit``ClusterRole`在`default`命名空间中创建一个并将其授予`spark`上面创建的服务帐户：

```
$ kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default
```

请注意，a`Role`只能用于授予对单个命名空间内资源（如pod）的访问权限，而a`ClusterRole`可用于授予对所有命名空间中的群集范围资源（如节点）以及命名空间资源（如pod）的访问权限。对于Kubernetes上的Spark，由于驱动程序总是在同一名称空间中创建执行程序窗格，因此`Role`用户可以使用a`ClusterRole`来完成。有关RBAC授权以及如何为pod配置Kubernetes服务帐户的更多信息，请参阅[使用RBAC授权](https://kubernetes.io/docs/admin/authorization/rbac/)和[配置](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)Pod的[服务帐户](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)。

## 客户端模式[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#client-mode)

目前不支持客户端模式。

## 未来的工作[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#future-work)

Kubernetes上有几个Spark功能，这些功能目前正在一个分支 -[apache-spark-on-k8s / spark中孵化](https://github.com/apache-spark-on-k8s/spark)，预计最终将成为spark-kubernetes集成的未来版本。

其中一些包括：

- PySpark
- [R
- 动态执行器缩放
- 本地文件依赖关系管理
- Spark应用程序管理
- 作业队列和资源管理

如果要尝试这些功能并向开发团队提供反馈，可以参考[文档](https://apache-spark-on-k8s.github.io/userdocs/)。

# 组态[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#configuration)

有关Spark配置的信息，请参阅[配置页面](http://spark.apache.org/docs/latest/configuration.html)。以下配置特定于Spark on Kubernetes。

#### Spark属性[](http://spark.apache.org/docs/latest/running-on-kubernetes.html#spark-properties)

| 物业名称                                                          | 默认                                            | 含义                                                                                                                                                                                                                                         |
| ------------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `spark.kubernetes.namespace`                                  | `default`                                     | 将用于运行驱动程序和执行程序窗格的命名空间。                                                                                                                                                                                                                     |
| `spark.kubernetes.container.image`                            | `(none)`                                      | 用于Spark应用程序的容器映像。这通常是形式`example.com/repo/spark:v1.0.0`。除非为每种不同的容器类型提供显式图像，否则此配置是必需的并且必须由用户提供。                                                                                                                                              |
| `spark.kubernetes.driver.container.image`                     | `(value of spark.kubernetes.container.image)` | 用于驱动程序的自定义容器映像。                                                                                                                                                                                                                            |
| `spark.kubernetes.executor.container.image`                   | `(value of spark.kubernetes.container.image)` | 用于执行程序的自定义容器映像。                                                                                                                                                                                                                            |
| `spark.kubernetes.container.image.pullPolicy`                 | `IfNotPresent`                                | 在Kubernetes中拉图像时使用的容器图像拉取策略。                                                                                                                                                                                                               |
| `spark.kubernetes.allocation.batch.size`                      | `5`                                           | 每轮执行器pod分配中一次启动的pod数。                                                                                                                                                                                                                      |
| `spark.kubernetes.allocation.batch.delay`                     | `1s`                                          | 在每轮执行器pod分配之间等待的时间。指定小于1秒的值可能会导致火花驱动程序的CPU使用率过高。                                                                                                                                                                                           |
| `spark.kubernetes.authenticate.submission.caCertFile`         | （没有）                                          | 启动驱动程序时通过TLS连接到Kubernetes API服务器的CA证书文件的路径。此文件必须位于提交计算机的磁盘上。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                                         |
| `spark.kubernetes.authenticate.submission.clientKeyFile`      | （没有）                                          | 客户端密钥文件的路径，用于在启动驱动程序时针对Kubernetes API服务器进行身份验证。此文件必须位于提交计算机的磁盘上。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                                     |
| `spark.kubernetes.authenticate.submission.clientCertFile`     | （没有）                                          | 客户端证书文件的路径，用于在启动驱动程序时对Kubernetes API服务器进行身份验证。此文件必须位于提交计算机的磁盘上。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                                      |
| `spark.kubernetes.authenticate.submission.oauthToken`         | （没有）                                          | 在启动驱动程序时对Kubernetes API服务器进行身份验证时使用的OAuth令牌。请注意，与其他身份验证选项不同，这应该是用于身份验证的令牌的确切字符串值。                                                                                                                                                          |
| `spark.kubernetes.authenticate.submission.oauthTokenFile`     | （没有）                                          | OAuth令牌文件的路径，其中包含在启动驱动程序时对Kubernetes API服务器进行身份验证时要使用的令牌。此文件必须位于提交计算机的磁盘上。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                           |
| `spark.kubernetes.authenticate.driver.caCertFile`             | （没有）                                          | 在请求执行程序时，通过TLS从驱动程序窗口连接到Kubernetes API服务器的CA证书文件的路径。此文件必须位于提交机器的磁盘上，并将上载到驱动程序窗格。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                     |
| `spark.kubernetes.authenticate.driver.clientKeyFile`          | （没有）                                          | 客户端密钥文件的路径，用于在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证。此文件必须位于提交机器的磁盘上，并将上载到驱动程序窗格。将其指定为路径而不是URI（即不提供方案）。如果指定了此项，则强烈建议为驱动程序提交服务器设置TLS，因为此值是以明文方式传递给驱动程序窗格的敏感信息。                                                                             |
| `spark.kubernetes.authenticate.driver.clientCertFile`         | （没有）                                          | 客户端证书文件的路径，用于在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证。此文件必须位于提交机器的磁盘上，并将上载到驱动程序窗格。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                   |
| `spark.kubernetes.authenticate.driver.oauthToken`             | （没有）                                          | 在请求执行程序时，从驱动程序窗格中对Kubernetes API服务器进行身份验证时使用的OAuth令牌。请注意，与其他身份验证选项不同，这必须是用于身份验证的令牌的确切字符串值。此标记值将上载到驱动程序窗格。如果指定了此项，则强烈建议为驱动程序提交服务器设置TLS，因为此值是以明文方式传递给驱动程序窗格的敏感信息。                                                                            |
| `spark.kubernetes.authenticate.driver.oauthTokenFile`         | （没有）                                          | OAuth令牌文件的路径，其中包含在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证时要使用的令牌。请注意，与其他身份验证选项不同，此文件必须包含用于身份验证的令牌的确切字符串值。此标记值将上载到驱动程序窗格。如果指定了此项，则强烈建议为驱动程序提交服务器设置TLS，因为此值是以明文方式传递给驱动程序窗格的敏感信息。                                                             |
| `spark.kubernetes.authenticate.driver.mounted.caCertFile`     | （没有）                                          | 在请求执行程序时，通过TLS从驱动程序窗口连接到Kubernetes API服务器的CA证书文件的路径。必须可以从驱动程序窗格访问此路径。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                                |
| `spark.kubernetes.authenticate.driver.mounted.clientKeyFile`  | （没有）                                          | 客户端密钥文件的路径，用于在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证。必须可以从驱动程序窗格访问此路径。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                              |
| `spark.kubernetes.authenticate.driver.mounted.clientCertFile` | （没有）                                          | 客户端证书文件的路径，用于在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证。必须可以从驱动程序窗格访问此路径。将其指定为路径而不是URI（即不提供方案）。                                                                                                                                              |
| `spark.kubernetes.authenticate.driver.mounted.oauthTokenFile` | （没有）                                          | 包含OAuth令牌的文件的路径，在请求执行程序时从驱动程序窗格中对Kubernetes API服务器进行身份验证时使用。必须可以从驱动程序窗格访问此路径。请注意，与其他身份验证选项不同，此文件必须包含用于身份验证的令牌的确切字符串值。                                                                                                                      |
| `spark.kubernetes.authenticate.driver.serviceAccountName`     | `default`                                     | 运行驱动程序窗格时使用的服务帐户。从API服务器请求执行程序窗格时，驱动程序窗格使用此服务帐户。请注意，这不能与CA证书文件，客户端密钥文件，客户端证书文件和/或OAuth令牌一起指定。                                                                                                                                              |
| `spark.kubernetes.driver.label.[LabelName]`                   | （没有）                                          | 将指定的标签添加`LabelName`到驱动程序窗格。例如，`spark.kubernetes.driver.label.something=true`。请注意，Spark还会在驱动程序窗格中添加自己的标签，以便进行簿记。                                                                                                                            |
| `spark.kubernetes.driver.annotation.[AnnotationName]`         | （没有）                                          | 将指定的注释添加`AnnotationName`到驱动程序窗格。例如，`spark.kubernetes.driver.annotation.something=true`。                                                                                                                                                    |
| `spark.kubernetes.executor.label.[LabelName]`                 | （没有）                                          | 将指定的标签添加`LabelName`到执行程序窗格。例如，`spark.kubernetes.executor.label.something=true`。请注意，Spark还会在驱动程序窗格中添加自己的标签，以便进行簿记。                                                                                                                          |
| `spark.kubernetes.executor.annotation.[AnnotationName]`       | （没有）                                          | 将指定的注释添加`AnnotationName`到执行程序窗格。例如，`spark.kubernetes.executor.annotation.something=true`。                                                                                                                                                  |
| `spark.kubernetes.driver.pod.name`                            | （没有）                                          | 驱动程序窗格的名称。如果未设置，则驱动程序窗格名称将设置为“spark.app.name”，后缀为当前时间戳，以避免名称冲突。                                                                                                                                                                            |
| `spark.kubernetes.executor.lostCheck.maxAttempts`             | `10`                                          | 驱动程序尝试确定特定执行程序的丢失原因的次数。丢失原因用于确定执行程序失败是由于框架还是应用程序错误，而错误又反过来决定执行程序是被删除和替换，还是处于失败状态以进行调试。                                                                                                                                                     |
| `spark.kubernetes.submission.waitAppCompletion`               | `true`                                        | 在群集模式下，是否在退出启动程序进程之前等待应用程序完成。当更改为false时，启动Spark作业时启动器会出现“即发即忘”行为。                                                                                                                                                                          |
| `spark.kubernetes.report.interval`                            | `1s`                                          | 群集模式下当前Spark作业状态报告之间的间隔。                                                                                                                                                                                                                   |
| `spark.kubernetes.driver.limit.cores`                         | （没有）                                          | 为驱动程序指定硬CPU [限制]（https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container）荚。                                                                                |
| `spark.kubernetes.executor.limit.cores`                       | （没有）                                          | 为每个执行者指定硬CPU [限制]（https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#resource-requests-and-limits-of-pod-and-container） pod为Spark应用程序启动。                                                                |
| `spark.kubernetes.node.selector.[labelKey]`                   | （没有）                                          | 添加到驱动程序窗格和执行程序窗格的节点选择器，使用键`labelKey`和值作为配置的值。例如，设置`spark.kubernetes.node.selector.identifier`为`myIdentifier`将导致驱动程序窗格和执行程序具有带键`identifier`和值的节点选择器`myIdentifier`。通过使用此前缀设置多个配置，可以添加多个节点选择器键。                                               |
| `spark.kubernetes.driverEnv.[EnvironmentVariableName]`        | （没有）                                          | 将指定的环境变量添加`EnvironmentVariableName`到Driver进程。用户可以指定其中的多个来设置多个环境变量。                                                                                                                                                                         |
| `spark.kubernetes.mountDependencies.jarsDownloadDir`          | `/var/spark-data/spark-jars`                  | 在驱动程序和执行程序中下载jar的位置。此目录必须为空，并将作为空目录卷挂载在驱动程序和执行程序窗格上。                                                                                                                                                                                       |
| `spark.kubernetes.mountDependencies.filesDownloadDir`         | `/var/spark-data/spark-files`                 | 在驱动程序和执行程序中下载jar的位置。此目录必须为空，并将作为空目录卷挂载在驱动程序和执行程序窗格上。                                                                                                                                                                                       |
| `spark.kubernetes.mountDependencies.timeout`                  | 300S                                          | 在中止尝试从远程位置下载和解压缩依赖关系到驱动程序和执行程序窗体之前的超时（以秒为单位）。                                                                                                                                                                                              |
| `spark.kubernetes.mountDependencies.maxSimultaneousDownloads` | 五                                             | 在驱动程序或执行程序窗格中同时下载的最大远程依赖项数。                                                                                                                                                                                                                |
| `spark.kubernetes.initContainer.image`                        | `(value of spark.kubernetes.container.image)` | 自定义容器映像，用于驱动程序和执行程序的init容器。                                                                                                                                                                                                                |
| `spark.kubernetes.driver.secrets.[SecretName]`                | （没有）                                          | 在名称中指定的路径上添加名为驱动程序窗格的[Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/)`SecretName`。例如，`spark.kubernetes.driver.secrets.spark-secret=/etc/secrets`。请注意，如果使用init-container，则秘密​​也将添加到驱动程序窗格中的init-container中。   |
| `spark.kubernetes.executor.secrets.[SecretName]`              | （没有）                                          | 在名称中指定的路径上添加名为执行程序窗格的[Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/)`SecretName`。例如，`spark.kubernetes.executor.secrets.spark-secret=/etc/secrets`。请注意，如果使用init-container，则秘密​​也将添加到执行程序窗格中的init-container中。 |
