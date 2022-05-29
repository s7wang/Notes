# Android NDK 开发

## Android NDK 简介

NDK 全称 Native Development Kit 本地语言（C/C++）开发包。而对应的Android开发中使用的是 Android-SDK，（Software Development Kit）软件开发包（只支持Java语言开发）。我们可以利用 NDK 将 C/C++ 开发的代码编译成 so 库，让 Java 程序调用。Java 使用的是 JNI（Java本地接口） 来调用。

NDK 开发优点：

* 利用 NDK 开发的库，不容易被反编译，保密性，安全性都提高了；
* 很多开源工程和大工程都是 C/C++ 代码写的；
* C/C++ 的代码运行速度和效率都比 Java 快很多。

Android NDK 开发步骤：

（1）JNI 接口设计；
（2）使用 C/C++ 本地实现方法；
（3）生成动态链接库；
（4）将动态链接库复制到Java工程，运行Java程序。


## JNI 接口设计与实现

在 Java 中有两种数据类型，一种是基本类型（primitive types），如，int，float，char；另一种是引用类型（reference types），如，类，实例，数组。【数组不管是对象数组还是基本类型数组，都作为reference types存在，有专门的JNI方法取数组中的每个元素】

基本数据类型表

|         Java类型        | JNI类型  |        C/C++类型          |
|------------------------|----------|--------------------------|
| boolean                | jboolean | unsigned char/uint8\_t   |
| byte                   | jbyte    | signed/int8\_t           |
| char                   | jchar    | unsigned short/unit16\_t |
| short                  | jshort   | short/int16\_t           |
| int                    | jint     | int/int32\_t             |
| long                   | jlong    | long long/int64\_t       |
| float                  | jfloat   | float                    |
| double                 | jdouble  | double                   |

JNI的数组类型在类型后加 `Array` 即可。


对象类型表
| Java类型| JNI类型 | 描述                                |
|--------|---------|------------------------------------|
| Object | jobject | 任意Java对象，或者没有对应java类型的对象 |
| Class  | jclass  | Class对象                           |
| String | jstring | String对象                          |


相比基本类型，对象类型的传递要复杂得得多。不能对 jstring 进行直接操作。

JNI String 相关函数

| 函数名  | 描述  |
|---|---|
| const jchar *GetStringChars(JNIEnv *env, jstring str, jboolean *isCopy); | 得到Unicode编码的String指针，返回值为string的copy值 |
| void ReleaseStringChars(JNIEnv *env, jstring str, const jchar *chars); | 释放  |
| const jbyte *GetStringUTFChars(JNIEnv *env, jstring str, jboolean *isCopy); | 得到UTF-8编码的String指针，返回值为string的copy值 |
| void ReleaseStringUTFChars(JNIEnv *env, jstring str, const jchar *utf); | 释放 |
| jsuze GetStringLength(JNIEnv *env, jstring str); | 得到Unicode编码的String长度  |
| jsuze GetStringUTFLength(JNIEnv *env, jstring str); | 得到UTF-8编码的String长度  |
| jstring NewString（JNIEnv *env, jchar *uchars, jsize len）; | 创建一个java.lang.String实例，长度参数中给出的Unicode编码格式的String相同 |
| jstring NewStringUTF（JNIEnv *env, const char *bytes, jsize len）; | 创建一个java.lang.String实例，长度参数中给出的UTF-8编码格式的String相同 |
| void GetStringRegion(JNIEnv *env, jstring str, jsize start， jsize len， jchar *buf);(*env)->SetStringRegion | 把String的内容复制出来，复制给Unicode编码格式的参数buf |
| void GetStringUTFRegion(JNIEnv *env, jstring str, jsize start， jsize len， jchar *buf);(*env)->SetStringUTFRegion | 把String的内容复制出来，复制给UTF-8编码格式的参数buf |


下面为 JNI 的接口设计，一般来说使用 JNI 的情况都会把调用 C/C++ 的部分封装到一个类中集中提供统一调用。下面看第一种方法，不使用注册功能，直接调用。

在cpp文件中

