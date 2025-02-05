========================
跑酷小车(RunGo)
========================

跑酷小车(RunGo)由HiiBot推出的智能小车底盘，包含以下资源：

  - 3x 底盘像素彩灯(位于小车底盘的底部/面向地面, pixels)，使用P1引脚
  - 1x 颜色识别传感器(位于小车底盘的底部，识别地面上色块的颜色, groundColor)，使用P2引脚
  - 2x 光线强度传感器(位于小车前部)，使用P3(小车右前部光强度, rightLightSensor)和P4(小车左前部光强度, leftLightSensor)引脚
  - 2x 黄色头灯(位于小车前部)，使用P6(小车右前灯, rightHeadLED)和P7(小车左前灯, leftHeadLED)引脚
  - 1x 超声波避障传感器(与前方障碍物之间的距离, distance)，使用P8(Trig)和P12(Echo)引脚
  - 2x 光电反射型循迹传感器，使用P9(rightTracker)和P10(leftTracker)引脚
  - 2x 直流有刷电机(N20型)，使用P13和P15(右马达)、P14和P16(左马达)引脚
  - 1x 车载锂电池包(3.7V,600mAh)及其充电和电量指示单元，使用USB Type C接口实现快速充电(20~30分钟即可充满)，满电后可连续使用1小时以上
  - 1x 高效的车载电源管理单元，小车底盘右前方的轻触按钮可开启/关闭小车电源，


---------------------------------

RunGo小车底盘的传感器和分布(示意图)，如下图所示：

.. image::  ../../_static/images/peripheral/rungo_car_sensors_layout_.jpg
  :scale: 45%
  :align: center

BlueFi+RunGo组成智能小车(实物图)，如下图所示。

.. image::  ../../_static/images/peripheral/rungo_car.jpg
  :scale: 45%
  :align: center

与microbit+RunGo组成的智能小车相比，使用BlueFi丰富的传感器你将能实现更多种交互，譬如，BlueFi的姿态传感器可以感知小车的水平或倾角，
使用BlueFi的声音传感器、按钮等与小车交互，使用BlueFi的彩色像素控制氛围光，使用BlueFi的无线电通讯、蓝牙通讯和WiFi通讯实现RunGo与手柄交互等。

----------------------------------

使用前准备工作：添加RunGo库模块
----------------------------------

使用RunGo小车前，请点击“下载按钮”下载RunGo的Python库文件 
:download:`hiibot_rungo.py <../../_static/lib_py/hiibot_rungo.py>`  到本地，
然后将该文件保存到BlueFi的"/CIRCUITPY/lib/"文件夹中。

注意，请使用USB数据线将BlueFi与你的电脑连接在一起，电脑的资源管理器中出现“CIRCUITPY”磁盘，然后将RunGo的Python库文件复制到"/CIRCUITPY/lib/"文件夹即可。
注意，部分BlueFi的lib文件夹中已有hiibot_rungo库文件，请忽略此步骤。检查方法：使用USB数据线将BlueFi与电脑连接，出现CIRCUITPY磁盘后，
展开"/CIRCUITPY/lib/"文件夹，检查是否已有“hiibot_rungo.py”或“hiibot_rungo.mpy”，其中“py”格式是Python脚本源码库，“mpy”格式是压缩的Python脚本源码库，
前者是文本格式具有可读性但文件较大，后者是二进制格式无可读性但文件较小。

--------------------------------

让RunGo动起来
--------------------------------

当你拿到RunGo和BlueFi之后，并做好前述的准备工作做，我们首先让RunGo小车动起来。
实现的效果：小车前进一段距离；然后开启右转灯并开始右转，然后关闭右转灯并停止右转；如此循环。

示例程序代码如下：

