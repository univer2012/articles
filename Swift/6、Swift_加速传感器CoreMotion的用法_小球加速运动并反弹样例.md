来自：[Swift - 加速传感器（CoreMotion）的用法，小球加速运动并反弹样例](https://www.hangge.com/blog/cache/detail_545.html)



**1，加速传感器可以监听到x,y,z三个方向的加速度，使用步骤如下：** 

（1）实例化`CMMotionManager`类

 （2）向`CMMotionManager`的`accelerometerUpdateInterval`属性中设置通知间隔时间值。



 **2，通知频率设置建议**

 `accelerometerUpdateInterval`表示通知频率，表示间隔多少秒通知一次。iPhone开发文档中推荐使用的通知间隔如下： 

（1）检测设备朝向：1/10 ~ 1/20 

（2）在游戏中需要实时使用加速传感器时：1/30 ~ 1/60 

（3）检测敲击设备或者剧烈摇动设备的情况下：1/70 ~ 1/100

 **3，x,y,z轴** 

（1）对于iphone手机来说，画面上下为y轴，左右为x轴，贯穿屏幕为z轴。 

（2）向上，向右，手机的前面分别是各轴的正方向。

[![原文:Swift - 加速传感器（CoreMotion）的用法，小球加速运动并反弹样例](https://www.hangge.com/blog_uploads/201702/2017020819193293224.png)](https://www.hangge.com/blog/cache/detail_545.html#)



 **4，加速度（原始加速度）** 

加速度不仅受震动手机时施加的作用力的影响，还会持续受到重力的影响。因此iphone手机如果垂直拿在手上的话，Y轴负方向将受重力作用，加速度y属性将一直为负值（最小值为-1.0） 

**5，Gravity和UserAcceleration**

> 网友cruise_H问：`motionManager.deviceMotion.userAcceleration.x`和`motionManager.accelerometerData!.acceleration.x`两个获取的`acceleration`有什么区别？



上面提到的原始的加速度（即通过`startAccelerometerUpdates`获取的那个值）实际上是由两种加速度合成而来的。一个是重力加速度（Gravity），一个是用户对手机施加的加速度（UserAcceleration）（当然我们也可以分别获取这两种加速度）。

所以当手机垂直静止时，虽然UserAcceleration是0，但由于有重力加速度，所以两个合成后加速度y属性便是为负值。



 **6，测试样例**
 该应用运行后，将在画面正中央显示一个球体。倾斜手机时，球体将向着倾斜的方向运动。碰撞到四壁后反弹回来。

[![原文:Swift - 加速传感器（CoreMotion）的用法，小球加速运动并反弹样例](https://www.hangge.com/blog_uploads/201611/2016111919283823490.png)](https://www.hangge.com/blog/cache/detail_545.html#)

（注意：由于需要用到设备的重力感应器，所以要使用真机调试，模拟器运行小球不会动。）

```swift
import UIKit

import CoreMotion

class SHCoreMotionDemoViewController: UIViewController, UIAccelerometerDelegate {

    var ball: UIImageView!
    var speedX: UIAccelerationValue = 0
    var speedY: UIAccelerationValue = 0
    var motionManager = CMMotionManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()

        //放一个小球在中央
        ball = UIImageView(image: UIImage(named: "ball"))
        ball.frame = CGRect(x: 0, y: 0, width: 50, height:50)
        ball.center = self.view.center
        self.view.addSubview(ball)
        
        motionManager.accelerometerUpdateInterval = 1/60
        
        if motionManager.isAccelerometerAvailable {
            let queue = OperationQueue.current!
            motionManager.startAccelerometerUpdates(to: queue) { (accelerometerData, error) in
                //动态设置小球位置
                self.speedX += accelerometerData!.acceleration.x
                self.speedY = accelerometerData!.acceleration.y
                var posX = self.ball.center.x + CGFloat(self.speedX)
                var posY = self.ball.center.y - CGFloat(self.speedY)
                //碰到边框后的反弹处理
                if posX < 0 {
                    posX = 0
                    //碰到左边的边框 后以0.4倍的速度反弹
                    self.speedX *= -0.4
                } else if posX > self.view.bounds.size.width {
                    posX = self.view.bounds.size.width
                    //碰到右边的边框后以0.4倍的速度反弹
                    self.speedX *= -0.4
                }
                
                if posY < 0 {
                    posY = 0
                    //碰到上面的边框不反弹
                    self.speedY = 0
                } else if posY > self.view.bounds.size.height {
                    posY = self.view.bounds.size.height
                    //碰到下面的边框后以1.5倍的速度反弹
                    self.speedY *= -1.5
                }
                self.ball.center = CGPoint(x: posX, y: posY)
            }
        }
    }
}
```

