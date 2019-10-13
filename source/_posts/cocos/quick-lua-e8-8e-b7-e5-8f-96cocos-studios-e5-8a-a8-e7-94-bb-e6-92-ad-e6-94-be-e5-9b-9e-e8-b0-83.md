---
title: quick-lua 获取Cocos Studios动画播放回调
categories:
- Cocos
url: 66.html
id: 66
date: 2016-09-01 18:34:12
tags:
---

quick-lua 版本是3.5 cocos studios是2.0以上，目前是最新版本 在cocos studios中回调使用帧事件也是可以的，就是在关键帧上设置回调事件： ![](http://img.blog.csdn.net/20150917123801699?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  设置的时候，要选中开始记录动画，然后选中关键帧，再设置事件名称。在动画执行到这一帧时会触发该事件，在lua中使用方式为：

    local eventFrameCall = function(frame)
        local eventName = frame:getEvent()
        if eventName == "wait" then
            end
        end
    end

    timeline:clearFrameEventCallFunc()
    timeline:setFrameEventCallFunc(eventFrameCall)

    timeline:play("wait", false)

但这种方式很繁琐，在修改动画的时候很容易忘记有这个事件而出错 在使用cocosbuilder的时候，可以设置动画回调函数。在cocos studios也有这个的回调，但在quick-lua 3.5中还没有支持。所以下面记录另外一种回调方式。 简单说就是获取动画执行时间，手动设置回调方法：

     timeline:play("wait", false)
     local duration = UIHelper.getAnimationDuration(timeline
     local block = cc.CallFunc:create( function(sender)
        callBack()
     end )

    self:runAction(cc.Sequence:create(cc.DelayTime:create(duration), block))

  通过UIHelper.getAnimationDuration方法获取动画执行的时间：

function UIHelper.getAnimationDuration(timeline)
    local speed = timeline:getTimeSpeed()
\-\-    local duration = timeline:getDuration()
    local startFrame = timeline:getStartFrame()
    local endFrame = timeline:getEndFrame()
    local frameNum = endFrame - startFrame

    return 1.0 /(speed * 60.0) * frameNum
end

这样的方法虽然增加一些代码，但减少因编辑器经常更新而造成的问题 完整的UIHelper.lua如下：

UIHelper = {}

\-\- tolua.cast(ccsLayout:getChildByName("server_list") ,"ListView") .

function UIHelper.getControl(root,parentNames)
    local newRoot = root
    for i = 1,#parentNames  do
        local name = parentNames\[i\]
        local child =  newRoot:getChildByName(name)
        if not child then
            newRoot = nil
            break
        else 
            newRoot = child
        end
    end
    
    return newRoot
end

function UIHelper.createNode(parent,file,pos)
    local layout = cc.CSLoader:createNode(file)
    if pos then layout:setPosition(pos) end
    parent:addChild(layout)

    local timeline = cc.CSLoader:createTimeline(file)
    layout:runAction(timeline)

    return layout,timeline
end

function UIHelper.getAnimationDuration(timeline)
    local speed = timeline:getTimeSpeed()
\-\-    local duration = timeline:getDuration()
    local startFrame = timeline:getStartFrame()
    local endFrame = timeline:getEndFrame()
    local frameNum = endFrame - startFrame

    return 1.0 /(speed * 60.0) * frameNum
end

function UIHelper.runTimeline(layout,timeline, animationName,loop)
    if not loop then loop = false end

    if animationName ~= nil and timeline:IsAnimationInfoExists(animationName) then
        timeline:play(animationName, loop)
    else
        timeline:gotoFrameAndPlay(0, loop)
    end
end

--
function UIHelper.runTimelineAndClear(layout,timeline, animationName)
    UIHelper.runAction(layout,timeline, animationName)
    local duration = UIHelper.getAnimationDuration(timeline)
    local block = cc.CallFunc:create( function(sender)
        layout:removeSelf()
    end )

    layout:runAction(cc.Sequence:create(cc.DelayTime:create(duration), block))
end

function UIHelper.runTimelineAndCall(layout,timeline, animationName,callBack)
    local duration = UIHelper.getAnimationDuration(timeline)
    local block = cc.CallFunc:create( function(sender)
        return callBack
    end )

    layout:runAction(cc.Sequence:create(cc.DelayTime:create(duration), block))
end

QQ群：239759131 cocos 技术交流 欢迎您