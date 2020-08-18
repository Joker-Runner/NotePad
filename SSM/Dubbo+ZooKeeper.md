# Dubbo、ZooKeeper 学习

![img](Dubbo+ZooKeeper.assets/architecture.png)

## 一、安装ZooKeeper

1. **下载zookeeper**

https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.9/

2. **解压，在conf目录下创建并配置zoo.cfg文件**

文件内容如下(zoo_sample.cfg默认配置)：

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
```

3. **到安装目录的bin目录运行 启动zookeeper**

```shell
./zookeeper.sh start
```

4. **停止zookeeper**

```shell
./zookeeper.sh stop
```

5. **前台启动zookeeper，方便查看日志**

```shell
zkServer.sh start-foreground
```

## 二、Dubbo

提供服务的远程调用

## 二、步骤

### 2.1 提供者提供服务

1. 导入依赖

   ```xml
   <dependency>
       <groupId>org.apache.dubbo</groupId>
       <artifactId>dubbo-spring-boot-starter</artifactId>
       <version>2.7.7</version>
   </dependency>
   
   <dependency>
       <groupId>com.github.sgroschupf</groupId>
       <artifactId>zkclient</artifactId>
       <version>0.1</version>
   </dependency>
   
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-framework</artifactId>
       <version>5.0.0</version>
   </dependency>
   
   <dependency>
       <groupId>org.apache.curator</groupId>
       <artifactId>curator-recipes</artifactId>
       <version>5.0.0</version>
   </dependency>
   
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.6.1</version>
       <exclusions>
           <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   ```
   
2. 配置注册中心的地址，以及服务发现名和要扫描的包

   ```properties
   # 服务应用名称
   dubbo.application.name=provider-server
   # 注册中心地址
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   # 那些服务要被注册
   dubbo.scan.base-packages=com.joker.service
   ```

3. 在想要被注册的服务上面加@DubboService注解


### 2.2 消费者消费

1. 导入依赖

   ```xml
   <!-- 同提供者 -->
   ```

2. 配置注册中心的地址，配置自己的服务名

   ```properties
   # 消费者去哪里拿服务，需要暴露自己的名字
   dubbo.application.name=consumer-server
   # 注册中心地址
   dubbo.registry.address=zookeeper://127.0.0.1:2181
   ```

3. 从远程注入服务，使用@DubboReference注解替换Spring的@AuroWired





