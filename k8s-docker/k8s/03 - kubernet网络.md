<!-- Kubernetes approaches networking somewhat differently than Docker does by default. There are 4 distinct networking problems to solve: -->
Kubernetes 实现网络的方式与docker不同。以下是需要解决的4个主要问题。

<!-- 1. Highly-coupled container-to-container communications: this is solved by[pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)and`localhost`communications. -->
1. 容器间高耦合的网络连接：pods解决了这个问题。

<!-- 2. Pod-to-Pod communications: this is the primary focus of this document. -->
2. pod到pod之间的通信: 这是这篇文档的主要目标（后文还会专门说明）。

<!-- 3. Pod-to-Service communications: this is covered by[services](https://kubernetes.io/docs/concepts/services-networking/service/). -->
3. pod到Service之间的通信: 这关系到service的实现。

<!-- 4. External-to-Service communications: this is covered by[services](https://kubernetes.io/docs/concepts/services-networking/service/). -->
4. 集群外部到Service的通信: 这关系到service的实现。

<!-- Kubernetes assumes that pods can communicate with other pods, regardless of which host they land on. Every pod gets its own IP address so you do not need to explicitly create links between pods and you almost never need to deal with mapping container ports to host ports. This creates a clean, backwards-compatible model where pods can be treated much like VMs or physical hosts from the perspectives of port allocation, naming, service discovery, load balancing, application configuration, and migration. -->
Kubernetes 假定所有的 pods 之间可以相互通信，并且不论 pods 部署在哪台机器上。每个 pods 有自己的 IP 地址，所以你不需要再 pods 之间创建连接，也不需要将 pods 映射到主机的端口上。这创造了一个整洁的，向后兼容的模式，从端口分配，命名，服务发现，负载均衡，应用程序配置和迁移的角度来看，pod可以像VM或物理主机一样。

<!-- There are requirements imposed on how you set up your cluster networking to achieve this. -->

这一切依赖于集群的网络配置。

## Docker model

<!-- Before discussing the Kubernetes approach to networking, it is worthwhile to review the “normal” way that networking works with Docker. By default, Docker uses host-private networking. It creates a virtual bridge, called`docker0`by default, and allocates a subnet from one of the private address blocks defined in[RFC1918](https://tools.ietf.org/html/rfc1918)for that bridge. For each container that Docker creates, it allocates a virtual Ethernet device (called`veth`) which is attached to the bridge. The veth is mapped to appear as`eth0`in the container, using Linux namespaces. The in-container`eth0`interface is given an IP address from the bridge’s address range. -->
在讨论 kubernetes 实现网络的方式之前，我们有必要回顾一下 Docker 的网络模式。默认情况下 Docker 使用一个 host-private （主机内私有）网络。Docker 默认创建一个叫`docker0`的虚拟网桥，同时为该网桥定义一个子网。每一个容器 Docker 的创建， 将分配一个虚拟的 Ethernet 设备（`veth`），这个设连接到`docker0`虚拟网桥。这个veth在容器内部被映射为`eth0`, 它将被分配一个与网桥网段相同的IP地址。

<!-- The result is that Docker containers can talk to other containers only if they are on the same machine (and thus the same virtual bridge). Containers on different machines can not reach each other - in fact they may end up with the exact same network ranges and IP addresses. -->
使用这种方式，在同一台服务器上的 Docker 容器可以与其他 Docker 容器相互通讯（他们连接到相同的虚拟网桥）。不同的服务器上的容器无法相互通讯。

<!-- In order for Docker containers to communicate across nodes, there must be allocated ports on the machine’s own IP address, which are then forwarded or proxied to the containers. This obviously means that containers must either coordinate which ports they use very carefully or ports must be allocated dynamically. -->
为了使 Docker 容器跨节点进行通信，必须在计算机自己的 IP 地址上分配端口，然后将这些端口转发或代理到容器。这意味着容器必须非常小心地协调它们使用的端口，或者必须动态分配端口。

## Kubernetes model[]

<!-- Coordinating ports across multiple developers is very difficult to do at scale and exposes users to cluster-level issues outside of their control. Dynamic port allocation brings a lot of complications to the system - every application has to take ports as flags, the API servers have to know how to insert dynamic port numbers into configuration blocks, services have to know how to find each other, etc. Rather than deal with this, Kubernetes takes a different approach. -->
在多个开发人员之间协调端口的难度非常大，并且需要用户去解决集群级别的问题，这些问题超出了他们的解决范围。 动态端口分配给系统带来了很多的困难 - 每个应用需要端口作为一个标记, API 服务器必须知道如何将动态端口加入到自己的配置文件， 服务需要知道如何去联系对方，等等。
Kubernetes采取了不同的解决方法。

<!-- Kubernetes imposes the following fundamental requirements on any networking implementation (barring any intentional network segmentation policies): -->

Kubernetes对任何网络实现都强加了以下基本要求（除非有任何有意的网络分段策略）：

<!-- - all containers can communicate with all other containers without NAT
- all nodes can communicate with all containers (and vice-versa) without NAT
- the IP that a container sees itself as is the same IP that others see it as -->
- 所有的容器可以在没有 NAT 的情况下 与所有其他容器通信。
- 所有的 nodes 可以在没有 NAT 的情况下与所有容器通信（反之亦然）
- 容器看到的自己的IP与其他人看到该容器的IP相同

<!-- What this means in practice is that you can not just take two computers running Docker and expect Kubernetes to work. You must ensure that the fundamental requirements are met. -->
这在实践中意味着您不能只使用两台运行Docker的计算机并期望Kubernetes工作。您必须确保满足基本要求。

<!-- This model is not only less complex overall, but it is principally compatible with the desire for Kubernetes to enable low-friction porting of apps from VMs to containers. If your job previously ran in a VM, your VM had an IP and could talk to other VMs in your project. This is the same basic model. -->
这种模式总体上不那么复杂，而且与Kubernetes希望将应用程序从VM轻松移植到容器的愿望兼容。如果您的程序以前在VM中运行，则您的 VM 具有 IP 并且可以与项目中的其他VM通信。他们的基础模型相同。

<!-- Until now this document has talked about containers. In reality, Kubernetes applies IP addresses at the`Pod`scope - containers within a`Pod`share their network namespaces - including their IP address. This means that containers within a`Pod`can all reach each other’s ports on`localhost`. This does imply that containers within a`Pod`must coordinate port usage, but this is no different than processes in a VM. This is called the “IP-per-pod” model. This is implemented, using Docker, as a “pod container” which holds the network namespace open while “app containers” (the things the user specified) join that namespace with Docker’s`--net=container:<id>`function.  -->

实际上，Kubernetes在 `Pod` 范围内应用IP地址 - `Pod` 共享其 network namespaces 内的容器 - 包括它的 IP 地址。这意味着 `Pod` 内的容器都Pod可以到达彼此的localhost端口。这确实意味着在相同 `Pod` 中的容器必须协调使用中的端口，但这与在 VM 中的进程没有什么不同。这称为 "IP-per-pod" 模型。这是使用Docker 实现的 "pod容器" ，它保持网络名称空间打开，而"app容器"（用户指定的东西）通过Docker的--net=container:<id>功能加入该名称空间。


<!-- As with Docker, it is possible to request host ports, but this is reduced to a very niche operation. In this case a port will be allocated on the host`Node`and traffic will be forwarded to the `Pod`. The`Pod`itself is blind to the existence or non-existence of host ports. -->




