# Keras安装（基于Win10）



安装Keras前，请先基于下面这篇文章，安装好TensorFlow

<a style="", href="../../PART9/tensorflow/tensorflow-01-env-install-set-build.html">TensorFlow2.1安装</a>

## 安装 Keras

输入命令开始安装keras：

`pip install keras==2.3.1 -i https://pypi.doubanio.com/simple`



接着我们在 *PyCharm* 中新建一个Python文件，复制以下Python代码在IDE中执行

```python
import tensorflow as tf
import keras as keras
gpu_available = tf.test.is_gpu_available()
print(tf.__version__)
print(keras.__version__)
print(gpu_available)
```

如果 *PyCharm* 控制台打印如下内容，则说明配置成功




# 参考
- [2020-08 亲测最方便详细可用！Win10下Anaconda（python3.8）+Tensorflow2.1.0-gpu版本+spyder安装教程](https://blog.csdn.net/zazazaz1/article/details/108064895)
- [python3.8下使用tensorflow2.0版本](https://jingyan.baidu.com/article/bea41d435e3e9af5c51be6e1.html)
- [更改Python的pip install 默认安装依赖路径方法详解](https://www.jb51.net/article/149625.htm)
- [pip install 默认安装路径修改](https://www.cnblogs.com/maggieq8324/p/12099068.html)

- [Anaconda下安装keras2.3.1和tensorflow2.1](https://my.oschina.net/u/4023145/blog/4496410)  这个好用





