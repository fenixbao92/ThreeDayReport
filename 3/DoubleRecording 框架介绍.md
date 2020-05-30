DoubleRecording 框架介绍
===
## 一.安装运行
1. git clone https://github.com/xmatrix-cn/DoubleRecording.git
2. AndroidStdio file->open文件根目录
3. 在AndroidStdio待自动下载相关依赖和配置后点击运行即可

## 二.框架功能
1. 底部tab栏
2. 顶部导航栏
    * 在主界面(即tab控制的四个界面:首页 待面签保单 我的保单 我的)显示标题。
	* 在其余的非主界面显示标题和back按钮。
3. 蓝湖ui第二期具体界面的定义和跳转逻辑(除去有改动的3-待面签-正在排队界面)。
4. 用户名密码登录注册的通用逻辑，调sdk或api的地方已预留。
5. 自定义的轻量级事件处理框架。

## 三.工程结构
![](https://github.com/fenixbao92/ThreeDayReport/blob/master/3/pic/WechatIMG8.jpeg)

## 四.外部依赖
1. 'me.imid.swipebacklayout.lib:library:1.1.0'  Activity右划退出库
2. 'com.jakewharton:butterknife:7.0.1' Android在java中控制布局文件中组件的注解库,使用方法:
   * xxxLayout.xml：
    ``` 
    <Button android:id="@+id/btn_login".../>
    ```
   * xxxActivity:
    ```
    @OnClick(R.id.btn_login) 
    public void login(){...}
    ```
## 五.拟定工作规划
1. 音视频分离
2. 视频抽帧
3. 录制是视频界面上画东西
4. 接sdk
5. TTS和ASR

## 六.下一步
   我拟调研下openCV我大概搜了下跟我们貌似有点关系
![](https://github.com/fenixbao92/ThreeDayReport/blob/master/3/pic/WechatIMG6.jpeg)
![](https://github.com/fenixbao92/ThreeDayReport/blob/master/3/pic/WechatIMG7.jpeg)
