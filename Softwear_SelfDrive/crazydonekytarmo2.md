# 星辰探路者自动驾驶系统手册（扩展篇）


本文描述基于 **Mediapipe** 识别目标增强探路者自动驾驶能力。
内容包括：  
1. 使用预训练模型 识别 `STOP` 标志，车辆执行动作： **停止 -> 等待 -> 前进**
2. 使用预训练模型 识别特定物件， 车辆执行动作：**绕过障碍物**
3. 使用 MediaPipe Maker 迁移学习训练自定义模型 

## Object Detection 测试

### 电脑上测试
* [MediaPipe studio](https://mediapipe-studio.webapps.google.com/home)
* [Labelmap](https://storage.googleapis.com/mediapipe-tasks/object_detector/labelmap.txt)
* [Models](https://ai.google.dev/edge/mediapipe/solutions/vision/object_detector#configurations_options)

### 树莓派上运行 Demo

1. Google 官方样例

  [MediaPipe for Raspberry Pi](https://github.com/google-ai-edge/mediapipe-samples/tree/main/examples/object_detection/raspberry_pi)


```
# SSH 连接树莓派
ssh luke@crazydonkeytarmo-1.local 

# 下载代码
cd ~
git clone https://github.com/google-ai-edge/mediapipe-samples.git

# 设置环境，下载模型
cd mediapipe-samples/examples/object_detection/raspberry_pi
sh setup.sh

# 安装缺少 OpenCV 依赖的 OpenGL 库 (Optinal)
sudo apt install -y libgl1-mesa-glx

# 运行
python3 detect.py   --model efficientdet.tflite

## 摄像头驱动问题，报错和解决参考
# ERROR: Unable to read from webcam. Please verify your webcam settings.
>> https://github.com/google-ai-edge/mediapipe-samples/issues/383

```

2. 星辰探路者 测试代码
   
```
# 上传 mediapipe_cdt
rsync -rv --progress --partial ./mediapipe_cdt/ luke@crazydonkeytarmo-1.local:~/mediapipe_cdt/

# SSH 连接树莓派
ssh luke@crazydonkeytarmo-1.local 
cd mediapipe_cdt

# 下载模型文件 efficientdet_lite0.tflite
wget -q -O efficientdet_lite0.tflite -q https://storage.googleapis.com/mediapipe-models/object_detector/efficientdet_lite0/int8/1/efficientdet_lite0.tflite

# 运行 
# Headless (Text Only)
python3 detect_cdt_text.py   --model efficientdet_lite0.tflite --maxResults 2 --scoreThreshold 0.51 --frameWidth 320 --frameHeight 240

```
```
#fix issue: module 'cv2' has no attribute 'flip'
pip install opencv-python

Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Collecting opencv-python
  Using cached opencv_python-4.10.0.84-cp37-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl.metadata (20 kB)
Requirement already satisfied: numpy>=1.21.2 in /home/luke/env/lib/python3.11/site-packages (from opencv-python) (1.26.4)
Using cached opencv_python-4.10.0.84-cp37-abi3-manylinux_2_17_aarch64.manylinux2014_aarch64.whl (41.7 MB)
Installing collected packages: opencv-python
Successfully installed opencv-python-4.10.0.84
```



## 使用预训练模型

👉 [MR to donkeycar](https://github.com/autorope/donkeycar/pull/1197)

工程改动说明
```
- donkeycar/donkeycar/parts/object_detector/
├── mediapipe_object_detetor.py   ## mediapipe object detection
├── detector_manager.py           ## 识别模块总调度
├── action_stop_and_go.py         ## StopAndGo  停止车辆 -> 等待 -> 前进
├── action_pass_object.py         ## PassObject 绕过障碍物

- donkeycar/donkeycar/templates/
├── complete.py         ## 加载 detector_manager 模块
├── cfg_complete.py     ## 增加 object detection 开关配置

- donkeycar/
├── setup.cfg           ## 增加 mediapipe 依赖

```

创建新应用
```
# SSH 连接树莓派
ssh luke@crazydonkeytarmo-1.local 
cd donkeycar

# 安装依赖 
pip install mediapipe

# 编译(optinal)
pip install -e .[pi]
# pip install -e .\[pi\]

# 创建新应用 crazydonkeytarmo_od
donkey createcar --path ~/crazydonkeytarmo_od
cd ~/crazydonkeytarmo_od

# 下载模型文件物件识别模型文件
# https://storage.googleapis.com/mediapipe-models/object_detector/efficientdet_lite0/int8/1/efficientdet_lite0.tflite
# 上传自定义物件识别模型
# scp android_toy_model.tflite luke@crazydonkeytarmo-1.local:/home/luke/crazydonkeytarmo/

# 上传文件到树莓派：修改后的配置, 自动驾驶驾驶模型，物件识别模型
rsync -rv --progress --partial ./crazydonkeytarmo_od/ luke@crazydonkeytarmo-1.local:~/crazydonkeytarmo_od/
```

# 运行自动驾驶

* 测试识别指定标签
* `STOP` 标志动作
* 绕过障碍物


```
cd crazydonkeytarmo_od

# 1. 配置 myconfig.py： OBJECT_DETECTOR 

# 2. 运行
python manage.py drive --model ./models/cdt1.tflite --type tflite_linear

# 3. 浏览器访问 http://crazydonkeytarmo-1.local:8887/drive
```





## Transefer 训练自己的模型

[MediaPipe model maker](https://ai.google.dev/edge/mediapipe/solutions/model_maker)


[Object detection model customization](https://ai.google.dev/edge/mediapipe/solutions/customization/object_detector)






## Ref

* [mediapipe-samples: object detection on raspberry](https://github.com/google-ai-edge/mediapipe-samples/tree/main/examples/object_detection/raspberry_pi)


* [MediaPipe maker exaple: Object_Detection_for_3_dogs](https://colab.research.google.com/github/googlesamples/mediapipe/blob/main/tutorials/object_detection/Object_Detection_for_3_dogs.ipynb)


