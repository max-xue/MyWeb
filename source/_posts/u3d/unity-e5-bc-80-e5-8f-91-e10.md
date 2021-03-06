---
title: Unity开发源码的加解密二一打包自动加密
categories:
  - U3D
url: 1326.html
id: 1326
comments: true
date: 2018-06-08 21:29:18
tags:
  - u3d
---

在前一篇[Unity开发源码的加解密](http://www.le-more.com/?p=1311) 涉及的内容比较多，最终得到了加密后的Assembly-CSharp.dll和解密libmono.so。如何能在打包的时候自动处理呢？下面介绍一种方式。 \[PostProcessBuild\]标签：Add this attribute to a method to get a notification just after building the player. 作用就是获取打包后通知，执行添加后的方法. 处理函数如下：
    
    \[PostProcessBuild\]
    public static void OnPostprocessBuild (BuildTarget target, string pathToBuiltProject)
    {
        if (target == BuildTarget.Android)
        {
            OnPostprocessBuildAndroid(pathToBuiltProject);
        }
        else if (target == BuildTarget.iOS)
        {
            OnPostprocessBuildIOS(pathToBuiltProject);
        }			
    }

看一下iOS具体的处理，可以添加引用库，修改info.plist等，在Build iOS项目的时生成xCode工程时会自动处理，方便iOS打包

     private static void OnPostprocessBuildIOS(string pathToBuiltProject)
        {
    #if UNITY\_EDITOR\_OSX
            PBXProject project = new PBXProject();
            string configFilePath = PBXProject.GetPBXProjectPath(pathToBuiltProject);
            project.ReadFromFile(configFilePath);
           
            string targetGuid = project.TargetGuidByName("Unity-iPhone");
            string debug = project.BuildConfigByName(targetGuid, "Debug");
            string release = project.BuildConfigByName(targetGuid, "Release");
            project.AddBuildPropertyForConfig(debug, "CODE\_SIGN\_RESOURCE\_RULES\_PATH", "$(SDKROOT)/ResourceRules.plist");
            project.AddBuildPropertyForConfig(release, "CODE\_SIGN\_RESOURCE\_RULES\_PATH", "$(SDKROOT)/ResourceRules.plist");
    
            project.AddFrameworkToProject(targetGuid, "GLKit.framework", true);
            project.SetBuildProperty(targetGuid, "ENABLE_BITCODE", "NO");
            project.WriteToFile(configFilePath);
    
            //plist
            string plistPath = pathToBuiltProject + "/Info.plist";
            PlistDocument plist = new PlistDocument();
            plist.ReadFromString(File.ReadAllText(plistPath));
    
            // Get root
            PlistElementDict rootDict = plist.root;
            
            /\* ipad 关闭分屏 */
            rootDict.SetBoolean("UIRequiresFullScreen", true);
            rootDict.SetString("CFBundleShortVersionString", App.Global.GlobalDef.BundleVer);
    
            /\* 设置Build值 */
            var now = System.DateTime.Now;
            string time = string.Format("{0}{1:D2}{2:D2}{3:D2}{4:D2}", now.Year, now.Month, now.Day, now.Hour, now.Minute);
            rootDict.SetString("CFBundleVersion", string.Format("{0}.{1}", App.Global.GlobalDef.BundleVer, time));
    
            /\* iOS9所有的app对外http协议默认要求改成https */
            // Add value of NSAppTransportSecurity in Xcode plist
            var atsKey = "NSAppTransportSecurity";
            PlistElementDict dictTmp = rootDict.CreateDict(atsKey);
            dictTmp.SetBoolean("NSAllowsArbitraryLoads", true);
    
            // location native development region 
            //rootDict.SetString("CFBundleDevelopmentRegion", "zh_CN");
            
            /\* for share sdk */
            rootDict.SetString("NSPhotoLibraryUsageDescription", "AR拍照保存与分享");
            rootDict.SetString("NSPhotoLibraryAddUsageDescription", "AR拍照保存与分享");
            
            /\* 地图定位 */
            rootDict.SetString("NSLocationWhenInUseUsageDescription"
            , "您的位置和方向信息用于乐园中导航,实现虚拟乐园和真实乐园的对应");
    
            /\* ITSAppUsesNonExemptEncryption 合规证明，未使用加密 */ 
            rootDict.SetBoolean("ITSAppUsesNonExemptEncryption", false);
    
            File.WriteAllText(plistPath, plist.WriteToString());
            AdapteiPhoneXCode(pathToBuiltProject);
    #endif
        }

类似的，Build Android项目时，处理如下：
    
    private static void OnPostprocessBuildAndroid(string pathToBuiltProject)
    {
        string exportedPath = Path.Combine(pathToBuiltProject, PlayerSettings.productName);
    
        //加密
        string\[\] filesToCrypt = new\[\] { "Assembly-CSharp.dll"};
        for (int i = 0; i < filesToCrypt.Length; i++)
        {
            string fileName = filesToCrypt\[i\];
            string filePath = UnityLibs.Common.FileUtils.SearchFile(exportedPath, fileName);
            if (!string.IsNullOrEmpty(filePath))
            {
                UnityLibs.DEncrypt.DEncrypt.EncryptFile(filePath, "", App.Global.GlobalDef.DEncrypt\_Data\_Key + "~");
            }
        }
    
        //替换
        string\[\] srcToReplace = new\[\] { "../Client/Publish/libmono.so" };
        string\[\] desToReplace = new\[\] { "libmono.so" };
        for (int i = 0; i < desToReplace.Length; i++)
        {
            string destName = desToReplace\[i\];
            string destPath = UnityLibs.Common.FileUtils.SearchFile(exportedPath, destName);
            if (!string.IsNullOrEmpty(destPath))
            {
                //delete
                UnityLibs.Common.FileUtils.DeleteFile(destPath);
    
                //copy
                string src = srcToReplace\[i\];
                string srcFull = Path.GetFullPath(src);
                UnityLibs.Common.FileUtils.CopyFile(srcFull, destPath, true);
            }
        }
    }

说明： 

1.需要实现互通的加解密方式：C#加密，C解密； 

2.文件仅做了加密，未做混淆处理； 先简单实现，后优化提高！