# ENV

修改pip源：

C盘用户目录下AppData\Roaming目录下创建pip文件夹，新建pip.ini文件，写入（清华源也可以）

```
[global]
index-url = <http://mirrors.aliyun.com/pypi/simple/>
[install]
trusted-host = mirrors.aliyun.com
```

linux

```bash
mkdir ~/.pip
vim .pip/pip.conf

# copy follow contents
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

临时使用：

pip3 install -i [https://pypi.tuna.tsinghua.edu.cn/simple](https://pypi.tuna.tsinghua.edu.cn/simple) numpy

# 模块/包/库

**模块(modules)：**一个`.py`文件就是一个模块，模块名称就是文件名。`.so, .pyc, .pyo, .pyd, .dll` 类型文件也可以作为一个模块。使用模块，可避免命名空间的冲突。

- **模块导入：**
    1. import 模块名1 [as 别名1], 模块名2 [as 别名2]。通过 “模块名1.成员名1“ 方式访问。
    2. from 模块名 import 成员名1 [as 别名1]，成员名2 [as 别名2]。可直接访问模块中成员。
- **模块导入路径：**
    1. 当前的工作目录
    2. PYTHONPATH 中的每一个目录
    3. Python 默认的安装目录
    
    也可以向`sys.path`变量中临时添加模块文件所在的完整路径
    
    ```python
    import sys
    sys.path.append(r'D:\xxx\yyy')
    ```
    
- **`if __name__ == '__main__':`**
    
    一般编写完一个模块，会对模块代码进行测试。对于测试代码希望只在这个py文件运行时执行，其它程序import这个文件时不执行。需要借助python内置变量`__name__`。
    
    1. py文件直接运行时，`__name__`的值为`__main__`
    2. py文件被import到其他程序时，`__name__`的值为这个py文件的名字

**包(package)：**简单理解为带有`__init__.py`的文件夹。是有层次的文件目录结构，定义了多个模块或多个子包。`__init__.py`作用就是将文件夹变为一个python模块，一般是一些初始化的代码。当包被import时，`__init__.py`会自动执行。

- `__all__` 是`__init__`模块中一个变量，用于指定包被 `import *` 时，哪些模块import到当前作用域，不在`__all__` 列表中的模块不会被引用
- `__path__` 也是python中的变量，保存当前包内的搜索路径的一个列表。默认情况下只有一个元素，即当前包（package）的路径
- 包导入
    1. import 包名[.模块名 [as 别名]]
    2. from 包名 import 模块名 [as 别名]
    3. from 包名.模块名 import 成员名 [as 别名]

**from…import * 语句与 import 区别**

**import** 导入模块：每次使用模块中的函数都要是定是哪个模块。

**from…import *** 导入：每次使用模块中的函数，直接使用函数就可以了。因为已经知道该函数是那个模块中的了。

**库(lib)：**python语法中有模块，包的概念，但没有库的概念。库可以是模块也可以是包

# WHEEL包

setup.py内容如下

```python
import setuptools
 
setuptools.setup(
    name="testwheel",
    version="1.0.0",
    author="testuser",
    author_email="testuser@test.com",
    description="test wheel",
    packages=setuptools.find_packages(),
    url="https://github.com/test",
    license="MIT",
    python_requires=">=3.5"

    classifiers = [
        # 发展时期,常见的如下
        #   3 - Alpha
        #   4 - Beta
        #   5 - Production/Stable
        'Development Status :: 3 - Alpha',

        # 开发的目标用户
        'Intended Audience :: Developers',

        # 属于什么类型
        'Topic :: Software Development :: Build Tools',

        # 许可证信息
        'License :: OSI Approved :: MIT License',

        # 目标 Python 版本
        'Programming Language :: Python :: 2',
        'Programming Language :: Python :: 2.7',
        'Programming Language :: Python :: 3',
        'Programming Language :: Python :: 3.3',
        'Programming Language :: Python :: 3.4',
        'Programming Language :: Python :: 3.5',
    ]
)
```

## 生成whl文件

```bash
# 打包为exe
python setup.py bdist_wininst
# 打包为rpm
python setup.py bdist_rpm
# 打成egg包
python setup.py bdist_egg

# universal wheel：在任何OS和平台都支持py2和py3
python setup.py bdist_wheel --universal
# pure-Python wheel：支持py2或者py3，但不能同时支持
python setup.py bdist_wheel
```

## 上传whl文件到PyPI

1. 注册账号
2. 创建`~/.pypirc`文件
    
    ```markdown
    [distutils]
    index-servers = pypi
    
    [pypi]
    repository = https://upload.pypi.org/legacy/
    username = <username>
    password = <password>
    ```
    
3. 信息注册 `python setup.py register`
4. 长传源码包 `python setup.py upload`

也可以使用 twine 工具（专门用于与 pypi 进行交互的工具）上传，`twine upload dist/`

Ref：[http://www.coolpython.net/python_senior/project/op_py_setup_install.html](http://www.coolpython.net/python_senior/project/op_py_setup_install.html)