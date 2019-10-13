---
title: Unity开发之设计模式---状态模式
categories:
  - U3D
url: 1292.html
id: 1292
comments: true
date: 2018-05-21 21:10:20
tags:
---

曾记录一篇关于设计模式的日志：[《Head First 设计模式》------学习总结](http://www.le-more.com/?p=92),在那篇文章中只是记录一些重要的概念，缺乏实战，也不好理解。下面记录在项目中使用到的设计模式，理论和实际相结合才有价值，“把模式装进脑子里，然后在你的设计和已有的应用中，寻找何处可以使用它们。”。 在回合制模式的战斗游戏中，为了很好控制角色的行为，使用有限状态机会容易的多。 quick-lua版本中提供有限状态机实现，是从js移植过来，很容易使用（时间有点久可能记错了），[js版本](https://www.npmjs.com/package/javascript-state-machine) 在C#使用状态机的问题就是类比较多。状态机的实现方式也有很多种，我选择其中一种来实现，一个子状态接口和状态机类，代码来源与网络

 public abstract class IState<Entity>
    {
        public abstract void Enter(Entity subject,object data = null);
        public abstract void Exit(Entity subject);
        public abstract void Update(Entity subject);
    }

public class StateMachine<Entity>
{
    private IState<Entity> _currentState;
    private IState<Entity> _previousState;
    private IState<Entity> _globalState;

    private Entity _Subject;

    public IState<Entity> CurrentState
    {
        get
        {
            return _currentState;
        }

        set
        {
            _currentState = value;
        }
    }

    public IState<Entity> PreviousState
    {
        get
        {
            return _previousState;
        }

        set
        {
            _previousState = value;
        }
    }

    public IState<Entity> GlobalState
    {
        get
        {
            return _globalState;
        }

        set
        {
            _globalState = value;
        }
    }

    // constructor
    public StateMachine(Entity subject)
    {
        this._Subject = subject;
    }

    public void ChangeState(IState<Entity> newState,object data = null)
    {
        //handle input
        if (newState != null)
        {
            PreviousState = CurrentState;

            if (CurrentState != null)
            {
                CurrentState.Exit(_Subject);
            }

            CurrentState = newState;
            CurrentState.Enter(_Subject, data);
        }

    }

    public void Update()
    {
        if (GlobalState != null)
        {
            GlobalState.Update(_Subject);
        }
        if (CurrentState != null)
        {
            CurrentState.Update(_Subject);
        }
    }
}

项目中的需求说明，对战有两方，两方各有几个队员。从战斗开始到结束都是自动运行。 把队员的完整的过程分解为攻击-走/跑-攻击结束，三个状态。因为攻击有近身和远程，可将攻击分为近身攻击和远程攻击状态，走/跑是在近身攻击时才有的状态分为前进和后退两个状态。 全部的状态有：

*   空闲(Idle)
*   近身攻击/远程攻击(AttackClose/AttackRemote)=
*   前进/回退(RunTo/RunBack)
*   受击(Hit)
*   死亡(Dead)
*   胜利(Win)
*   复活(Relive)
*   防御(Defense)
*   停止(Stop)

灰色状态在这个项目中未实现，列出来只是为了后面扩展使用，使用举例：

//状态机
private StateMachine<ActorModel> _fsm;

//开始
public void OnIdle()
{
    _fsm.ChangeState(ActorState.Idle);
}

public class Idle : IState<ActorModel>
{
    public override void Enter(ActorModel subject, object data)
    {
        CustomEvent customEvent = new CustomEvent(ActorModel.ACTOR\_START\_EVENT);
        subject.EventDispatcher.dispatchEvent(customEvent);
    }

    public override void Exit(ActorModel subject)
    {

    }

    public override void Update(ActorModel subject)
    {
    }
}

在Idle状态进入时发送消息给订阅者 其他状态的和Idle都是类似的实现，不同的是发送消息时也可以传参数，由订阅者处理。 最终实现的效果： \[video width="180" height="298" mp4="http://www.le-more.com/wp-content/uploads/2018/05/patten-state.mp4"\]\[/video\]