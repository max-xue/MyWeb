---
title: Unity开发之设计模式---观察者模式
categories:
  - U3D
url: 1285.html
id: 1285
comments: true
date: 2018-05-19 21:05:22
tags:
---

曾记录一篇关于设计模式的日志：[《Head First 设计模式》------学习总结](http://www.le-more.com/?p=92),在那篇文章中只是记录一些重要的概念，缺乏实战，也不好理解。下面记录在项目中使用到的设计模式，理论和实际相结合才有价值，“把模式装进脑子里，然后在你的设计和已有的应用中，寻找何处可以使用它们。”。 观察者模式也是比较常见的设计模式，也容易理解。观察者模式的实现方式也有很多种，先列出项目中使用的方式

*   观察者模式实现方式一Java中的实现

定义订阅者行为接口

public interface IObserver
{
    void update(Observable obs, Object arg);
}

出版者父类，类中包含订阅者对象超类型列表，OO编程原则“针对接口编程，而不是针对实现编程”示例。

public class Observable
{
    private bool _changed = false;
    private List<IObserver> _observerList;
    public bool Changed
    {
        get
        {
            return _changed;
        }

        set
        {
            _changed = value;
        }
    }

    public Observable()
    {
        _observerList = new List<IObserver>();
    }

    public void addObserver(IObserver item) {
        _observerList.Add(item);
    }

    public void deleteObserver(IObserver item) {
        _observerList.Remove(item);
    }

    public void notifyObservers(Object arg) {
        if (Changed) {
            for(int i=0;i< _observerList.Count;++i)
            {
                var observer = _observerList\[i\];

                observer.update(this, arg);
            }

            Changed = false;
        }
    }

    protected void setChanged() {
        Changed = true;
    }
}

使用示例，具体的出版者(网络数据管理分发类)和订阅者(用户消息处理类)

 public class HandlerObservable : Observable
{
    ..........................
    未列出函数
    
    //数据分发
    public void Update(float dt)
    { 
        //数据转发
        if (ClientSocketHelper.Instance.RecvQueue.Count > 0)
        {
            for (int i = 0; i < ClientSocketHelper.Instance.RecvQueue.Count; ++i)
            {
                setChanged();

                var data = ClientSocketHelper.Instance.RecvQueue.Dequeue();
                bool result = _resReconnect(data);
                if (!result)
                {
                    notifyObservers(data);
                }
            }
        }
    }
}

public class UserHandler : BaseHandler,IObserver 
{
    public UserHandler()
    {
        //订阅消息
        HandlerObservable.Instance.addObserver(this);
    }

    //消息接收
    public void update(Observable obs, object arg)
    {
        SocketData socketData = arg as SocketData;
    }
}

 

*   观察者模式实现EventDispatcher

观察者模式还有其他表现形式，像iOS中的NotificationCenter，cocos2d中的EventDispatcher事件分发组件,下面介绍一下EventDispatcher 因从cocos2d转到unity开发，习惯使用EventDispatcher对程序功能进行分层设计，故找到C#的实现，代码来源于网络。主要包括IEvent,IEventDispatcher,Event,EventDispatcher等接口或类，使用的方式在模块处理中声明：

 //事件分发
 private EventDispatcher _eventDispatcher;

在UI模块中注册

MsgModule.Instance.EventDispatcher.addEventListener(MsgModule.EVENT\_GAME\_NOTICE, _onGameNoticeEvent);

获取网络消息后推送

CustomEvent customEvent = new CustomEvent(EVENT\_GAME\_NOTICE);
customEvent.Data = GameNotice;
EventDispatcher.dispatchEvent(customEvent);

实现应用逻辑层和UI层代码的解耦，减少维护复杂度

*   C#中的委托与事件

先看一下委托与事件的[总结](https://www.cnblogs.com/darrenji/archive/2014/09/11/3967381.html):

1.  委托就是一个类，也可以实例化，通过委托的构造函数来把方法赋值给委托实例
2.  触发委托有2种方式: 委托实例.Invoke(参数列表)，委托实例(参数列表)
3.  事件可以看作是一个委托类型的变量
4.  通过+=为事件注册多个委托实例或多个方法
5.  通过-=为事件注销多个委托实例或多个方法
6.  EventHandler就是一个委托

通过委托定义的事件，也能通过+=或-=订阅或取消订阅事件。 定义委托

public delegate bool OnSocketConnectEvent(int iErrorCode, string pszErrorDesc);
public delegate bool OnSocketReadEvent(CMD_Command Command, byte\[\] pBuffer, short wDataSize);
public delegate bool OnSocketCloseEvent();
public delegate bool OnSocketErrorEvent(object data);

事件定义

public event OnSocketConnectEvent onSocketConnect;
public event OnSocketCloseEvent onSocketClose;
public event OnSocketErrorEvent onSocketError;

注册事件和事件处理

//网络异常处理
HandlerObservable.Instance.onSocketConnect += OnSocketConnect;
HandlerObservable.Instance.onSocketClose += OnSocketClose;
HandlerObservable.Instance.onSocketError += OnSocketError;

 // 网络异常处理
    private bool OnSocketConnect(int iErrorCode, string pszErrorDesc)
    {

        return false;
    }

    private bool OnSocketClose()
    {
        return true;
    }

    private bool OnSocketError(object data)
    {
        return false;
    }

取消事件

HandlerObservable.Instance.onSocketConnect -= OnSocketConnect;
HandlerObservable.Instance.onSocketClose -= OnSocketClose;
HandlerObservable.Instance.onSocketError -= OnSocketError;