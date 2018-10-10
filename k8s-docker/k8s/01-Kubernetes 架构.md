# Kubernetes architecture

[原文地址](https://github.com/kubernetes/kubernetes/blob/release-1.3/docs/design/architecture.md)
<!-- A running Kubernetes cluster contains node agents (kubelet) and master components (APIs, scheduler, etc), on top of a distributed storage solution. This diagram shows our desired eventual state, though we're still working on a few things, like making kubelet itself (all our components, really) run within containers, and making the scheduler 100% pluggable. -->

一个运行的 kubernetes 集群，包涵 agents 节点（`kubelet`）和主节点组件（APIs，调度器等等），这些构建于分布式存储之上。下图展示了一个我们期望的架构，尽管我们还在完善，比如让`kebelet`（所有的组件）运行在容器当中，并使所有的调度程序100％可插拔。

![Architecture Diagram](https://raw.githubusercontent.com/kubernetes/kubernetes/release-1.3/docs/design/architecture.png)

## The Kubernetes Node

<!-- When looking at the architecture of the system, we'll break it down to services that run on the worker node and services that compose the cluster-level control plane. -->

当我们审视整个系统架构时，我们将其分解为两部分，在工作节点上运行的服务和群级控制层面的服务。

<!-- The Kubernetes node has the services necessary to run application containers and be managed from the master systems. -->

Kubernetes node 节点具有运行应用程序容器所需的服务，并可从 master 系统进行管理。

<!--Each node runs Docker, of course. Docker takes care of the details of downloading images and running containers. -->

当然，每个节点都要运行Docker。Docker负责下载镜像，管理容器运行的细节。

### `kubelet`

<!--The kubelet manages pods and their containers, their images, their volumes, etc.-->

`kubelet` 管理 pods 和他们的容器， 镜像， 磁盘卷等资源。

### `kube-proxy`

<!-- Each node also runs a simple network proxy and load balancer (see the services FAQ for more details). This reflects services (see the services doc for more details) as defined in the Kubernetes API on each node and can do simple TCP and UDP stream forwarding (round robin) across a set of backends. -->

每个节点还运行一个简单的网络代理和负载均衡器（有关详细信息，请参阅[服务常见问题解答](https://github.com/kubernetes/kubernetes/wiki/Services-FAQ)）。这反映了`services`（在每个节点上的Kubernetes API中定义[的服务文档](https://github.com/kubernetes/kubernetes/blob/release-1.3/docs/user-guide/services.md)以获取更多详细信息），并且可以跨一组后端执行简单的TCP和UDP流转发（循环）。

<!--Service endpoints are currently found via [DNS](../admin/dns.md) or through
environment variables (both
[Docker-links-compatible](https://docs.docker.com/userguide/dockerlinks/) and
Kubernetes `{FOO}_SERVICE_HOST` and `{FOO}_SERVICE_PORT` variables are
supported). These variables resolve to ports managed by the service proxy.-->

Service endpoints 目前通过DNS或环境变量找到。这些变量解析为 service proxy 管理的端口。

## The Kubernetes Control Plane

<!-- The Kubernetes control plane is split into a set of components. Currently they
all run on a single _master_ node, but that is expected to change soon in order
to support high-availability clusters. These components work together to provide
a unified view of the cluster. -->

Kubernetes控制层分为几部分组件。目前它们都在一个主节点上运行（single _master_ node），但预计很快就会改变，以支持集群的高可用。这些组件协同工作以提供群集的统一视图。



### `etcd`

<!-- All persistent master state is stored in an instance of `etcd`. This provides a
great way to store configuration data reliably. With `watch` support,
coordinating components can be notified very quickly of changes. -->

集群中 master 节点的所有持久状态都存储在`etcd`中。它提供了一种可靠的方法，存储配置的数据。通过对`watch`的支持，可以非常快速地将变更通知到各个组件。

### Kubernetes API Server

<!-- The apiserver serves up the [Kubernetes API](../api.md). It is intended to be a
CRUD-y server, with most/all business logic implemented in separate components
or in plug-ins. It mainly processes REST operations, validates them, and updates
the corresponding objects in `etcd` (and eventually other stores). -->

apiserver提供Kubernetes API。它旨在成为一个 CRUD-y 服务器，大多数/所有业务逻辑在单独的组件或插件中实现。apiserver主要处理 REST 操作，验证它们，并更新etcd（或者其他存储）中的相应对象。

### Scheduler

<!-- The scheduler binds unscheduled pods to nodes via the `/binding` API. The
scheduler is pluggable, and we expect to support multiple cluster schedulers and
even user-provided schedulers in the future. -->

Scheduler 程序通过 `/binding` API 将未调度的 pod 绑定到节点。调度程序是可拔插的，我们希望将来支持多个集群调度程序甚至用户提供的调度程序。

### Kubernetes Controller Manager Server

<!--All other cluster-level functions are currently performed by the Controller
Manager. For instance, `Endpoints` objects are created and updated by the
endpoints controller, and nodes are discovered, managed, and monitored by the
node controller. These could eventually be split into separate components to
make them independently pluggable. -->


所有其他集群级功能当前由 Controller Manager 执行。例如，`Endpoints` 对象由 `endpoints controller` 创建和更新，节点由 `node controller` 发现，管理和监视。这些最终可以分成单独的组件，使它们可以独立插入。

<!-- The [`replicationcontroller`](../user-guide/replication-controller.md) is a
mechanism that is layered on top of the simple [`pod`](../user-guide/pods.md)
API. We eventually plan to port it to a generic plug-in mechanism, once one is
implemented. -->

`replication controller`是一种机制，它在simple pod API 之上的层级。一旦实现，我们最终计划将其移植成为通用插件机制。

