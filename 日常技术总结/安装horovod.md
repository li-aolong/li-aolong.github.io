## 安装OpenMPI

下载https://download.open-mpi.org/release/open-mpi/v4.0/openmpi-4.0.0.tar.gz

安装

```
gunzip -c openmpi-4.0.0.tar.gz | tar xf -
cd openmpi-4.0.0
./configure --prefix=/usr/local ## 很多输出 ##
make all install ## 很多输出 ##
```

环境变量

```
vim ~/.bashrc

export PATH=/usr/local/lib:$PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

source ~/.bashrc
```

## 安装NCCL

```
git clone https://github.com/NVIDIA/nccl.git
```

按照readme安装

## 安装mxnet

```
pip install mxnet-cu100
```

## 安装horovod

要指定NCCL三个路径

```
HOROVOD_NCCL_HOME=/home/xxxx/build HOROVOD_NCCL_LIB=/home/xxxx/build/lib HOROVOD_NCCL_INCLUDE=/home/xxxx/build/include HOROVOD_GPU_ALLREDUCE=NCCL pip install --no-cache-dir horovod
```

## 测试

```
python -c "import horovod.tensorflow as hvd;"
```

