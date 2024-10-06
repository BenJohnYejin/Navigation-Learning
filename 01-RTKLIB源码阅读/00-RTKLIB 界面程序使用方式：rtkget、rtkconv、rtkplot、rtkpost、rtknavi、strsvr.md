

[TOC]

> 推荐资料：
>
> * [RTKLIB2.4.3界面程序中文说明书.pdf](RTKLIB2.4.3界面程序中文说明书.pdf)
> * 比起苍白的图文介绍，看视频教程来学习界面程序更为清晰，推荐看B站[赵乐文](https://space.bilibili.com/479790048)和[欧阳明俊](https://space.bilibili.com/1394219706)老师的视频。

### 1、RTKGET 数据下载

> 不推荐用 RTKGET 下载数据，推荐使用周峰老师开源的 [GAMP-GOOD](https://github.com/zhouforme0318/GAMPII-GOOD) 和常春涛博士开源的 [FAST](https://github.com/ChangChuntao/FAST)，都带有界面。 

1. 观测值下载

   * 选择下载数据的时间：起始时间，结束时间

   * options 设置 URL_LIST，可以用 RTKLIB 默认配置，选择 rtklib 中 data 文件夹下 URL_LIST.txt 文件（我在 bin 版的 rtklib 里没找到，用的源码版的 rtklib 里的文件），加载进来，左边就有了两列内容。

   * OBS 为观测值文件，NAV 为导航电文，EPH 为精密轨道，CLK 精密钟差、ATX 天线文件

   * 做相对定位要下观测值
     * 先选择分析中心，IGS、MGES 等。

     * 后在右边选测站，点...把测站加上去，ALIC、KARR

     * 把测站名点选，再点 Download，理论上就可把数据下到指定目录，但会比较慢

       * 可复制 FTP 路径直接进网页下载

       * 要在 Linux 下大量处理，可写脚本

   ![bf7b9a35f8b2c5f149215023bfd25831](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/bf7b9a35f8b2c5f149215023bfd25831.png)

2. 用 RTKGET 做时间转换：输入年月日时分秒，点问号 ？，就可看各种时间，下载观测值需要年积日 DOY，改链接的日期就可下载对应的观测值文件

   ![image-20231025204151284](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/image-20231025204151284.png)

3. 数据命名格式：**测站名（4位）+机构信息+年+年积日+采样间隔**，crx 是压缩格式，gz 也是压缩格式，还有一种是 o.z 结尾只进行一次压缩

4. 广播星历文件和精密星历文件：也可用 rtkget 和 ftp下载



### 2、RTKCONV 数据转换

1. 为啥要介绍此模块：老师刚刚拿到了ublox 接收机连天宝天线采集的数据，想分析一下数据的质量。

2. ublox 通过串口导出的二进制文件，COM3 开头，.ublox 结尾，除了原始观测数据之外还有 NMEA 数据。通过 notepad++ 打开查看，开头乱码是二进制数据，后面是 NMEA 文本格式。

3. ublox 数据还可用 ucenter 接收机配置软件查看

4. RTKCONV 使用

   * 先选择需要转换数据的起止时间，采样率 Interval。
   * 输入原始数据地址。
   * 选格式，u-blox、RINEX、RTCM3...，不知道格式可选自动 Auto。
   * 勾选选输出数据，一般得要obs观测数据，如果实时数据从网上不能在网上下导航电文，需要转换出的 nav 文件。
   * 配置信息：RINEX 版本号、测站 ID 可以不写、RunBy 可以写自己、天线类型接收机类型有需要可以写，近视坐标，加哪些改正信息，输出哪些系统、观测值类型可都勾上，观测频率，信号通道。
   * 点 Convert 转换。 

   ![19792c3d48403c733efcf69677e4863b](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/19792c3d48403c733efcf69677e4863b.png)

   * 点 Plot 可以直观展示卫星数据质量
     * Sat Vis：卫星可见性，选频率，颜色代表信噪比 SNR。
     * Skyplot：卫星天空视图，站心地平级坐标系，可看出低高度角卫星信号差
     * DOP：上面是可视卫星数，下面是 DOP 值
     * SNR：载噪比、多路径，可选某一颗卫星指定频率，横坐标可选时间、高度角

### 3、RTKPOST 数据后处理

1. 主界面

   * 设置解算起止时间，解算间隔
   * 加载 RINEX OBS 数据：Rover 流动站、Base 基准站，右上角点天空图标开 RTKPLOT 看数据状态。基准站整天的数据非常大，截取流动站对应部分即可。
   * 加载其它数据：NAV、CLK、SP3 等。每个接收机输出的 NAV 只有它能观测到的卫星星历，从网上可下全部所有卫星所有系统的导航电文。
   * 输出默认在流动站文件路径，后缀为 .pos。 

   ![image-20231026163551141](https://pic-bed-1316053657.cos.ap-nanjing.myqcloud.com/img/image-20231026163551141.png)

2. Options设置

   * **定位模式**

     * Single：伪距单点定位
     * DGPS/DGNSS：伪距差分
     * Kinematic：载波动态相对定位，动态RTK，假设流动站是移动的，可以做车载定位
     * Static：载波静态相对定位，静态RTK，两站都是静止的，可以得到很高的精度
     * Static-Start：（demo5 才有）冷启动：先在比较开阔的地方，进行短时期的静态定位，模糊度固定，再动起来
     * Moving-Base：两站都动，主要用来定姿
     * Fixed：固定坐标，解算模糊度、对流层、电离层等参数
     * PPP Kinematic、PPP-Static、PPP Fixed

   * **频率**：可选不同频率组合，如L1+L2

   * **滤波**：前向（后面结果更可靠）、后向（可使刚开始的时候有高的精度）、Combind（正向一个结果，反向一个结果，根据方差加权平均），RTKLIB里除了SPP都用卡尔曼滤波，滤波有一个收敛的过程，后面更准。

   * **设置截止高度角**：可以看天空视图，如果低高度角数据很差，可设置更高的截止高度角。质量好可以不管，有残差检验也可剔除一些数据。

   * **设置截止信噪比**：做工程一般环境都不会很好，想做的序列稳定，要设置截止信噪比。RTK有很多算法，但其实传统算法效果已经很好了，算法不用做的太复杂，把数据质量控制做好就行，RTK就不会有太大的问题。

   * **Rec Dynamics**：动力学模式，选 ON 会估计速度加速度参数，，选 OFF 就只估算动态坐标参数

     * Kinematic 动态模式：把位置参数当白噪声估计。
     * Dynamics 动力学模型：建立 CA 模型，同时估计位置、速度、加速度。

   * **RCV、潮汐改正等**：PPP 才用的到

   * **电离层、对流层改正**：双差已经可以消除部分电离层对流层误差，可以关闭此改正，也可以直接采用广播星历的模型改正。RTKLIB 做 RTK 最好用非组合模式，短基线电离层可以关闭，对流层可以用 saastamoinen 模型直接修正。

   * **卫星星历**：RTK 相对定位距离近可以直接用广播星历，长距离相对定位可选精密星历。

     * SSR APC：参考天线相位中心
     * SSR CoM：参考质心，还需要天线相位中心改正

   * **剔除卫星**：写卫星号，空格隔开，如：C01 C02

   * **RAIM FDE完好性检验**：算法不是很稳健，不选

   * **模糊度固定模式 ARMODE**

     * **OFF**：浮点解，不固定

     * **Continues**：认为模糊度是连续解，通过前面历元的解算结果滤波提高后续历元模糊度固定精度。

     * **Instantaneous**：瞬时模糊度固定，单历元模糊度固定，每个历元都初始化一个参数，这个历元和上个历元模糊度不相关，用伪距和载波计算的整周模糊度作为模糊度的状态。

     * **Fix and Hold**：先 Continues，在不发生周跳情况下都采用之前模糊度固定的结果作为约束，表示用上一个历元求得的模糊度固定解作为量测，上一 个历元求得的模糊度实数解为状态，进行卡尔曼滤波，融合后的模糊度作为当前模糊度的状态。也有问题：固定错了，时间序列会一直飘，到一定程度变成浮点解，会重置模糊度重新算。

       > * 与 continuous 和 fix-and-hold 方法相比，instantaneous 方案的误差曲线突刺较多，定位误差较大，这主要是因为伪距噪声较大，用伪距求得的模糊度精度较差；
       >* 当无周跳发生时，continuous 和 fix-and-hold 方法精度应该会高于 instantaneous 方法；当有周跳发生并且没有探测出来时，instantaneous 方法精度可能优于 continuous 和 fix-and-hold 方法；
       > 
       > * 当模糊度固定正确时，fix-and-hold 方法精度应该高于 instantaneous 和 continuous 方法；固定错误时，instantaneous 和 continuous 方法精度应该好于 fix-and-hold 方法。
       
       > 做工程可做两套，Instantaneous 和 Fix and Hold，发现 Fix and Hold 错了，就用 Instantaneous 的解把它替换掉，相当于把模糊度和方差初始化了一次，避免漂移和模糊度重新收敛的过程。

     * **PPP-AR**：PPP 时固定模糊度，不支持，需要额外产品。

   * **Ratio值**：用于检验模糊度是否固定成功，设为 3 即可。

   * **最小LOCK**：连续锁定这颗卫星几次，才用于计算模糊度固定。

   * **用于模糊度固定的最低高度角设置**：可设 15°

   * **最小Fix**：这个历元最少固定多少个模糊度才认为模糊度是固定的，可设 10，现在卫星系统多了，而且组合模式，双频一颗卫星就 2 个模糊度，5 颗卫星固定就能凑 10 个。

   * **Fix hold**：选择哪些模糊度固定结果用于约束后续。

   * **输出结果**：可选  LLH、XYZ、ENU、NMEA

   * **输出解算状态**：可选 OFF、Residuals 残差、State

   * **Debug Trace等级**：1-5级，level 越高输出越多

   * **基准站坐标**：可输入、也可选伪距单点定位

   * **天线类型**：选*，自动获取 O 文件里的

3. 算完之后

   * **Plot**：对解算结果可视化分析，黄色没固定，绿色固定
   * **view**：查看解算结果，类似记事本
   * **KML**：转为 GoogleXML 可把地图展示到地图上

> 建议：下静态数据，找动态车载数据，分别处理静态相对定位和处理动态相对定位，设置不同处理模式，分析定位结果的差异。

4. PPP 数据处理
   * 实时PPP：IGS/MGEX 分析中心播发的实时卫星轨道和钟差产品，结合广播星历
   * 事后或近实时：下载精密星历、钟差产品，结合其它精密改正信息实现定位
   * RTKLIB 使用必须给广播星历，因为解算前都会先进行一次伪距单点定位

### 4、SRTSVR 数据流收发

1. 功能概述

   * **TCP Server**：等待来自客户端的连接请求，处理请求并返回结果。

   * **TCP Client**：主动角色，发送连接请求，等待服务器响应。

   * **Ntrip Server**：将本地接收机的 RTCM 数据推送到 Ntrip Caster。

   * **Ntrip Caster**：用户管理和播发 RTCM 数据。

   * **Ntrip Client**：登录 Ntrip Caster 获取 RTCM 数据。


2. 界面

   * 一个输入，多个输出

   * 输入输出可以是：Serial，TCP Client、TCP Server、Ntrip Client、Ntrip Server、UDP Server、File、FTP、HTTP


3. RTK2GO

   * 相当于免费的 Ntrip Caster，所有的用户都可把自己的数据源上传到 Caster 中，其它的用户都可以用 Caster 接受数据

   * 连接：输入模式选 Ntrip Client，通过网址和端口，点 Ntrip 就会弹出弹出数据源

4. 输出

* 点左下角 □ 框，开 Input Stream Monitor 查看数据流状态，可选很多种格式
* 输出也可选很多种，比如 Ntrip Server 可把自己的数据作为 Caster，别人可以通过网络接受你的数据，选 File 把数据存成文件

### 5、RTKNAVI 实时定位解算

