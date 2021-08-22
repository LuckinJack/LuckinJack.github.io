# TensorFlow安装须知

学习Tensorflow 这个框架时，推荐使用 *Tensorflow 2.1* + *Pycharm* +*Anconda3* 的组合：

* *Tensorflow* 选用2.1版本；

- *Pycharm* 作为开发用的IDE；

- *Anconda3* 用作Python的包管理和Python1虚拟环境的创建；
- *Python* 选用3.7版本（由Anconda安装）

*Tensorflow* 有CPU版本和GPU两个版本，不过GPU的加速效果要比CPU好很多



在此之前，我们需要先安装好 *Pycharm* 和 *Anconda3*。

> **[!TIP]**
> 关于 *Pycharm* 和 *Anconda3* 的安装请参考下面这两篇教程：
>
> - [Pycharm2019.3.5安装]()
> - [Anconda3安装]()

# TensorFlow2.1 安装（基于Win10）

### 创建虚拟环境

打开Anconda Prompt （以管理员模式打开），创建一个 名为 ` tf21` 虚拟环境，并且配置python版本为3.7

`conda create -n tf21_cpu python=3.7`

进入这个环境

`conda activate tf21`

> `conda deactivate`可以退出改环境

### 配置国内镜像

- conda 配置清华源

```bash
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
	conda config --set show_channel_urls yes #设置搜索时显示通道地址
```

- 输入`conda config --show channels`显示镜像源
- 输入：`conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/`移除镜像源



### 使用 pip 命令安装tensorflow

- 方案一：使用 pip 命令在线安装tensorflow

  - ```bash
    pip install tensorflow==2.1
    ```

- 方案二：使用国内镜像源（二选一）

  - ```bash
    ## 这里使用了清华的镜像，速度比默认源更快（如果网络不佳，多执行几次）
    pip install tensorflow==2.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
    ```
  - ```bash
    ## 这里使用了豆瓣的镜像，速度比默认源更快（如果网络不佳，多执行几次）
    pip install tensorflow==2.1 -i https://pypi.doubanio.com/simple
    ```

- 方案三：使用 *tensorflow* 的 `.whl` 文件进行离线版安装

  - 进入tensorflow官网 https://tensorflow.google.cn/install/pip
  - 将页面跳转到这个目录，下载我们需要的 `.whl` 文件：https://storage.googleapis.com/tensorflow/windows/cpu/tensorflow_cpu-2.1.0-cp37-cp37m-win_amd64.whl
  - 下载完成后，我们将 `.whl` 文件放入 `tf21` 虚拟环境的Scripts文件夹下
  - 接着用命令提示符，进入Scripts文件夹： cd 路径\Scripts
  - 执行命令(以具体文件名＋后缀为标准) ：`pip install tensorflow-2.1.0-cp37-cp37m-win_amd64.whl`



### 命令行中进行CPU版测试

命令行中输入Python，然后复制如下Python代码回车执行

```python
import tensorflow as tf
version = tf.__version__
gpu_ok = tf.test.is_gpu_available()
print("tf version:",version,"\nuse GPU",gpu_ok)
```

如果没有问题，会看到打印结果为：
```Python
tf version: 2.1.0
use GPU False
```

至此，CPU版本安装完毕，接下来我们继续安装GPU版本的部分



### 确认NVCUDA 显卡驱动版本

紧接着，在CPU版的基础上，我们开始安装GPU版需要的依赖，即 cudatoolkit 和 cuDnn ，在此之前，我们需要确认下自己的显卡驱动版本，请保证 NVCUDA ≥ 10.1，低于10.1请升级，否则后面安装会报错。

进入目录 `NVIDIA控制面板\帮助\系统信息\组件` ，查看你的NVCUDA版本：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/see-nvcuda-version.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">如果NVCUDA ≥ 10.1 则不用进行升级，反之则需要</div>
    <br>
    <br>
</center>



安装GPU版本，拥有Nvidia的GPU的windows一般都有默认驱动的，只需要安装cudatoolkit 与 cudnn包就可以了。我们下载的TensorFlow版本是2.1.0，python版本是3.7，对照下面的版本对应图，cudatoolkit 版本应该是10.1，cuDnn版本应该是7.6。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/tensorflow-python-gcc-bazel-cudnn-cuda.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">TensorFlow与其依赖工具的版本对应图</div>
    <br>
    <br>
</center>




> **[!TIP]**
> 该图来自于（搭梯子才能看）：
> https://www.tensorflow.org/install/source_windows



### 安装cudatoolkit和cudnn



执行命令：

`conda install cudatoolkit=10.1 cudnn=7.6`



如果 Conda 使用了清华源还是会下载失败，抛出类似`CondaHTTPError: HTTP 000 CONNECTION FAILED`的报错，那么网络有问题，我们需要改变安装方式。下面这两个分别是 cudatoolkit 和 cudnn 的源，我们可以手动用其他下载工具（比如迅雷）来下载源

https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/win-64/cudatoolkit-10.1.243-h74a9793_0.conda

https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/win-64/cudnn-7.6.5-cuda10.2_0.conda



将下载完成的包移动到Anaconda的pkgs文件夹下（比如我的是G:\Anaconda3\pkgs），在命令终端进入该目录（比如我就是 `cd G:\Anaconda3\pkgs`），然后执行 Conda 命令进行离线安装



`conda install --use-local cudatoolkit-10.1.243-h74a9793_0.conda`

`conda install --use-local cudnn-7.6.5-cuda10.2_0.conda`



### 命令行中进行GPU版测试

命令行中输入Python，然后复制以下 Python代码回车执行

```python
import tensorflow as tf
tensorflow_version = tf.__version__
gpu_available = tf.test.is_gpu_available()
print("tf version:", tensorflow_version, "\nuse GPU:", gpu_available)
```

如果没有问题，会看到打印结果为：

```Python
tf version: 2.1.0
use GPU True
```

至此， GPU版本也安装成功



# 将搭建好的环境配置到PyCharm

打开 *PyCharm*，选择 *Esisting Interprete*

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/set-tf2.1-env-to-pycharm-01.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">打开PyCharm，选择已有的Interpreter</div>
    <br>
	<br>
</center>


选中 *Conda* 下我们刚创建好的 `tf21` 环境下的 python.exe 文件

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/set-tf2.1-env-to-pycharm-02.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">选中Conda下我们刚创建好的tf21环境下的python.exe文件</div>
    <br>
	<br>
</center>




接着我们在 *PyCharm* 中新建一个Python文件，复制以下Python代码在IDE中执行

```python
import tensorflow as tf
gpu_available = tf.test.is_gpu_available()
print(tf.__version__)
print(gpu_available)
```

如果 *PyCharm* 控制台打印如下内容，则说明配置成功



[此处应有配图]()



# 参考

[Anaconda下安装keras和tensorflow - Atlantis-Brook的个人空间 - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/u/4023145/blog/4496410)

[Tensorflow 2.1 安装（CPU版本＋GPU版本）_小只视觉的博客-CSDN博客](https://blog.csdn.net/qq_46083603/article/details/105365826) 这个好用

 [Ubuntu安装 cuda10 + cudnn7.5 + Tensorflow2.0](https://zhuanlan.zhihu.com/p/65557545)

[TensorFlow2.0教程-Windows10安装tensorflow2.0（GPU、CPU）](https://zhuanlan.zhihu.com/p/62036280)