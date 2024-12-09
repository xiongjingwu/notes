# Compiler

静态编译：程序运行前编译为机器码，称为AOT（Ahead of time），提前编译

动态解释：程序一边解释一边运行，称为JIT（Just in time），即时编译

LLVM IR practice

[https://www.zhihu.com/column/c_1458687300104830976](https://www.zhihu.com/column/c_1458687300104830976)

[ELF](Compiler%2093910dde16c943738a45beb07530727d/ELF%2081a03b978f1445368ae2b14913205931.md)

linux下默认导出所有符号，隐藏符号方式：[https://www.cnblogs.com/zzqcn/p/3640353.html](https://www.cnblogs.com/zzqcn/p/3640353.html)

例子：

```cpp
// clang-format off

// Definitions for shared library support
#if defined _WIN32
#define VACM_DLL_IMPORT __declspec(dllimport)
#define VACM_DLL_EXPORT __declspec(dllexport)
#else
#if __GNUC__ >= 4
#define VACM_DLL_IMPORT __attribute__((visibility("default")))
#define VACM_DLL_EXPORT __attribute__((visibility("default")))
#else
#define VACM_DLL_IMPORT
#define VACM_DLL_EXPORT
#endif
#endif

#define VACM_API        VACM_DLL_EXPORT
#define VACM_API_IMPORT VACM_DLL_IMPORT

VACM_API vacmErr vacmGetVersion(const char** ver, uint32_t* major, uint32_t* minor, uint32_t* rev, uint32_t* bn);
```

[lc](Compiler%2093910dde16c943738a45beb07530727d/lc%2011a4b39f25d580118b15d844ca06d8da.md)