```c++
jtype Java_packagename_classname_functionname(JNIEnv *env, jobject thiz, ...);
```
这里jtype表示函数返回值，packagename表示包名，functionname表示函数名，点用下划线代替。这是很严格的命名规范必须这样命名。

在对应的class.java文件中，需要这样调用

```java
public class ClassName {
    ...
    // 声明方法
    public native type functionname(...);
    // 加载编译好的库
    static {
        System.loadLibrary("targetname");
    }
    ...
}
```

这种方法还有一种快生成cpp的头文件的方法，首先编写好需要的方法的类声明好方法，然后在将java文件复制到工程目录中的bin下面，然后用命令 `javac classname.java` 会生成一个 class 文件，把这个class文件复制到对应的class目录下覆盖原来的文件，然后在命令进入bin 输入 `javah -jni packagename.classname` 此时在当前目录下就会产生一个头文件，文件中将会把所有你定义过的方法用 JNI 的格式在头文件中定义好，只需要把这个头文件copy出来放到对应文件夹中并用c文件实现即可。


这种方法是最直观也是最简单的，但是可维护性差。一种更好的方式是采用 JNINativeMethod 对函数进行注册。首先先来看一下 JNINativeMethod 结构图的官方定义。

```c++
typedef struct {
    const char *name; //Java中函数的名字
    const char *signature; //描述Java函数的参数和返回值
    void *fnPtr; //函数指针，指向native函数，前面都要接 (void *)
} JNINativeMethod;
```

示例：

* jni.c
```c++
//jni.c
#include "jniUtils.h"

static const char *const kClassPathName = "packagename/classname";

static JNINativeMethod gMethods[] = {
    {"NativeFileOpen",  "(Ljava/lang/String;I)I",   (void*)java_packagename_class_NativeFileOpen},
    {"NativeFileRead",  "(I[BI)I",                  (void*)java_packagename_class_NativeFileRead},
    {"NativeFileWrite", "(I[BI)I",                  (void*)java_packagename_class_NativeFileWrite},
}

int register_packagename_class(JNIEnv *env) {
    return jniRegisterNativeMethods(env, kClassPathName, gMethods, sizeof(gMethods) / sizeof(gMethods[0]));
}
```

* jniUtils.h 
```c++
// jniUtils.h 
#ifndef _JNI_UTILS_H_
#define _JNI_UTILS_H_

#include <stdlib.h>
#include <jni.h>

#ifdef __cplusplus
extern "C"
{
#endif

int jniThrowException(JNIEnv *env, const char *className, const char *msg);

JNIEnv *getJNIEnv();

int jniRegisterNativeMethods(JNIEnv *env, 
                             const char *className
                             const JNINativeMethod *gMethods
                             int numMethods);
                             
#ifdef __cplusplus
}
#endif

#endif /* _JNI_UTILS_H_ */
```

