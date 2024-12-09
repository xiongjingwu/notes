# Snippets

```cpp
// chrono 库
auto start = std::chrono::high_resolution_clock::now();
func();
auto end = std::chrono::high_resolution_clock::now();
long time = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();

// 进程线程
#include <unistd.h>
std::cout << "pid = " << getpid() << std::endl;
#include <thread>
std::cout << "tid=" << std::this_thread::get_id() << std::endl;
std::this_thread::sleep_for(std::chrono::milliseconds(x));

// std::ios::beg, std::ios::cur, std::ios::end
void ReadFile(const std::string& path, void *data) {
  std::ifstream ifs(path, std::ios::binary | std::ios::ate); // std::ios::in
  int file_size = ifs.tellg();
  std::cout << "File size: " << file_size << std::endl;
  ifs.seekg(0, std::ios::beg);
  ifs.read(reinterpret_cast<char*>(data), file_size);
  ifs.close();
  return;
}
void WriteFile(const std::string& path, const void *data, size_t size) {
  std::ofstream ofs(path, std::ios::binary | std::ios::out);
  ofs.write(reinterpret_cast<const char*>(data), size);
  ofs.close();

  // std::ofstream ofs("out.txt");
  return;
}

// 取偶
int width_pitch = width & ~0x1;

#define ALIGN_UP(num, align) (((num) + (align)-1) & ~((align)-1))

#define ALIGN_TO_32(v) ((v + (32 - 1)) & (~(32 - 1)))

inline uint32_t Align(uint32_t size, uint32_t align) {
  return ((size + align - 1) & (~(align - 1)));
}

// dynamic batch
std::vector<int> batches{8, 4, 2, 1};
size_t size = batches.size();
int n = 100; // total
int done = 0;
for (size_t i = 0; i < size; i++) {
  int batch = batches[i]; // current batch
  if ((n - done) >= batch) {
    int batch_cnt = (n - done) / batch;
    for (int j = 0; j < batch_cnt ; j++) {
      infer(batch);
      done += batch;
    }
  }
}

// fp32 --> fp16
inline uint32_t as_uint(const float x) { return *(uint32_t *)&x; }

// IEEE-754 16-bit floating-point format (without infinity): 1-5-10, exp-15,
// +-131008.0, +-6.1035156E-5, +-5.9604645E-8, 3.311 digits
static uint16_t float_to_half(const float x) {
  // round-to-nearest-even: add last bit after truncated mantissa
  const uint32_t b = as_uint(x) + 0x00001000;
  // exponent
  const uint32_t e = (b & 0x7F800000) >> 23;
  // mantissa; in line below: 0x007FF000 = 0x00800000-0x00001000 = decimal
  // indicator flag - initial rounding
  const uint32_t m = b & 0x007FFFFF;
  // sign : normalized : denormalized : saturate
  return (b & 0x80000000) >> 16 |
         (e > 112) * ((((e - 112) << 10) & 0x7C00) | m >> 13) |
         ((e < 113) & (e > 101)) *
             ((((0x007FF000 + m) >> (125 - e)) + 1) >> 1) |
         (e > 143) * 0x7FFF;
}

inline uint32_t AsUint(const float x) { return *(uint32_t *)&x; };

inline float AsFloat(const uint32_t x) { return *(float *)&x; };

// IEEE-754 16-bit floating-point format (without infinity): 1-5-10,
// exp-15, +-131008.0, +-6.1035156E-5, +-5.9604645E-8, 3.311 digits
static float half_to_float(const uint16_t x) {
  const uint32_t e = (x & 0x7C00) >> 10;  // exponent
  const uint32_t m = (x & 0x03FF) << 13;  // mantissa
  const uint32_t v =
      AsUint((float)m) >>
      23;  // evil log2 bit hack to count leading zeros in denormalized format
  return AsFloat(
      (x & 0x8000) << 16 | (e != 0) * ((e + 112) << 23 | m) |
      ((e == 0) & (m != 0)) *
          ((v - 37) << 23 | ((m << (150 - v)) &
                             0x007FE000)));  // sign : normalized : denormalized
}
```

### Read&Write File

[https://blog.csdn.net/kingstar158/article/details/6859379](https://blog.csdn.net/kingstar158/article/details/6859379)

```cpp
// std::ios::beg, std::ios::cur, std::ios::end
void ReadFile(const std::string& path, void *data) {
  std::ifstream ifs(path, std::ios::binary | std::ios::ate); // std::ios::in
  int file_size = ifs.tellg();
  std::cout << "File size: " << file_size << std::endl;
  ifs.seekg(0, std::ios::beg);
  ifs.read(reinterpret_cast<char*>(data), file_size);
  ifs.close();
  return;
}
void WriteFile(const std::string& path, const void *data, size_t size) {
  std::ofstream ofs(path, std::ios::binary | std::ios::out);
  ofs.write(reinterpret_cast<const char*>(data), size);
  ofs.close();

  // std::ofstream ofs("out.txt");
  return;
}

// txt
int x,sum=0;
ifstream srcFile("in.txt", ios::in); //以文本模式打开in.txt备读
if (!srcFile) { //打开失败
    cout << "error opening source file." << endl;
    return 0;
}
ofstream destFile("out.txt", ios::out); //以文本模式打开out.txt备写
//可以像用cin那样用ifstream对象
while (srcFile >> x) {
    sum += x;
    //可以像 cout 那样使用 ofstream 对象
    destFile << x << " ";
}
cout << "sum：" << sum << endl;
destFile.close();
srcFile.close();
```

调试日志

```c
#ifdef DEBUG_LOGGING
#define RT_LOG(message, ...) \
  printf("[RT LOG] %s:%d - " message "\n", __FILE__, __LINE__, ##__VA_ARGS__)
#else
#define RT_LOG(message, ...)
#define LOG(INFO)
#endif
```

```cmake
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  target_compile_options(rt PRIVATE -DDEBUG_LOGGING)
endif()
```