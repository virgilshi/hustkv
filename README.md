# 简介

hustkv是一个基于leveldb原型修改的键值存储，其底层存储引擎为SPDK，hustkv使用方法及测试步骤如下：

```bash
# clone hustkv
git clone https://github.com/virgilshi/hustkv.git
# update submodule
git submodule update --init --recursive
# make spdk
cd spdk-backend
git submodule update --init
./scripts/pkgdep.sh
make
# mkfs
HUGEMEM=5120 scripts/setup.sh
test/blobfs/mkfs/mkfs /path/to/bdev.conf nvme0
# make hustkv with spdk
cd ..
make SPDK_DIR=./spdk_backend
# run and test
out-static./db_bench --spdk=/root/bdev_restore.conf --spdk_bdev=nvme0 --benchmarks=fillrandom,readrandom
```

# 文件改动

本质上leveldb是通过系统调用实现数据持久化，IO会穿过漫长的IO栈实现落盘，此过程性能次优，为将leveldb的IO经过用户态落盘，我们用SPDK Blobfs提供的API改写了leveldb底层的系统调用，实现IO走用户态。相对于leveldb改动的文件如下，其中env_spdk.cc参考了spdk/lib/rocksdb写法：

```
db/db_bench.cc
util/env_posix.cc
spdk/env_spdk.cc
spdk/spdk.leveldb.mk
```

性能测试结果如下(随机写1GB数据)：
```
leveldb: fillrandom   :      22.005 micros/op;    5.0 MB/s
hustkv:  fillrandom   :      10.162 micros/op;   10.9 MB/s
```