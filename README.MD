# Android版本EasyLink SDK
### **请注意，Easylink AWS请使用v0.2.6版本**
EasyLink SDK 主要功能:    

  * 将手机端设置的Wi-Fi网络参数发送到设备端
    *  支持EasyLink模式
    *  支持EasyLink—AWS模式
  * 到新入网的设备，并且交互。实现参数设置，OTA等功能  
**注意：**EasyLink模式仅支持在IPv4网络中进行操作。

### 示例程序

1. 简单EasyLink的示例程序：  
1）[简单的示例程序：](https://github.com/MXCHIP/EasylinkAndroid_Demo)  
提供了一个简单的UI，展示EasyLink SDK的使用方法 ，主要功能是配网和设备发现。  
app链接如下：[easylink-app-url](https://fir.im/eckg)


##### Android Studio引入的资源

```js
dependencies {
    // 只发广播的版本
    compile 'io.fogcloud.sdk:easylinkv3:0.1.4'
    // 广播组播同时发的版本
    compile 'io.fogcloud.sdk:easylinkv3:0.1.5'
    // AWS配网版本
    compile 'io.fogcloud.sdk:easylinkv3:0.1.6'
}
```

一般情况使用只发广播的版本就可以了，特殊需求，比如广播收不到的情况可以使用0.1.5版本，广播组播一起发。

##### 开启服务

需要现在manifest.xml中开启服务


```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```


## **功能列表**

* [获取SSID](#getSSID)
* [开始配网](#startEasyLink)
* [停止配网](#stopEasyLink)
* [设备发现信息](#设备发现)


<div id="getSSID"></div>

## getSSID

    获取当前手机连接的WIFI的名称，即ssid

    String getSSID()

##### callback
ssid
- 类型：String
- 描述：当前WIFI的名称

##### 示例代码

```java
EasylinkP2P elp2p = new EasylinkP2P(mContext);
elink.getSSID()
```

<div id="startEasyLink"></div>

## startEasyLink

    发送数据包(包含ssid和password)给设备，连续发10s，再停止3s，再继续发，如此反复

    startEasyLink(EasyLinkParams easylinkPara, EasyLinkCallBack easylinkcb)

##### params
参数名 | 类型 | 描述
:-----------  | :-------------:|:-----------
easylinkPara     | EasyLinkParams       | EasyLinkParams包含以下的信息

##### EasyLinkParams
参数名 | 类型 | 默认值 | 描述
:-----------  | :-------------:| :-----------|:-----------
ssid        | String       | 无默认值，不可为空 | 当前wifi的名称
password     | String       | 无默认值，可为空 | 当前wifi的密码(8-64个字节，越长配网速度越慢)
isSendIP     | Boolean       | 默认值为false，可为空|是否发送手机的IP，默认不发送，如果此参数为false，那么extraData也不可用
runSecond         | int       | 默认值50，可为空 | 发送持续的时间，到点了就停止发送, 单位s
sleeptime         | int       | 默认值50，可为空 | 每包数据的间隔时间，建议20-200, 单位ms
extraData         | String     | 无默认值，可为空   | 需要发送给设备的额外信息
rc4key     | String       | 无默认值，可为空 | 如果需要RC4加密，这里就输入字符串密钥

##### callback
easylinkcb
- 类型：EasyLinkCallBack
- 描述：接口调用成功后的回调函数

##### 示例代码

```java
EasyLinkParams easylinkPara = new EasyLinkParams();
easylinkPara.ssid = "mxchip";
easylinkPara.password = "12345678";
easylinkPara.runSecond = 60000;
easylinkPara.sleeptime = 20;

elink.startEasyLink(easylinkPara, new EasyLinkCallBack() {
    @Override
    public void onSuccess(int code, String message) {
        Log.d(TAG, code + message);
    }
    @Override
    public void onFailure(int code, String message) {
        Log.d(TAG, code + message);
    }
});
```

<div id="stopEasyLink"></div>

## stopEasyLink
    
    停止发送数据包

    stopEasyLink(EasyLinkCallBack easylinkcb)

##### callback
easylinkcb
- 类型：EasyLinkCallBack
- 描述：接口调用成功后的回调函数

##### 示例代码

```java
elink.stopEasyLink(new EasyLinkCallBack() {
    @Override
    public void onSuccess(int code, String message) {
        Log.d(TAG, code + message);
    }
    @Override
    public void onFailure(int code, String message) {
        Log.d(TAG, code + message);
    }
});
```

## 实时更新SSID

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    ...
    listenwifichange();
}

private void listenwifichange() {
    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(WifiManager.NETWORK_STATE_CHANGED_ACTION);
    registerReceiver(broadcastReceiver, intentFilter);
}

BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action.equals(WifiManager.NETWORK_STATE_CHANGED_ACTION)) {
            NetworkInfo info = intent.getParcelableExtra(WifiManager.EXTRA_NETWORK_INFO);
            if (info.getDetailedState() == NetworkInfo.DetailedState.CONNECTED) {
                Log.d(TAG, "---heiheihei---");
                wifissid.setText(elink.getSSID());
            }
        }
    }
};

@Override
protected void onDestroy() {
    super.onDestroy();
    unregisterReceiver(broadcastReceiver);
}
```

## 设备发现

设备会在连上路由器后通过广播方式通知到手机，可以在easylink success callback  
里面获取到相应的信息，在message中。

```java   
elp2p.startEasyLink(elp, new EasyLinkCallBack() {
                    @Override
                    public void onSuccess(int code, String message) {
                        Log.d(TAG, message);
                    }

                    @Override
                    public void onFailure(int code, String message) {
                        Log.d(TAG, message);
                    }
                });
```


# EasyLink SDK with Android
Main function of EasyLink SDK:    

  * Send the Wi-Fi network parameters set on mobile APP to module
    *  Support EasyLink mode
    *  Support EasyLink—AWSS mode
**Notice：**EasyLink mode support in IPv4

### Sample Code

1. Simple EasyLink Sample:  
1）[Simple example：](https://github.com/MXCHIP/EasylinkAndroid_Demo)  
Provided a simple UI to show how to use EasyLink SDK, main function would be Wi-Fi pairing and device discovery.  
app link：[easylink-app-url](https://fir.im/eckg)


##### Android Studio resource

```js
dependencies {
    // Boardcast only version
    compile 'io.fogcloud.sdk:easylinkv3:0.1.4'
    // boardcast and muilticast version
    compile 'io.fogcloud.sdk:easylinkv3:0.1.5'
    // AWSS version
    compile 'io.fogcloud.sdk:easylinkv3:0.1.6'
}
```

In general, version with only broadcast can be used. For special needs, such as when the broadcast cannot be received, the 0.1.5 version can be used with broadcast and multicast can be sent together.

##### Turn on service

It is require to turn on setvice in manifest.xml


```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```


## **Function list**

* [Get SSID](#getSSID)
* [Start Wi-Fi pairing](#startEasyLink)
* [Stop Wi-fi Pairing](#stopEasyLink)
* [Information of device discovery](#Device discovery)


<div id="getSSID"></div>

## getSSID

    Get ssid from mobile phone

    String getSSID()

##### callback
ssid
- Type: String
- Description: SSID of current Wi-Fi
 
##### Sample Code

```java
EasylinkP2P elp2p = new EasylinkP2P(mContext);
elink.getSSID()
```

<div id="startEasyLink"></div>

## startEasyLink

    Send data packets (including SSID and password) to the device, send them continuously for 10s, stop for 3S, and continue sending them again and again

    startEasyLink(EasyLinkParams easylinkPara, EasyLinkCallBack easylinkcb)

##### params
Parameters | Type | Description
:-----------  | :-------------:|:-----------
easylinkPara     | EasyLinkParams       | EasyLinkParams contains below information

##### EasyLinkParams
Parameters | Type | Default Value | Description
:-----------  | :-------------:| :-----------|:-----------
ssid        | String       | No default value, cannot be NULL | SSID of current Wi-Fi
password     | String       | No default value, can be NULL | Current WiFi password (8-64 bytes, the longer the length, the slower the distribution network speed)
isSendIP     | Boolean       | default value: false. can be NULL|Whether to send the IP of the mobile phone. It is not sent by default. If this parameter is false, extradata is not available
runSecond         | int       | default value: 50. can be NULL | Duration of sending. Stop sending when the point is reached, unit: second
sleeptime         | int       | default value: 50. can be NULL | The interval time of each packet of data is recommended to be 20-200, in MS
extraData         | String     | No default value, can be NULL   | Additional information that needs to be sent to the device
rc4key     | String       | No default value, can be NULL | If RC4 encryption is required, enter the string key here

##### callback
easylinkcb
- Type: EasyLinkCallBack
- Description: Callback function after successful API call

##### Sample Code

```java
EasyLinkParams easylinkPara = new EasyLinkParams();
easylinkPara.ssid = "mxchip";
easylinkPara.password = "12345678";
easylinkPara.runSecond = 60000;
easylinkPara.sleeptime = 20;

elink.startEasyLink(easylinkPara, new EasyLinkCallBack() {
    @Override
    public void onSuccess(int code, String message) {
        Log.d(TAG, code + message);
    }
    @Override
    public void onFailure(int code, String message) {
        Log.d(TAG, code + message);
    }
});
```

<div id="stopEasyLink"></div>

## stopEasyLink
    
    Stop send package

    stopEasyLink(EasyLinkCallBack easylinkcb)

##### callback
easylinkcb
- Type: EasyLinkCallBack
- Description: Callback when API success

##### Sample Code

```java
elink.stopEasyLink(new EasyLinkCallBack() {
    @Override
    public void onSuccess(int code, String message) {
        Log.d(TAG, code + message);
    }
    @Override
    public void onFailure(int code, String message) {
        Log.d(TAG, code + message);
    }
});
```

## Real-time SSID update

```java
@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    ...
    listenwifichange();
}

private void listenwifichange() {
    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(WifiManager.NETWORK_STATE_CHANGED_ACTION);
    registerReceiver(broadcastReceiver, intentFilter);
}

BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action.equals(WifiManager.NETWORK_STATE_CHANGED_ACTION)) {
            NetworkInfo info = intent.getParcelableExtra(WifiManager.EXTRA_NETWORK_INFO);
            if (info.getDetailedState() == NetworkInfo.DetailedState.CONNECTED) {
                Log.d(TAG, "---heiheihei---");
                wifissid.setText(elink.getSSID());
            }
        }
    }
};

@Override
protected void onDestroy() {
    super.onDestroy();
    unregisterReceiver(broadcastReceiver);
}
```

## Device Discovery

The device will notify the mobile phone by broadcasting after connecting to the router. Information could be check in message from easylink success callback  

```java   
elp2p.startEasyLink(elp, new EasyLinkCallBack() {
                    @Override
                    public void onSuccess(int code, String message) {
                        Log.d(TAG, message);
                    }

                    @Override
                    public void onFailure(int code, String message) {
                        Log.d(TAG, message);
                    }
                });
```
