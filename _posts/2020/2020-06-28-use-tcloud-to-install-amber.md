# 缘起

自己的一个朋友是做科研工作的，不久前他找到我向我咨询一个关于科学计算的需求：

他在做蛋白和药物对接相关的研究，希望使用分子动力学模拟软件***Amber*** （https://ambermd.org/）。

这款软件在运算时可以利用GPU加速极大提升计算效率，所以一开始他和我咨询的是关于GPU显卡相关的问题，但聊着聊着发现如果自行购买GPU显卡维护主机有如下问题：

1. 单台主机购买及维护成本很高，GPU硬件通常需要单独购买，更新换代不易。
1. ***Amber***的使用并不高频，感觉有些浪费。
1. 计算量大时，单台机器性能瓶颈严重，但搞多台首先成本高，安装部署更是麻烦。
1. 机器一旦出问题，修理期间服务相当于不可用。
1. 硬件环境搞定的话，软件环境的安装部署对我朋友来说有点困难。

我朋友说有同事使用超算来作为解决方案，单等待时间很长，且使用成本也不低。

上面这些问题，听起来不就是云服务可以解决的经典问题吗？

# 使用云服务解决这类问题的优点

1. 无须购买及维护硬件。
1. 即买即用，无需等待。
1. 针对使用频率不高的场景，可以按量服务，节约成本。
1. 云服务高可用，无需担心主机故障带来的服务不可用问题。
1. 单机性能不够可快速
1. 软件环境部署仅需一次，之后可以制作为镜像，未来不再会有软件环境部署成本。

朋友欣然接受了我的提议，并拜托我帮他部署好整个***Amber***环境。

这里我记录下使用腾讯云（https://cloud.tencent.com/）部署***Amber***环境的整个过程。

# 部署GPU云服务器环境

这里我参考了[【玩转腾讯云】GPU云服务器(驱动篇)](https://cloud.tencent.com/developer/article/1608269) 这篇文章，成功部署好GPU云服务器环境。但因为我对CentOS更为熟悉，所以操作系统使用的CentOS 7.6版本。

部署机器选择：



# 部署Amber

这里部署的是***Amber18***这个版本。

***Amber18***本身有两个需要安装的包，分别是：

1. AmberTools18.tar.bz2

2. Amber18.tar.bz2

其中AmberTools是免费的，但不提供GPU加速功能，如果想利用GPU加速，就需要额外付费购买Amber18。

我在部署过程中使用`root`账号在`/root`目录下操作。

## 依赖环境部署

### yum安装

```
yum install -y gcc gcc-gfortran gcc-c++ flex tcsh zlib-devel \
     bzip2-devel libXt-devel libXext-devel libXdmcp-devel \
     tkinter openmpi openmpi-devel perl perl-ExtUtils-MakeMaker \
     patch bison boost-devel
```

### MPICH安装

```
tar zxvf ~/amber_pkgs/mpich-3.3.2.tar.gz
cd mpich-3.3.2/
./configure
make -j8
make install
```

### 解压Amber

我这里解压到

```
tar jxvf amber_pkgs/Amber18.tar.bz2
tar jxvf amber_pkgs/AmberTools18.tar.bz2
```

### 安装conda环境

```
yum install -y conda

conda init bash
source ~/.bashrc
conda create -n amber18
conda activate amber18

conda install --file amber18/AmberTools/src/python_requirement.txt
```

### 设置环境变量

在~/.bashrc中添加：

```
export CUDA_HOME=/usr/local/cuda
export PATH=$PATH:$CUDA_HOME/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CUDA_HOME/lib64

export AMBER_PREFIX=$HOME/amber18
export AMBERHOME=$AMBER_PREFIX
export PATH="${AMBER_PREFIX}/bin:${PATH}"

# Add location of Amber Python modules to default Python search path
if [ -z "$PYTHONPATH" ]; then
    export PYTHONPATH="${AMBER_PREFIX}/lib/python3.8/site-packages"
else
    export PYTHONPATH="${AMBER_PREFIX}/lib/python3.8/site-packages:${PYTHONPATH}"
fi
if [ -z "${LD_LIBRARY_PATH}" ]; then
   export LD_LIBRARY_PATH="${AMBER_PREFIX}/lib"
else
   export LD_LIBRARY_PATH="${AMBER_PREFIX}/lib:${LD_LIBRARY_PATH}"
fi
```

之后source一下：

```
source ~/.bashrc
```

### 编译Amber

```
cd $AMBERHOME

# 编译串行版本

./configure --with-python /root/.conda/envs/amber18/bin/python gnu
make install

# 编译并行版本

./configure --with-python /root/.conda/envs/amber18/bin/python -mpi gnu
make install

# 编译gpu版本

./configure --with-python /root/.conda/envs/amber18/bin/python -cuda gnu
make install

# 编译gpu并行版本

./configure --with-python /root/.conda/envs/amber18/bin/python -cuda -mpi gnu
make install
```

### 测试

```
export DO_PARALLEL="mpirun --allow-run-as-root -np 8"

make test.cuda
make test.cuda_parallel
```

测试时可以观察gpu的运行状况：

```
watch -n 10 nvidia-smi
```





