# wifihotspot
使用WiFi热点实现简单的通信

WiFi(Wireless Fidelity)又称802.11b标准,是IEEE定义的一个无线网络通信的工业标准。该技术使用的是2.4GHz附近的频段。
Wi-Fi主要特征如下
1.速度快，最高宽带为11Mbps
2.可靠性强，在信号较弱或者有干扰的情况下，宽带可自动调整为5.5Mbps、2Mbps或者1Mbps，有效的保障了网络的稳定性和可靠性。
3.距离较远，在开放性区域，通信距离可达305米，在封闭性区域，通信距离可达100米左右。
4.方便与现有的有线以太网整合，组网的成本更低。

Android Wi-Fi相关类
Android中Wi-Fi按照层次结构设计的，如下所示。

        ------------------             ---------------            --------------------
       | WirelessSetting |             | WifiSetting |-----------+| AccessPointDialog|
       -------------------             ---------------            --------------------
               |                             |     |                        |
               ------------------------------       --------------          |  
                            |                                    |          |
                     -------+-------                       ------+------    |
                 |--+| WifiEnabler |     ------------------| WifiLayer |+----
                 |   ---------------     |                 ------+---+--  
                 |----------|------------|-----------------------|   |------------- NETWORK_STATE_CHANGED_ACTION
      WIFI_STATE_|CHANGED_AC|TIO         |                                        | SCAN_RESULTS_AVAILABLE_ACTION
                 |   -------+-------     |                 --------------------   | SUPPLICANT_CONNECTION_CHANGE_ACTION
                 |   | WifiManager |+----|  |-------------+| WifiState Tracker|----
                 |   ---------------        |              --------+-----------
                 |          |               |                      |
                 |   -------+-------        |              --------+------
                 ----| WifiService |---------              | WifiMonitor |
                     -------+-------                       ---------------
                            |                                     |
                            |-----------------|-------------------- 
                                      --------+--------
                                      |   WifiNative  |                JAVA VM
                                      -----------------
        ______________________________________|_________________________________________
                                              |JNI                                     
                                   -----------+------------
                                   | android_net_wifi_wifi|
                                   ------------------------
                                              |
                                              |
                                       -------+------
                                       |    wifi    |
                                       --------------
                                              | Socket
                                    ----------+-------
                                    | wpa_supplicant |
                                    ------------------
                                    
Android Wi-Fi模块自下向上可以分为5层：硬件驱动程序、wpa_supplication、JNI、WiFi API、WifiSetting应用程序。
1.wpa_supplicant是一个开源库，是Android实现Wi-Fi功能的基础。它从上层接到命令后，通过Socket与硬件驱动进行通信，操作硬件完成需要的操作。
2.JNI(Java Native Interface)实现了Java代码与其他代码的交互，使得在Java虚拟机中运行的Java代码能够与其他语言编写的应用程序和库进行交互。
在Android中，JNI可以让Java程序调用C程序。
3.Wi-Fi API使应用程序可以使用WiFi功能。
4.Wifi Settings应用程序是Android中自带的一个应用程序，选择手机的Settings->Wireless&networks->Wi-Fi,可以让用户手动打开或关闭Wi-Fi功能，
当用户打开Wi-Fi功能后，它会自动搜索周围的无线网络，并以列表的形式显示，供用户选择，默认会连接用户上一次成功连接的无线网络。

在android.net.wifi包中提供了一些类管理设备的Wi-Fi功能，主要包括ScanResult、wifiConfiguration、WifiInfo和WifiManager.

(1)ScanResult类
ScanResult类主要是通过Wi-Fi硬件的扫描来获取一些周边的Wi-Fi热点(Access Point,如无线网络路由器等)的信息，该类含有以下5个类

                                           ScanResult类包含的域
           --------------------------------------------------------------------------------------------------
           |  返回类型   |      域名        |                       解释                                    |
           --------------------------------------------------------------------------------------------------
           |public String|     BSSID        |接入点的地址                                                   |
           --------------------------------------------------------------------------------------------------
           |public String|      SSID        |网路的名称                                                     |
           --------------------------------------------------------------------------------------------------
           |public String|   capabilities   |网络性能，包括接入点支持的认证，密钥管理，加密机制等           |
           --------------------------------------------------------------------------------------------------
           |public int   |  frequency       |以MHz为单位的接入频率                                          |
           --------------------------------------------------------------------------------------------------
           |public int   |   level          |以dBm为单位的信号强度                                          |
           --------------------------------------------------------------------------------------------------
