# 文件解释

magic_wand 正确项目
data_collector 网页数据采集器  
FengRu 使用了错误模型的 arduino 记录  
stm 不同实现方法，未改为 arduino 形式

# tinyML 模型要求

1. 硬件选择 arduino / raspberry pie  
   raspberry 性能更强  
   默认选择 arduino

2. 工具选择 TensorFlow / Keras  
   默认TensorFlow Lite

3. 限制  
   支持的 TensorFlow 操作有限  
   支持的设备有限  
   需要手动管理内存的低阶 C++ API  
   不支持设备端直接训练  

## 输入  
IMU 加速度计/陀螺仪

## 输出
手机蓝牙接受数据（可写APP，目前略）实时提醒健身者调整姿势

## 工作流程
若要在微控制器上部署并运行 TensorFlow 模型，必须执行以下步骤：

+ 训练模型 Python：  
    + 生成小型 TensorFlow 模型，该模型适合目标设备并包含支持的操作  
    + 使用 TensorFlow Lite 转换器转换为 TensorFlow Lite 模型  
    + 使用标准工具转换为 C 语言字节数组，以将其存储在设备上的只读程序内存中  

+ 使用 C++ 库在 Arduino 上进行推断并处理结果。

## 具体硬件代码逻辑

1. define 宏定义环境 BLE 蓝牙 UUID(板子蓝牙连接时的ID)，include库文件  
2. 设定 namespace 相关运行参数  
   + IMU(Inertial Measurement Unit 惯性测量模块) 模块 stroke 二维化预备参数设定（用于后续 ML 模型输入计算）
   + BLE(Bluetooth Low Energy 低功耗蓝牙) 蓝牙通讯模块预备设定
3. SetupIMU()   
   + 初始化 IMU，设定数据采集模式，acceleration 加速度计 & gyroscope 陀螺仪 初始化设定
   + 定义相关函数，用以处理IMU数据。如 读取加速度&陀螺仪数据，运动判定，重力方向估计，速度更新，陀螺仪漂移，运动方向更新，更新stroke(二维化)数据逻辑
4. setup()  
   + IMU、BLE、TF模型具体初始化启动
   + TF 模型 version 检查
   + resolver 解析器初始化。只调用4个，防止后续占用内存过大
   + interpreter 解释器初始化，包含配置 model resolver tensor_arena(模型运行内存分配)
   + 将模型输入入口对准 interpreter 。此时还会检查模型输入 tensor(张量) 的 shape(形状，也就是张量的维度和格式信息)
   + 将模型输出入口对准 interpreter 。类似的，检查输出形状， n 个 kTfLiteInt8 整数(取决于要识别的动作数量，100为上限的识别率)
5. loop()  
   + 检测 IMU 读取的数据有效性并读取数据
   + 使用 UpdateStroke 函数判断动作是否做完了。 true(eDone) 则开启下列流程，否则跳回上一步读取继续读数据 (使用 enum 枚举了状态 eWaiting = 0, eDrawing = 1, eDone = 2, )
   + 将 Buffer 缓冲区内的数据放入模型 input 中
   + 从 TF model 中取出 output ，串口 print 输出最高识别率，并传输通过 BLE 将识别数据传输到手机端(可实现自定义APP，目前为简单用 nRf connect 接受蓝牙数据) 