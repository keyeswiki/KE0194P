
### 项目十五 多功能桌面小车

**项目介绍：**

在前面课程中，我们只是让智能车实现单个功能，那我们能不能把所有功能合在一起呢？能，在这一课程中，我们利用一个代码测试智能车，智能车包含前面课程中讲到的所有功能，我们利用手机蓝牙APP上按钮自动切换各种功能,简单方便。

**项目组件：**

| 组装好的智能车(<span style="color: rgb(255, 76, 65);">未插上蓝牙模块</span>) *1 |USB线 *1 |18650电池 *2（电池自备） |
| --- | --- | --- | 
| ![](../media/image4.png) | ![](../media/image8.png)| ![](../media/battery.png)| 
| 蓝牙模块  *1 | 手机/平板 *1|  |
| ![](../media/image38_2.png)|![](../media/WAX.png)| |

**接线图：**

**⚠️特别注意：坦克智能车已经组装好了，这里不需要把传感器模块和其他的都拆下来又重新组装和接线，这里再次提供接线图，是为了方便您编写代码！**

![image203](../media/5db7458e37c2b2966f8a70dc8f6cf658.png)

![image203](../media/5db7458e37c2b2966f8a70dc8f6cf6581.png)

⚠️ <span style="color: rgb(255, 76, 65);">**特别注意：**</span>

- <span style="color: rgb(172, 57, 255);">**上传示例代码前，蓝牙模块可以先不直插到电机驱动扩展板上！因为蓝牙模块也占用Arduino的串口通信（TX/RX），如果连接到电机驱动扩展板上，示例代码上传会失败。示例代码上传成功后，再插回蓝牙模块。**</span>

**测试代码：**

（**特别提醒：在上传程序代码前，需要把蓝牙模块取下，否则代码会上传失败。需要上传代码成功后，再连接蓝牙模块。**）

