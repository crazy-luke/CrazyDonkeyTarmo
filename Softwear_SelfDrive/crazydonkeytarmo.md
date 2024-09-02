# 星辰探路者自动驾驶系统手册

本文描述基于 **Donkeycar** 搭建 **星辰探路者** 自动驾驶搭建全过程

## Hardware 
  * CrazyDonkeyTarmo 车
  * 树莓派
    * [Raspberry Pi](https://docs.donkeycar.com/guide/robot_sbc/setup_raspberry_pi/): 4B 4GB
    * [Camera](https://docs.donkeycar.com/parts/cameras/): 160 degree wide angle
    * microSD: 128G A2 U3 
      
------
##  一、安装系统
### Raspberry Pi

DonkeycarDoc:[Get Your Raspberry Pi Working](https://docs.donkeycar.com/guide/robot_sbc/setup_raspberry_pi/)

1. **安装树莓派 OS**
  * 操作： SD卡插入电脑
  * 安装工具: [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
    * Choose
      * Choose divice: `raspberry pi 4`
      * Choose OS: `Raspberry Pi OS Lite 64-bit Bookworm`
      * Choose Storage: `SD`
    * Edit Settings
      * hostname: `crazydonkeytarmo-1`
      * username: `luke`, password: `123456`
      * SSID,  Password, Country
      * Enable SSH
  * 操作：SD插入树莓派，树莓派和电脑处于同一个网络，通电

  * 修改莓派修改配置
    
      ```
      # 电脑通过 SSH 连接树莓派
      # ssh username@ip
      # ssh username@hostname.local

      ssh luke@crazydonkeytarmo-1.local 

      # 进入配置
      sudo raspi-config
      ```
      配置界面依次选择：Advanced Options -> Expand Filesystem -> Finish -> reboot
      
      


2. **升级环境**

  * 升级软件包
    ```
    # Update and Upgrade
    sudo apt-get update --allow-releaseinfo-change
    sudo apt-get upgrade
    ```

  * 设置 Python 虚拟环境
    ```
    #创建一个名为env的虚拟环境，命令执行完毕后会出现一个env文件夹
    python3 -m venv env --system-site-packages

    #每次开机都会自动进入env的虚拟环境
    echo "source ~/env/bin/activate" >> ~/.bashrc
    source ~/.bashrc
    ```

  * 提前安装后续可能遇到错误的库（optinal)
    ```
    # libcap：捕获 过滤 分析 存储网络数据包
    sudo apt install libcap-dev

    # psutil：获取系统的信息， h5py：读写HDF5文件格式
    sudo apt-get install gcc python3-dev
    pip install psutil
    pip install h5py --only-binary h5py

    # python3-libcamera: 摄像头
    sudo apt install -y python3-libcamera
    
    # python3-kms++
    sudo apt install -y python3-kms++
    ```

3. **Donkeycar Code**

  * 下载源码
    ```
    # 安装git库
    sudo apt install git

    # 下载Git代码
    cd ~
    git clone https://github.com/autorope/donkeycar.git
    cd donkeycar
    
    # 切换版本
    git checkout 5.1.0

    # 编译安装 
    pip install -e .[pi]
    # zsh shell
    pip install -e .\[pi\]
    ```

  * 验证 tensorflow (optinal)
  ```
  python -c "import tensorflow; print(tensorflow.__version__)"
  ```



### 创建 CrazyDonkeyTarmo

1. [Create your car application 创建应用](https://docs.donkeycar.com/guide/create_application/)

    ```
    donkey createcar --path ~/crazydonkeytarmo

    cd ~/crazydonkeytarmo
    ```


2. 配置 [控制器](https://docs.donkeycar.com/parts/rc/)

  * 启用 pigpiod
    ```
    sudo systemctl enable pigpiod & sudo systemctl start pigpiod
    ```

  * 编辑 ~/crazydonkeytarmo/myconfig.py

    1. 从树莓派下载 myconfig.py 文件到电脑
    ```
    #电脑开一个新的终端
    #scp <username>@<ip>:<src_file> <des_file>

    scp luke@crazydonkeytarmo-1.local:/home/luke/crazydonkeytarmo/myconfig.py .
    ```


    2. 电脑打开 myconfig.py 文件并修改下面字段

    * 开启 pigpio
      ``` 
      CONTROLLER_TYPE = 'pigpio_rc'
      USE_JOYSTICK_AS_DEFAULT = True
      DRIVE_TRAIN_TYPE = "PWM_STEERING_THROTTLE"
      ```
    * 修改 PWM 输入信号（RC接收机 -> 树莓派）
      ```
      #PIGPIO RC control
      STEERING_RC_GPIO = 3              # CrazyDonkeyTarmoBus - s_in
      THROTTLE_RC_GPIO = 15             # CrazyDonkeyTarmoBus - t_in
      DATA_WIPER_RC_GPIO = 4            # CrazyDonkeyTarmoBus - ext
      PIGPIO_STEERING_MID = 1500         # Adjust this value if your car cannot run in a straight line
      PIGPIO_MAX_FORWARD = 2000          # Max throttle to go fowrward. The bigger the faster
      PIGPIO_STOPPED_PWM = 1500
      PIGPIO_MAX_REVERSE = 1000          # Max throttle to go reverse. The smaller the faster
      PIGPIO_SHOW_STEERING_VALUE = False
      PIGPIO_INVERT = False
      PIGPIO_JITTER = 0.025   # threshold below which no signal is reported
      ```
    * 打开 PWM 输出配置 （树莓派 -> 方向舵机和动力电机）
      ```
      # 打开PWM_STEERING_THROTTLE字段，修改PIN
      PWM_STEERING_THROTTLE = {
          "PWM_STEERING_PIN": "PIGPIO.BCM.2",   # CrazyDonkeyTarmoBus - s-out
          "PWM_THROTTLE_PIN": "PIGPIO.BCM.14",  # CrazyDonkeyTarmoBus - t-out
          .....
      }
      ```
    
    * 其他配置  Optinal
      ```
      # 修改摄像头
      CAMERA_TYPE = "PICAM"
      IMAGE_W = 320
      IMAGE_H = 240

      # 关闭自动录制
      AUTO_RECORD_ON_THROTTLE = False
      ```

    * 修改后的 myconfig.py 文件上传到树莓派
    ```
    #电脑 -> 树莓派
    #scp <src_file> <username>@<ip>:<des_file>

    scp myconfig.py luke@crazydonkeytarmo-1.local:/home/luke/crazydonkeytarmo/
    ```    

4. 测试
  注意：**关闭ESC电调开关**

  * 启动 crazydonkeytarmo应用
    ```
    #终端连接树莓派
    ssh luke@crazydonkeytarmo-1.local 

    #启动
    cd ~/crazydonkeytarmo
    python manage.py drive
    ```
  * 电脑浏览器访问
    http://ip:8887/drive
   
    如 http://crazydonkeytarmo-1.local:8887/drive

  * 关闭crazydonkeytarmo应用
  终端 `ctrl+c`



### 校准 

1. 官方[驴车参数](https://docs.donkeycar.com/parts/rc_hat/#controlling-esc-and-steering-servo-with-rc-hat)


2. CrazyDonkeyTarmo 参考参数
```
# 注：RC 接收机58hz,官方默认参数60hz, 修改 SCALE 为1.03
PWM_STEERING_THROTTLE = {
    "PWM_STEERING_PIN": "PIGPIO.BCM.2",   # PWM output pin for steering servo
    "PWM_STEERING_SCALE": 1.03,             # used to compensate for PWM frequency differents from 60hz; NOT for adjusting steering range
    "PWM_STEERING_INVERTED": False,         # True if hardware requires an inverted PWM pulse
    "PWM_THROTTLE_PIN": "PIGPIO.BCM.14",   # PWM output pin for ESC
    "PWM_THROTTLE_SCALE": 1.0,              # used to compensate for PWM frequence differences from 60hz; NOT for increasing/limiting speed
    "PWM_THROTTLE_INVERTED": False,         # True if hardware requires an inverted PWM pulse
    "STEERING_LEFT_PWM": 200,               #pwm value for full left steering
    "STEERING_RIGHT_PWM":400,              #pwm value for full right steering
    "THROTTLE_FORWARD_PWM": 400,            #pwm value for max forward throttle
    "THROTTLE_STOPPED_PWM": 300,            #pwm value for no movement
    "THROTTLE_REVERSE_PWM": 200,            #pwm value for max reverse throttle
}
```

------
## 二、自动驾驶

### 收集训练数据
* 树莓派连接手机[网络(Bookworm)](https://www.raspberrypi.com/documentation/computers/configuration.html#configuring-networking)
  ```
  #终端连接树莓派
  ssh luke@crazydonkeytarmo-1.local 

  #查看网络
  sudo nmcli dev wifi list

  #连接网络 (例如 AiPhone)
  sudo nmcli --ask dev wifi connect AiPhone

  #查看网络连接优先级
  nmcli --fields autoconnect-priority,name connection

  #调整优先级（数字越大优先级越高）
  sudo nmcli connection modify AiPhone connection.autoconnect-priority 2
  ```
* 手机安装SSH应用 APP ： `Termius`

2. 采集训练数据

* SSH 连接树莓派
  ```
  ssh luke@crazydonkeytarmo-1.local 
  cd crazydonkeytarmo

  #清理历史数据(optinal)
  #rm -rf ./data/*

  #开始驾驶
  python manage.py drive
  ```
* 开始录制
浏览器访问 http://crazydonkeytarmo-1.local:8887/drive
User-> Start Reconrding
**Let's Go~~~~**


### 训练数据
1. 树莓派训练数据上传到电脑
```
## 树莓派crazydonkeytarmo 目录同步下载到电脑上（初次）
rsync -rv --progress --partial luke@crazydonkeytarmo-1.local:~/crazydonkeytarmo/  ./crazydonkeytarmo

## 树莓派 data目录同步下载到电脑上 crazydonkeytarmo 目录中
# cd crazydonkeytarmo 
# rsync -rv --progress --partial luke@crazydonkeytarmo-1.local:~/crazydonkeytarmo/data/  ./data
```


2. 电脑安装 donkeycar （初次)
```
git clone https://github.com/autorope/donkeycar.git
cd donkeycar
git checkout 5.1.0

#删除已经存在的环境
#conda activate base
#sudo conda env remove -n donkey

conda create -n donkey python=3.11
conda activate donkey
#确认当前已经处于 donkey
conda env list  

#安装
#pip install donkeycar\[pc\]
pip install -e .\[pc\]
```

3. [训练模型](https://docs.donkeycar.com/guide/deep_learning/train_autopilot/)
  
  * 启动donkey ui
    ```
    conda activate donkey
    sudo donkey ui
    ``` 
    * Load car directory:  选择 crazydonkeycar 目录
    * Load tube: 选择 crazydonkeycar 中的data目录（训练数据）
    * Delete: 删除不需要的数据区域(optinal)

  * 通过 [donkey ui](https://docs.donkeycar.com/utility/ui/) 训练
    `Trainer -> Train`
  * 通过命令行训练（optinal, 适合数据量较大)
    ```
    # 用 data 目录下的数据训练一个名为 cdt1 的模型
    donkey train --tub ./data  --model ./models/cdt1
    ```
  * [Train model using GPU for FREE on google golab](https://colab.research.google.com/drive/1CALMB7GioBEUcWNCuHQjrWCT4r01p6Qh)
4. 可能遇到的错误(optinal)
     
     ```
     ## v5.1.0  
     ## ValueError: Key image is not in available keys.
     ## fix: 设置 albumentations==1.4.6
     sudo pip uninstall -y albumentations && pip install albumentations==1.4.6
     ```


### 开始自动驾驶

1. 上传训练完成的 Model 到树莓派
   ```
   #上传模型到树莓派
   #进入电脑的 crazydonkeytarmo 目录执行
   rsync -rv --progress --partial ./models/ luke@crazydonkeytarmo-1.local:~/crazydonkeytarmo/models/
   ```
2. 自动驾驶
    ```
    # 连接 树莓派
    ssh luke@crazydonkeytarmo-1.local 
    cd ~/crazydonkeytarmo

    # 启动模型
    python manage.py drive --model ./models/cdt1.tflite --type tflite_linear
    ```




### 扩展篇
  
 [物件识别](./crazydonekytarmo2.md)

