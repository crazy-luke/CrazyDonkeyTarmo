# CrazyDonkeyTarmo

`CrazyDonkeyTarmo` 项目目标是保留高性能RC车乐趣的同时能开展学习自动驾驶技术。  
项目深度改造 Tarmo5 RC 车， 结合 DonkeyCar 自动驾驶项目。

## 视频介绍
* 《造车篇》： https://www.bilibili.com/video/BV1qM4m1U7vm
* 《自动驾驶篇》: https://www.bilibili.com/video/BV124HyeUER8

---
## 车辆制作 (Hardwear_Car)
* [CDT_Assemble.pdf](./Hardwear_Car/CDT_Assemble.pdf) ： 组装说明书
* STL： 3D 打印文件 （打印参数见 `stl/README.md` )
* [Parts.md](./Hardwear_Car/Parts.md): 视频中使用的部分零件

![main_car](res/main_car.png) 

## PCB电路 （Optinal)
* PCB_CrazyDonkeyTarmoBoard：超级电源板 
* PCB_CrazyPowerBoard： 探路者控制板 
    * BOM.csv: 采购元件清单
    * ibom.html：元件摆放(细节见PNG图片)
    * Gerber：PCB生产
    * Schematic：原理图
* 电路连接: [Wiring Diagram.jpg](./Hardwear_Car/wiring_diagram.jpg)

---
## 自动驾驶篇 (Softwear_SelfDrive)
* [星辰探路者自动驾驶系统手册](./Softwear_SelfDrive/README.md)
* [扩展篇-物件识别](./Softwear_SelfDrive/crazydonekytarmo_od.md) （Optinal)
* myconfig: 探路者配置参考文件

![self_drive](res/self_drive.png)



## Reference
* [tarmo5](https://www.reddit.com/r/EngineeringNS/comments/zvellk/tarmo5/)
* [donkeycar](https://www.donkeycar.com/)
  


## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International License</a>.

Copyright © 2024, 疯狂豆® 