* onLoad.cpp
```c++
//onLoad.cpp
#define TAG "class_onLoad"  
  
#include <android/log.h>  
#include "jniUtils.h"  
  
extern "C" {  
  
extern int register_packagename_class(JNIEnv *env);  
  
}  
  
static JavaVM *sVm;  
  
/* 
 * Throw an exception with the specified class and an optional message. 
 */  
int jniThrowException(JNIEnv* env, const char* className, const char* msg) {  
    jclass exceptionClass = env->FindClass(className);  
    if (exceptionClass == NULL) {  
        __android_log_print(ANDROID_LOG_ERROR,  
                TAG,  
                "Unable to find exception class %s",  
                        className);  
        return -1;  
    }  
  
    if (env->ThrowNew(exceptionClass, msg) != JNI_OK) {  
        __android_log_print(ANDROID_LOG_ERROR,  
                TAG,  
                "Failed throwing '%s' '%s'",  
                className, msg);  
    }  
    return 0;  
}  
  
JNIEnv* getJNIEnv() {  
    JNIEnv* env = NULL;  
    if (sVm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {  
        __android_log_print(ANDROID_LOG_ERROR,  
                            TAG,  
                            "Failed to obtain JNIEnv");  
        return NULL;  
    }  
    return env;  
}  
  
/* 
 * Register native JNI-callable methods. 
 * 
 * "className" looks like "java/lang/String". 
 */  
int jniRegisterNativeMethods(JNIEnv* env,  
                             const char* className,  
                             const JNINativeMethod* gMethods,  
                             int numMethods)  
{  
    jclass clazz;  
  
    __android_log_print(ANDROID_LOG_INFO, TAG, "Registering %s natives\n", className);  
    clazz = env->FindClass(className);  
    if (clazz == NULL) {  
        __android_log_print(ANDROID_LOG_ERROR, TAG, "Native registration unable to find class '%s'\n", className);  
        return -1;  
    }  
    if (env->RegisterNatives(clazz, gMethods, numMethods) < 0) {  
        __android_log_print(ANDROID_LOG_ERROR, TAG, "RegisterNatives failed for '%s'\n", className);  
        return -1;  
    }  
    return 0;  
}  
//Dalvik虚拟机加载C库时，第一件事是调用JNI_OnLoad()函数  
jint JNI_OnLoad(JavaVM* vm, void* reserved) {  
    JNIEnv* env = NULL;  
    jint result = JNI_ERR;  
    sVm = vm;  
  
    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {  
        __android_log_print(ANDROID_LOG_ERROR, TAG, "GetEnv failed!");  
        return result;  
    }  
  
    __android_log_print(ANDROID_LOG_INFO, TAG, "loading . . .");  
  
  
    if(register_packagename_class(env) != JNI_OK) {  
        __android_log_print(ANDROID_LOG_ERROR, TAG, "can't load register_com_conowen_fs_FsActivity");  
        goto end;  
    }  
  
    __android_log_print(ANDROID_LOG_INFO, TAG, "loaded");  
  
    result = JNI_VERSION_1_4;  
  
end:  
    return result;  
}  
```

`const char *signature` 与Java对象类型的关系：

|   Field Descriptor     |   Java Language Type   |
|------------------------|------------------------|
| Z                      | boolean                |
| B                      | byte                   |
| C                      | char                   |
| S                      | short                  |
| I                      | int                    |
| J                      | long                   |
| F                      | float                  |
| D                      | double                 |
| "Ljava/lang/String;"   | String                 |
| "[I"                   | int[]                  |
| "[Ljava/lang/Object;"  | Object[]               |
| "()Ljava/lang/String;" | String f()             |
| "(ILjava/lang/Class;)J"| long f(int i, Class c);|
| "([B)V"                | String(byte[] bytes);  |

Android.mk文件
```makefile
# 表示此时正在位于工程目录的根目录中，（call my-dir）
# 的功能由编译器提供，被用来返回当前目录的地址
LOCAL_PATH := $(call my-dir)  

# CLEAR_VARS 这个变量有编译系统提供，
# 并指明了一个 GNU makefile 文件，
# 这个功能回清理掉所有以LOCAL_开头的内容，
# 除了LOCAL_PATH这句话是必须的，
# 因为如果所有的变量都是全局变量，
# 所有可控的编译文件都需呀在一个单独的 GNU 中被解析执行。
include $(CLEAR_VARS)  

# LOCAL_MODULE必须被定义 ，用来区分Android.mk中的每一个模块  
LOCAL_MODULE    := class 

# 该变量必须包含一个 c 或 c++ 源文件的列表
# 这些列表会被编译并聚合到一个模块中
LOCAL_SRC_FILES := class.c jni.c onLoad.cpp  

# 链接参数
LOCAL_LDLIBS  += -llog  

# 哟欧系统提供，并且指定给 GNU makefile
# 收集所有include $(CLEAR_VARS) 中定义的变量，
# 并决定哪些应该被编译
include $(BUILD_SHARED_LIBRARY)  
```

其中 class.c 是真正的基础实现的c文件。除了使用.mk文件来构建，还可以用CMakeLists.txt来构建。

## 编译
一般来说，可以直接使用IDE在项目中直接构建编译，但是也可以使用本编译工具来实现，比如使用Cyin进入工程目录直接执行`cmake`，或者新建一个Application.mk，然后执行 `make App=$appmodulename`。









