# SECS GEM

SECS GEM HSMS 驱动通过 TCP/IP 协议访问支持 SEMI E37 HSMS 标准的设备，目前支持设备 PASSIVE 模式，驱动作为 Host 主动连接。

## 设备设置

| 字段     | 说明                 |
| -------- | -------------------- |
| host     | 设备 IP 地址         |
| port     | 设备端口号, 默认5000 |
| deviceid | 设备 ID, 默认0       |

## 支持的数据类型

* uint8
* int8
* uint16
* int16
* uint32
* int32
* uint64
* int64
* float
* double
* bool
* string
* bytes


## 地址格式

> SxFy([_1][_2][_3][_4])

## Streams 和 Functions

SECS-II 消息称为流和功能。每条消息都有一个流值（ Sx ）和一个功能值（ Fy ）。对于流1功能1，它写为 S1F1，读为 “S1F1” 。流是消息的类别，而功能是类别中的特定消息。主消息中的功能值始终是奇数，关联的二次答复中的功能值始终比主消息中的功能值大1或偶数。

| Stream | 说明                 |
| ------ | -------------------- |
| 1      | 设备状态             |
| 2      | 设备控制             |
| 3      | 材料状态             |
| 4      | 材料控制             |
| 5      | 报警处理             |
| 6      | 数据收集             |
| 7      | 配方管理             |
| 8      | 控制程序传输         |
| 9      | 系统错误             |
| 10     | 终端服务             |
| 11     | 未启用               |
| 12     | 晶片图形布置         |
| 13     | 未格式化的数据集传输 |


**常用 SxFy**

| Stream Function | 说明                 |
| --------------- | -------------------- |
| S1F1            | Hello消息            |
| S1F2            | 在线数据             |
| S1F3            | 选定设备状态数据     |
| S1F4            | 格式化的设备状态数据 |
| S6F7            | 数据传输请求         |
| S6F8            | 数据传输数据         |
| S6F15           | 事件报告请求         |
| S6F16           | 事件报告数据         |


## 特殊类型处理

SECS-II 消息定义了 LIST 类型，插件也通过 string 支持该类型，但是有一个反序列化的过程，具体的规则如下所示。

| SECS-II 类型 | 序列化          | 说明                                                      |
| ------------ | --------------- | --------------------------------------------------------- |
| LIST         | `<L[n] xxx>(.)` | n 为 LIST 长度，空 LIST n 为 0， 最外层 LIST 最后包括 *.* |
| ASCII        | `<A[n] xxx> `   | n 为 ASCII 长度，空 ASCII n 为 0                          |
| Binary       | `<B[n] xxx> `   | n 为 BInary 长度，空 BInary n 为 0                        |
| Boolean      | `<Boolean x> `  |                                                           |
| UINT8        | `<U1 x> `       |                                                           |
| UINT16       | `<U2 x> `       |                                                           |
| UINT32       | `<U4 x> `       |                                                           |
| UINT64       | `<U8 x> `       |                                                           |
| INT8         | `<I1 x> `       |                                                           |
| INT16        | `<I2 x> `       |                                                           |
| INT32        | `<I4 x> `       |                                                           |
| INT64        | `<I8 x> `       |                                                           |
| 32FLOAT      | `<F4 x> `       |                                                           |
| 64FLOAT      | `<F8 x> `       |                                                           |

LIST 类型支持使用下标访问具体的元素，目前支持嵌套四层 LIST。

## 常见问题
* 设备 ID 如何填写？
设备 ID 一般在设备的 SECS GEM HSMS 协议配置页面可以查看。

* LIST 中 ASCII 类型如何序列化？
本插件 LIST 中 ASCII 序列化为 `<A[n] xxx>`，内容不带引号。例如 "ABC" 序列化为 `<A[3] ABC>`，而不是 `<A[3] "ABC">`。

* LIST 中 Binary 类型如何序列化？
本插件 LIST 中 Binary 序列化为 `<B[n] xx>`，内容不带引号。例如 0xFF 序列化为 `<B[1] FF>`，而不是 `<B[2] "FF">`。

* LIST 类型数据新建点位的时候应该选择什么类型？
设备数据类型为 **LIST** 的点位，在新建的时候选择 **string** 类型。

* 设备主动上报的事件如何新建点位？
对于例如 `S6F11` 等设备主动上报的点位，直接新建相应的点位即可。

* 对于需要传递参数进行数据获取的点位如何新建点位？
例如 `S1F3`，需要携带 **LIST** 类型的参数，其返回值会通过 `S1F4` 返回，类型也为 **LIST**。那么对于这个点位，我们需要新建两个点位，一个地址为 `S1F3`，属性配置 **Write**，类型为 **string**。另一个地址为 `S1F4`，属性配置为 **Read** 或者 **Sub**，类型为 **string**。使用第一个点位写入参数，第二个节点获取返回值。

* 对于不需要传递参数进行数据获取的点位如何新建点位？
例如 `S1F1`，其不需要传递参数，那么只需要新建一个点位即可，其返回值会直接在此点位显示。

