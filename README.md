# FengRu
BUAA FengRu Cup 2023.2
初始化仓库


以下为项目交接内容
1. 在arduino IDE内安装 Arduino_TensorFlowLite库。
   github网址 https://github.com/tensorflow/tflite-micro-arduino-examples

2. 请用arduino IDE 开发版管理器中安装 Arduino Nano 33 BLE

3. 在FengRu.ino文件中，我与实例magic_wand（文件->示例->第三方库 中打开示例文件）的区别为
   
    line 97 调整大小
    ```c++
    constexpr int kTensorArenaSize = 50 * 1024; //初始值为 30*1024  60有第一条失败 80无第一条失败 100失败 向下调整 20失败
    ```

    line 103 104 修改label
    ```c++
    constexpr int label_count = 5;
    const char* labels[label_count] = {"normal", "up", "down", "short", "negative"};
    ```

    line 156 157 158 训练模型时数据都乘以100，故读出的加速度也要*100

    ```c++
    current_acceleration_data[0]=current_acceleration_data[0]*100;//模型数据乘以100
    current_acceleration_data[1]=current_acceleration_data[1]*100;//模型数据乘以100
    current_acceleration_data[2]=current_acceleration_data[2]*100;//模型数据乘以100
    ```
   修改模型文件，magic_wand_model_data.cpp换为我们训练好的模型

4. 计划中我们还需要使用蓝牙模块，将匹配结果发送到手机APP上，具体实现应该不难。
   现阶段只需要解决上面的invoke bug跑通模型。

下面是串口监视器的部分报错信息
```c++
MicroAllocator: Model allocation started before finishing previously allocated model
Failed starting model allocation.

Invoke failed

Invoke() called after initialization failed //位于 micro_interpreter.cpp line 275

Invoke failed
```

google等检索结果为 可能是 kTensorArenaSize 设置不对，内存分配问题，但是我调整无果。