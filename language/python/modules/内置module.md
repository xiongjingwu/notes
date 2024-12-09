# Modules

### **argparse**

用于解析命令行参数和选项的标准模块，使用方式：

- 导入argparse模块，创建解析对象`parser = argparse.ArgumentParser()`
- 给parser对象添加命令行参数`parser.add_argument()`
- 调用`parser.parse_args()`进行解析

可以将以上步骤封装在函数中，main函数调用函数得到args对象并使用其中的参数，代码示例：

```python
import argparse

def argument_parser():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "-d",
        "--device_id",
        default=0,
        type=int,
        help="device id to run",
    )
    parser.add_argument(
        "--input_file",
        default="../data/images/word_336.png",
        help="input file",
    )
    parser.add_argument(
		    "-v",
		    "--verbosity",
		    type=int,
		    choices=[0,1,2],   # 设置参数的可选值
		    help="increase output verbosity"
		)
    args = parser.parse_args()
    return args
    
if __name__ == "__main__":
    args = argument_parser()
    print(args.input_file)    
```

```python
# 互斥参数
group=parser.add_mutually_exclusive_group()
group.add_argument("-v","--verbose",action="store_true")
group.add_argument("-q","--quiet",action="store_true")
# -q 和 -v 不出现，或仅出现一个都可以，同时出现就会报错
```

### os

| 方法 | 说明 |
| --- | --- |
| `os.path.abspath(path)` | 返回绝对路径 |
| `os.path.basename(path)` | 返回文件名 |
| `os.path.dirname(path)` | 返回文件路径 |
| `os.path.exists(path)` | 如果路径 path 存在，返回 True；如果路径 path 不存在，返回 False。 |
| `os.path.join(path1[, path2[, …]])` | 把目录和文件名合成一个路径 |
| `os.path.split(path)` | 把路径分割成 dirname 和 basename，返回一个元组 |
| `os.path.walk(path, visit, arg)` | 遍历path，划分为根目录，文件夹名，文件名 |

### opencv

```python
import cv2 as cv

img = cv.imread("data.jpg", cv.IMREAD_UNCHANGED)
#拆分通道
b, g, r = cv.split(img) # split比较耗时
# 等同于
b = img[:, :, 0]
g = img[:, :, 1]
r = img[:, :, 2]

# 合并图像通道
m = cv.merge([r, g, b])

# 分别显示三个通道的图像
cv.imshow("B", b)
cv.imshow("G", g)
cv.imshow("R", r)

# 等待显示
cv.waitKey(0)
cv.destroyAllWindows()

# 图像类型转换
result = cv.cvtColor(img, cv.COLOR_RGB2GRAY)
```