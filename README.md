# jni

## using in Android studio

1.新建一个CMakeLists.txt 

### 注意 一定要加SHARED选项，不然不会生成so动态库

```
add_library(name SHARED cppname.cpp)
```

2.修改build.gradle

defaultConfig中添加

```

 ndk {
      abiFilters "armeabi-v7a"
    }
    
```

android中添加

```
externalNativeBuild {
    cmake {
      path "CMakeLists.txt"
    }
  }
```

整体示例如下：
```
android {
  compileSdkVersion 25
  buildToolsVersion "25.0.2"
  defaultConfig {
    minSdkVersion 21
    targetSdkVersion 25
    ndk {
      abiFilters "armeabi-v7a"
    }
  }
  externalNativeBuild {
    cmake {
      path "CMakeLists.txt"
    }
  }
}
```

3.新建cpp和.h文件
函数名称为包名加类名，例如：

可以用javah命令自动生成
在terminal进入到src/main/java目录，执行``javah org.ros2.examples.android.talker.TestJni``,生成的头文件在当前的src/main/java目录中


.h ``这里cpp编译要加extern "c"`` 2.要包含jni.h头文件

```
#include <jni.h>

extern "C" {
JNIEXPORT jstring JNICALL Java_org_ros2_examples_android_talker_jniUtile_getJniString
        (JNIEnv *, jclass);
}
```
.cpp
```
JNIEXPORT jstring JNICALL Java_org_ros2_examples_android_talker_jniUtile_getJniString
        (JNIEnv * env, jclass zz)
{
    return env-> NewStringUTF("hello jni...");
}

```

4.java文件中加载so库 这个库名称就是cmake中指定的名称

```
 static {
        System.loadLibrary("jiantu");
    }

```
整体效果如下：

```
package org.ros2.examples.android.listener;

public class JniUtils {

    static {
        System.loadLibrary("jiantu");
    }



    public static native String getJniString();
}


```