``` c
/*
  迷你履带坦克机器人
  课程 15
  蓝牙控制多功能智能坦克车
  http://www.keyes-robot.com
*/

#include <Servo.h>
Servo myservo;  // create servo object to control a servo
//数组，用于储存图案的数据，可以自己算也可以从取摸工具中得到
unsigned char start01[] = {0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x80, 0x40, 0x20, 0x10, 0x08, 0x04, 0x02, 0x01};
unsigned char front[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x24, 0x12, 0x09, 0x12, 0x24, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
unsigned char back01[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x24, 0x48, 0x90, 0x48, 0x24, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
unsigned char left[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x44, 0x28, 0x10, 0x44, 0x28, 0x10, 0x44, 0x28, 0x10, 0x00};
unsigned char right[] = {0x00, 0x10, 0x28, 0x44, 0x10, 0x28, 0x44, 0x10, 0x28, 0x44, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
unsigned char STOP01[] = {0x2E, 0x2A, 0x3A, 0x00, 0x02, 0x3E, 0x02, 0x00, 0x3E, 0x22, 0x3E, 0x00, 0x3E, 0x0A, 0x0E, 0x00};
unsigned char speed_a[] = {0x00, 0x40, 0x20, 0x10, 0x08, 0x04, 0x02, 0xff, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x00, 0x00};
unsigned char speed_d[] = {0x00, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0xff, 0x40, 0x20, 0x10, 0x08, 0x04, 0x02, 0x00, 0x00};
unsigned char clear[] = {0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00};
#define SCL_Pin  A5  //设置时钟引脚为 A5
#define SDA_Pin  A4  //设置数据引脚为 A4
int MA = 2; //定义电机A方向控制引脚为D2
int PWMA = 6; //定义电机A速度控制引脚为D6
int MB = 4; //定义电机A方向控制引脚为D4
int PWMB = 5; //定义电机A速度控制引脚为D5
int speeds = 150; //初始化速度为150
char blue_val;
int trigPin = 12; //TRIG引脚接D12
int echoPin = 13; //ECHO引脚接D13
int distance, distance_l, distance_r;
int left_light = 0; //定义左边传感器的变量
int right_light = 0; //定义右边传感器的变量

void setup() {
  Serial.begin(9600);  //设置波特率为9600
  myservo.attach(10);  // attaches the servo on pin 10 to the servo object
  myservo.write(90);  //舵机角度为90
  delay(500);
  pinMode(MA, OUTPUT); //配置电机引脚为输出模式
  pinMode(PWMA, OUTPUT);
  pinMode(MB, OUTPUT);
  pinMode(PWMB, OUTPUT);
  pinMode(trigPin, OUTPUT); //定义TRIG为输出模式
  pinMode(echoPin, INPUT); //定义ECHO为输入模式
  //设置引脚为输出
  pinMode(SCL_Pin, OUTPUT);
  pinMode(SDA_Pin, OUTPUT);
  //清屏
  matrix_display(clear);
  matrix_display(start01);
}
void loop() {
  if (Serial.available() > 0) { //接收到蓝牙信号
    blue_val = Serial.read(); //接收到的信号赋给blue_val
    Serial.println(blue_val);  //串口监视器显示蓝牙信号
    switch (blue_val) {
      case  'F':  advance();   break;  //接收到‘F’前进
      case  'B':  back();      break;  //接收到‘B’后退
      case  'L':  turnL();     break;  //接收到‘L’左旋
      case  'R':  turnR();     break;  //接收到‘R’右旋
      case  'S':  stopp();     break;  //接收到‘S’电机停止转动，功放停止
      case  'Y':  follow();    break;  //接收到‘U’，进入跟随模式
      case  'U':  avoid();     break;  //接收到‘Y’，进入避障模式
      case  'X':  light_follow();   break;  //接收到‘X’，寻光模式
    }
  }

}

void advance() { //小车前进
  matrix_display(front);  //显示向前的图案
  digitalWrite(MA, HIGH); //电机A逆时针转
  analogWrite(PWMA, speeds); //电机A速度为speeds
  digitalWrite(MB, HIGH); //电机B顺时针转
  analogWrite(PWMB, speeds); //电机B速度为speeds
}

void back() { //小车后退
  matrix_display(back01);  //显示后退的图案
  digitalWrite(MA, LOW); //电机A顺时针转
  analogWrite(PWMA, speeds); //电机A速度为speeds
  digitalWrite(MB, LOW); //电机B逆时针转
  analogWrite(PWMB, speeds); //电机B速度为speeds
}

void turnL() { //小车左转
  matrix_display(left);  //显示左转的图案
  digitalWrite(MA, LOW); //电机A顺时针转
  analogWrite(PWMA, speeds); //电机A速度为speeds
  digitalWrite(MB, HIGH); //电机B顺时针转
  analogWrite(PWMB, speeds); //电机B速度为speeds
}

void turnR() { //小车右转
  matrix_display(right);  //显示右转的图案
  digitalWrite(MA, HIGH); //电机A逆时针转
  analogWrite(PWMA, speeds); //电机A速度为speeds
  digitalWrite(MB, LOW); //电机B逆时针转
  analogWrite(PWMB, speeds); //电机B速度为speeds
}

void stopp() { //小车停止
  matrix_display(STOP01);  //显示停止的图案
  analogWrite(PWMA, 0); //电机A速度为0
  analogWrite(PWMB, 0); //电机B速度为0
}

int get_distance() {
  int distance = 0;
  digitalWrite(trigPin, LOW);     // 通过Trig/Pin 发送脉冲，触发 HC-SR04 测距，使发出发出超声波信号接口低电平2μs
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);    // 使发出发出超声波信号接口高电平10μs，这里是至少10μs
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);     // 保持发出超声波信号接口低电平
  distance = pulseIn(echoPin, HIGH) / 58; // 读出脉冲时间,将脉冲时间转化为距离（单位：厘米）
  delay(20);
  //  Serial.println(distance);        //输出距离值
  return distance;
}

void follow() {
  int follow_flag = 1;
  while (follow_flag) {
    distance = get_distance(); //调用测距函数
    if (distance < 8 ) {//如果距离小于8
      back();//后退
    }
    else if (distance >= 8 && distance < 13) { //如果距离大于等于8，小于13
      stopp();//停止
    }
    else if (distance >= 13 && distance <= 35 ) { //如果距离大于等于13，小于35
      advance();//跟随
    }
    else {//如果以上都不是
      stopp();//停止
    }
    blue_val = Serial.read();
    if (blue_val == 'S') { //接收到‘S’退出循环，小车停止
      follow_flag = 0;
      stopp();
    }
  }
}

void avoid() {
  int avoid_flag = 1;
  while (avoid_flag) {
    distance = get_distance(); //调用测距函数

    if (distance > 0 && distance < 20) { //如果距离小于20且大于0
      stopp();//停止
      delay(100);
      myservo.write(180); //舵机转到180度
      delay(500);
      distance_l = get_distance(); //获取左边的距离
      delay(100);
      myservo.write(0); //舵机转到0度
      delay(500);
      distance_r = get_distance(); //获取右边的距离
      delay(100);
      if (distance_l > distance_r) { //比较距离，如果左边大于右边
        turnL();  //向左转
        delay(500);
        myservo.write(90);//舵机回到90度
        delay(100);
        matrix_display(front);   //点阵显示前进图案

      }
      else { //否则如果右边大于左边
        turnR();//向右转
        delay(500);
        myservo.write(90);//舵机回到90度
        delay(100);
        matrix_display(front);   //显示前进图案
      }

    }
    else { //前方距离小于等于10cm时
      advance();//前进

    }
    blue_val = Serial.read();
    if (blue_val == 'S') { //接收到‘S’退出循环，小车停止
      avoid_flag = 0;
      stopp();
    }
  }
}

void light_follow() {
  int light_flag = 1;
  while (light_flag) {
    left_light = analogRead(A1); //左边光敏传感器接A1
    right_light = analogRead(A2); //右边光敏传感器接A2
    if (left_light > 650 && right_light > 650) { //左右超过650
      advance();   //前进
    }
    else if (left_light > 650 && right_light <= 650) {
      turnL(); //左转
    }
    else if (left_light <= 650 && right_light > 650) {
      turnR(); //右转
    }
    else if (left_light <= 650 && right_light <= 650) {
      stopp();//停止
    }
    blue_val = Serial.read();
    if (blue_val == 'S') { //接收到‘S’退出循环，小车停止
      light_flag = 0;
      stopp();
    }
  }
}

//这个函数用于点阵屏显示
void matrix_display(unsigned char matrix_value[])
{
  IIC_start();  //调用数据传输开始条件的函数
  IIC_send(0xc0);  //选择地址
  for (int i = 0; i < 16; i++) //图案数据有16个字节
  {
    IIC_send(matrix_value[i]); //传输图案的数据
  }
  IIC_end();   //结束图案数据传输
  IIC_start();
  IIC_send(0x8A);  //显示控制，选择脉宽为4/16
  IIC_end();
}
//传输数据开始的条件
void IIC_start()
{
  digitalWrite(SCL_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, LOW);
  delayMicroseconds(3);
}
//传输数据
void IIC_send(unsigned char send_data)
{
  for (char i = 0; i < 8; i++) //每个字节有8位
  {
    digitalWrite(SCL_Pin, LOW); //将时钟引脚SCL_Pin拉低，才可以改变SDA的信号
    delayMicroseconds(3);
    if (send_data & 0x01) //根据字节的每一位是1还是0来设置SDA_Pin的高低电平
    {
      digitalWrite(SDA_Pin, HIGH);
    }
    else
    {
      digitalWrite(SDA_Pin, LOW);
    }
    delayMicroseconds(3);
    digitalWrite(SCL_Pin, HIGH); //将时钟引脚SCL_Pin拉高，停止数据的传输
    delayMicroseconds(3);
    send_data = send_data >> 1;  //一位一位的检测，所以将数据右移一位
  }
}
//数据传输结束的标志
void IIC_end()
{
  digitalWrite(SCL_Pin, LOW);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, LOW);
  delayMicroseconds(3);
  digitalWrite(SCL_Pin, HIGH);
  delayMicroseconds(3);
  digitalWrite(SDA_Pin, HIGH);
  delayMicroseconds(3);
}
```

