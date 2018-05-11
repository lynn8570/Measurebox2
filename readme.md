使用Android studio导入opencv几个sample示例代码

- calibration
- colorblob
- facedection

其中facedection用到jni，环境配置需要配置ndk，以及编写cmakelist文件。

## 简单导入

1. ### 下载opencv sdk

   [下载](https://sourceforge.net/projects/opencvlibrary/files/opencv-android/) opencv 的 Android sdk，并解压到电脑某个路径

2. ### 模块导入opencv sdk

   参考 [opencv 开发环境搭建](https://mp.weixin.qq.com/s/sPjs7KC0yOV-bVZTgpeqUA) 将sdk/java中的代码通过 file-new-Import Module导入。

3. ### 添加项目依赖

   在 file-Project Structure中选择应用模块，在dependencies 标签下添加dependence，选择刚才导入的opencv模块openCVLibrary341 

4. ### 添加jniLibs

   如果此时项目不加jniLibs，项目可以编译，但是运行时，回先提示安装open cv manager。就是要另外安装 sdk 目录中apk文件夹下对应的apk，例如 OpenCV_3.4.1_manager_arm64-v8a.apk。

   如果不想要另外安装的，则需要将 sdk-native-libs下的 so 文件复制导入项目的jniLibs中。

   这样编译之后的apk比较大，要好几十M。

   如果是简单的使用opencv 的sdk，例如sample中的color bob示例，这这里的配置就可以了，以下是涉及到ndk编程，项目需要编写cpp文件然后再参与编译，例如示例代码中的facedetection。

## 涉及jni

1. ### 下载ndk

   File-Project structure - sdk Location。 查看Android NDK location，如果为空，点击底部的Download 即可自动下载，并配置好nkd的位置，如 `…../sdk/android-sdk-macosx/ndk-bundle`

   还需要下载cmake：Android studio打开preference对话框，查找 android sdk，在sdk tools标签下勾选下载 cmake。

2. ### c/c++源码文件夹

   在main目录下创建cpp文件夹，用于存放cpp文件。

3. ### 编写CMakeLists.txt

   cmake文件可以参考在Android studio中新建一个c++ support项目而自动生成的cmakelist文件。参考如下：

   ```
   
   cmake_minimum_required(VERSION 3.4.1)
   
   #include open cv sdk中的jni include路径，不然
   include_directories(/Users/zowee-laisc/lynn/opencv/OpenCV-android-sdk/sdk/native/jni/include)
   
   #导入项目jniLibs文件夹下载 so库
   add_library( # Sets the name of the library.
                lib_opencv
   
                # Sets the library as a shared library.
                SHARED
   
                IMPORTED)
   
   set_target_properties(lib_opencv
                         PROPERTIES IMPORTED_LOCATION
                         ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI}/libopencv_java3.so)
   
   #编译添加自己的jni 库
   add_library( # Sets the name of the library.
                detection_based_tracker
   
                # Sets the library as a shared library.
                SHARED
   
                # Provides a relative path to your source file(s).
                src/main/cpp/DetectionBasedTracker_jni.cpp )
   
   # Searches for a specified prebuilt library and stores the path as a
   # variable. Because CMake includes system libraries in the search path by
   # default, you only need to specify the name of the public NDK library
   # you want to add. CMake verifies that the library exists before
   # completing its build.
   
   find_library( # Sets the name of the path variable.
                 log-lib
   
                 # Specifies the name of the NDK library that
                 # you want CMake to locate.
                 log )
   
   # Specifies libraries CMake should link to your target library. You
   # can link multiple libraries, such as libraries you define in this
   # build script, prebuilt third-party libraries, or system libraries.
   
   target_link_libraries( # Specifies the target library.
                          detection_based_tracker
   
                          lib_opencv
   
                          # Links the target library to the log library
                          # included in the NDK.
                          ${log-lib} )
   
   ```

   

4. ### 修改gradle文件

   修改应用gradle文件，主要是

   ```
    externalNativeBuild {
               cmake {
                   cppFlags "-frtti -fexceptions"
                   abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
                   arguments "-DANDROID_STL=gnustl_static"
               }
           }
   ```

   完整文件如下：

   

   ```
   apply plugin: 'com.android.application'
   
   android {
       compileSdkVersion 27
   
   
   
       defaultConfig {
           applicationId "com.zowee.facedection"
           minSdkVersion 21
           targetSdkVersion 27
           versionCode 1
           versionName "1.0"
   
           testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
   
           externalNativeBuild {
               cmake {
                   cppFlags "-frtti -fexceptions"
                   abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
                   arguments "-DANDROID_STL=gnustl_static"
               }
           }
       }
   
       sourceSets {
           main {
               jniLibs.srcDirs = ['src/main/jniLibs']
           }
       }
   
       buildTypes {
           release {
               minifyEnabled false
               proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
           }
       }
   
       externalNativeBuild {
           cmake {
               path "CMakeLists.txt"
           }
       }
   
   }
   
   dependencies {
       implementation fileTree(include: ['*.jar'], dir: 'libs')
       implementation 'com.android.support:appcompat-v7:27.1.1'
       testImplementation 'junit:junit:4.12'
       androidTestImplementation 'com.android.support.test:runner:1.0.2'
       androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
       implementation project(':openCVLibrary341')
   }
   
   ```

   

5. ### 注意事项

   构建项目的时候，遇到的主要报错信息和解决方案

   - 用<>来导入文件出错，该用双引号导入

     ```
     Error:(1, 10) error: 'DetectionBasedTracker_jni.h' file not found with <angled> include; use "quotes" instead
     ```

     

   - 无法找到 opencv2/core.hpp

     ```
     Error:(2, 10) fatal error: 'opencv2/core.hpp' file not found
     Error:(2, 10) fatal error: 'opencv2/core.hpp' file not found
     ```

     在cmakelist文件中添加

     ```
     include_directories(/Users/zowee-laisc/lynn/opencv/OpenCV-android-sdk/sdk/native/jni/include)
     ```

     

   - Linker命令错误

     ```
     /Measurebox2/facedection/src/main/cpp/DetectionBasedTracker_jni.cpp
     Error:(36) undefined reference to `cv::CascadeClassifier::detectMultiSc
     Error:error: linker command failed with exit code 1 (use -v to see invocation)
     
     或者
     Error:(36) undefined reference to `cv::CascadeClassi
     ```

     解决方案

     检查gradle文件

     ```
     cmake {
         cppFlags "-frtti -fexceptions"
         abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
         arguments "-DANDROID_STL=gnustl_static"
     }
     ```

     

   - 删除部分libs文件夹

     ```
     
     Error:Execution failed for task ':facedection:transformNativeLibsWithStripDebugSymbolForDebug'.
     > A problem occurred starting process 'command '/Users/zowee-laisc/lynn/sdk/android-sdk-macosx/ndk-bundle/toolchains/mips64el-linux-android-4.9/prebuilt/darwin-x86_64/bin/mips64el-linux-android-strip''
     ```

     解决方法将jniLibs下的mips64的so 库文件夹删除 

   

   

   



