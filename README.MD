# 明日方舟未成年版 更新至1.1.50 沃伦姆德的薄暮
## 注意事项
事先申明，这个东西对客户端有一定的篡改，如果鹰小姐铁了心要查是可以查出来的。虽然目前没发现检查的迹象，但未来的事情谁知道呢？如果担心鹰小姐狠心检查然后封号，就绝对不要使用，免得到时候来找我说**啊啊啊我被封号了嘤嘤嘤**。  
  
一句话就是我不负这个责，你需要考虑的就是鹰小姐会不会这么狠心，不过这种东西不涉及游戏数据，她估计也懒得对付你。

***已度过大版本：3 ***

***目前封号人数：0***

## 使用说明
直接安装，安装完之后名称为明日方舟未成年版，可以和原版共存。登录进去之后免掉未成年限制，直接可以玩。
    
充值限制有没有去掉我不知道，因为没钱去试，如果去掉了的话创建个issue告诉我（不用试了，没去掉，懒得动手了，有需要可以参考说明自己上）。
    
使用限制是仅限armeabi-v7a架构，不过不用担心，绝大部分手机是这个，就是模拟器不一定，不过我用的夜神支持，其它不晓得。  
  
使用期限到下一次大更新，大概。现在方舟加了lua热更新系统，保不齐给你加点东西然后就用不了了，不过概率不大，除非它故意针对了这个版本。

## 下载
百度云  
链接：https://pan.baidu.com/s/1NpntUv9_IBdVI4CKUmNzqA 提取码：gjam  

## 原理说明
好像有人想要知道怎么做的，就讲一下原理吧

YJ的防沉迷系统实际上是包含在游戏的Ping中的，游戏会不停地Ping服务器，其中返回的值中就包含了是否要把你踢下线

返回结构如下：

```csharp
class PingResponse{
	int result;	//Offset = 0x8
	int interval;	//Offset = 0xC
	string message;	//Offset = 0x10
}
```

如果你是未成年且到限制了，就直接踢下线

所以拦下这个消息，把result修改为0就行。

方法随便，网截也好，注入也好，看你喜好

我这里介绍我的这个笨办法，写一个so，让游戏启动时加载进去就行

加载进去之后Hook两个函数，在执行前修改参数就行。

Hook点如下：

登录时验证：HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0

过程中验证：HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0

这两参数都是void *_this, void *response，我直接这样修改：

```cpp
void *HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	*(int *)(p+0x8)=0;
	return old_HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0(_this,response);
}

void *HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	*(int *)(p+0x8)=0;
	return old_HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0(_this,response);
}
```

