# 启用 FQDN 访问

本文介绍如何启用基于完全限定域名（fully qualified domain name，FQDN）访问 StarRocks 节点。 FQDN 是指可以在 Internet 上访问的特定实体的**完整域名**。 FQDN 由两部分组成：主机名和域名。

StarRocks v2.4 之前版本仅支持通过 IP 地址访问每个节点。即便使用 FQDN 添加节点到集群，该 FQDN 仍会被强制转换为 IP 地址。如果更改集群中某些节点的 IP 地址，将会导致无法访问该节点。在 v2.4 版中，StarRocks 将每个节点与其 IP 地址解耦。启用 FQDN 访问后，您可以直接通过节点的 FQDN 管理对应的节点。

## 前提条件

如需为 StarRocks 集群启用 FQDN 访问，请确保满足以下要求。

- 集群中的所有机器都必须配置有主机名。

- 必须在集群中每台机器的 **/etc/hosts** 文件中指定集群中其他机器对应的 IP 地址和 FQDN。

- 不可在 **/etc/hosts** 文件中重复的 IP 地址。

## 新集群启用 FQDN 访问

如需建立启用 FQDN 访问的新集群，您需要安装 StarRocks v2.4.0 或更高版本。有关如何安装 StarRocks 的详细说明，请参阅 [手动部署 StarRocks](../quick_start/Deploy.md)。

新集群中的 FE 节点在启动时默认启用 IP 地址访问。

新集群中的 BE 节点将通过 FE 元数据中定义的 `BE Address` 判定使用 FQDN 或 IP 地址标识自身。例如，如果 `BE Address` 记录了一个 BE 节点的 FQDN，则该 BE 节点使用这个 FQDN 来标识自身。

因此，如需在启动新节点时启用 FQDN 访问，您无需在 FE 和 BE 配置文件 **fe.conf** 和 **be.conf** 中指定属性 `priority_networks`。在满足前提条件中的要求后，您可以按照 [手动部署 StarRocks](../quick_start/Deploy.md) 中的说明启动各节点。

如果您需要在启动新节点时启用 IP 地址访问，您仍然需要在启动节点之前在相应的 FE 和 BE 配置文件中指定属性 `priority_networks`。除此之外，由于启动时默认不采用 IP 地址访问，您必须通过以下命令启动 FE 节点：

```Shell
sh bin/start_fe.sh --host_type IP --daemon
```

`--host_type` 属性用于指定该节点的访问方式。有效值包括 `FQDN` 和 `IP`。您只需在第一次启动节点时指定该属性。

## 旧集群启用 FQDN 访问

如需为旧集群启用 FQDN 访问，您首先需要将集群 **升级** 至v2.4.0 或更高版本。

### 为 FE 节点启用 FQDN 访问

在为 FE 节点启用 FQDN 访问时，您需要先修改所有非 Leader Follower FE 节点，最后修改 Leader FE 节点。

> **注意**
>
> 请您确保集群中至少有 3 个 Follower FE 节点，否则无法为 FE 节点启用 FQDN 访问。

#### 修改非 Leader Follower FE 节点

1. 进入 FE 节点的部署目录，执行以下命令停止 FE 节点。

    ```Shell
    sh bin/stop_fe.sh --daemon
    ```

2. 通过 MySQL 客户端执行以下语句，查看该 FE 节点的 `Alive` 状态直至变为 `false`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

3. 执行以下语句，将该 FE 节点的 IP 地址变更为 FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令启动该 FE 节点。

    ```Shell
    sh bin/start_fe.sh --host_type FQDN --daemon
    ```

    `--host_type` 属性用于指定该节点的访问方式。有效值包括 `FQDN` 和 `IP`。您只需在修改后第一次启动节点时指定该属性。

5. 查看该 FE 节点的 `Alive` 状态直至变为 `true`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

6. 在当前FE节点的 `Alive` 状态为 `true` 后，重复上述步骤，依次为其他非 Leader Follower FE 节点启用 FQDN 访问。

#### 修改 Leader Follower FE 节点

在修改完所有非 Leader Follower FE 节点并成功重启后，您就可以为 Leader FE 节点启用 FQDN 访问了。

> **说明**
>
> 在 Leader FE 节点启用 FQDN 访问之前，用于添加新节点的 FQDN 仍会被转换为 IP 地址。在集群重新选出启用了 FQDN 访问的 Leader FE 节点后，添加新节点的 FQDN 将不会被转换为 IP 地址。

1. 进入 Leader FE 节点的部署目录，执行以下命令停止该节点。

    ```Shell
    sh bin/stop_fe.sh --daemon
    ```

2. 执行以下语句，检查集群是否选举出新的 Leader FE 节点。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    任何 `Alive` 和 `isMaster` 为 `true` 的 FE 节点即为正常运行的 Leader FE 节点。

3. 执行以下语句，将该 FE 节点的 IP 地址变更为 FQDN。

    ```SQL
    ALTER SYSTEM MODIFY FRONTEND HOST "<fe_ip>" TO "<fe_hostname>";
    ```

4. 运行以下命令启动该 FE 节点。

    ```Shell
    sh bin/start_fe.sh --host_type FQDN --daemon
    ```

    `--host_type` 属性用于指定该节点的访问方式。有效值包括 `FQDN` 和 `IP`。您只需在修改后第一次启动节点时指定该属性。

5. 查看该 FE 节点的 `Alive` 状态直至变为 `true`。

    ```SQL
    SHOW PROC '/frontends'\G
    ```

    如果 `Alive` 状态变为 `true`，则当前 FE 节点被成功修改并作为 Follower FE 节点添加到集群中。

### 为 BE 节点启用 FQDN 访问

执行以下语句，为 BE 节点启用 FQDN 访问。

```SQL
ALTER SYSTEM MODIFY BACKEND HOST "<be_ip>" TO "<be_hostname>";
```

> **说明**
>
> 为 BE 节点启用 FQDN 访问后，您无需重启该 BE 节点。

## 常见问题

**问：在为 FE 节点启用 FQDN 访问时返回错误：“required 1 replica. But none were active with this master”。该如何处理？**

答：请确保集群中至少有 3 个 Follower FE 节点，否则无法为 FE 节点启用 FQDN 访问。

**问：是否可以通过 IP 地址将新节点添加到启用了 FQDN 访问的集群？**

答：可以。
