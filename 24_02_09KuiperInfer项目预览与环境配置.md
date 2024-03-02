# 学习目标

- 实现一个深度学习推理框架
- 设计、编写一个计算图
- 实现常见的算子，例如卷积、池化、全连接
- 学会如何进行算子的优化加速
- 使用自己的推理框架推理常见模型，检查结果是否能够和torch对齐



# 什么是推理框架？

推理框架用于**对已经训练完成的模型进行加载**，并根据模型文件中的网络结构和权重参数，对输入图像进行预测。

推理框架没有反向传播，因为推理过程中权重不需要更新。这也是和训练框架的最大的不同。

推理框架的运行流程可参照下图：

![image-20240209085251275](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240209085251275.png)



# 技术全景

KuiperInfer包括以下几个模块：

1. `Operator`：计算图中的计算节点，包括：
   - **存储输入输出的张量**，用于存放各层的输入输出
   - **节点的类型和名称**，名称是唯一的，用于区分任意一个节点，例如`Convolution`
   - **节点的参数信息**，例如卷积步长、卷积核大小
   - **节点的权重信息**，例如`weight`, `bias`
2. `Graph`：多个`Operator`串联得到的有向无环图，规定了节点的执行顺序
3. `Layer`：运算的具体执行者，首先读入输入张量中的数据，然后对输入帐量进行计算，并将结果放入输出张量中
4. `Tensor`：存放多维数据，方便在节点中传递，该结构同时也封装矩阵运算

![image-20240209090440415](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240209090440415.png)



# 环境配置

主要库：

1. 数学计算：Armadillo，该库是Open Blas的封装
2. 加速库：OpenMP
3. 单元测试：Google Test
4. 性能测试：Google Benchmark

第二次开课提供了docker框架，更省事一些。



# 什么是Docker

## 为什么使用Docker

出现背景：不同的电脑的环境配置不同，导致在一个系统上运行正常的程序，在另一个系统上不能正常运行。

解决这个问题的一个方法是，构建和源系统一样的虚拟机，这种方法通常会占用大量内存来支持Guest OS。与之相对，Docker在这方面省略了大量内存占用。且Docker允许在不同的容器之间共享和重用数据空间，也方便在不同平台之间移植。

## DevOps模式

DevOps模式：开发（development）和运维（operation）团队合作，使得可以对边缘用户持续交付（continuous delivery）。

![image-20240213103855582](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240213103855582.png)

![image-20240213103959971](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240213103959971.png)

## Docker是什么

Docker是用于自动化部署应用程序到轻量级的容器中的工具，使得应用可以在不同的运行环境中高效运行。

容器（container）是一种软件包，包含所有运行依赖。 

![image-20240213104250687](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240213104250687.png)

Docker为每个软件都对应在容器中提供其依赖的框架（framework），使得不同框架的软件，甚至冲突框架的软件，可以在同一宿主机上运行，甚至可以进行数据共享。

## Docker是如何工作的

- Docker是安装在宿主机上的基础引擎，主要功能是build和运行容器
- 使用client-server架构
- Client和Server使用REST API交互
- Client运行指令，指令通过REST API转译，发送到Server
- Server检查Client请求，在操作系统上响应操作

![image-20240213105213672](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240213105213672.png)



# Win环境下Docker环境配置

首先安装Docker，进入[Docker官网](www.docker.com)，点击选择products中的Docker DeskTop下载并安装。注意安装后需要重启，记得提前关闭其他应用并保存。

打开Docker DestTop，我我这里出现卡starting的问题，推测可能是Hyper-V的原因，查阅[microsoft手册](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)，首先尝试在powershell中启用Hyper-V

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

提示`Microsoft-Hyper-V`未知，说明没安装，尝试安装。将下面的文本存入一个cmd文件中，管理员身份运行，然后重启

```
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```

再次启动Docker-DeskTop，这次没有卡住。

验证安装，命令行中输入：

```
docker run hello-world
```

出现以下文本，说明安装成功

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

现在开始拉环境

1. 拉取镜像

   ```
   docker pull registry.cn-hangzhou.aliyuncs.com/hellofss/kuiperinfer:datawhale
   ```

