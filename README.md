### 功能说明

Home assistant中配合HassAPP的位置上报插件。

Hass APP中通过该插件可以上报经纬度、手机电量、是否充电、当前是否在使用、当前前台APP、是否连接WIFI以及WIFI名称等信息（部分敏感信息可以开关）。

可以通过服务调用查询指定device_tracker的当前位置并通过tts组件播放出来。

### 部署配置

* 将ihasslogger文件夹复制到home assistant配置目录下的custom_components中。
* 修改配置文件configuration.yaml，增加配置：

``` yaml
device_tracker: 
  - platform: ihasslogger
    password: 'password'
    baidu_lbsak: 'baidu lbsak'
    tts_domain: 'hello_miai'
    tts_service: 'force_play'
    sensors:
      gogogo:
        value_template: >-
          {%- if state_attr('device_tracker.gogogo', 'interactive') == 'true' -%}
            home
          {%- else -%}
            not_home
          {%- endif -%} 
```

其中参数意义：

* password：需要在APP上配置访问该组件的密匙，避免别人盗用伪造数据；
* baidu_lbsak：百度LBS的AK，初次使用到https://lbsyun.baidu.com/去申请一个，“应用类型”选择“浏览器端“， ”在“启用服务”中勾选“地点检索”应该就可以了。
* tts_domain、tts_service：如果需要通过脚本触发播放家庭成员的位置，则需要配置tts的domain和service那么，需要注意的是在调用tts的时候，将固定通过message传递需要播放的内容（配合hello_miai没毛病），如果不是则手动修改一下插件
* sensors：可选，如果最终要显示device_tracker的state不是home not_home以及匹配到的zone的话，可以在这儿增加指定成员的state的取值模板，上述例子中gogogo就是显示当前是否在使用手机，如果在使用则是home，否则为not _home

##### 添加device

如果家里没有其他的device_tracker源，比如router之类的来创建device，可以通过以下途径解决：

* 编辑known_devices.yaml，如果没有就创建他；
* 添加一段以下内容：

``` yaml
androidecd09ff07bb4:
  mac: EC:D0:9F:F0:7B:B4
  name: DeviceName
  track: true
```

其中的mac可以随便填，这个是router模式的device要用的，我们用不到，只是是这种格式就行了（用冒号分割的6个两位16进制数字），如果需要多个设备就添加多个，上边第一行的名字不能重复。

重启home assistant后，应该就能在APP中选择了。

### 位置播报

#### 服务

device_tracker.report_address

#### 参数

* entity_id：需要播报device的ID
* friendly_name：播报时的称谓

比如调用服务参数为：

``` yaml
entity_id: device_tracker.7c49eb713ddb
friendly_name: 牛大傻
```

则播报的内容则是：牛大傻当前位于四川省成都市XXXXX街道的XXXX大厦

### 自动化

可以通过小爱同学间接触发播报（比如通过小爱同学控制万能遥控发送一个特定指令，然后利用esp的红外接收收到指定指令后播报位置），场景：

你：牛大傻在哪儿？
小爱：发送一条红外指令
ESP：收到红外指令，控制home assistant
HASS：查询牛大傻的位置，然后通过百度LBS逆地址解析到位于四川省成都市XXXXX街道的XXXX大厦，利用配置的tts控制小爱播放
小爱：牛大傻当前位于四川省成都市XXXXX街道的XXXX大厦

脚本例子：

``` yaml
- id: '15701750265114554'
  alias: xiaoai_dashazi_location
  trigger:
  - payload: 8415:36975
    topic: /esp1/sensor/ir/nec
    platform: mqtt
  condition: []
  action:
  - data:
      entity_id: device_tracker.7c49eb713ddb
      friendly_name: 牛大傻
    data_template: {}
    service: device_tracker.report_address
  max: 10
  mode: single
```

 Enjoy it!