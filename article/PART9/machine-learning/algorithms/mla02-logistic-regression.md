**逻辑回归（Logistic Regression, LR）** 模型其实仅在线性回归的基础上套用了一个逻辑函数，但就是由于这个逻辑函数，使得逻辑回归模型成为机器学习领域一个重要的模型。



| **算法名称** | 逻辑回归（Logistic Regression, LR）                          |
| :----------: | ------------------------------------------------------------ |
|  **问题域**  | 有监督学习的分类问题                                         |
|   **输入**   | 向量X，向量Y                                                 |
|   **输出**   | 预测模型是否为正类的概览                                     |
|   **优点**   | 线性模型形式简单，可解释性强，容易理解和实现，是计算代价较低的分类模型 |
|   **缺点**   | 分类的效果有时候不好，容易欠拟合                             |
| **应用领域** | 适用于二分领域，或者作为其他算法的“部件”，如作为神经网络算法的激活函数 |



## Logistic回归的数学表达式



Logistic函数的数学表达式如下：


$$
Logistic(z) = \frac{1}{1 + e^{-Z}}
$$


把线性方程带入上式的 $$z$$ ,可以达到下面的表达式：
$$
H(x) = \frac{1}{1 + e^{-(w^{t}x_i+b)}}
$$


## Logistic回归的损失函数


$$
L(x) = -ylogH(x) - (1-y)log(1-H(x))
$$


 分类问题的预测值是离散的，其损失函数确实与回归函数的很不一样。



## 在Python中使用Logistic回归算法



```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris


def logistic_regression1():
    '''
    载入鸢尾花数据集（一个分类问题的经典数据集）
    '''
    iris = load_iris(return_X_y=True)

    x, y = iris

    '''
    训练模型
    
    如果写作clf = LogisticRegression().fit(x, y)时会sklearn报如下的警告
    ----------
    ConvergenceWarning: lbfgs failed to converge (status=1):
    STOP: TOTAL NO. of ITERATIONS REACHED LIMIT.
    ----------
    因为默认运行了 LogisticRegression(... solver='lbfgs', max_iter=100 ...)，
    这是说训练模型的时候，参数的迭代次数达到了限制(默认max_iter=100)，
    但是两次迭代参数变化还是比较大，仍然没有在一个很小的阈值以下，这就叫没有收敛
    ----------
    因为这只是一个警告，我们可以选择：
        1. 忽略
            import warnings
            warnings.filterwarnings("ignore")
        2. 增大最大迭代次数
            clf = LogisticRegression(max_iter=1000)
        3. 更换其他的模型或者参数'solver'
            clf = LogisticRegression(solver="sag")
        4. 将数据进行预处理
            ...
    '''
    clf = LogisticRegression(max_iter=1000).fit(x, y)

    # 打印鸢尾花数据集
    print(iris)

    # 使用模型进行分类预测
    clf.predict(x)

    # 打印性能评估
    print("score: ", clf.score(x, y))


if __name__ == "__main__":
    logistic_regression1()

```

打印出的预测结果和性能评估如下：

```Python
[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 1 1 1
 1 1 1 2 1 1 1 1 1 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 1 2 2 2 2
 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2
 2 2]
score:  0.9733333333333334
```





 [【机器学习】逻辑回归（非常详细） - 知乎](https://zhuanlan.zhihu.com/p/74874291)

 [逻辑回归——理论简介 - 知乎](https://zhuanlan.zhihu.com/p/157519604)

