---
title: C++开发之--问题汇总
comments: true
categories:
  - C++
tags:
  - 总结
---

*   1\. 'strcpy': This function or variable may be unsafe. Consider using strcpy\_s instead. To disable deprecation, use \_CRT\_SECURE\_NO\_WARNINGS. 解决：1> 根据下面的warning提示,使用strcpy\_s 2> 项目|属性|配置属性|C/C++|命令行|附加选项,加入【/D "\_CRT\_SECURE\_NO\_DEPRECATE" 】