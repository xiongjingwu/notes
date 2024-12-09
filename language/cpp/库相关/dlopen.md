# dlopen

## dlopen系列接口

```c
/**
 * \brief 在进程运行时，打开so，将其加载到进程的地址空间，并完成初始化过程
 * \param filename[in]: 动态库路径，可以是绝对路径或相对路径
 * \param flag[in]: 打开方式
 * \return 成功返回指向库的句柄的指针，该句柄可以用于后续的函数调用和符号查找；失败返回NULL，可以通过dlerror函数获取失败信息
 */
void *dlopen(const char *filename, int flag);

/**
 * \brief 将动态库从进程的地址空间中移除，释放相关资源，包括符号表、全局变量、代码和数据段等
 * \param handle[in]: 指向已经打开的共享库句柄的指针
 * \return 成功返回0，失败返回非0。可以通过dlerror函数获取失败信息
 */
int dlclose(void *handle);

/**
 * \brief 运行时动态获取共享库中的符号（函数或变量）地址
 * \param handle[in]: 指向已经打开的共享库句柄的指针。handle为NULL，会在所有已经打开的共享库中搜索符号
 * \param symbol[in]: 需要获取的符号的名称，可以是函数名或变量名
 * \return 成功指向符号地址的指针；失败返回NULL，可以通过调用dlerror函数来获取错误信息
 */
void *dlsym(void *handle, const char *symbol);

/**
 * \brief 运行时动态获取共享库中的符号（函数或变量）地址
 * \param handle[in]: 指向已经打开的共享库句柄的指针。handle为NULL，会在所有已经打开的共享库中搜索符号
 * \param symbol[in]: 需要获取的符号的名称，可以是函数名或变量名
 * \param version[in]: 指定需要获取的符号的版本号，如果不需要指定版本号，则可以将该参数设为NULL。如果指定了版本号，则只会返回与该版本号匹配的符号地址，如果没有找到匹配的符号，则会返回NULL
 * \return 成功指向符号地址的指针；失败返回NULL，可以通过调用dlerror函数来获取错误信息
 */
void *dlvsym(void *handle, const char *symbol, const char *version);

/**
 * \brief 获取动态链接库加载错误信息的函数
 * \return 错误信息的C字符串的指针；如果没有错误发生返回NULL
 */
char *dlerror();
```

dlopen不同flag解释：

- **`RTLD_LAZY`**: 共享库中的符号并不会立即被解析，只有在程序调用共享库中的函数或访问共享库中的变量时，才会进行符号解析。
- **`RTLD_NOW`**: 立即解析所有符号，这样可以确保在dlopen函数返回之后，程序可以立即调用共享库中的函数或访问共享库中的变量。但是，这种方式可能会增加共享库加载的时间和内存占用量。
- **`RTLD_GLOBAL`**: 符号全局可见，即共享库中定义的符号可以被其他共享库和主程序使用。如果没有设置该标志，则共享库中定义的符号只能在该共享库内部使用。
- **`RTLD_LOCAL`**: 符号局部可见，即共享库中定义的符号只能在该共享库内部使用。如果没有设置该标志，则共享库中定义的符号可以被其他共享库和主程序使用。
- **`RTLD_NODELETE`**: 程序退出时，不自动卸载共享库，而是将共享库一直保留在内存中。这种方式可能会导致内存泄漏，因此需要谨慎使用。
- **`RTLD_NOLOAD`**: 不加载共享库文件，仅检查共享库文件是否已经加载到内存中。如果共享库已经加载，则dlopen函数会返回该共享库的句柄，否则会返回NULL。

dlopen 查找顺序：

- 如果flags包含RTLD_GLOBAL，按以下顺序查找：
    1. LD_LIBRARY_PATH环境变量指定的路径
    2. 使用/etc/ld.so.cache文件中缓存的路径列表。ldconfig命令会更新该文件
    3. 使用默认的系统路径，/lib和/usr/lib。
- 如果flags不包含RTLD_GLOBAL，按以下顺序查找：
    1. LD_LIBRARY_PATH环境变量指定的路径
    2. 使用dlopen函数调用时指定的路径
    3. 使用默认的系统路径，/lib和/usr/lib。

dlsym和dlvsym：

- 如果符号是函数和变量，返回符号和变量的地址；如果符号是常量，就返回常量的值
- 如果共享库中存在多个版本的符号，且需要动态获取特定版本的符号地址，就需要使用dlvsym函数。如果只需要获取默认版本的符号地址，则可以使用dlsym函数

dlclose：

- 如果多次打开同一个共享库，必须调用多次dlclose函数才能完全关闭该共享库

示例：

```c
#include <stdio.h>
#include <dlfcn.h>

int main() {
    void *handle;

    // 打开动态库
    handle = dlopen("libm.so", RTLD_LAZY);
    if (!handle) {
        printf("Failed to open dynamic library: %s\n", dlerror());
        return 1;
    }

    // 获取动态库中的cos函数符号
    // double (*cosine)(double);
    typedef double(*cosine)(double);
    cosine get_cos = (cosine)dlsym(handle, "cos");

    // 检查是否出错
    char *error;
    error = dlerror();
    if (error != NULL) {
        printf("Failed to find symbol: %s\n", error);
        return 1;
    }

    // 使用cos函数计算cos(0.5)
    double result = get_cos(0.5);
    printf("cos(0.5) = %f\n", result);

    // 关闭动态库句柄
    dlclose(handle);

    return 0;
}
```