---
title: Unity开发之工具一技能参数配置
categories:
  - U3D
url: 1376.html
id: 1376
comments: true
date: 2018-07-05 21:07:24
tags:
  - u3d
---

在开发游戏过程中，涉及到各种数值和特效的调整，就需要程度开发个工具方便策划或特效调整数值，下面介绍一下调整技能数值的工具。 说是工具，其实就是一个场景，提供一参数配置，就长这样： ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_skill_1.jpg) 就这样？还是那句话：能解决问题的工具就是好工具 看一技能表中的数据配置有哪些需要调整的： ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_skill_2.png) 可以看到需要调整的条目还是很多的，从对象的角度看就两个，一个是具体的技能，一个是所属的角色，所以工具脚本中提供 Fairy File和Skill SID,确定这两个其他的数值都有了，然后在运行过程中调整即可 从Excel中导表到程序中可以查看**[Unity开发之导表（Excel）工具的制作分析](http://www.le-more.com/?p=1154)，**可方便在Excel配置数据然后导入到程序中使用。 如何在程序运行时调整，在程序逻辑中找一个时点，加载最新的配置即可，比如在这个工具中每个回合开始从界面上加载一下，然后开始新的回合。 看一运行时的画面 ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_skill_3.png) 动图 ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_skill_4.gif) 这个工具是在游戏中战斗场景基础上修改的，加了测试的逻辑，可模拟真实战斗的情况