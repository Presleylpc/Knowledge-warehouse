# 05-docker状态查看命令集

## docker inspect

用法 : docker inspect [docker名称/docker short id] 
作用 ：docker inspect 获取容器/镜像的元数据。主要参数有：
-f ：指定返回值的模板文件。
-type ：为指定类型返回JSON。
以下为常用输出信息

### 查看容器完整id

```
# docker inspect pymom_managerselect_dev -f '{{.Id}}'
cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63
```

### 查看容器的各种状态

```
# docker inspect pymom_managerselect_dev -f '{{.State.Status}}'
running
# docker inspect pymom_managerselect_dev -f '{{.State.Running}}'
true
# docker inspect pymom_managerselect_dev  -f '{{.State.Paused}}'
false
# docker inspect pymom_managerselect_dev  -f '{{.State.Restarting}}'
false
# docker inspect pymom_managerselect_dev  -f '{{.State.OOMKilled}}'
false
# docker inspect pymom_managerselect_dev  -f '{{.State.Dead}}'
false
# docker inspect pymom_managerselect_dev  -f '{{.State.Pid}}'
135329
# docker inspect pymom_managerselect_dev  -f '{{.State.Health.Status}}'
healthy
```

### 查看容器相关文件的文件路径

```
# docker inspect pymom_managerselect_dev -f '{{.ResolvConfPath}}'
/var/lib/docker/containers/cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63/resolv.conf
# docker inspect pymom_managerselect_dev -f '{{.HostnamePath}}'
/var/lib/docker/containers/cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63/hostname
# docker inspect pymom_managerselect_dev -f '{{.HostsPath}}'
/var/lib/docker/containers/cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63/hosts
# docker inspect pymom_managerselect_dev -f '{{.LogPath}}'
/var/lib/docker/containers/cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63/cd6ff29a4db67653909edf28eb46761842983f89b0ea36d9db7a81399e99bf63-json.log
```

## docker stats

显示容器资源占用的实时信息
--no-stream标识关闭实时状态，仅显示当前时间点的状态。

```
# docker stats pymom_managerselect_dev --no-stream
CONTAINER ID        NAME                      CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
cd6ff29a4db6        pymom_managerselect_dev   0.07%               486.1MiB / 125.6GiB   0.38%               982kB / 2.35MB      13.1MB / 0B         83
```

## docker info

docker info用于显示系统信息，主要有下面这些：

```
# docker info
Containers: 175
 Running: 175
 Paused: 0
 Stopped: 0
Images: 515
Server Version: 18.02.0-ce
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 9b55aab90508bd389d7654c4baf173a981477d55
runc version: 9f9c96235cc97674e935002fc3d78361b696a69e
init version: 949e6fa
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-693.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 48
Total Memory: 125.6GiB
Name: operation_server.pycf
ID: NSPI:LQWX:4CWD:WZIX:VTNA:RBDI:MV5R:BZLU:XXJP:7WXS:2U4L:Y4LT
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 192.168.0.156
 127.0.0.0/8
Registry Mirrors:
 http://617811b8.m.daocloud.io/
Live Restore Enabled: false
```

## docker top

显示容器中正在运行的进程。

```
# docker top pymom_managerselect_dev 
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                135329              135229              0                   18:47               ?                   00:02:15            java -Xms128m -Xmx256m -jar /java8/app.jar --server.port=5000 --spring.profiles.active=dev
```

## docker port

```
# docker port pymom_managerselect_dev 
5000/tcp -> 0.0.0.0:6502
```
