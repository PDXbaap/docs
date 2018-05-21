# PDX blockchain deployment guide

Note that PDX blockchain only supports Linux X64 with a kernel version of 3.10.0 or above.

### 1. Download installation package

http://download.pdxtech.biz:8181/pdx-baap/pdx-baap.tar.gz

### 2. Set environment variable

`PDX_BAAP_HOME=/data/pdx/baap`

And make it effective immediately

### 3. Untar to install PDX blockchain

```shel
tar -zxvf pdx-baap.tar.gz -C $PDX_BAAP_HOME
```

### 4. Node initialization

```shell
cd /data/pdx/baap

./baap.py -acct public-ip-of-local-node

./eth.py -seed 739 -ip public-ip-of-seed-node
```

### 5. Start blockchain engine

```shell
cd /data/pdx/baap
./eth.py -start 739
```

### 6. Start blockchain platform

```shell
cd /data/pdx/baap

./baap.py -start
```

### 7.Stop blockchain platform

```shell
cd /data/pdx/baap

./baap.py -stop
```

### 8. Stop blockchain engine 

```shell
cd /data/pdx/baap

./eth.py -stop 739
```

### 9. View status of blockchain engine

```shell
cd /data/pdx/baap

./eth.py -stat 739
```

### 10. View status of blockchain platform

```shell
cd /data/pdx/baap

./baap.py -stat
```

Notes:

* *1 Step 5) & step 6) must be run in order to start PDX blockchain*
* *2 The blockchain platform must be stopped when stopping the blockchain engine*
