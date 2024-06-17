---
date: 2024-04-10
---
https://nacos.io/
https://nacos.io/docs/latest/quickstart/quick-start/
# 集群模式部署
https://nacos.io/docs/latest/guide/admin/cluster-mode-quick-start/
## 官网说明如下：
推荐用户把所有服务列表放到一个vip下面，然后挂到一个域名下面
[http://nacos.com:port/openAPI](http://nacos.com:port/openAPI) 域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式

| 端口   | 与主端口的偏移量 | 描述                              |     |
| ---- | -------- | ------------------------------- | --- |
| 8848 | 0        | 主端口，客户端、控制台及OpenAPI所使用的HTTP端口   |     |
| 9848 | 1000     | 客户端gRPC请求服务端端口，用于客户端向服务端发起连接和请求 |     |
| 9849 | 1001     | 服务端gRPC请求服务端端口，用于服务间同步等         |     |
| 7848 | -1000    | Jraft请求服务端端口，用于处理服务端间的Raft相关请求  |     |
**使用VIP/nginx请求时，需要配置成TCP转发，不能配置http2转发，否则连接会被nginx断开。** **9849和7848端口为服务端之间的通信端口，请勿暴露到外部网络环境和客户端测。**
### 1. 预备环境准备
1. 64 bit OS Linux/Unix/Mac，推荐使用Linux系统。
2. 64 bit JDK 1.8+；[下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html). [配置](https://docs.oracle.com/cd/E19182-01/820-7851/inst_cli_jdk_javahome_t/)。
3. Maven 3.2.x+；[下载](https://maven.apache.org/download.cgi). [配置](https://maven.apache.org/settings.html)。
4. nginx作为负载均衡
5. mysql集群
6. 3个或3个以上Nacos节点才能构成集群。
### 2. 下载源码或者安装包
你可以通过两种方式来获取 Nacos。我使用第二种：
#### 下载编译后压缩包方式
您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 `nacos-server-$version.zip` 包 （windows环境）或 `nacos-server-$version.tar.gz`（linux环境）
https://github.com/alibaba/nacos/releases/download/1.4.1/nacos-server-1.4.1.tar.gz
https://github.com/alibaba/nacos/releases/download/1.4.6/nacos-server-1.4.6.tar.gz
https://github.com/alibaba/nacos/releases/download/2.2.1/nacos-server-2.2.1.tar.gz
https://github.com/alibaba/nacos/releases/download/2.3.2/nacos-server-2.3.2.tar.gz   截止2024年4月18日的最新版本

```
  unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz  cd nacos/bin
```
注：如果想单机搭建伪集群，可复制nacos安装包，修改为nacos8849，nacos8850，nacos8851
### 3. 配置集群配置文件
在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip。（请配置3个或3个以上节点）

\# ip:port
200.8.9.16:8848
200.8.9.17:8848
200.8.9.18:8848

![[Pasted image 20240418031818.png|325]]
![[Pasted image 20240418031933.png]]
![[Pasted image 20240418032622.png|750]]
![[Pasted image 20240418032609.png|500]]














