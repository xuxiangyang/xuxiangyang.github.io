+++
title = "如何用RaspberryPI控制Arduino"
author = ["徐向阳"]
date = 2021-08-14T14:39:00+08:00
tags = ["diy", "raspberrypi", "arduino"]
categories = ["diy"]
draft = false
+++

## 方案整体思路 {#方案整体思路}

我希望能够通过Arduino来接收感应器和控制舵机等，RaspberryPI来控制网络业务等复杂交互逻辑。Arduino就像一个超级设备，对上层业务隐藏硬件控制细节。所以 Arduino会通过USB链接到RaspberryPI，由RaspberryPI负责Arduino供电。RaspberryPI部分使用Python来做业务代码，通过Serial于Arduino交互。


## 环境安装 {#环境安装}

1.  安装Arduino IDE环境

    ```bash
    sudo apt-get install arduino
    ```
2.  允许 `pi` 访问USB和串口，这里假设你的账户名称为 `pi`

    ```bash
    sudo usermod -a -G dialout pi
    ```
3.  使用Python3环境安装pySerial库

    ```bash
    python3 -m pip install pyserial
    ```


## 从Arduino读取数据 {#从arduino读取数据}

我们需要编写2个设备上的代码

1.  编写Arduino代码

    ```arduino
    void setup() {
      Serial.begin(9600);
    }
    void loop() {
      Serial.println("Hello");
      delay(1000);
    }
    ```
2.  点击Arduino IDE UpLoad，通过 Serial Monitor 应该可以看到 `Hello` 的输出
3.  编写Python代码

    ```python
    import serial
    if __name__ == '__main__':
        ser = serial.Serial('/dev/ttyUSB0', 9600, timeout=1)
        ser.flush()
        while True:
            if ser.in_waiting > 0:
                line = ser.readline().decode('utf-8').rstrip()
                print(line)
    ```

    运行代码，注意这里要关闭上一步开启的 `Serial Monitor` 。就可以看到Hello的输出了


## 控制Arduino LED 闪烁 {#控制arduino-led-闪烁}

1.  编写Arduino代码后Upload

    ```arduino
    #define LED_PIN 13

    void setup() {
      Serial.begin(9600);
      pinMode(LED_PIN, OUTPUT);
      digitalWrite(LED_PIN, LOW);
    }

    void loop() {
      if(Serial.available() == 0) return;

      String cmd = Serial.readStringUntil('\n');

      if(cmd == "on") {
        digitalWrite(LED_PIN, HIGH);
      } else if(cmd == "off") {
        digitalWrite(LED_PIN, LOW);
      }
    }
    ```
2.  编写Python代码

    ```python
    import serial
    import time
    if __name__ == '__main__':
        ser = serial.Serial('/dev/ttyUSB0', 9600, timeout=1)
        ser.flush()
        while True:
            ser.write(b"on\n")
            time.sleep(1)
            ser.write(b"off\n")
            time.sleep(1)
    ```

    运行代码，此时Arduino板子上的LED应该就开始闪烁了，如果你的Arduino板子上没有内置LED，那么接一个LED在13 pin口上即可。


## 控制的同时从Arduino读取数据 {#控制的同时从arduino读取数据}

1.  编写Arduino代码后Upload

    ```arduino
    #define LED_PIN 13

    void setup() {
      Serial.begin(9600);
      pinMode(LED_PIN, OUTPUT);
      digitalWrite(LED_PIN, LOW);
    }

    void loop() {
      if(Serial.available() == 0) return;

      String cmd = Serial.readStringUntil('\n');

      if(cmd == "on") {
        digitalWrite(LED_PIN, HIGH);
        Serial.println("led on");
      } else if(cmd == "off") {
        digitalWrite(LED_PIN, LOW);
        Serial.println("led off");
      }
    }
    ```
2.  编写Python代码

    ```python
    import serial
    import time
    if __name__ == '__main__':
        ser = serial.Serial('/dev/ttyUSB0', 9600, timeout=1)
        ser.flush()
        while True:
            ser.write(b"on\n")
            line = ser.readline().decode('utf-8').rstrip()
            print(line)
            time.sleep(1)
            ser.write(b"off\n")
            line = ser.readline().decode('utf-8').rstrip()
            print(line)
            time.sleep(1)
    ```

    运行代码，此时Arduino板子上的LED应该就开始闪烁了，并且python应该交替的出现 `led on` 和 `led off`


## 为什么不用RaspberryPI直接操作硬件 {#为什么不用raspberrypi直接操作硬件}

1.  我自己手头正好有一个Arduino和一个树莓派。以前接触过Arduino觉得也很简单，用起来也很容易。所以如果你只有一个树莓派，我不认为在没有明确的可衡量的理由时采用这套方案
2.  我的树莓派希望做的事情会更多些，更多的会是家庭的服务器用途，所以我不希望在树莓派上插好多线来充当DIY环节的一部分，我希望树莓派是Arduino功能的补充，比如网络控制等。


## 最后 {#最后}

多折腾折腾