该类还提供了一个方法toString()方法，可以将结果转化为简洁，易读的字符串形式。
(2)WifiConfiguration类
通过该类可以获取Wi-Fi网络的网络配置，包括安全配置等。该类包含6个子类。

                                       WifiConfiguration子类列表
           --------------------------------------------------------------------------------------------------
           |        子类                              |                         解释                        |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.AuthAlgorthm            |获取IEEE 802.11的加密方法                            |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.GroupCiper              |获取组密钥                                           |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.KeyMgmt                 |获取密码管理体制                                     |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.PairwiseCipher          |获取WPA方式的成对密钥                                |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.Protocol                |获取加密协议                                         |
           --------------------------------------------------------------------------------------------------
           |WifiConfiguration.Status                  |获取当前网络状态                                     |
           --------------------------------------------------------------------------------------------------
(3)WifiInfo类
通过该类可以获取已经建立的或处于活动状态的Wi-Fi网络的状态信息。

                                              WifiInfo类中的部分方法
           --------------------------------------------------------------------------------------------------
           | 方法                      |获取当前接入点BSSID(basic service set identifier)                   |
           --------------------------------------------------------------------------------------------------
           |getBSSID()                 |获取IP地址                                                          |
           --------------------------------------------------------------------------------------------------
           |getLinkSpeed()             |获取当前连接的速度                                                  |
           --------------------------------------------------------------------------------------------------
           |getMacAddress()            |获取MAC地址                                                         |
           --------------------------------------------------------------------------------------------------
           |getRssi()                  |获取802.11n网络的信号强度指示                                       |
           --------------------------------------------------------------------------------------------------
           |getSSID()                  |获取网络SSID(service set identifier)                                |
           --------------------------------------------------------------------------------------------------
           |getSupplicanState()        |返回客户端状态的信息(SupplicantState对象的形式)                     |
           --------------------------------------------------------------------------------------------------
(4)WifiManager类
该类用于管理Wi-Fi连接，其定义了26个常量和23个方法。
                                          WifiManager类中部分方法
					  --------------------------------------------------------------------------------------------------
					  | 方法                                             |                         解释                |
					  --------------------------------------------------------------------------------------------------
					  |addNetwork(WifiConfiguration config)              |向设置好的网络里添加新网络                   |           
					  --------------------------------------------------------------------------------------------------
					  |calculateSignalLevel(int rssi,int numLevels)      |计算信号的等级                               |
					  --------------------------------------------------------------------------------------------------
					  |compareSignalLevel(int rssiA, int rssiB)          |对比两个信号的强度                           |
					  --------------------------------------------------------------------------------------------------
					  |createWifiLock(int lockType, String tag)          |创建一个Wi-Fi锁，锁定当前的Wi-Fi连接         |
					  --------------------------------------------------------------------------------------------------
					  |disableNetwork(int netId)                         |让一个网络连接失败                           |
					  --------------------------------------------------------------------------------------------------
					  |disconnect()                                      |与当前接入点断开连接                         |
					  --------------------------------------------------------------------------------------------------
					  |enableNetwork(int netId, Boolean disableOthers)   |与之前设置好的网络建立连接                   |
					  --------------------------------------------------------------------------------------------------
            |getConfiguredNetworks()                           |客户端获取网络连接的状态                     |
            --------------------------------------------------------------------------------------------------
            |getConnectionInfo()                               |获取当前连接的信息                           |
            --------------------------------------------------------------------------------------------------
            |getDhcpInfo()                                     |获取DHCP的信息                               |
            --------------------------------------------------------------------------------------------------
            |getScanResulats()                                 |获取扫描测试的结果                           |
            --------------------------------------------------------------------------------------------------
 