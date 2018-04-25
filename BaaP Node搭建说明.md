# BaaP 搭建节点使用说明文档

备注：`仅限于ubuntu系统`

#### 1. 下载pdx节点安装包
**下载地址：http://124.204.47.69/pdx-baap/pdx-baap.tar.gz**  

#### 2. 设置环境变量PDX_BAAP_HOME
**增加变量：vi /etc/profile，在最后一行增加export PDX_BAAP_HOME=/data/pdx/baap**   
**`备注：目录自定义,此文档中的环境变量目录均为/data/pdx/baap`**  
**变量生效：source /etc/profile**  
**验证是否生效：echo $PDX_BAAP_HOME**

#### 3. 解压pdx安装文件到环境变量PDX_BAAP_HOME目录下
**tar -zxvf pdx-baap.tar.gz -C /data/pdx/baap**

#### 4. 节点初始化
**cd /data/pdx/baap**   
**./baap.py -acct 本机公网ip**    
**./eth.py -seed 739 -ip 主节点ip**

#### 5. 启动协议栈
**cd /data/pdx/baap**    
**./eth.py -start 739**

#### 6. 启动pdx-baap
**cd /data/pdx/baap**    
**./baap.py -start**  

#### 7.停止pdx-baap
**cd /data/pdx/baap**    
**./baap.py -stop** 

#### 8. 停止协议栈
**cd /data/pdx/baap**    
**./eth.py -stop 739**

#### 9. 查看协议栈启动状态
**cd /data/pdx/baap**    
**./eth.py -stat 739**

#### 10. 查看pdx-baap启动状态
**cd /data/pdx/baap**    
**./baap.py -stat**

**注意项：**
* *1 第5和第6按顺序执行*
* *2 停止协议栈必须停止pdx-baap*
