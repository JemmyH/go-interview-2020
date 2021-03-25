### 1. 安装 `Java`

```bash
brew cask install homebrew/cask-versions/adoptopenjdk8
```

### 2. 安装 `kafka`

```bash
brew install kafka
```

>   没必要单独安装 `zookeeper` ，`kafka` 中自带了 `zk`，自己安装启动会徒增复杂度。

### 3. 配置 `kafka`

修改 `kafak` 配置文件：`/usr/local/etc/kafka/server.properties`，找到 `listeners=PLAINTEXT://:9092`，注释掉那一行（你甚至可以修改 `kafka` 启动的端口）

### 4. 启动

```bash
brew services start zookeeper
brew services start kafka
```

### 5. 创建 topic

```bash
kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic my-topic
```

### 6. 生产消息

```bash
$ kafka-console-producer --broker-list localhost:9092 --topic my-topic
> 1
> 2
```

### 7. 消费消息

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic my-topic --from-beginning
```