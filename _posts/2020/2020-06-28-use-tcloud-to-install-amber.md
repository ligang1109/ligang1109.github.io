Amber是一套分子动力学模拟程序，我们今天来说下如何使用云服务器安装部署这套程序。

https://ambermd.org/images/amber_mol_fitted.jpg

# 缘起

自己的一个朋友是做科研工作的，不久前他找到我向我咨询一个关于科学计算的需求：

他在做蛋白和药物对接相关的研究，希望使用分子动力学模拟软件***Amber*** （https://ambermd.org/），这款科学计算软件也在材料科学中有着广泛的应用。

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
1. 针对使用频率不高的场景，可以按量付费，节约成本。
1. 云服务高可用，无需担心主机故障带来的服务不可用问题。
1. 单机性能不够可快速扩容。
1. 软件环境部署仅需一次，之后可以制作为镜像，未来不再会有软件环境部署成本。

朋友欣然接受了我的提议，并拜托我帮他部署好整个***Amber***环境。

这里我记录下使用腾讯云（https://cloud.tencent.com/）部署***Amber***环境的整个过程。

# 部署GPU云服务器环境

这里我参考了[【玩转腾讯云】GPU云服务器(驱动篇)](https://cloud.tencent.com/developer/article/1608269) 这篇文章，成功部署好GPU云服务器环境。但因为我对CentOS更为熟悉，所以操作系统使用的CentOS 7.6版本。

部署机器选择：

![instance](https://raw.githubusercontent.com/ligang1109/ligang1109.github.io/master/images/2020-06-28/instance.png)

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

# 编译gpu并行版本

./configure --with-python /root/.conda/envs/amber18/bin/python -cuda -mpi -noX11 gnu
make -j8 install
```

### 测试

```
export DO_PARALLEL="mpirun -np 8"

make test.cuda_parallel
```

测试时可以观察gpu的运行状况：

```
watch -n 10 nvidia-smi
```

可以看到：

```
Every 10.0s: nvidia-smi                                                                               Sun Jun 28 17:18:57 2020

Sun Jun 28 17:18:57 2020
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.33.01    Driver Version: 440.33.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:08.0 Off |                    0 |
| N/A   41C    P0    55W /  70W |   1219MiB / 15109MiB |    100%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0     16583      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16584      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16585      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16586      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16587      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16588      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16589      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
|    0     16590      C   /root/amber18/bin/pmemd.cuda_DPFP.MPI        151MiB |
+-----------------------------------------------------------------------------+
```

到这里***Amber***的软件环境我们就部署完成了。

# 后续工作

做好环境后，我们可以利用云服务器的镜像制作功能为部署好的软件环境制作自定义镜像，这样做有如下好处：

1. 可随时使用该镜像创建新的计算实例。
1. 之后机器上的软件环境有问题随时可用该镜像恢复。
1. 可以使用腾讯云提供的 [批量计算](https://cloud.tencent.com/document/product/599) 及 [弹性伸缩](https://cloud.tencent.com/document/product/377) 服务解决算力不足问题。
1. 可使用镜像的分享功能分享给其他需要的人。（这里也要注意软件授权问题）

# 参考资料

- [nvidia developer](https://developer.nvidia.com/)
- [【玩转腾讯云】GPU云服务器(驱动篇)](https://cloud.tencent.com/developer/article/1608269) 
- [Amber](https://ambermd.org/)
- [镜像服务](https://cloud.tencent.com/document/product/213/4940)