**测试结果：**

外接电源，将电机驱动扩展板上的拨码开关拨至ON端。选择好正确的开发板板型和适当的串口端口（COMxx），上传代码。手机APP连接蓝牙，蓝牙连接成功后，我们可以通过按下对应按钮实现对应功能，通过停止钮来停止功能。

| 按钮:![image180](../media/1233714e0234b6245cedbc3eff07864d.png) |                            | 功能：配对连接HM-10蓝牙模块                                                               |
|--------------------------------------------------------------|----------------------------|-------------------------------------------------------------------------------------------|
| 按钮:![image181](../media/35b811ab85a240ba2eabeffd8379b337.png) |                            | 功能：进入蓝牙控制界面                                                                    |
| 按钮:![image182](../media/38f4b9a388eb3633c4b0f5c04545c130.png) |                            | 功能：断开蓝牙连接                                                                        |
| 按钮:![image183](../media/720164d43dbe50d388f5887eafec078b.png) | 控制字符：按下：F；松开：S | 功能：按下，小车前进；松开就停止                                                          |
| 按钮:![image184](../media/dd17f04276c6ec578ce69bdd6b709ab0.png) | 控制字符：按下：B；松开：S | 功能：按下，小车后退；松开就停止                                                          |
| 按钮:![image185](../media/5f178f0b7c951228333fdc5ed4917ed2.png) | 控制字符：按下：L；松开：S | 功能：按下，小车左旋转；松开就停止                                                        |
| 按钮:![image186](../media/e311a61ee103c36b8c79121db9de6fb2.png) | 控制字符：按下：R；松开：S | 功能：按下，小车右旋转；松开就停止                                                        |
| 按钮:![image187](../media/558a091d1dff040217a8c8ec8e3b7cdb.png) | 控制字符： 点击发送：S     | 功能：小车停止，停止所有功能                                                              |
| 按钮:![image188](../media/1cde47833aec02a65075b02c136d3d4a.png) | 控制字符：                 | 功能：点击一下开启手机方向感应控制，再点击一下退出方向感应控制                            |
| 按钮:![image189](../media/aabdabfe88c7ab40944f1b59f2b902c6.png) | 控制字符： 点击发送：U     | 功能：开启避障功能，点击![image190](../media/cd86ca672f19353b4ce9a1720895c2c2.png)退出       |
| 按钮:![image191](../media/23183b80cb7a19fa78cc085299430ff7.png) | 控制字符： 点击发送：X     | 功能：开启寻光功能，点击![image192](../media/cd86ca672f19353b4ce9a1720895c2c2.png)退出       |
| 按钮:![image193](../media/418535b9bdcc25b1fb706b90205869b2.png) | 控制字符： 点击发送：Y     | 功能：开启超声波跟随功能，点击![image194](../media/cd86ca672f19353b4ce9a1720895c2c2.png)退出 |