.. code-block::  python
  :linenos:

  import time
  from hiibot_rungo import RunGo
  car = RunGo()
  carspeed = 80
  while True:
      car.motor(carspeed, carspeed)
      time.sleep(0.5)
      car.stop()
      car.rightHeadLED = 1   # turn on right head lamp
      car.motor(carspeed//2, -carspeed//2)
      time.sleep(0.65)
      car.rightHeadLED = 0   # turn off right head lamp
      car.stop()

请将上述示例代码保存到BlueFi的/CIRCUITPY/code.py文件，并将BlueFi插入到RunGo小车上，然后打开RunGo小车的电源，
请观察RunGo小车的动作是否达到我们的预期效果。

上述示例程序非常容易理解。前两行语句是导入Python模块，第3行程序是将RunGo类实例化为“car”，第4行设置一个常数。
在无穷循环程序块中，我们使用“car.motor(左轮速度和方向, 右轮速度和方向)”接口控制小车前进、后退、左转和右转，该接口的
两个参数分别代表左轮速度和方向、右轮速度和方向，负数表示反转，正数表示正转，数值绝对值的大小代表速度，速度取值0～255。

此外，本示例也展示如何控制RunGo小车的左/右头灯。

--------------------------------

识别地面颜色(色块)
--------------------------------

RunGo小车的底部有一个颜色识别传感器，可用于识别地面的颜色或色块，有效识别区域是巡线传感器的区域(颜色识别传感器在标注“P2”文字的地方)。
本示例程序实现的效果：让小车置于白、红、黄、绿、青、蓝或紫色的地面或贴有这些颜色色块之上，按下A按钮后，BlueFi的5颗彩灯将显示对应的地面颜色，
并在LCD屏幕上显示颜色的名称字符串(White、Red、Yellow、Green、Cyan、Blue或Purple)。

示例程序代码如下：

.. code-block::  python
  :linenos:

  from hiibot_bluefi.basedio import Button, NeoPixel
  from hiibot_rungo import RunGo
  button = Button()
  rgb = NeoPixel()
  car = RunGo()
  rgb.brightness = 0.2
  rgb.fillPixels((0,0,0))
  car.stop() # stop motors
  print("Press Button-A to sense ground color")
  car.pixels.fill(0)
  car.pixels.show()
  while True:
      button.Update()
      if button.A_wasPressed:
          cid = car.groundColorID
          print(car.groundColor_name[cid])
          rgb.fillPixels(car.groundColor_list[cid])


请将上述示例代码保存到BlueFi的/CIRCUITPY/code.py文件，并将BlueFi插入到RunGo小车上，然后打开RunGo小车的电源，
每次按下A按钮即可执行一次“地面颜色”识别，并将识别出来的颜色名字字符串显示到LCD屏幕上，同时BlueFi的5颗彩灯也显示出同样的颜色。

上述示例程序非常容易理解。前两行语句是导入Python模块；第3～5行程序是将Button类、NeoPixel类、RunGo类分别实例化为“button”、“rgb”、“car”；
第6～7行将BlueFi上的5颗彩灯熄灭(即显示黑色)；第8行代码让小车停止。
在无穷循环程序块中，我们使用“button.Update()”接口检测A按钮是否被按下，如果被按下则开始识别地面颜色并返回颜色识别结果(颜色ID)，
使用“car.groundColor_name[color_id]”列表返回该颜色ID对应的颜色名称(字符串)并打印到屏幕上，
然后使用“car.groundColor_list[id]”列表返回该颜色ID对应的颜色的RGB分量值(元组类型)，并让BlueFi的5颗彩灯显示这种颜色。

-------------------------------

电子围栏
-------------------------------

前面的示例中都没有逻辑的问题，只是简单的顺序执行固定动作，下面我们来实现一个稍微复杂一点的动作效果：地上画个圆作为电子围栏的边界，
RunGo小车就在围栏内随意行驶。准备工作：在白色地面或纸上贴上宽度大于1公分以上的黑色胶带或不干胶，确保黑色胶带围成一个封闭的图案，
并将RunGo小车放在图案内。图案可以参考下图所示：

.. image::  ../../_static/images/peripheral/rungo_corral.jpg
  :scale: 30%
  :align: center

执行下面的示例代码，你会看到RunGo小车在电子围栏内随意地行驶，但始终不会跑出围栏。


.. code-block::  python
  :linenos:

  import time
  from hiibot_bluefi.basedio import Button
  button = Button()
  print("I am BlueFi")
  # import RunGo module from hiibot_rungo.py
  from hiibot_rungo import RunGo
  car = RunGo()
  print("RunGo")
  # speed=100, 0, forward; 1, backward; 2, rotate-left; 3, rotate-right
  car.stop() # stop motors
  print("we do it, press Button-A")

  car.pixels.brightness = 0.5
  car.pixels[0] = (255,0,0)
  car.pixels[1] = (0,255,0)
  car.pixels[2] = (0,0,255)
  car.pixels.show()

  car.rightHeadLED = 0
  car.leftHeadLED = 0

  carSpeed_fast = 100
  carSpeed_slow = 70

  carrun = False
  while True:
      button.Update()
      if button.A_wasPressed:
          carrun = True
          print("running")
      if button.B_wasPressed:
          car.stop()
          print("stop")
          carrun = False
      lt = car.leftTracker   # left sensor
      rt = car.rightTracker  # right sensor
      if carrun:
          if lt ==1 and rt ==1 :  # dual sensor above back-line
              car.stop()
              car.move(1, 0-carSpeed_fast)  # backward
              time.sleep(0.2)
              car.stop()
              car.move(2, carSpeed_fast)  # turn left
              time.sleep(0.2)
              car.stop()
          elif lt ==1 :  # left sensor above back-line only
              car.stop()
              car.rightHeadLED = 1
              car.move(3, carSpeed_fast)  # turn right
              time.sleep(0.2)
              car.stop()
              car.rightHeadLED = 0
          elif rt ==1 :   # right sensor above back-line only
              car.stop()
              car.leftHeadLED = 1
              car.move(2, carSpeed_fast)  # turn left
              time.sleep(0.2)
              car.stop()
              car.leftHeadLED = 0
          else: 
              car.move(0, carSpeed_slow)  # forward
              time.sleep(0.02)
      pass


将示例程序保存到BlueFi的/CIRCUITPY/code.py文件中，并将BlueFi插入到RunGo小车，打开RunGo小车的电源，
等待我们的程序正式开始运行后，按下BlueFi的A按钮，并将整个小车放在黑色胶带围成的封闭图案内，
你将看到RunGo小车始终在围栏内行驶。当你想要让RunGo小车停下时，请按下BlueFi的B按钮即可，或者直接关闭电源。

为什么RunGo小车不会越过黑色胶带围成的“围栏边界”呢？我们使用RunGo小车底部的一对循迹传感器来侦测小车是否到达
“围栏边界”，如果遇到边界则根据这对传感器的状态来调整行驶方向：如果两个传感器都侦测到黑色边界，则先后退一段距离
再左转；如果只有左侧传感器侦测到黑色边界则右转；如果右侧传感器侦测到黑色边界则左转；如果传感器都未侦测到黑色边界
则继续前进。

这是本示例程序的无穷循环程序块中的关键逻辑，或者说这就是实现“电子围栏”效果的关键逻辑。本示例中增加2个按钮做交互
实现开始行驶和停止行驶的功能，也属于无穷循环程序块的一部分逻辑。

为了达到更好的视觉效果，我们可以使用RunGo小车底盘的3颗彩灯来指示行驶、停车状态：在围栏内行驶期间3颗彩灯的颜色不断地转动；
当停车时彩灯颜色全部保持白色。

你可以根据本向导底部的接口库介绍来掌握RunGo小车的控制接口，然后设计更加有趣的示例。

-------------------------------

循迹小车
-------------------------------

AGV(Automatic Guided Vehicle，无人搬运车)小车已经是很多现代车间里最重要的物料“搬运工”！沿着预先规划好的
路线能够无人且自动驾驶的货车能够将仓库的物料自动地运送到指定工位，并从指定工位将产品自动运送会成品仓库。这些曾
经依靠人力或依靠司机开着货车来完成的工作，现在逐步被AGV代替。

AGV如何实现“沿着规定路线行驶到指定停靠点”呢？有很多种方法可以实现AGV的功能，本向导给出一种循迹的方法。使用循迹
传感器反馈的状态信号控制RunGo小车运动来模拟AGV。

我们采用地面贴黑色胶带或黑色不干胶来“指定路线”，编程控制RunGo小车沿着该路线行驶(允许弯曲的路线)，到达路线末端后自动
调头并原路返回。为了更好地理解循迹的程序逻辑，我们先分析下图的三种情况：

.. image::  ../../_static/images/peripheral/rungo_tracking1.jpg
  :scale: 40%
  :align: center

根据上图所示，狠容易回答以下问题：如果小车向右偏离路线我们应该如何纠偏呢？向左偏离时又如何纠偏呢？
此外，当我们达到道路末端时循迹传感器的状态是怎样？如何让RunGo小车绕自身中心调头呢？

.. image::  ../../_static/images/peripheral/rungo_tracking2.jpg
  :scale: 40%
  :align: center

简单地分析这几种特殊情况将有助于掌握下面的示例程序中的关键逻辑和代码。本示例的准备工作非常简单，
使用前示例所用的黑色胶带围成的封闭边界作为本次循迹的“指定路线”。

循迹小车的示例程序如下：

.. code-block::  python
  :linenos:

  import time
  import random
  from hiibot_bluefi.basedio import Button, NeoPixel
  from hiibot_rungo import RunGo
  car = RunGo()
  button = Button()
  pixels = NeoPixel()

  #  stop car one second
  car.stop()
  carspeed = 50
  start = True
  colors = [(255,0,0), (0,255,0), (0,0,255)] # R, G, B
  st = time.monotonic()

  def roundColors():
      global st
      if (time.monotonic() - st) < (0.2 if start==True else 0.6):
          return
      st = time.monotonic()
      t=colors[0]
      colors[0] = colors[1]
      colors[1] = colors[2]
      colors[2] = t
      for ci in range(3):
          car.pixels[ci] = colors[ci]
      car.pixels.show()

  def  searchBackLine():
      global car
      car.stop()
      rdir = random.randint(0, 2)
      if rdir==0:
          car.move(2, carspeed)
      else:
          car.move(3, carspeed)
      for steps in range(360):
          time.sleep(0.005)
          if car.rightTracker or car.leftTracker:
              car.stop()
              return True
      car.stop()
      return False

  def start_or_stop():
      global start
      button.Update()
      if start and button.B_wasPressed:
          print("stop")
          start = False
          return 2  # stop
      if not start and button.A_wasPressed:
          print("start")
          start = True
          return 1  # start
      return 0      # hold the current status
      

  while True:
      start_or_stop()
      sbl = searchBackLine()
      if sbl and start:
          print("We Run Go!")
          while True:
              start_or_stop()
              if not start:
                  car.stop()
                  time.sleep(0.1)
                  continue
              # two sensors is above backline, go on
              if car.rightTracker and car.leftTracker:
                  car.motor(carspeed, carspeed)
              # left sensor is above backline, but right sensor missed backline, thus turn left
              elif car.leftTracker:
                  car.motor(carspeed//2, carspeed)
              # right sensor is above backline, but left sensor missed backline, thus turn right
              elif car.rightTracker:
                  car.motor(carspeed, carspeed//2)
              # two sensors missed backline, thus stop car and search backline
              else:
                  car.stop()
                  print("black line is missing, need to search the black line")
                  break
              time.sleep(0.01)
              roundColors()
      else:
          print("failed to search backline")
          while True:
              pass

看起来程序代码很长！为了帮助你理解程序语句的作用，请分析下面的流程图，并对照程序代码、执行程序时RunGo小车的行为。

.. image::  ../../_static/images/peripheral/rungo_tracking_flowchart.jpg
  :scale: 40%
  :align: center

将上面的示例程序保存到BlueFi的/CIRCUITPY/code.py文件中，并将BlueFi插入到RunGo小车，并打开RunGo小车的电源，
然后将RunGo小车放在黑色胶带上方，等待我们的程序正式开始运行后，观察程序的执行效果。
如果你想要让RunGo小车停下来，按下B按钮即可。如果想要RunGo小车继续巡线行驶，按下A按钮即可。

虽然本示例程序看起来很长，我们增加的彩光效果和按钮控制开启/停车等逻辑占用将近一半的代码，真正的循迹控制逻辑只是在嵌套循环的内循环体中。

此外，本示例程序中包含一个容错处理，被定义成子程序searchBackLine。该子程序可以实现：
当RunGo小车的两个循迹传感器都未检测到“指定路线”的黑色道路时，小车将自动开始绕自身中心旋转，找到黑色道路后再继续沿路行驶。
如果你未将小车放在黑色道路上方，该容错程序将控制RunGo小车原地打转几圈来尝试找黑色道路，如果尝试几圈都未找到黑线则自动停车。

你也可以试一试如下图所示的“指定路线”，你能预测自己的RunGo小车会如何行驶？

.. image::  ../../_static/images/peripheral/rungo_tracking3.jpg
  :scale: 40%
  :align: center

事实上，企业车间的仓库分为原料仓库、半成品仓库、成品仓库等多种，生产工位较多，如何实现多点物料搬运？需要我们去探索，
下图是多点物料搬运问题的抽象图例，你可以使用黑色胶带或不干胶绘制这些图中的“指定路线”，编程实现沿着这些“指定路线”自动搬运物料的小车。

.. image::  ../../_static/images/peripheral/rungo_tracking4.jpg
  :scale: 40%
  :align: center

或许你觉得单纯使用巡线传感器的信息并不足以实现自己的想法，RunGo小车底盘带有颜色识别传感器，可以用来识别地面的颜色，
如果我们在道路的分叉口的地面贴上一些特殊颜色，譬如红、黄、绿、青、蓝和紫色等，每种颜色代表不同的旋转方向，
或许实现上图的多点之间货物运输会变得非常简单。动手试一试吧。

-------------------------------

AGV避障
--------------------------------

如果AGV行驶过程中遇到障碍物怎么办？譬如有人正好站在AGV行驶路线上，此时AGV绝对不能直接撞上去。问题是，AGV
如何知道前方有人？RunGo小车带有一个超声波传感器，能够检测2公分到4米距离内的障碍物。下面我们修改前一个示例实现
这一功能：当RunGo小车的行驶方向有障碍物时，让RunGo小车自动停下来，直到障碍物被移除。


--------------------------------

让RunGo配合你扮演“气功大师”
--------------------------------

武林高手能隔山打牛、隔空取物，气功大师能用气击倒对手。本示例的执行效果：让RunGo当个“托儿”帮助我们表演气功大师的绝招。
气功大师不仅能用手掌“发气”隔空推动RanGo小车后退，还能用手掌隔空“吸引”RunGo小车，其中的奥秘是什么呢？

请注意，本示例程序需要使用超声波传感器，请将超声波传感器模块正确地插在RunGo小车上。

本示例程序的代码如下：

.. code-block::  python
  :linenos:

  import time
  from hiibot_bluefi.basedio import Button, NeoPixel
  from hiibot_rungo import RunGo
  car = RunGo()
  button = Button()
  pixels = NeoPixel()
  start = False
  while True:
      button.Update()
      if button.A_wasPressed:
          start = True
          pixels.fillPixels( (255,255,0) )
      if button.B_wasPressed:
          start = False
          pixels.fillPixels( (0,0,128) )
      d = car.distance  # cm
      if start:
          if d<2 or d>400:
              car.stop()
              print("too close, or too far!")
          elif d >= 12:
              s = d-10
              s = s/60
              cs = s*255
              cs = min(40, cs)
              cs = max(cs, 220)
              cs = 255-cs 
              car.move(0, cs)
          elif d <= 8:
              car.move(1, 40)
          else:
              car.stop()
          time.sleep(0.005)

将示例程序保存到BlueFi的/CIRCUITPY/code.py文件中，并将BlueFi插入到RunGo小车，打开RunGo小车的电源，
等待我们的程序正式开始运行后，按下BlueFi的A按钮，然后用手掌靠近或远离RunGo小车的超声波，
观察程序的执行效果是否有“武林高手”、“气功大师”隔空推车、隔空取物等效果。

--------------------------------

帮助RunGo走出“巨石阵”
--------------------------------

三国演义中诸葛亮在长江边摆的“巨石阵”让诸多敌人有进无出。你能使用今天的科技手段帮助RunGo走出纸杯模拟的“诸葛巨石阵”吗？


--------------------------------

RunGo闻声而动
--------------------------------

灭霸想打个响指想让宇宙半数生命灰飞烟灭，我想打个响指让RunGo前进几步。BlueFi的声音传感器能“听”到响指声，进而
控制RunGo小车前进，我们如此可以实现用声音交互的智能小车。随着你的深入学习，或许有一天你还可以编程实现：对着RunGo
喊“走你”，小车就开始前进，再喊“停”，小车就自动停车。虽然BlueFi支持机器学习能够对你说的短语进行分类，并实现
相应的动作，本向导仅仅实现侦测到“很大的声音”并作出相应的简单逻辑。


--------------------------------

RunGo的“趋光性”
--------------------------------

RunGo的前部带有一对光线强度传感器能够识别前方光线的方向(哪个方向的光线更亮)





------------------------------------

.. Important::
  **RunGo类的小车底盘接口**

    - car (自定义的RunGo类实例化对象)：

      - car = RunGo() # "car"可以自定义为其他名称


    - pixels (底盘像素彩灯子类，默认3颗RGB(兼容WS2812B)/50%亮度/GRB模式)支持的接口方法和属性包括：

      - car.pixels.fill( (R,G,B) ): 填充全部像素为设定颜色
      - car.pixels.show(): 刷新全部像素
      - car.pixels.brightness: 全部像素的亮度属性值(可读可写的)，属性值范围：0.0(灭)~1.0(最亮)
      - car.pixels[index]: 指定某个像素的颜色属性(可读可写的), index有效值范围：0~2；属性值为(R, G, B)


    - groundColor (地面颜色传感器)支持的接口方法和属性包括：

      - car.groundColorID: 地面颜色ID属性值(只读的)，地面颜色ID属性值有效范围：0~6
      - car.groundColorValue: 地面颜色的RGB值属性(只读的)，该属性值为“元组型”颜色分量值：(R, G, B)
      - car.groundColor: 地面颜色的名称属性(只读的)，地面颜色的名称有效值为：'white' (ID=0), 'Red', 'Yellow', 'Green', 'Cyan','Blue','Purple' (ID=6)


    - LightSensor (小车前部光线强度传感器)支持的接口方法和属性包括：

      - car.rightLightSensor: 右前部光线强度的属性值(只读的)，该属性值有效范围：0~1023
      - car.leftLightSensor: 右前部光线强度的属性值(只读的)，该属性值有效范围：0~1023


    - HeadLED (小车(黄色)前灯)支持的接口方法和属性包括：

      - car.rightHeadLED: 右(黄色)前灯的属性值(可读可写的)，该属性值有效范围：1或0, True或False；1或True: On, 0或False: Off
      - car.leftHeadLED: 左(黄色)前灯的属性值(可读可写的)，该属性值有效范围：1或0, True或False；1或True: On, 0或False: Off


    - distance (超声波测距传感器)获取的小车与障碍物之间的距离属性值(只读的)，有效值范围：2~400,量纲为cm(厘米)


    - Tracker (小车底盘的巡线传感器)支持的接口方法和属性包括：

      - car.rightTracker: 右前部巡线传感器的状态属性值(只读的)，该属性值有效范围：1或0, True或False；1或True: 黑线, 0或False: 非黑线
      - car.leftTracker: 左前部巡线传感器的状态属性值(只读的)，该属性值有效范围：1或0, True或False；1或True: 黑线, 0或False: 非黑线
      - car.tracking(mode): 巡线传感器对儿的状态属性值(只读的)，该属性值有效范围：1或0, True或False；1或True: 小车在线上, 0或False: 小车偏离线；mode有效值：0:使用较宽(线宽大于两个巡线传感器的间距[1cm])的黑色线，左右巡线传感器同时在黑线上；1:使用较窄(线宽小于两个巡线传感器的间距[1cm])的黑色线，仅左巡线传感器在黑线上；2:使用较窄(线宽小于两个巡线传感器的间距[1cm])的黑色线，仅右巡线传感器在黑线上；3:使用较宽(线宽大于两个巡线传感器的间距[1cm])的白色线，左右巡线传感器同时在白线上


    - motor (两个直流马达)支持的接口方法和属性包括：

      - car.stop(): 停止两个马达
      - car.motor(l_speed, r_speed): 设置左右马达的转速，转速的有效值为：-255~-1(反转的转速), 0(停转), 1~255(正转的转速)
      - car.move(dir, speed): 设置小车的移动方向和速度，方向dir的有效值为：0(向前)、1(向后)、2(向左)、3(向右)；速度有效值为：0(停转), 1~255
      - car.moveTime(dir, speed, mt): 设置小车的移动方向、速度和时间，方向dir的有效值为：0(向前)、1(向后)、2(向左)、3(向右)；速度有效值为：0(停转), 1~255；时间单位为秒(second)。注意，使用该接口控制小车移动时，在运动结束时会自动停止
