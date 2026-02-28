> 边缘计算网关与ThingsBoard平台的MQTT连接边缘计算网关作为物联网系统中的关键设备，负责实现设备端与云端平台之间的数据交互。在基于MQTT协议的连接方案中，ThingsBoard平台提供了便捷的设备接入方式。
>



##   
一、MQTT协议在物联网中的作用  
MQTT（Message Queuing Telemetry Transport）是一种轻量级的机器对机器(M2M)协议，特别适用于资源受限的环境。它采用 publish/subscribe 模型，支持设备与平台之间的双向通信机制，具有以下特点：
+ 低带宽占用
+ 低延迟
+ 可靠的消息传输
+ 支持多种QoS级别

## 二、边缘计算网关与ThingsBoard的MQTT连接配置
### 网关注册与凭证管理
+ 在ThingsBoard平台创建设备
+ 通过<font style="background-color:#DF2A3F;">MQTT Basic配置获取设备凭证</font>，包括：
    - 客户端ID
    - 用户名
    - 密码
    - 服务器地址

### 网关配置步骤
+ 配置MQTT客户端参数
+ 设置设备标识和认证信息
+ 配置数据上报主题和接收主题

### 数据传输机制
+ 上行数据：边缘计算网关收集设备数据，通过MQTT协议发送到ThingsBoard平台
+ 下行数据：平台指令通过MQTT发送到网关，再转发至目标设备



```python
import json
import time
from common.Logger import logger
from quickfaas.remotebus import publish

def main(message, wizard_api, cloudName):
    # 初始化一个列表，存储所有找到的测点（名称+值）
    point_list = []

    # 遍历所有设备和变量，复刻原代码的嵌套逻辑，收集所有测点
    for device, val_dict in message["values"].items():
        for point_id, val in val_dict.items():
            # 直接提取所有测点的raw_data（原代码仅提取CO2，此处去掉判断）
            point_value = val["raw_data"]
            # 将测点名称和值存入列表，等待发布
            point_list.append({
                "name": point_id,
                "value": point_value
            })

    # 遍历所有测点，按原代码的方式逐个发布（保持单测点发布的一致性）
    for point in point_list:
        # 构造与原代码CO2格式完全相同的遥测数据
        telemetry = {
            point["name"]: point["value"]
        }

        # 发布到ThingsBoard，完全复用原代码的publish参数
        publish(__topic__, json.dumps(telemetry),__qos__, cloud_name=cloudName)
        time.sleep(3)
```





