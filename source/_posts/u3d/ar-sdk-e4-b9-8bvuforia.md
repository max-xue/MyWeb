---
title: Unity 开发之---问题汇总
categories:
  - U3D
url: 296.html
id: 296
date: 2017-04-01 10:17:16
tags:
  - u3d
  - 总结
---

- 无法识别：参照官方提供的示例Vuforia-3-ImageTargets，实现自定义的Target时确保下面选项勾选： ![](http://www.le-more.com/wp-content/uploads/2017/04/Vuforia-_AR_DataLoad.png)
    如果不选中，是无法成功识别的！

- Vuforia导出为iOS工程运行报错 错误信息为:

        /BuildRoot/Library/Caches/com.apple.xbs/Sources/Metal/Metal-85.83/ToolsLayers/Debug/![](file:///C:\Users\maxx\AppData\Local\Temp\%W@GJ$ACOF(TYDYECOKVDYB.png)MTLDebugCommandBuffer.mm:314: failed assertion `MTLRenderPassDescriptor MTLStoreActionMultisampleResolve store action requires resolve texture'

    问题原因：项目工程导出环境为Windows 10 解决办法：将项目在Mac OS X下导出不再报错

- 打包安装APK，报错：

        Failure to initialize! Your hardware does not support this application, sorry! 
        
    Unity项目中导入MovieTexture资源，移动端是不支持MovieTexture的。移除后正常 还有其他原因，这个是我遇到的问题！
    
- 打包安装APK,报错：Android解析包时出现问题Player Settings->Other Settings->设置适当的Mininum API Level

- 打包APK,报错：Unable to merge android manifests Player Settings->Other Settings->设置适当的Mininum API Level 以前还遇到API Level小于要合并targetSdkVersion,就是编辑器的选项最大的也不满足，搜索所有Manifest文件，修改一致！
- 查看Android Debug Log 工具：\\AndroidSDK\\platform-tools\\adb.exe 命令：
    
    **-s 指定过滤器**
    
            //-s 指定过滤器
            adb logcat -s Unity ActivityManager PackageManager dalvikvm DEBUG
    

//如果出现error: more than one device/emulator，需要adb -s deviceName指定设备,
        adb devices 得到设备名 MyAndroid
        adb -s MyAndroid logcat -s Unity

//-f 输出log到指定文件
adb -s deviceName logcat -s Unity  -f c:\\unity_log.txt

- 将 Java 代码做成 Unity 插件 参考：[在 Unity 中使用 Android SDK](http://www.tuicool.com/articles/qmmeemR)

- Unity 打包Android贴图模糊 1.Edit->Player Settings->QualitySettings设置Quality Level 2.纹理属性设置（属性详情参考：[Unity3D Shader学习之九—纹理](http://www.le-more.com/?p=620)） ![](http://www.le-more.com/wp-content/uploads/2016/10/unity_export_01.png) ![](http://www.le-more.com/wp-content/uploads/2016/10/unity_export_02.png)

- 自定义Skybox有接缝

修改纹理Wrap Mode为Clamp

- 设置相同的对象不碰撞 设置对象所属Layer，在Edit--Project Setting--Physics 设置Layer之间是否有相互碰撞关系
- Unable to install APK to device. Please make sure the Android SDK is installed and is properly configured in the Editor! adb: error: failed to copy Package.apk: no response: Connection reset by peer 更新Android SDK
- Skybox
    
    //修改
    void Update () {
            float num = RenderSettings.skybox.GetFloat("_Rotation");
            RenderSettings.skybox.SetFloat("_Rotation", num + 0.01f);
    }
    
    //动态加载 资源需放到Resources目录
    RenderSettings.skybox = Resources.Load<Skybox>(skyboxName).material;
    RenderSettings.skybox = Resources.LoadAll(skyboxName)\[0\] as Material;
    
- 修改程序材质
    
    ProceduralMaterial mat = uiObject.GetComponent<Renderer>().material as ProceduralMaterial; ;
    //mat.SetProceduralFloat("Luminence", 0);
    mat.SetProceduralFloat("Coral\_Shape\_Tiling", life + 1);
    
    mat.RebuildTextures();
    //mat.RebuildTexturesImmediately();
    
- 制作天空盒 全景图创建天空盒：pano2vr 星空创建天空盒：[Spacescape](http://alexcpeterson.com/spacescape/)
- VR 躺着看场景，视角和正常一样（MojingSDK ） x轴旋转90度，在代码中动态设置，否则视点偏移
    
    mojingMain = GameObject.Find("MojingMain");
    mojingMain.transform.rotation = Quaternion.Euler(90, 0, 0);
    
    启动应用前，手机屏要朝下
- Canves层次关系 设置Sort Order
- Vuforia 相机对焦 [Vuforia应用之相机自动对焦功能](http://blog.csdn.net/caizedong1990/article/details/47169513)
- 模型动画找不到，无法播放 （The animation state "xxx" could not be played because it couldn't be found"） 选择模型->Rig->Animation Type 设置为Legacy
- android plugin添加权限 在Assets\\Plugins\\Android目录下创建AndroidManifest.xml文件，package要和unity设置一致。如果不知道如何创建，可以在Temp文件夹下搜索到（要打包一次后才生成），复制AndroidManifest.xml，修改添加权限即可！
- 暴风魔镜SDK,程序中切成单屏，使用小米4测试，陀螺仪会卡住 设置-电池和性能-神隐模式关掉
- 相同的场景，显示的效果不一样 选择一个色彩空间(CHOOSING A COLOR SPACE)，场景使用不同的色彩空间Gamma 或Linear
-   Serializable属性->Inspector中显示自定义类或结构
    
    \[System.Serializable\]
    public class ChairColor {
       public Color inColor;
       public Color outColor;
    }
    
    \[System.NonSerialized\] 不被序列化该变量，且不显示在检视面板中。 \[HideInInspector\] 在检视面板中隐藏 RequireComponent 必须要有相应的组建 SerializeField 序列化域(强制序列化) ExecuteInEditMode 在Editor模式下运行 \[ContextMenu\] 上下文菜单 \[AddComponentMenu\] 添加组件菜单
- VS 中的快捷键 Ctr+R,Ctr+E 属性自动生成拾取器 引入命名空间：光标移动到目标，Shilt+Alt+F10 移除不用空间：右键 -> 组织 using -> 移除和排序
- CullMask

        camera.cullingMask = ~(1 << x);  // 渲染除去层x的所有层    
              
        camera.cullingMask &= ~(1 << x); // 关闭层x    
              
        camera.cullingMask |= (1 << x);  // 打开层x    
            
        camera.cullingMask = 1 << x + 1 << y + 1 << z; // 摄像机只显示第x层,y层,z层.
    
     
- Ran out of trampolines of type 2 in Player Settings AOT中输入：
    
        nrgctx-trampolines=8096,nimt-trampolines=8096,ntrampolines=4048
    
    参考：["Ran out of trampolines of type 0/1/2" 运行时间错误](http://blog.csdn.net/yupu56/article/details/77888071)
- UnityEngine.dll 路径 Mac:Applications/Unity.app/Contents/Frameworks/Managed/UnityEngine.dll Win:Program Files\\Unity\\Editor\\Data\\Managed\\UnityEngine.dll
- VS 生成后事件 copy "$(TargetDir)*.dll" "..\\..\\bin\\"   //整个文件夹拷贝 copy "$(TargetDir)$(TargetFileName)" "..\\..\\bin\\$(TargetFileName)"   //单文件拷贝
- Python import xlrd 出错，需要安装xlrd模块 Python3命令：pip3 install xlrd Pyhone2命令：pip install xlrd pip位于\\Python27\\Scripts目录
- Unity导出iOS支持arm64位 player settings设置Scripting Backend 为IL2CPP,Architecture选择ARM64或Universal
- Unity 在macOS下打开黑屏 复制正常Mac电脑下的目录 ~/资源库/Unity/Packages到有问题的电脑
- Unity打开项目为空 新建立一个磁盘分区，格式为Mac OS日志扩展类型，将项目移到这个磁盘 注意Unity也需要安装（或复制）到这个磁盘
- 打包iOS报错："AssemblyResolutionException: Failed to resolve assembly: 'UnityEngine, Version=0.0.0.0, Culture=neut" 注意Unity也需要安装（或复制）到这个磁盘 ,运行这个磁盘的Unity
- 验证签名 jarsigner -verify -verbose -certs xxx.apk
- 破解 使用DisUnity拆包，DotNetReflector很简单地进行反编译，再通过dot4net进行反混淆
- android-ndk-r10e-darwin-x86\_64.bin 解压 更改权限，然后解压 chmod a+x android-ndk-r10e-darwin-x86\_64.bin ./android-ndk-r10e-darwin-x86_64.bin