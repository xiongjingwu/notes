# numpy

创建ndarray对象

```python
# 通过 list 创建
array2 = np.array([[1, 2, 3], [4, 5, 6]])
# arange 指定范围(0~20)和跨度2
array3 = np.arange(0, 20, 2)
# 生成等差数列，指定范围(-1~1)和个数11
array4 = np.linspace(-1, 1, 11)
# 生成等比数列，2^1~2^10的10个数
array5 = np.logspace(1, 10, num=10, base=2)
#
array6 = np.fromstring('1, 2, 3, 4, 5', sep=',', dtype='i8')
#
def fib(how_many):
    a, b = 0, 1
    for _ in range(how_many):
        a, b = b, a + b
        yield a
gen = fib(20)
array7 = np.fromiter(gen, dtype='i8')
#
#
array8 = np.random.rand(10)
#
array9 = np.random.randint(1, 100, 10)
#
array10 = np.random.normal(50, 10, 20)
#
array11 = np.random.rand(3, 4)
#
array12 = np.random.randint(1, 100, (3, 4, 5))
#
array13 = np.zeros((3, 4))
#
array14 = np.ones((3, 4))
#
array15 = np.full((3, 4), 10)
#
array16 = np.eye(4)
#
array16 = plt.imread('res/guido.jpg')
```

ndarray 的属性

```python
print(array16.size)  # 获取元素个数
print(array16.shape)
print(array16.dtype)
print(array16.ndim)
print(array16.itemsize) # 单个元素占用字节数
print(array16.nbytes)  # 所有元组占用字节数

```

获取统计信息

```python
array1 = np.random.randint(1, 100, 10)
print(array1)
print(array1.sum())
print(np.sum(array1))
print(array1.mean())
print(np.mean(array1))
print(np.median(array1))
print(np.quantile(array1, 0.5))

print(array1.max())
print(np.amax(array1))
print(array1.min())
print(np.amin(array1))
print(array1.ptp())
print(np.ptp(array1))
q1, q3 = np.quantile(array1, [0.25, 0.75])
print(q3 - q1)

print(array1.var())
print(np.var(array1))
print(array1.std())
print(np.std(array1))
print(array1.std() / array1.mean())

# 绘制箱线图
plt.boxplot(array1, showmeans=True)
plt.ylim([-20, 120])
plt.show()
```

