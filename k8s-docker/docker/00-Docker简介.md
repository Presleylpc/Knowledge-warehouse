## 00-Docker基础概念

## Docker是什么

在官网上没有找到直接对docker的定义，docker官网上只有对容器的定义。要了解docker必须先对容器有充分的了解。

## 容器是什么

### 定义

标准化的软件单元（ A standardized unit of software）

### 将软件打包成标准化单元，用于开发，装运和部署

![](https://www.docker.com/sites/default/files/styles/large/public/container-what-is-container.png?itok=1Wir2KmG)容器是一个标准的软件单元，它将代码及其所有依赖关系打包，以便应用程序从一个计算环境快速可靠地运行到另一个计算环境。Docker容器映像是一个轻量级，独立的可执行软件包，包含运行应用程序所需的一切：代码，运行时，系统工具，系统库和设置。

容器映像（container images）在运行时成为容器（docker container），在Docker容器的情况下 - 映像在[Docker Engine](https://www.docker.com/products/docker-engine)上运行时成为容器。适用于基于Linux和Windows的应用程序，无论基础架构如何，容器化软件都将始终运行相同。容器将软件与其环境隔离开来，并确保它可以统一工作，尽管开发和分段之间存在差异。

在Docker Engine上运行的Docker容器：

- **标准：**Docker创建了容器的行业标准，因此它们可以随处携带
- **轻量级：**容器共享机器的操作系统内核，因此不需要每个应用程序的操作系统，从而提高服务器效率并降低服务器和许可成本
- **安全：**应用程序在容器中更安全，Docker提供业界最强大的默认隔离功能

> 关键点：
> 
> 1. 容器镜像是对软件以及软件运行环境的封装。软件代表软件代码，运行环境代表运行时，系统工具，系统库和设置。
> 
> 2. 容器镜像运行时成为容器。
> 
> 3. 容器运行在操作系统之上，与操作系统无关。
> 
> 4. docker与docker，docker与操作系统之间相互隔离。

### 容器与虚拟机的区别

> 容器和虚拟机具有类似的资源隔离和分配优势，但功能不同。因为容器是虚拟化操作系统而不是虚拟化硬件。容器更便携，更高效。

#### Docker 容器

![容器架构](https://www.docker.com/sites/default/files/styles/content_6_6/public/compare/docker-containerized-appliction-blue-border_2.png?itok=lsxRQ9HU)

容器是应用层的抽象，它将代码和依赖关系打包在一起。多个容器可以在同一台机器上运行，并与其他容器共享操作系统内核，每个容器在用户空间中作为独立进程运行。容器占用的空间比VM少（容器映像的大小通常为几十MB），可以处理更多的应用程序，并且需要更少的VM和操作系统。

#### 虚拟机

![虚拟机架构](https://www.docker.com/sites/default/files/styles/content_6_6/public/compare/container-vm-whatcontainer_2.png?itok=0eNn5aap)

虚拟机（VM）是物理硬件的抽象，将一台服务器转变为多台服务器。虚拟机管理程序允许多台虚拟机在一台计算机上运行。每个VM都包含操作系统的完整副本，应用程序，必要的二进制文件和库 - 占用数十GB。虚拟机也可能很慢启动。



## containerd

[了解有关containerd的更多信息](https://containerd.io/)

containerd是一个行业标准的核心容器运行时，强调简单性，健壮性和可移植性。它可用作Linux和Windows的守护程序，可以管理其主机系统的完整容器生命周期：映像传输和存储，容器执行和监视，低级存储和网络附件等。

containerd旨在嵌入到更大的系统中，而不是由开发人员或最终用户直接使用。

containerd包含一个守护进程，通过本地UNIX套接字暴露gRPC API。API是一种低级API，旨在用于更高层的包装和扩展。它还包括`ctr`专为开发和调试目的而设计的准系统CLI()。它使用[runC](https://github.com/opencontainers/runc)根据[OCI规范](https://www.opencontainers.org/about)运行容器。

![containerd架构](https://containerd.io/img/chart-a.png)

> docker的容器管理主要是通过containerd完成的，containerd提供了一个行业标准的核心容器运行时。
> 
> 关键点：
> 
> 1. containerd API是一种低级API，不能被用户直接使用。
> 
> 2. 我们使用Docker的Docker命令，其实是使用的DockerClient。DockerClient属于上图的ctr部分。
> 
> 3. ctr部分可以有多种实现，而不仅仅是DockerClient，比如k8s的运行时管理。

## 总结

通过本节应该完成以下目标：

1. 对容器有了基本的了解，容器是标准化的软件单元，对软件及软件环境进行了封装。

2. 容器的管理是通过containerd完成的，containerd提供了一个行业标准的核心容器运行时。

3. docker提供了对containerdAPI的封装，提供了用户访问的CLI接口。