找Hook的具体地址参考P大的工具：[IL2CPPDUMPER](https://github.com/Perfare/Il2CppDumper "IL2CPPDUMPER")

一个挺好用的Hook框架：[Android-Hooking](https://github.com/joeyjurjens/Android-Hooking-Template "Android-Hooking")

某个版本的hook.cpp代码，自己参考：
```cpp
#include "hook.h"

void *(*old_HGSDK_HGSDK__CallIdentityVerifyService_c__AnonStoreyA____m__0)(void *_this, void *response);
void *(*old_HGSDK_HGSDK__CallLoginService_c__AnonStorey3____m__0)(void *_this, void *response);
void *(*old_HGSDK_HGSDK__CallAuthService_c__AnonStorey4____m__0)(void *_this, void *response);
void *(*old_HGSDK_HGSDK__CallSmsCodeLoginService_c__AnonStorey7____m__0)(void *_this, void *response);
void *(*old_HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0)(void *_this, void *response);
void *(*old_HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0)(void *_this, void *response);

void *HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	//int result=*(int *)(p+0x8);
	//LOGW("result:%d",result);
	*(int *)(p+0x8)=0;
	//result=*(int *)(p+0x8);
	//LOGW("result2:%d",result);
	//int interval=*(int *)(p+0xC);
	//LOGW("interval:%d",interval);
	//CString *message=(CString *)(*(int *)(p+0x10));
	//CString_Print(message);

	return old_HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0(_this,response);
}

void *HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	//int result=*(int *)(p+0x8);
	//LOGW("result:%d",result);
	*(int *)(p+0x8)=0;
	//result=*(int *)(p+0x8);
	//LOGW("result2:%d",result);
	//int interval=*(int *)(p+0xC);
	//LOGW("interval:%d",interval);
	//CString *message=(CString *)(*(int *)(p+0x10));
	//CString_Print(message);
	return old_HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0(_this,response);
}
/*
void *Torappu_UI_UIPopupWindow___Alert(void *_this, void *content, void *onConfirm){
	CString_Print((CString *)content);
	return old_Torappu_UI_UIPopupWindow___Alert(_this,content,onConfirm);
}

void *HGSDK_HGSDK__CallSmsCodeLoginService_c__AnonStorey7____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	*(p+0x18)=1;
	unsigned char isAuthenticate=*(p+0x18);
	*(p+0x19)=0;
	unsigned char isMinor=*(p+0x19);
	LOGW("isAuthenticate:%d",isAuthenticate);
	LOGW("isMinor:%d",isMinor);
	return old_HGSDK_HGSDK__CallSmsCodeLoginService_c__AnonStorey7____m__0(_this,response);
}

void *HGSDK_HGSDK__CallAuthService_c__AnonStorey4____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	unsigned char *isAuthenticate=(p+0xC);
	unsigned char *isMinor=(p+0xD);
	unsigned char *needAuthenticate=(p+0xE);
	
	*(p+0xC)=0;
	*(p+0xD)=0;
	*(p+0xE)=0;
	
	LOGW("isAuthenticate:%d",*isAuthenticate);
	LOGW("isMinor:%d",*isMinor);
	LOGW("needAuthenticate:%d",*needAuthenticate);
	
	return old_HGSDK_HGSDK__CallAuthService_c__AnonStorey4____m__0(_this,response);
}

void *HGSDK_HGSDK__CallLoginService_c__AnonStorey3____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	*(p+0x18)=1;
	unsigned char isAuthenticate=*(p+0x18);
	*(p+0x19)=0;
	unsigned char isMinor=*(p+0x19);
	int result=*(int *)(p+0x8);
	LOGW("isAuthenticate:%d",isAuthenticate);
	LOGW("isMinor:%d",isMinor);
	//LOGW("result:%d",result);
	return old_HGSDK_HGSDK__CallLoginService_c__AnonStorey3____m__0(_this,response);
}

void *HGSDK_HGSDK__CallIdentityVerifyService_c__AnonStoreyA____m__0(void *_this, void *response){
	unsigned char *p =(unsigned char *)response;
	*(int *)(p+0xC)=0;
	unsigned char isMinor=*(int *)(p+0xC);
	LOGW("isMinor:%d",isMinor);
	CString *message=(CString *)(*(int *)(p+0x10));
	CString_Print(message);
	*(int *)(p+0x8)=0;
	int result=*(int *)(p+0x8);
	LOGW("result:%d",result);
	return old_HGSDK_HGSDK__CallIdentityVerifyService_c__AnonStoreyA____m__0(_this,response);
}
*/
__attribute__((constructor))
void libhook_main() {
    while(libBase == 0) { 
        libBase = get_libBase(libName); 
        sleep(1); 
    }   
    /*MSHookFunction((void *) getRealOffset(0x1188268),
                   (void *) HGSDK_HGSDK__CallIdentityVerifyService_c__AnonStoreyA____m__0,
                   (void **) &old_HGSDK_HGSDK__CallIdentityVerifyService_c__AnonStoreyA____m__0);*/
	/*MSHookFunction((void *) getRealOffset(0x1188850),
                   (void *) HGSDK_HGSDK__CallLoginService_c__AnonStorey3____m__0,
                   (void **) &old_HGSDK_HGSDK__CallLoginService_c__AnonStorey3____m__0);	
	MSHookFunction((void *) getRealOffset(0x1187A34),
                   (void *) HGSDK_HGSDK__CallAuthService_c__AnonStorey4____m__0,
                   (void **) &old_HGSDK_HGSDK__CallAuthService_c__AnonStorey4____m__0);		
	MSHookFunction((void *) getRealOffset(0x1188DAC),
                   (void *) HGSDK_HGSDK__CallSmsCodeLoginService_c__AnonStorey7____m__0,
                   (void **) &old_HGSDK_HGSDK__CallSmsCodeLoginService_c__AnonStorey7____m__0);
	MSHookFunction((void *) getRealOffset(0x108E534),
                   (void *) Torappu_UI_UIPopupWindow___Alert,
                   (void **) &old_Torappu_UI_UIPopupWindow___Alert);*/
	MSHookFunction((void *) getRealOffset(0x140A200),
                   (void *) HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0,
                   (void **) &old_HGSDK_HGSDK_PingManager__FirstPing_c__AnonStorey1____m__0);
	MSHookFunction((void *) getRealOffset(0x1409C24),
                   (void *) HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0,
                   (void **) &old_HGSDK_HGSDK_PingManager____DoPeriodicPing_m__0);
}
```
