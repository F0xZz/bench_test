测试时间 2022 0324

赛扬J1900 4x1.99Ghz

tf > 2.0 illegal instruction (core dump)

不能该主机——转换模型

yolov4-tiny

cpuinfo

```
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:27] Info From CPUID:
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:28] Vendor: GenuineIntel
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:29] VNNI  : 0
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:30] AVX512: 0
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:31] AVX2  : 0
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:32] FMA3  : 0
[INFO][2022-03-24 13:01:52.810][cpuinfo.cc:33] AVX   : 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:34] SSE   : 1
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:35] L1 D-Cahce: 24576
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:36] L2 Cahce: 1048576
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:37] L3 Cahce: 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:39] Info From RUN:
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:40] Vendor: GenuineIntel
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:41] VNNI  : 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:42] AVX512: 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:43] AVX2  : 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:44] FMA3  : 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:45] AVX   : 1
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:46] SSE   : 0
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:47] L1 D-Cahce: 24576
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:48] L2 Cahce: 1048576
[INFO][2022-03-24 13:01:52.811][cpuinfo.cc:49] L3 Cahce: 0
```

## Tengine

reference code 来自OAID 

(finish test in yolov3-tiny ----> 820ms) 

install

https://github.com/OAID/Tengine

Convert-tools

https://github.com/OAID/Tengine-Convert-Tools

编译流程参考文档，需要protobuf

```
sudo apt install libprotobuf-dev protobuf-compiler
```

按照流程手动编译即可

！！！！ J1900 **AVX2**指令优化CMakeLists中需要**OFF** 否则出现illeage instruction (core dump) 仅能使用未优化版本

对应issues：https://github.com/OAID/Tengine/issues/1306

**利用convert-tools**

编译完可进行转换darknet framework下

```
./install/bin/convert_tool -f darknet -p /home/ubuntu/moduletest/darknet/cfg/yolov4-tiny.cfg -m /home/ubuntu/moduletest/yolov4-tiny.weights -o yolov4-tiny.tmfile
```

benchmark test

only run the backbone part

```
squeezenet_v1.1  min =  224.05 ms   max =  224.05 ms   avg =  224.05 ms
mobilenetv1  min =  494.76 ms   max =  494.76 ms   avg =  494.76 ms
mobilenetv2  min =  443.05 ms   max =  443.05 ms   avg =  443.05 ms
mobilenetv3  min =  302.04 ms   max =  302.04 ms   avg =  302.04 ms
shufflenetv2  min =  140.08 ms   max =  140.08 ms   avg =  140.08 ms
resnet18  min =  748.62 ms   max =  748.62 ms   avg =  748.62 ms
resnet50  min = 2139.88 ms   max = 2139.88 ms   avg = 2139.88 ms
googlenet  min =  891.00 ms   max =  891.00 ms   avg =  891.00 ms
inceptionv3  min = 3491.12 ms   max = 3491.12 ms   avg = 3491.12 ms
vgg16  min = 2833.26 ms   max = 2833.26 ms   avg = 2833.26 ms
mssd  min = 1007.65 ms   max = 1007.65 ms   avg = 1007.65 ms
retinaface  min =  145.30 ms   max =  145.30 ms   avg =  145.30 ms
yolov3_tiny  min =  867.64 ms   max =  867.64 ms   avg =  867.64 ms
mobilefacenets  min =  205.38 ms   max =  205.38 ms   avg =  205.38 ms
```

## Openvino

Intel旗下对Intel开放的使用工具

offical support 

https://docs.openvino.ai/latest/openvino_docs_OV_UG_Working_with_devices.html

don't include the J1900

```
-i https://pypi.tuna.tsinghua.edu.cn/simple
```

python 使用pip方式进行安装

https://docs.openvino.ai/latest/get_started.html

open_model_zoo

https://github.com/openvinotoolkit/open_model_zoo/

use guide :https://docs.openvino.ai/latest/omz_models_group_intel.html

demo command

!! **copy common folder to path exec** 

```
python3 object_detection_demo.py -m ./yolo-v4-tiny-tf.xml -i /home/ubuntu/moduletest/openvino/img/1.jpg --no_show
python3 object_detection_demo.py -m ./FP32/yolo-v4-tiny-tf.xml -i /home/ubuntu/moduletest/openvino/img/1.jpg --no_show
```

test yolov4-tiny-tf  in 963ms

```
[ INFO ] OpenVINO Runtime
[ INFO ]        build: 2022.1.0-7019-cdb9bec7210-releases/2022/1
[ INFO ] Reading model ./FP32/yolo-v4-tiny-tf.xml
[ WARNING ] The parameter "input_size" not found in YOLOV4 wrapper, will be omitted
[ INFO ]        Input layer: image_input, shape: [1, 3, 416, 416], precision: f32, layout: NCHW
[ INFO ]        Output layer: conv2d_20/BiasAdd/Add, shape: [1, 255, 26, 26], precision: f32, layout: 
[ INFO ]        Output layer: conv2d_17/BiasAdd/Add, shape: [1, 255, 13, 13], precision: f32, layout: 
[ INFO ] The model ./FP32/yolo-v4-tiny-tf.xml is loaded to CPU
[ INFO ]        Device: CPU
[ INFO ]                Number of streams: 4
[ INFO ]                Number of threads: AUTO
[ INFO ]        Number of model infer requests: 5
[ INFO ] Metrics report:
[ INFO ]        Latency: 963.3 ms
[ INFO ]        FPS: 1.0
[ INFO ]        Decoding:       22.1 ms
[ INFO ]        Preprocessing:  16.9 ms
[ INFO ]        Inference:      918.6 ms
[ INFO ]        Postprocessing: 24.0 ms
[ INFO ]        Rendering:      1.5 ms
```

