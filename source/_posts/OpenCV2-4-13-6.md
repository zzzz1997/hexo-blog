---
title: OpenCV2.4.13.6在Android上的采坑（native扩展nonfree）
date: 2019-09-11 10:52:33
cover: http://qiniu.zzzz1997.com/opencv-nonfree/image_result.jpg
categories: 
- 技术
tags:
- OpenCV
- Android
---

最近在做一个小组作业，利用SUFT实现图像拼接。采用的是[OpenCV2.4.13.6](https://opencv.org/releases.html)，而在搭建的时候发现Android pack中并没有`nonfree`文件夹（包括`nonfree/features2d.hpp`头文件），而`libs/**abi/`文件夹中也没有相关的`libnonfree.so`文件，所以并不能使用SUFT。

<!--more-->

所以只好利用NDK自己构建.so文件。参考[前辈的使用方法](https://github.com/bkornel/opencv_android_nonfree)

**使用工具：**

* ndk-bundle工具
* opencv-2.4.13.6-android-sdk
* opencv-2.4.13.6-vc14（注意版本必须和android-sdk相同）

## 开始构建libnonfree.so文件

1.首先解压下载的opencv-2.4.13.6-android-sdk.zip文件，注意文件目录不能含有中文（否则后面.mk可能会编码错乱），如：F:\opencv-2.4.13.6-android-sdk

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/android-sdk.png">

2.安装opencv-2.4.13.6-vc14.exe，得到如下文件夹

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/vc14.png">

3.打开目录`sources/modules/nonfree/include/opencv2`，将其中的`nonfree`文件夹复制到`F:/opencv-2.4.13.6-android-sdk/OpenCV-android-sdk/sdk/native/jni/include/opencv2`文件夹中

4.在任意位置新建一个`libnonfree`文件夹，并在其中新建`jni`文件夹，例如我新建在`F:/opencv-2.4.13.6-android-sdk/OpenCV-android-sdk/sdk/native/jni/include/libnonfree`处

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/jni.png">

再将opencv-2.4.13.6-vc14文件夹`sources/modules/nonfree/src`位置的`nonfree_init.cpp`，`precomp.hpp`，`sift.cpp`和`surf.cpp`四个文件复制到之前新建的`libnonfree/jni`文件夹中。

5.在`libnonfree/jni`文件夹中新建两个文件`Android.mk`，`Application.mk`，此时你的文件目录应该是这样

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/libnonfree.png">

编辑`Android.mk`文件：

```mk
LOCAL_PATH  := $(call my-dir)
OPENCV_PATH := F:\opencv-2.4.13.6-android-sdk\OpenCV-android-sdk\sdk\native\jni

include $(CLEAR_VARS)
OPENCV_INSTALL_MODULES := on
OPENCV_CAMERA_MODULES  := off
include $(OPENCV_PATH)/OpenCV.mk

LOCAL_C_INCLUDES :=             \
    $(LOCAL_PATH)               \
    $(OPENCV_PATH)/include

LOCAL_SRC_FILES :=              \
    nonfree_init.cpp            \
    sift.cpp                    \
    surf.cpp

LOCAL_MODULE := nonfree
LOCAL_CFLAGS := -Werror -O3 -ffast-math
LOCAL_LDLIBS := -llog -ldl

include $(BUILD_SHARED_LIBRARY)
```

编辑`Application.mk`文件：

```mk
APP_STL := gnustl_shared
APP_CPPFLAGS := -std=c++11 -frtti -fexceptions
APP_ABI := armeabi,armeabi-v7a,mips,x86
APP_PLATFORM := android-21
```

6.完成上面的步骤后，就可以使用命令行工具进入`F:/opencv-2.4.13.6-android-sdk/OpenCV-android-sdk/sdk/native/jni/include/libnonfree`文件夹，再使用`ndk-build`命令，如遇到“‘ndk-build’不是内部或外部命令，也不是可运行的程序或批处理文件”，请检查下载ndk-bundle工具并在环境变量中配置好ndk-build

此时执行，会下列一系类报错，例如

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/ndk-build-error.png">

>一次性解决方案：
第一步，打开`precomp.hpp`注释掉第46行`#include "cvconfig.h"`和第66行`#  include "opencv2/ocl/private/util.hpp"`；第二步，打开`nonfree_init.cpp`文件，注释掉第66到第77行，`OPENCV_OCL`部分，完成

再次执行‘ndk-build’，通过！

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/ndk-build-success.png">

此时你的`libnonfree`文件夹里多了两个文件夹`libs`，`obj`，其中`libs`里面的全部.so文件就是我们需要的文件啦！

## 开始使用SUFT

1.在任意位置新建一个`libnonfree_demo`文件夹，并在其中新建`jni`文件夹，例如我建立在`libnonfree`文件夹同级目处

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/libnonfree_demo_src.png">

以armeabi架构为例，将上面的`libnonfree/libs/armeabi`中所有的.so文件拷贝到`jni`文件中

2.在文件夹中新建一个.cpp文件，例如`test.cpp`

```c
#include <jni.h>
#include <android/log.h>

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/nonfree/features2d.hpp>
#include <opencv2/nonfree/nonfree.hpp>
#include <iostream>

using namespace cv;
using namespace std;

#define  LOG_TAG    "nonfree_demo"
#define  LOGI(...)  __android_log_print(ANDROID_LOG_INFO,LOG_TAG,__VA_ARGS__)

int run_demo();

extern "C" {
JNIEXPORT void JNICALL Java_com_zzapp_nonfree_MainActivity_runDemo(JNIEnv * env, jobject obj);
};

JNIEXPORT void JNICALL Java_com_zzapp_nonfree_MainActivity_runDemo(JNIEnv * env, jobject obj)
{
    LOGI( "开始demo\n");
    run_demo();
    LOGI( "结束demo\n");
}

int run_demo()
{
    // 输入输出图片地址
    const char * imgInFile = "/sdcard/nonfree/image.jpg";
    const char * imgOutFile = "/sdcard/nonfree/image_result.jpg";

    Mat image;
    image = imread(imgInFile, CV_LOAD_IMAGE_COLOR);
    if(! image.data )
    {
        LOGI("图片不存在\n");
        return -1;
    }

    vector<KeyPoint> keypoints;
    Mat descriptors;


    // 计算特征描述
    SiftFeatureDetector detector;
    LOGI("检测到%d个特征点\n", (int) keypoints.size());
    detector.detect(image, keypoints);
    LOGI("计算特征描述\n");
    detector.compute(image,keypoints, descriptors);

    // 存储特征描述
    FileStorage fs;
    LOGI("以写入方式打开文件\n");
    fs.open("descriptors.des", FileStorage::WRITE);
    LOGI("写入文件\n");
    fs << "descriptors" << descriptors;
    LOGI("释放文件\n");
    fs.release();

    // 显示图片关键点
    Mat outputImg;
    Scalar keypointColor = Scalar(255, 0, 0);
    LOGI("绘制图片关键点\n");
    drawKeypoints(image, keypoints, outputImg, keypointColor, DrawMatchesFlags::DRAW_RICH_KEYPOINTS);

    LOGI("存储生成图片\n");
    imwrite(imgOutFile, outputImg);

    LOGI("完成\n");
    return 0;
}
```

**注意其中的方法名`Java_com_zzapp_nonfree_MainActivity_runDemo`，你需要将其中的`com_zzapp_nonfree_MainActivity`更改为你的包名加上使用方法的Activity名（包名在后面的使用中应用到，应提前想好）**

3.在`libnonfree_demo/jni`文件夹中新建两个文件`Android.mk`，`Application.mk`，此时你的文件目录应该是这样

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/libnonfree_demo.png">

编辑`Android.mk`文件：

```mk
LOCAL_PATH  := $(call my-dir)
OPENCV_PATH := F:\opencv-2.4.13.6-android-sdk\OpenCV-android-sdk\sdk\native\jni

include $(CLEAR_VARS)
LOCAL_MODULE     := nonfree
LOCAL_SRC_FILES  := libnonfree.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
OPENCV_INSTALL_MODULES := on
OPENCV_CAMERA_MODULES  := off
include $(OPENCV_PATH)/OpenCV.mk

LOCAL_C_INCLUDES :=             \
    $(LOCAL_PATH)               \
    $(OPENCV_PATH)/include

LOCAL_SRC_FILES :=              \
    test.cpp

LOCAL_MODULE := nonfree_demo
LOCAL_CFLAGS := -Werror -O3 -ffast-math
LOCAL_LDLIBS := -llog -ldl
LOCAL_SHARED_LIBRARIES += nonfree

include $(BUILD_SHARED_LIBRARY)
```

编辑`Application.mk`文件：

```mk
APP_STL := gnustl_shared
APP_CPPFLAGS := -std=c++11 -frtti -fexceptions
APP_ABI := armeabi
APP_PLATFORM := android-21
```

4.同上，使用命令行工具进入`F:/opencv-2.4.13.6-android-sdk/OpenCV-android-sdk/sdk/native/jni/include/libnonfree_demo`文件夹，再使用`ndk-build`命令，成功后，在`libnonfree_demo/libs/armeabi`里生成了几个.so文件，包括一个`libnonfree_demo.so`文件

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/armeabi.png">

此时自己的.so文件已生成完毕，即可使用啦！

## Android使用.so文件

1.新建项目，对于我们的上面的.so文件，我们新建一个项目，要和上面`test.cpp`文件方法`Java_com_zzapp_nonfree_MainActivity_runDemo`中一致

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/application.png">

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/activity.png">

2.使用.so有两种方法，在`build.gradle`文件中添加和制作.jar文件添加，这里我更偏向于后者。所以来制作.jar文件。

新建`lib`文件夹（注意是“lib”，而不是“libs”），在其中新建一个文件夹`armeabi`，将`libnonfree_demo/libs/armeabi`中的所有.so文件拷贝到`armeabi`文件夹中。以zip方式压缩`lib`文件夹，并重命名，改为.jar后缀名，例如`so.jar`

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/so.png">

3.将`so.jar`文件拷贝到项目的`libs`下，右键选择`Add As Library...`作为库添加到项目。

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/add_library.png">

4.修改`MainActivity`文件

```kotlin
package com.zzapp.nonfree

import android.support.v7.app.AppCompatActivity
import android.os.Bundle

/**
 * Project Nonfree
 * Date 2018/4/25
 *
 * 主活动
 *
 * @author zzzz
 */
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 使用方法
        runDemo()
    }

    // 库里面的方法
    external fun runDemo()

    companion object {
        init {
            // 加载.so库文件
            System.loadLibrary("nonfree_demo")
        }
    }
}
```

5.构建前的最后一步，添加权限。因为在`test.cpp`中，我们有读写外部文件的操作，所以应在`AndroidManifest.xml`文件中添加两条权限

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

在手机`/sdcard/nonfree`目录下新建文件`image.jpg`，打包运行应用。成功！！

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/image.jpg">

<img class="img-fluid" src="http://qiniu.zzzz1997.com/opencv-nonfree/image_result.jpg">
