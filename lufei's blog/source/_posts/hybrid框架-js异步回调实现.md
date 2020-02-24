---
title: 'hybrid框架:js异步回调实现'
date: 2019-11-15 16:03:49
tags: iOS
categories: iOS
---

# 前言
最近学习了一段时间`hybrid`，写一点收获，本文主要关注的是`js`调用原生的时候，异步回调，如何处理，这是实现一个`JSBridge`的关键。

下面，我们开始

# js消息体
代码类似这样：
```
var msgBody = {};
msgBody.handler = 'common';
msgBody.action = 'nativeLog';
msgBody.params = params; //任意json对象，用于传参.
msgBody.callbackId = '';
msgBody.callbackFunction = '';
```
`callbackId`: 对每一次消息需要发起回调都会生成一个唯一ID，用来当回调发生时，找到最初的发起调用的 `JS Callback`

`callbackFunction`: 客户端主动 `Call JS` 的唯一函数入口，客户端会用这个字符串来拼接回调注入的 JS 头，一般设计下，每个消息这个值都应该不变，不过也可以灵活处理（本来这个值可以不需要传递，写死在客户端，只要前端客户端约定好，但如果这个值不写死，而由前端可控操作，那么灵活性会更大，不必担心前端大规模修改 `Call JS` 唯一入口的时候，还得等客户端发版）

代码大概如下：
```
sendMessage: function (data,callback) {
    if (callback && typeof (callback) === 'function') {
        var callbackid = this.getNextCallbackID();
        this.msgCallbackMap[callbackid] = callback;
        params.callbackId = callbackid;
        params.callbackFunction = 'window.callbackDispatcher';
    }
    
    if (this.isIOS) {
        try {
            window.webkit.messageHandlers.WKJSBridge.postMessage(data);
        }
        catch (error) {
            console.log('error native message');
        }
    }

    if (this.isAndroid) {
        try {
            prompt(JSON.stringify([data]));
        }
        catch (error) {
            console.log('error native message');
        }
    }
},
    
sendHaHaHaMessage(msgBody,function(result){
	console.log('回调触发');
});
```
可以看到我们着手修改 `sendMessage` 函数，如果在调用的时候多写了一个`callback`函数，那么就会认为该次通信需要回调，因此对 `callbackId` 与 `callbackFunction` 进行赋值，callbackId 是一个保证每次通信都唯一的一个id值 `getNextCallbackID` ，大概思路可以是用时间戳+一定程度的随机小数来进行生成，思路不深入展开了。 `callbackFunction` 这里我们先写 `window.callbackDispatcher` 。

这里有一步最最重要的操作就是:
```
this.msgCallbackMap[callbackid] = callback; 
```
会把 JS 业务的回调函数，保存在一个全局可处理的回调字典之中，而 Key 就是这个唯一ID `callbackId`，这样当 `OC` 发起回调的时候，你才能找到对应的 `JS Function`

# 原生处理（以OC做示例）

首先是OC的消息类
```
typedef void (^JSResponseCallback)(NSDictionary* responseData);

@interface msgObject : NSObject

- (instancetype)initWithDictionary:(NSDictionary *)dict;

@property (nonatomic, copy, readonly) NSString * handler;
@property (nonatomic, copy, readonly) NSString * action;
@property (nonatomic, copy, readonly) NSDictionary * parameters;
@property (nonatomic, copy, readonly) NSString * callbackID;
@property (nonatomic, copy, readonly) NSString  *callbackFunction;

-(void)setCallback:(JSResponseCallback)callback; //block 作为属性，保存在msgObject的.m文件里

-(void)callback:(NSDictionary *)result;//在msgObject的.m文件里 调用保存在消息体里的block

@end
```
所以我们继续修改 OC 这边收到 JS 消息的函数体，当判断消息体含有回调信息的时候，就会生成用于回调的 OC Block，当OC业务处理完毕，准备回调回传数据的时候使用, 代码如下：
```
-(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    NSDictionary *msgBody = message.body;
    if (msgBody) {
        msgObject *msg = [[msgObject alloc]initWithDictionary:msgBody];
        NSDictionary *handlerDic = [self.handlerMap objectForKey:msg.handler];
        HandlerBlock handler = [handlerDic objectForKey:msg.action];
        //处理回调
        if (msg.callbackID && msg.callbackID.length > 0) {
            //生成OC的回调block，输入参数是，任意字典对象的执行结果
            JSResponseCallback callback = ^(id responseData){
                //执行OC 主动 Call JS 的编码与通信
                [weakSelf injectMessageFuction:callbackFunction withActionId:callbackId withParams:responseData];
            };
            [msg setCallback:callback];
        }
        if (handler){
            handler(msg);
        }
    }
}
```

那业务在注册 OC 消息处理函数的时候，就可以使用这个block 进行回调，大概是这样：
```
[self registerHandler:@"common" Action:@"nativeLog" handler:^(msgObject *msg) {
    NSLog(@"webview log : \n%@",msg)
    NSDictionary *result = @{@"result":"result"};
    //回调一个key value均为 result 字符串的字典当做数据
    [msg callback:result];
}];
```
以上就是大概的思路。

如果只是需要js和原生通信的话，`UIWebView`，`WKWebView`，`JavaScriptCore`都提供了很多方法，感兴趣的可以去了解一下，嘻嘻嘻QAQ

# 参考
[从零收拾一个hybrid框架](http://awhisper.github.io/2018/03/06/hybrid-webcontainer/)




