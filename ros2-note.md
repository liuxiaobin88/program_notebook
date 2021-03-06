### ndk foxy
```
# define paths
export ANDROID_NDK=/Users/liushichao/Library/Android/sdk/ndk/21.0.6113669
ROOT_DIR=${HOME}/new_ros
AMENT_WORKSPACE=${ROOT_DIR}/ament_ws
ROS2_ANDROID_WORKSPACE=${ROOT_DIR}/ros2_ws
# android build configuration
export PYTHON3_EXEC="$( which python3 )"
export ANDROID_ABI=armeabi-v7a
export ANDROID_NATIVE_API_LEVEL=android-28
export ANDROID_TOOLCHAIN_NAME=arm-linux-androideabi-clang
cd ${ROS2_ANDROID_WORKSPACE}
source ${AMENT_WORKSPACE}/install/local_setup.sh
ament build --isolated \
  --cmake-args \
  -DBUILD_TESTING=OFF \
  -DPYTHON_EXECUTABLE=${PYTHON3_EXEC} \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_FUNCTION_LEVEL_LINKING=OFF \
  -DANDROID_NATIVE_API_LEVEL=${ANDROID_NATIVE_API_LEVEL} \
  -DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME} \
  -DANDROID_STL=c++_shared \
  -DCMAKE_CXX_STANDARD=17 \
  -DANDROID_ABI=${ANDROID_ABI} \
  -DANDROID_NDK=${ANDROID_NDK} \
  -DENABLE_SSL=NO \
  -DProtobuf_DIR=/Users/liushichao/grpc/install/lib/cmake/protobuf \
  -DgRPC_DIR=/Users/liushichao/grpc/install/lib/cmake/grpc \
  -DTHIRDPARTY=ON \
  -DCOMPILE_EXAMPLES=OFF \
  -DCMAKE_FIND_ROOT_PATH="$AMENT_WORKSPACE/install;$ROS2_ANDROID_WORKSPACE/install_isolated"
```

## 笔记
### rmw全称
出处：https://index.ros.org/doc/ros2/Concepts/DDS-and-ROS-middleware-implementations/
```
“ROS Middleware interface” (a.k.a. rmw interface or just rmw)
```

## 错误记录

### Qt5

```
export Qt5_DIR=/usr/local/opt/qt5
```


### Could NOT find OpenSSL

  colcon build 的时候，有时候会提示这个错误
  ```
   Could NOT find OpenSSL, try to set the path to OpenSSL root folder in the
  system variable OPENSSL_ROOT_DIR (missing: OPENSSL_INCLUDE_DIR)
  ```
原因：

没有找到OpenSSL

解决方法：

将OpenSSL的路径添加到环境变量``OPENSSL_ROOT_DIR``中，如果没装过OpenSSL就安装一下

```
export OPENSSL_ROOT_DIR=/usr/local/Cellar/openssl@1.1/1.1.1g/
```

如果继续提示出错
```
cannot link directly with dylib/framework, your binary is not an allowed client of /usr/lib/libcrypto.dylib for architecture x86_64
```

则清空所编译包的build 文件和install文件，再次尝试，原因是cmake找到一次错误的位置就会一直用这个位置，出处https://stackoverflow.com/questions/58446253/xcode-11-ld-error-your-binary-is-not-an-allowed-client-of-usr-lib-libcrypto-dy


### link error

  出现以下类似的提示，可以考虑是不是某个新添加的cpp，没有添加到cmakelist.txt的 add_library中
```
/build_isolated/my_package'
Scanning dependencies of target my_package
[  8%] Building CXX object CMakeFiles/my_package.dir/src/node/frame_update_node/frame_update_node.cpp.o
[ 16%] Linking CXX shared library libmy_package.so
/Users/liushichao/workspace/ros2/ros2_android/ros2_android_ws/src/ros2_example/my_package/src/manager/node_manager/node_manager.cpp:31: error: undefined reference to 'sgmm::sensor_update::SensorUpdateNode::publish_gps(double, double, double, double, double)'
/Users/liushichao/Library/Android/sdk/ndk/21.0.6113669/toolchains/llvm/prebuilt/darwin-x86_64/sysroot/usr/include/c++/v1/memory:2155: error: undefined reference to 'sgmm::sensor_update::SensorUpdateNode::SensorUpdateNode(std::__ndk1::basic_string<char, std::__ndk1::char_traits<char>, std::__ndk1::allocator<char> > const&, std::__ndk1::basic_string<char, std::__ndk1::char_traits<char>, std::__ndk1::allocator<char> > const&)'
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
make[2]: *** [libmy_package.so] Error 1
make[1]: *** [CMakeFiles/my_package.dir/all] Error 2
make: *** [all] Error 2
```

添加sensor_update_node.cpp后 如下

```
add_library(${PROJECT_NAME} SHARED 
            src/my_package.cpp 
            src/node/frame_update_node/frame_update_node.cpp
            src/node/object_detector_node/object_detector_node.cpp
            src/object_detector/object_detector.cpp
            src/manager/assets_manager/assets_manager.cpp
            src/manager/node_manager/node_manager.cpp
            src/util/file_util/file_util.cpp
            src/instantiation/instantiation.cpp
            src/instantiation/hungarian.cpp
            src/instantiation/kalman_tracker.cpp
            src/instantiation/sort.cpp
            src/node/sensor_update_node/sensor_update_node.cpp)
```



### Ros2创建msg依赖其他msg

0.创建xxx.msg文件，这里要注意，必须大写字母开头

1.在xxx.msg中添加要依赖的msg，例如：
```
#msg文件中可以用#作为注释
std_msgs/Header header # Header timestamp should be acquisition time of image
                             # Header frame_id should be optical frame of camera
sensor_msgs/CameraInfo camera_info
int64 frame_addr
```

2.在CMakeList.txt文件中，加入下边的代码，主要是DEPENDENCIES那一项，如果不指定会编译不通过

```
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)


rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/DetectedObject.msg"
  "msg/DetectedObjects.msg"
  "msg/InstantiatedObject.msg"
  "msg/InstantiationLabel.msg"
  "msg/UpdateFrame.msg"
  "msg/Gps.msg"
  DEPENDENCIES std_msgs sensor_msgs    ##这行很重要
)
```

3.在package.xml中添加下边的内容

```
  <build_depend>rosidl_default_generators</build_depend>
  <exec_depend>rosidl_default_runtime</exec_depend>
  <build_depend>std_msgs</build_depend>
  <exec_depend>std_msgs</exec_depend>
  <build_depend>sensor_msgs</build_depend>
  <exec_depend>sensor_msgs</exec_depend>
```

### ros2-java的版本

ros2-java中用的ros2应该是crystal版本的，最新的是foxy，ros2版本列表如下

```
http://docs.ros2.org/eloquent
http://docs.ros2.org/dashing
http://docs.ros2.org/crystal
http://docs.ros2.org/bouncy
http://docs.ros2.org/ardent
http://docs.ros2.org/beta3
http://docs.ros2.org/beta2
http://docs.ros2.org/beta1
http://docs.ros2.org/alpha8

```
android版的ros2可以调试运行了

```
build.gradle文件中，加入

android{

  //...

  ndk{
    abiFilters "armeabi-v7a"
  }
  
  //...

}

在运行按钮左边选择ros2_talker_android，点debug按钮就可以了

```
