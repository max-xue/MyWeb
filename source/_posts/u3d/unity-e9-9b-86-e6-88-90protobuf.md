---
title: Unity开发之网络---集成Protobuf
categories:
  - U3D
url: 1100.html
id: 1100
comments: true
date: 2017-10-09 18:31:23
tags:
  - u3d
  - network
---

1.  下载protobuf 源码　[mgravell/protobuf-net](https://github.com/mgravell/protobuf-net)
2.  复制protobuf-net-master\\src\\protobuf-net 目录到Assert下的任意目录
3.  Assert目录下创建smcs.rsp和gmcs.rsp 文件内容都是：-unsafe (可用文本文件创建然后改名，后缀为rsp)
4.  打开protobuf-net-master\\src下的protobuf-net.sln 编译生成ProtoGen 可执行文件(使用2015打开报错，可将protobuf-net-master\\assorted\\ProtoGen复制替换protobuf-net-master\\src下的同名目录，2015可打开，修改该项目下的引用protobuf-net-master\\assorted\\protobuf-net.Enyim\\packages\\protobuf-net.2.0.0.602\\lib\\net20下的protobuf.dll)
5.  测试脚本，将下面脚本存为generator.bat复制到protogen.exe相同目录，创建protobuf目录后双击执行，成功会在protobuf目录生成descriptor.cs源文件
    
    @echo off  
    set out_path=%cd%/protobuf
    set in_path = %cd%/protos
    rem cd ProtoGen  
    rem 查找文件   
    for /R "%cd%" %%i in (%in_path%*.proto) do echo %%~ni     
    for /R "%cd%" %%i in (%in\_path%*.proto) do protogen -i:%%i -o:%out\_path%/%%~ni.cs    
    pause
    
6.  编写自动化脚本，脚本目录请根据项目情况而定
    
    %cd%\\..\\..\\..\\Tools\\ProtoGen\\protogen.exe -i:protos\\Common.proto -o:protobuf\\Common.cs
    
    xcopy protobuf %cd%\\..\\..\\..\\Code\\ProjName\\Assets\\Scripts\\App\\Net\\Proto /s /f /q
    xcopy protobuf  %cd%\\..\\..\\..\\Code\\Server\\GameServer\\App\\Net\\Proto /s /f /q
    
7.  序列化
    
       /// 将消息序列化为二进制的方法
            /// < param name="model">要序列化的对象< /param>
            public static byte\[\] Serialize<T>(T model)
            {
                try
                {
                    //涉及格式转换，需要用到流，将二进制序列化到流中
                    using (MemoryStream ms = new MemoryStream())
                    {
                        //使用ProtoBuf工具的序列化方法
                        ProtoBuf.Serializer.Serialize<T>(ms, model);
                        //定义二级制数组，保存序列化后的结果
                        byte\[\] result = new byte\[ms.Length\];
                        //将流的位置设为0，起始点
                        ms.Position = 0;
                        //将流中的内容读取到二进制数组中
                        ms.Read(result, 0, result.Length);
                        return result;
                    }
                }
                catch (Exception ex)
                {
                    UnityLibs.Debuger.Log("序列化失败: " + ex.ToString());
                    return null;
                }
            }
    
    测试实例：
    
     ReqLogin reqLogin = new ReqLogin();
            reqLogin.user_name =userName;
            reqLogin.password = passWord;
            reqLogin.type =accoutType; 
     byte\[\] buffer = Utils.Serialize<ReqLogin>(req);
    
8.  反序列化
    
    /// 将收到的消息反序列化成对象
            /// < returns>The serialize.< /returns>
            /// < param name="msg">收到的消息.</param>
            public static T Deserialize<T>(byte\[\] msg, int dataSise)
            {
                try
                {
                    using (MemoryStream ms = new MemoryStream())
                    {
                        //将消息写入流中
                        ms.Write(msg, 0, dataSise);
                        //将流的位置归0
                        ms.Position = 0;
                        //使用工具反序列化对象
                        T result = ProtoBuf.Serializer.Deserialize<T>(ms);
                        return result;
                    }
                }
                catch (Exception ex)
                {
                    UnityLibs.Debuger.Log("反序列化失败: " + ex.ToString());
                    return default(T);
                }
            }
    
    测试实例
    
    ReqLogin req = FileUtils.Deserialize<ReqLogin>(pBuffer, wDataSize);
    

为了便于重用，可将源文件放到自定义类库，请参考：[Unity开发类库封装与调用](http://www.le-more.com/?p=1186)