2. 克隆课程代码

   ```
   git clone https://github.com/zjhellofss/kuiperdatawhale.git
   ```

3. 创建并运行容器

   ```
   docker run -it -p 7860:22 registry.cn-hangzhou.aliyuncs.com/hellofss/kuiperinfer:datawhale /bin/bash
   ```

4. 使用ssh连接容器

   ```
   ssh -p 7860 me@127.0.0.1
   ```

   > 如果连不上的话可以在Docker DeskTop里面重启一下容器试试。



## VS配置Docker

先补一点自己不熟悉的知识：

1. CMake. CMake是一个跨平台build system，用于在不同平台上创建build。
2. Visual Studio with CMake. VS内嵌了CMake，可以创建CMake的项目，会自动检测CMakeLists.txt文件，并生成必要的项目文件。
3. VS使用Docker，调试 > 选项 > 跨平台 > 连接管理器 > 添加。Win平台本机启动的docker主机名为127.0.0.1，端口、用户名和密码按照自己设置的填入即可。在跨平台 > 开发容器中，设置用于运行容器的主机为127.0.0.1

> 因为这个docker是linux的，所以需要在VS installer中，安装用于Linux的C++开发组件，参考：https://devblogs.microsoft.com/cppblog/build-c-applications-in-a-linux-docker-container-with-visual-studio/。
>
> 不装这个的话，即使能够连接到docker，也不能设置调试主机为docker容器。

4. 设置调试主机为docker容器

5. 配置新的debuger

   ![image-20240216212625232](D:\College\projects\KuiperInfer_notes\24_02_09KuiperInfer项目预览与环境配置.assets\image-20240216212625232.png)

6. 尝试生成，出现新的错误：`无法创建目录，mkdir 退出代码: 1`，用户权限问题？

   定位，在docker容器中，打开终端管理，`su me`登录me账户，尝试在`./root/home/me`文件夹下创建文件，报`Permission denied`，说明me用户没有足够权限。

   切root账户，在根目录下，`chown -R me home`，一定要给`home`的权限，否则还是不能mkdir

7. cmake没找到ninja，在debug高级配置中，改用Unix Makefiles，镜像里是没装Ninja的，且这个CMakefile也不支持Ninja，如果不小心用Ninja生成过一次，那改Unix Makefiles也还会报错，需要重新clone

8. 编译完成后，就可以直接run了，会提示FAILED，这个是后面要配置的

整理一下使用VS配置环境的关键问题：

1. 用户权限要给到home文件夹，确保在docker控制台，使用me用户（或者自己创建的用户）能够在home路径下创建文件
2. 在选项 > 跨平台 > 开发容器中，连接配置好的远程容器，如果容器是Linux环境，则需要先安装Linux组件；设置调试主机为该容器，在管理配置中设置新的CMake配置，修改配置中的主机为该容器，修改generator为UNIX Makefiles。



# 编写单元测试

使用`GoogleTest`编写单元测试，测试`armadillo`的计算接口。接口可参考`armadillo`的[手册](https://arma.sourceforge.net/docs.html)。

在`test/test1.cpp`中，包含了对加减乘和点积运算的接口，作业要求实现`axby.cpp`中的接口。

查手册找算子，对照实现即可。

1. 实现$y = w \times x + b$​

   ```c++
   void Axby(const arma::fmat &x, const arma::fmat &w, const arma::fmat &b,
             arma::fmat &y) {
       y = w * x + b;
     // 把代码写这里 完成y = w * x + b的运算
   }
   ```

   

2. 实现$y = e^{-x}$​

   ```c++
   void EPowerMinus(const arma::fmat &x, arma::fmat &y) {
     // 把代码写这里 完成y = e^{-x}的运算
       arma::fmat E(224, 224, arma::fill::value(arma::datum::e));
       y = pow(E, -x);
   }
   ```

   

3. 实现$Y = a \times x + y$​

   ```c++
   void Axpy(const arma::fmat &x, arma::fmat &Y, float a, float y) {
     // 编写Y = a * x + y
       Y = a * x + y;
   }
   ```

编译运行，PASSED，\^_\^





