# LAINCTL

集群的常规管理操作，如节点的添加与删除等。都可用 `lainctl` 完成。

lainctl 需要运行在 lain master 节点上，因为它要可能需要使用ansible playbook。


## 安装

```sh
git clone --depth=1 https://github.com/laincloud/lainctl.git
cd lainctl
python setup.py install
```


## 命令手册

### node

集群的节点管理。

- **list**

  `lainctl node list`

  列出集群的所有节点。

- **inspect**

  `lainctl node inspect NODE_NAME`

  查看一个集群节点的详细信息

- **add**
  
  `lainctl node add -p PLAYBOOKS [-P SSH_PORT] [-d DOCKER_DEVICE] [-c CID] [-s SECRET] [-r REDIRECT_URI] [-u SSO_URL] [-q] NDOE_NAME:NODE_IP [NODE_NAME:NODE_IP ...]`

  - `--p/--playbooks`: ansible playbooks 目录路径
  - `-d/--docker-device`: 初始化 `devicemapper` 的磁盘。若不指定，docker 使用 loopback device 作为存储启动。
  - `-c/--cid`: TODO
  - `-s/--secret`: TODO
  - `-r/--redirect-uri`: TODO
  - `-u/--sso-url`: TODO
  - `-q/--quiet`: 安静模式，不需要登录。

  为集群增加一个节点。在这之前需要确保当前机器到新 node 的 ssh 自动认证是通过的，可通过下面命令完成:

  ```sh
  ssh-copy-id -i /root/.ssh/lain.pub root@NODE_IP
  ```

  假设新增 node 节点的 IP 地址是 `192.168.77.22`，我们将其命名为 node2。则增加节点的命令为:

  ```sh
  lainctl node add node2:192.168.77.22
  ```

- **remove**

  `lainctl node remove [-t TARGET] -p PLAYBOOKS NODE_NAME`

  - `-t/--target`: 如果节点上有swarm-manager，则 `--target` 需要被指定。
  - `--p/--playbooks`: ansible playbooks 目录路径
  - `NODE_NAME`: 节点名

  将一个节点从集群中移除。需要确保要移除的 node 上没有业务的 container，否则操作会被拒绝。业务的 container 可通过 `lainctl drift` 命令将其移到其他节点上。

  若被删除节点是一个 `swarm-manager`，需要对其进行迁移，`--target` 指定迁移的目标节点。

- **clean**

  `lainctl node clean -p PLAYBOOKS nodes [nodes ...]`

  - `--p/--playbooks`: ansible playbooks 目录路径

  清理一个节点上没用的image，释放磁盘空间。

  每个节点上会为业务保留最新3个版本的image，更老的image会被清空。以及已经不在这太机器上运行的业务的 image 也会被清空。


- **health**

  `lainctl node cluster`

  检查节点上各个基础组件运行是否正常。

### cluster

  `lain cluster health`

  检查集群的基础组件是否正常，包括 `etcd`, `swarm`, `deployd`, `console`。

### drift

  `lain drift [--with-volume] [--ignore-volume] -p PLAYBOOKS [-t TARGET] CONTAINER [CONTAINER ...]`

  - `--p/--playbooks`: ansible playbooks 目录路径
  - `--with-volume`: 是否迁移 volume 数据。如果设置，则在迁移 container 之前会把数据 rsync 到指定节点。
  - `--ignore-volume`: 是否忽略 volume 数据。如果设置，则 container 的 volume 会直接被忽略掉，直接迁移 container。
  - `-t/--target`: 要迁移到的节点。如果不设置，container 会随机迁移到一个节点上。当 container 存在 volume 且需要迁移，则 `--target` 不能被省略。
  - `CONTAINER`: container ID 或 名字


  把一个 container 迁移到另外一个节点上。

  在 container 有 volume 的情况下，`--with-volume` 和 `--ignore-volume` 有且只有一个可以被设置。

### auth

  `lain auth {open,close}`

  - `open`: 打开 auth
  - `close`: 关闭 auth


  集群认证功能的开关设置。

### version

  `lainctl version`

  显示 lainctl 版本信息

