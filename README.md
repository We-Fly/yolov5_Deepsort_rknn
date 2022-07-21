# Yolov5_DeepSORT_rknn

[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

Yolov5_DeepSORT_rknn是基于瑞芯微Rockchip Neural Network(RKNN)开发的目标跟踪部署仓库，除了DeepSORT还支持SORT算法，可以根据不同的嵌入式平台选择合适的跟踪算法。本仓库中的DeepSORT在Rk3588上测试通过，SORT在Rk3588和Rk3399Pro上都可运行。（不好意思 因为使用了Rknn-toolkit2所以rk3399pro应该不支持 需要换掉rknn_fp.cpp相关接口）

<div align="center">
  <img src="https://github.com/Zhou-sx/yolov5_Deepsort_rknn/blob/deepsort/detect.gif" width="45%" />&emsp; &emsp;<img src="https://github.com/Zhou-sx/yolov5_Deepsort_rknn/blob/deepsort/deepsort.gif" width="45%" />
  <br/>
  <font size=5>Detect</font>
	&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
	&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
	&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;
  <font size=5>Track</font>
  <br/>
</div>

DeepSORT率先上线！SORT请稍等。除了这两个算法之外，可能还会更新其他SOTA跟踪算法，多多关注~！

我是野生程序猿，如果在代码编写上存在不规范的情况，请多多见谅。

## 文档内容

- [文件目录结构描述](#文件目录结构描述)
- [安装及使用](#安装及使用)
- [数据说明](#数据说明)
- [性能测试](#性能测试)
- [参考仓库](#参考仓库)

## 文件目录结构描述

```
├── Readme.md                   // help
├── data						// 数据
├── model						// 模型
├── build
├── CMakeLists.txt			    // 编译Yolov5_DeepSORT
├── include						// 通用头文件
├── src
├── 3rdparty                    
│   ├── linrknn_api				// rknn   动态链接库
│   ├── rga		                // rga    动态链接库
│   ├── opencv		            // opencv 动态链接库(自行编译并在CmakeLists.txt中设置相应路径)
├── yolov5           			
│   └── include
│       └── decode.h            // 解码
│       └── detect.h            // 推理
│       └── videoio.h           // 视频IO
│   └── src
│       └── decode.cpp    
│       └── ...
├── deepsort
│   └── include
│       └── deepsort.h     		// class DeepSort
│       └── featuretensor.h     // Reid推理
│       └── ...
│   └── src
│       └── deepsort.cpp
│       └── ...
│   └── CMakeLists.txt			// 编译deepsort子模块

```

## 安装及使用

+ RKNN-Toolkit

  这个项目需要使用RKNN-Toolkit2(Rk3588)或者RKNN-Toolkit1(Rk3399Pro)，请确保librknnrt.so和rknn_server正常运行。可以先运行瑞芯微仓库中的Demo来测试。

    ```
    rknpu2
    https://github.com/rockchip-linux/rknpu2
    ```

+ opencv的编译安装

  可以选择直接在板子上编译，Rk3588编译速度很快，不到十分钟。
  也可以选择交叉编译，我使用的交叉编译工具链：gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu

    ```
    注意！！！
    请根据自己的OpenCV路径修改CMakeLists.txt文件

    搜索 set(OpenCV_DIR /home/linaro/workspace/opencv/lib/cmake/opencv4)
    将路径替换成你的OpenCVConfig.cmake所在的文件夹

    本项目中有两个CMakeLists.txt请同时修改！！！
    ```

+ DeepSort选用的模型是TorchReID中的osnet_x0_25 ，输入尺寸是256x512

  目前还没有针对于自己的数据集重新训练

  ```
  Torchreid
  https://kaiyangzhou.github.io/deep-person-reid/MODEL_ZOO
  ```

+ 项目编译与运行

  参考Cmake的使用，会在build目录下生成可执行文件。

## 数据说明

+ 使用的是红外车辆、行人数据集，自行拍摄的，暂时不公开。

+ 测试的视频存在严重的抖动，影响了跟踪性能，不代表DeepSORT的跟踪性能。

## 性能测试

目前只对模型速度进行了测试，首先送上瑞芯微官方的benchmark

+ 瑞芯微rknn_model_zoo

| platform（fps）          | yolov5s-relu | yolov5s-silu | yolov5m-relu | yolov5m-silu |
| ------------------------ | ------------ | ------------ | ------------ | ------------ |
| rk1808 - u8              | 35.24        | 26.41        | 16.27        | 12.60        |
| rv1109 - u8              | 19.58        | 13.33        | 8.11         | 5.45         |
| rv1126 - u8              | 27.54        | 19.29        | 11.69        | 7.86         |
| rk3566 - u8              | 15.16        | 10.60        | 8.65         | 6.61         |
| rk3588 - u8(single core) | 53.73        | 33.24        | 22.31        | 14.74        |

+ Ours

  DeepSORT的ReID网络单次推理耗时约3ms，但是由于每个检测框都需要推理一次网络故受目标个数影响很大。根据我之前在Rk3399Pro上的部署经验知SORT受目标个数影响小一些且耗时很短，先放上一个预估值。
  
| platform（ms）          | yolov5s-relu | yolov5s-relu+Deepsort |yolov5s-relu+Sort   |
| :-------------------------: | :--: | :--: | :--: |
| rk3588 - u8(single core)    | 24.58|   -  |   -  |
| rk3588 - u8(double core)    | 13.13| 33.24(infulenced)|  15(predict)  |
| rk3399Pro - u8(single core) |   -  |   -  |   -  |

## 参考仓库

本项目参考了大量前人的优秀工作，在最后放上一些有用的仓库链接。

1. https://github.com/ultralytics/yolov5
2. https://github.com/airockchip/rknn_model_zoo
3. https://github.com/airockchip/librga
4. https://github.com/RichardoMrMu/yolov5-deepsort-tensorrt
5. https://github.com/KaiyangZhou/deep-person-reid
