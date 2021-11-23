---
title: Docker容器中JVM资源限制
date: 2021-11-23 15:33:42
tags:
---

## 问题背景

众所周知，Docker 容器时利用 [CGroup](https://docs.docker.com/config/containers/resource_constraints/) 对进程使用的资源进行限制的。
而旧版本的JVM（低于 8u131）与top/free等系统命令有类似的问题，并不会自动识别CGroup的资源限制。这将导致JVM读取和分配的是整台机器的资源，一旦进程使用的资源超过容器的限制就会被Docker杀死，造成Java应用OOM。
很明显，Java社区很快也意识到了[这个问题](https://blogs.oracle.com/java/post/java-se-support-for-docker-cpu-and-memory-limits)，在后续的版本里进行了支持。

## 8u131版本

从 [8u131](https://www.oracle.com/java/technologies/javase/8u131-relnotes.html) 版本开始支持 UseCGroupMemoryLimitForHeap 和 MaxRAMFraction 这两个选项，用 CGroup 中限制的内存资源来作为分配的依据。选项默认是不开启的，需要开启 UnlockExperimentalVMOptions 才能使用。

下面通过 Docker 对容器内的 JVM 限制 100MB 的内存，对比是否开启选项的效果。

### 未开启UseCGroupMemoryLimitForHeap

可以看到 JVM 并未感知到 Docker(Cgroup) 对内存的限制，仍然为JVM Max. Heap Size 分配 (443.00MB) 超过资源限制。

```shell
(base) ➜  ~ docker run -m 100MB openjdk:8u131-alpine java -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 443.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_131"
OpenJDK Runtime Environment (IcedTea 3.4.0) (Alpine 8.131.11-r2)
OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

### 开启UseCGroupMemoryLimitForHeap

JVM感知到 Docker(Cgroup) 对内存的限制，根据比例分配JVM Max. Heap Size 为 44.50MB。

```shell
(base) ➜  ~ docker run -m 100MB openjdk:8u131-alpine java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 44.50M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_131"
OpenJDK Runtime Environment (IcedTea 3.4.0) (Alpine 8.131.11-r2)
OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

## 8u191版本

从 [8u191](https://www.oracle.com/java/technologies/javase/8u191-relnotes.html) 版本开始引入了 UseContainerSupport 选项，而且是默认启用的。该功能不仅能像 UseCGroupMemoryLimitForHeap 感知内存的资源限制，还能感知 CPU 的限制。

### 关闭UseContainerSupport

可以看到 JVM 并未感知到 Docker(Cgroup) 对内存的限制，仍然为JVM Max. Heap Size 分配 (443.00MB) 超过资源限制。

```shell
(base) ➜  ~ docker run -m 100MB openjdk:8u191-alpine java -XX:-UseContainerSupport  -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 443.00M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_191"
OpenJDK Runtime Environment (IcedTea 3.10.0) (Alpine 8.191.12-r0)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

### 开启UseContainerSupport (默认)

JVM 默认能感知到 Docker(Cgroup) 对内存的限制，根据比例分配JVM Max. Heap Size 为 48.38MB。

```shell
(base) ➜  ~ docker run -m 100MB openjdk:8u191-alpine java -XshowSettings:vm -version
VM settings:
    Max. Heap Size (Estimated): 48.38M
    Ergonomics Machine Class: server
    Using VM: OpenJDK 64-Bit Server VM

openjdk version "1.8.0_191"
OpenJDK Runtime Environment (IcedTea 3.10.0) (Alpine 8.191.12-r0)
OpenJDK 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## 结论

1. 对于 8u191 及以上的版本，JVM已经能够比较好的感知 Docker 通过 CGroup 对容器的资源限制。
2. 对于 8u131 至 8u191 的版本，需要显式的开启 UseCGroupMemoryLimitForHeap 选项，来让 JVM 感知 Docker 对容器的资源限制。
3. 对于 8u131 以下的版本，需要用户根据Docker对资源的限制手动配置JVM参数，以防止出现非预期的OOM问题。