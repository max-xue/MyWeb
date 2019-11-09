---
title: Unity开发之Shader—AR涂涂乐项目实战
categories:
  - U3D
url: 1093.html
id: 1093
comments: true
date: 2017-10-09 15:39:59
tags:
  - u3d
  - ar
---

[Unity3D Shader示例之—AR涂涂乐实现原理](http://www.le-more.com/?p=280) [Unity3D Shader示例之—AR涂涂乐实现方法](http://www.le-more.com/?p=369) 最近一段时间忙于项目，日志好久没有更新了，现在挤点时间来一发！ 曾经对AR涂涂乐的原理进行了分析（详见开头链接），但和实际项目还有一些差距。

*   主角显示在屏幕中间
*   脱卡
*   模型接受光照

下面分别解决这些问题！ 要实现追踪目标显示的屏幕中间很简单，一句话搞定：

    _trackingObj.transform.parent = VuforiaManager.Instance.ARCameraTransform;

如何实现脱卡呢，需要解决两个问题，第一个是离开识别卡，追踪目标不消失，第二个是成功完成一次着色后，不再着色，否则获取的颜色就不是我们涂的卡片上的颜色了。 第一个问题好解决，在接收到Lost事件时，不清除追踪目标就行了
    
    public void OnTrackingLost()
        {
            //Clear();
        }

第二个问题解决的原理是，在目标被识别时获取摄像头所拍摄的画面作为纹理，然后将纹理传给材质球。 获取摄像机纹理：

        if (_texture)
            DestroyImmediate(_texture);

        _texture = new Texture2D((int)rect.width, (int)rect.height, TextureFormat.RGB24, false);
        //读取屏幕像素信息并存储为纹理数据  
        _texture.ReadPixels(rect, 0, 0);
        _texture.Apply();

在Shader的顶点着色器代码中，将世界坐标转为屏幕坐标：

    o.fixedPos = ComputeScreenPos(mul(UNITY\_MATRIX\_VP, fixedPos));

UNITY\_MATRIX\_VP会根据Camera实时计算最新的转换矩阵，所以在获取纹理的现时将转换矩阵保存并转给材质球

      //获取VP值
            Matrix4x4 P = GL.GetGPUProjectionMatrix(Camera.main.projectionMatrix, false);
            Matrix4x4 V = Camera.main.worldToCameraMatrix;
            Matrix4x4 VP = P * V;

完整代码 [TrackingBehaviour](https://github.com/max-xue/Coloring3D/blob/master/Assets/Coloring3D/Scripts/TrackingBehaviour_VP.cs) 光照就不介绍了直接从入门精要复制来的Blinn-Phong光照模型 [Coloring3D\_VP\_Lighting.shader](https://github.com/max-xue/Coloring3D/blob/master/Assets/Coloring3D/Shaders/Coloring3D_VP_Lighting.shader) 完整的实现方式请查看Github上的ARProject项目! 下面总结一下开发过程中会遇到一些问题：

*   ARCamera的World Center Mode必须是First Target
*    获取识别图4个角的世界坐标，注意调用TransformPoint函数的对象，示例使用_controller
    
            Vector3 targetAnglePoint1 = _controller.transform.TransformPoint(new Vector3(-halfSize.x, 0, halfSize.y));
            Vector3 targetAnglePoint2 = _controller.transform.TransformPoint(new Vector3(-halfSize.x, 0, -halfSize.y));
            Vector3 targetAnglePoint3 = _controller.transform.TransformPoint(new Vector3(halfSize.x, 0, halfSize.y));
            Vector3 targetAnglePoint4 = _controller.transform.TransformPoint(new Vector3(halfSize.x, 0, -halfSize.y));
    
*   识别成功时，注意重置目标的坐标
    
        //重置坐标
        //this.transform.parent.localPosition = Vector3.zero;
        //this.transform.parent.localRotation = Quaternion.Euler(0, 0, 0);
        this.transform.localPosition = Vector3.zero;
        this.transform.localRotation = Quaternion.Euler(0, 0, 0);
    
*   初始化TrackableEventHandler状态为NOT_FOUND
    
        mTrackableBehaviour.OnTrackerUpdate(TrackableBehaviour.Status.NOT_FOUND);
    

Show一下在项目中实现的效果： ![](http://www.le-more.com/wp-content/uploads/2017/10/ar_coloring_project.gif) [示例工程](https://github.com/max-xue/Coloring3D)