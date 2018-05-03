# BaaP 搭建节点使用说明文档

备注：`仅限于ubuntu系统`

### 1. 下载安装包

下载地址：http://124.204.47.69:8181/pdx-baap/pdx-baap.tar.gz

### 2. 设置环境变量

增加变量 `PDX_BAAP_HOME=/data/pdx/baap`

且使之生效

### 3. 解压

```shel
tar -zxvf pdx-baap.tar.gz -C /data/pdx/baap
```

### 4.节点初始化

```shell
cd /data/pdx/baap

./baap.py -acct 本机公网ip

./eth.py -seed 739 -ip 主节点ip
```

### 5. 启动协议栈

```shell
cd /data/pdx/baap
./eth.py -start 739
```

### 6. 启动pdx-baap

```shell
cd /data/pdx/baap

./baap.py -start
```

### 7.停止pdx-baap

```shell
cd /data/pdx/baap

./baap.py -stop
```

### 8. 停止协议栈

```shell
cd /data/pdx/baap

./eth.py -stop 739
```

### 9. 查看协议栈启动状态

```shell
cd /data/pdx/baap

./eth.py -stat 739
```

### 10. 查看pdx-baap启动状态

```shell
cd /data/pdx/baap

./baap.py -stat
```

注意项：

* *1 第5和第6按顺序执行*
* *2 停止协议栈必须停止pdx-baap*