How to use demo

https://docs.openvino.ai/latest/openvino_docs_get_started_get_started_demos.html

https://github.com/openvinotoolkit/open_model_zoo/blob/master/demos/common/python/openvino/model_zoo/model_api/README.md#model-api-adapters

自带omz-downloader

can use the `omz-downloader -h` to see the help doc

```
omz_downloader --name googlenet-v1 --output_dir ~/models 
```

如果需要安装open_model_zoon的api

```
pip install <omz_dir>/demos/common/python[ovms]
# makesur install whether
python -c "from openvino.model_zoo import model_api"
```

if use tiny openvino with tf 

需要安装tensorflow

this will download the model and test demo

需要安装models

特殊层级需要c++

c++ 需要采用源码的方式进行编译

https://github.com/openvinotoolkit/openvino/wiki/BuildingForLinux

GPU 需要install opencl

CPU 需要改设置 -DENABLE_CLDNN=OFF

```
 cmake -DCMAKE_BUILD_TYPE=Release -DNCNN_VULKAN=ON -DNCNN_BUILD_EXAMPLES=ON ..
```



## openppl

ppl.nn

https://github.com/openppl-public/ppl.nn

openmmlab open source 

install method

basic module

```
sudo apt-get install build-essential cmake git python3 python3-dev
```

Building from source:

```
./build.sh -DHPCC_USE_X86_64=ON -DPPLNN_ENABLE_PYTHON_API=ON
```

openppl

complie complete

**but some error occur in the process of the python demo**

```
error:get unsupported isa 0
```

报错代码指向指令集选择片段，尝试关闭AVX等指令优化选项

complie complete but also has occured error in the process

don't continue

已经提issues：https://github.com/openppl-public/ppl.nn/issues/410



## Ncnn

reference code

https://github.com/Tencent/ncnn/wiki/how-to-build

dependence

```
sudo apt install build-essential git cmake libprotobuf-dev protobuf-compiler libvulkan-dev vulkan-utils libopencv-dev
```

vulkan 和 cpu版本

vulkan采用 类似openGL和openCL 通用显存调用机制

确认vulkaninfo

如果vulkaninfo存在

```
cmake -DCMAKE_BUILD_TYPE=Release -DNCNN_VULKAN=ON -DNCNN_SYSTEM_GLSLANG=ON -DNCNN_BUILD_EXAMPLES=ON -DNCNN_PYTHON ..
```

test

```
cd ../examples
../build/examples/squeezenet ../images/256-ncnn.png
```

test2 **vulkan use abort**

```
cd ../benchmark
../build/benchmark/benchncnn 10 $(nproc) 0 0 -1
```

only run the backbone part

```
          squeezenet  min =   43.18  max =   45.00  avg =   43.88
     squeezenet_int8  min =   67.48  max =   69.62  avg =   68.22
           mobilenet  min =   68.91  max =   70.83  avg =   69.87
      mobilenet_int8  min =   94.99  max =  341.66  avg =  125.79
        mobilenet_v2  min =   55.80  max =   57.40  avg =   56.62
        mobilenet_v3  min =   44.86  max =   49.13  avg =   46.04
          shufflenet  min =   32.61  max =   34.25  avg =   33.36
       shufflenet_v2  min =   30.12  max =   32.00  avg =   30.74
             mnasnet  min =   48.13  max =   51.01  avg =   49.75
     proxylessnasnet  min =   52.99  max =   55.05  avg =   54.08
     efficientnet_b0  min =   86.03  max =   89.33  avg =   87.35
   efficientnetv2_b0  min =   96.50  max =   98.54  avg =   97.71
        regnety_400m  min =   67.12  max =   69.89  avg =   68.81
           blazeface  min =    8.92  max =    9.21  avg =    9.07
           googlenet  min =  156.73  max =  158.98  avg =  158.02
      googlenet_int8  min =  232.63  max =  236.75  avg =  234.00
            resnet18  min =  123.84  max =  127.28  avg =  125.84
       resnet18_int8  min =  182.42  max =  187.56  avg =  184.00
             alexnet  min =  101.45  max =  104.95  avg =  103.51
               vgg16  min =  691.41  max =  948.02  avg =  739.15
          vgg16_int8  min =  817.89  max = 1092.48  avg =  874.65
            resnet50  min =  319.08  max =  558.43  avg =  356.77
       resnet50_int8  min =  487.14  max =  792.22  avg =  520.23
      squeezenet_ssd  min =  115.04  max =  118.51  avg =  116.89
 squeezenet_ssd_int8  min =  152.60  max =  155.66  avg =  154.66
       mobilenet_ssd  min =  152.00  max =  583.26  avg =  198.56
  mobilenet_ssd_int8  min =  194.91  max =  606.24  avg =  264.24
      mobilenet_yolo  min =  356.94  max =  374.19  avg =  365.60
  mobilenetv2_yolov3  min =  199.99  max =  232.07  avg =  209.03
         yolov4-tiny  min =  249.49  max =  264.35  avg =  256.41
           nanodet_m  min =   73.11  max =   76.17  avg =   74.56
    yolo-fastest-1.1  min =   38.30  max =   40.80  avg =   39.36
      yolo-fastestv2  min =   33.30  max =   36.14  avg =   34.03
```