```python
# 加载文件
data_bin= np.fromfile('/home/xjw/iter0_128.bin', np.float16).reshape(19, 32000)
data_txt = np.loadtxt('/home/xjw/16_128.txt').astype("float32")

# 保存为二进制
data = np.array([1, 2, 3, 4, 5], dtype=np.float32)
data.tofile('data.bin')

# 保存为txt
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
np.savetxt('data.txt', data, fmt='%d', delimiter=',')

# bin -> txt
import numpy as np
import os

folder_path = '/home/xjw/dev/debug_log/cjc'  # 替换为你的文件夹路径

for root, dirs, files in os.walk(folder_path):
    for file_name in files:
        file_path = os.path.join(root, file_name)
        if (os.path.splitext(file_name)[1] == '.bin'):
            print('open file: ', file_path)
            data = np.fromfile(file_path, dtype=np.float16).astype(np.float16)
            savetxt = os.path.splitext(file_name)[0] + '.txt'
            np.savetxt(savetxt, data, fmt='%.8f')
            print('save file: ', savetxt)

# vacc convert

# compare embedding
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

def gemm_align_z_to_hw(data, shape = [512,256]):
  h,w = shape[0], shape[1]
  data = np.reshape(data, (h//16, w//16, 16, 16))
  data = np.transpose(data, [0,2,1,3])
  data = np.reshape(data, (h, w))
  return data

vacc_res = np.fromfile('/home/xjw/dev/test_gpt/build/embedding/emb_layernorm.bin', np.float16).reshape(-1)
vacc_res = gemm_align_z_to_hw(vacc_res, shape=[128, 5120]).reshape(128, 5120)

torch_res = np.fromfile('/home/xjw/dev/debug_log/13b/iter1_2_emb_layernorm.bin', np.float32).reshape(1, 5120)

print('cos_silm: pytorch_tvm: ', cosine_similarity(vacc_res[:1, :].reshape(1,-1), torch_res[:1, :].reshape(1,-1)))

# for i in range(256):
#     data1 = vacc_res[i*256:i*256+16]
#     data2 = vacc_res2[i*256:i*256+16]
#     print('cos_silm: pytorch_tvm: ', cosine_similarity(data1[:].reshape(1,-1), data2[:].reshape(1,-1)))

# compare logits
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

#GEMM A, align with Z
def restore_align_z_hw(data, shape = [128,768], core_num = 4):
  h,w = shape[0], shape[1]
  if w%(16*core_num):
    raise ValueError("w should be multiple of 16*core_num, but w = ",w)
  if h%16:
    raise ValueError("h should be multiple of 16, but h = ",h)
  data = data.reshape(core_num, h//16, w//16//core_num, 16, 16)
  data = data.transpose(1, 3, 0, 2, 4) # [ h//16, 16, core_num, w//16//core_num, 16]
  data = data.reshape(h,w)
  return data

def get_logits_data(vacc_logits, num_block = 46, height = 16, block_size = 704):
  logit_out = []
  for i in range(0,num_block):
    data_align = vacc_logits[height*block_size*i : height*block_size*(i+1)].reshape(height, -1)
    # print(data_align.shape)
    data = restore_align_z_hw(data_align, shape=[height, data_align.shape[-1]], core_num=4)
    data = data.reshape(height, -1)
    # print(i, ' cos_silm: pytorch_tvm: ', cosine_similarity(logits_torch[0, 832*i : 832*(i+1)].reshape(1,-1), data[0, :].reshape(1,-1)))
    logit_out.append(data)
  logit_out = np.concatenate(tuple(logit_out), axis=1)
  # print(logit_out.shape)
  return logit_out

vacc_logits = np.fromfile('/home/xjw/dev/test_gpt/build/0713/iter_21_logits.bin', np.float16).reshape(-1)
logit_out = get_logits_data(vacc_logits, 46, 16, 704)

logits_torch = np.fromfile('/home/xjw/dev/debug_log/cjc/dfmz/2.bin', np.float16).reshape(1, 32000)

idx = np.argmax(logit_out[:1, :])
print(idx, logit_out[:1, idx:idx+1])
idx2 = np.argmax(logits_torch[:1, :])
print(idx2, logits_torch[:1, idx2:idx2+1])

print('cos_silm: pytorch_tvm: ', cosine_similarity(logits_torch[:1, :].reshape(1,-1), logit_out[:1, :].reshape(1,-1)))

# compare cache_kv
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

def gemm_align_z_to_hw(data, shape = [512,256]):
  h,w = shape[0], shape[1]
  data = np.reshape(data, (h//16, w//16, 16, 16))
  data = np.transpose(data, [0,2,1,3])
  data = np.reshape(data, (h, w))
  return data

input_seq_len = 128
block = 40
logit_iter0_cache_align = np.fromfile('/home/xjw/dev/test_gpt/build/0713/iter_30_cache_kv.bin', np.float16).reshape(-1, 128)
logit_iter0_cache_align = gemm_align_z_to_hw(logit_iter0_cache_align, shape=[block*2*1*40*input_seq_len, 128])
logit_iter0_cache_align = logit_iter0_cache_align.reshape([block, 2, 1, 40, input_seq_len, 128]).astype("float32") 

cache_golden = np.fromfile('/home/xjw/dev/debug_log/cjc/dfmz/cache/21_cache.bin', np.float16).reshape(block, 2, 1, 40, input_seq_len, 128)

for i in range(0, 29):
  for m in range(0, block):
    for n in range(0, 2):
      diff = np.abs(cache_golden[m,n,:,:,i,:].reshape(1,-1)-logit_iter0_cache_align[m,n,:,:,i,:].reshape(1,-1))
      diff_max = np.max(diff)
      print(i, m, n, ' cos_silm: pytorch_tvm: ', cosine_similarity(cache_golden[m,n,:,:,i,:].reshape(1,-1), logit_iter0_cache_align[m,n,:,:,i,:].reshape(1,-1)), diff_max)
```

**ndarray的属性**

- base：如果一个ndarray是通过其他ndarray经过某种操作创建出来的，那么其base就会指向最初的源头
- strides：每一个维度（axis）都有一个strides，表示从数组在某个维度进行遍历的内存偏移量。strides都是相对于base数组而言进行遍历的

任何一个array只要shape和strides信息确定就知道如何访问数据

reshape/transpose 执行很快，因为没有在内存中重新排列数据，只是更改了shape和strides信息。

```python
import numpy as np

a = np.arange(24).reshape([2, 3, 4])
print(a.base)  # ==> None
print(a.shape)  # ==> (2, 3, 4)
print(a.strides)  # ==> (48, 16, 4)

b = a.transpose([1, 2, 0])
print(b.base)   # ==> [ 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23]
print(b.shape)  # ==>(3, 4, 2)
print(b.strides)  # ==> (16, 4, 48)
print(b)
b[0, 0, 0] = 100
print(a)
print(b)
```