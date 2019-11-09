---
title: Unity开发源码的加解密一mono.dll和libmono.so编译
categories:
  - U3D
url: 1311.html
id: 1311
comments: true
date: 2018-06-07 21:02:14
tags:
  - u3d
---

源码的加密，只是加了一层防护，不是绝对的安全，参考网络大部份类似的实现方式：

1.  混淆自定义库
2.  加解密Assembly-CSharp.dll
3.  自动化工具集成

#### 一、使用各种加密方法制作C++动态库DEncrypt.dll

加密方法,使用xxtea示例
    
    DLL\_INTERFACE\_API void Encrypt(const char * key, const char * fileName, const char* fileSuffix)
    {
        //带后缀的文件名
        char fileNameExt\[64\] = { 0 };
        //没有后缀的文件名
        char name\[64\] = { 0 };
        //文件后缀
        char fileExt\[8\] = { 0 };
        //路径名，不含有文件名
        char filePath\[512\] = { 0 };
        const char*pFind = nullptr;
        strcpy(fileNameExt, (pFind = strrchr(fileName, '\\\')) ? pFind + 1 : fileName);
        strncpy(filePath, fileName, strlen(fileName) - strlen(fileNameExt));
    
        pFind = nullptr;
        strcpy(fileExt, (pFind = strrchr(fileNameExt, '.')) ? pFind : fileNameExt);
        strncpy(name, fileNameExt, strlen(fileNameExt) - strlen(fileExt));
    
        //判断命令行是否正确	
        FILE *infp = 0;
        if ((infp = fopen(fileName, "rb")) == NULL)
        {
            //打开操作不成功
            printf("%s Read Error \\n", fileNameExt);
            //结束程序的执行
            return;
        }
    
        char* buffer = (char*)malloc(sizeof(char)*SIZE);
        memset(buffer, 0, sizeof(char)*SIZE);
    
        int rc = 0;
        int total_len = 0;
    
        total_len = fread(buffer, sizeof(unsigned char), SIZE, infp);
        printf("Read %s Successfully and total\_len : %d  \\n", fileNameExt, total\_len);
    
        //加密DLL
        size_t len;
        char \*encrypt\_data = (char\*)xxtea\_encrypt(buffer, total_len, key, &len);
        printf("Encrypt %s Successfully and len : %d \\n", fileNameExt, len);
    
        //写Dll
        strcat(name, fileSuffix);
        strcat(name, fileExt);
        strcat(filePath, name);
    
        FILE* outfp = 0;
        if ((outfp = fopen(filePath, "wb+")) == NULL)
        {
            //打开操作不成功
            printf("%s Read Error \\n", fileNameExt);
            //结束程序的执行
            return;
        }
    
        int rstCount = fwrite(encrypt_data, sizeof(unsigned char), len, outfp);
    
        fflush(outfp);
    
        printf("Write len : %d \\n", rstCount);
    
        fclose(infp);
        fclose(outfp);
    
        free(buffer);
        free(encrypt_data);
    }

解密方法
    
    //解密
    //fileName:路径+文件名
    DLL\_INTERFACE\_API void Decrypt(const char * key, const char * fileName, const char* fileSuffix)
    {
        //带后缀的文件名
        char fileNameExt\[64\] = { 0 };
        //没有后缀的文件名
        char name\[64\] = { 0 };
        //文件后缀
        char fileExt\[8\] = { 0 };
        //路径名，不含有文件名
        char filePath\[512\] = { 0 };
        const char*pFind = nullptr;
        strcpy(fileNameExt, (pFind = strrchr(fileName, '\\\')) ? pFind + 1 : fileName);
        strncpy(filePath, fileName, strlen(fileName) - strlen(fileNameExt));
    
        pFind = nullptr;
        strcpy(fileExt, (pFind = strrchr(fileNameExt, '.')) ? pFind : fileNameExt);
        strncpy(name, fileNameExt, strlen(fileNameExt) - strlen(fileExt));
    
        //判断命令行是否正确
        FILE *infp = 0;
        if ((infp = fopen(fileName, "rb")) == NULL)
        {
            //打开操作不成功
            printf("%s Read Error \\n", fileNameExt);
            //结束程序的执行
            return;
        }
    
        char* buffer = (char*)malloc(sizeof(char)*SIZE);
        memset(buffer, 0, sizeof(char)*SIZE);
    
        int rc = 0;
        int total_len = 0;
    
        total_len = fread(buffer, sizeof(unsigned char), SIZE, infp);
        printf("Read File %s Successfully and total\_len : %d  \\n", fileNameExt, total\_len);
    
        //解密DLL
        size_t len;
        char \*decrypt\_data = (char\*)xxtea\_decrypt(buffer, total_len, key, &len);
        printf("Decrypt %s Successfully and len : %d \\n", fileNameExt, len);
    
        //写Dll
        //去除_encrypt
        char newName\[64\] = { 0 };
        pFind = nullptr;
        if (pFind = strstr(name, fileSuffix))
            strncpy(newName, name, strlen(name) - strlen(fileSuffix));
        //全路径
        strcat(newName, fileExt);
        strcat(filePath, newName);
        FILE* outfp = 0;
        if ((outfp = fopen(filePath, "wb+")) == NULL)
        {
            //打开操作不成功
            printf("%s Read Error \\n", fileNameExt);
            //结束程序的执行
            return;
        }
    
        int rstCount = fwrite(decrypt_data, sizeof(unsigned char), len, outfp);
        fflush(outfp);
    
        printf("Write len : %d \\n", rstCount);
    
        fclose(infp);
        fclose(outfp);
    
        free(buffer);
        free(decrypt_data);
    }

检查导出的方法

    dumpbin -exports DEncrypt.dll

除了xxtea，也可以使用des,aes等对称加密方式 ![](http://www.le-more.com/wp-content/uploads/2018/06/u3d_dencrypt_01.png) 参考：[C++动态库制作](https://blog.csdn.net/a7055117a/article/details/47733247)

#### 二、使用C#调用DEncrypt.dll加密Assembly-CSharp.dll

C#代码引用DLL
    
    \[DllImport("DEncrypt.dll", CallingConvention = CallingConvention.Cdecl)\]
    public static extern void Encrypt(string key, string fileName, string fileSuffix);
    
    \[DllImport("DEncrypt.dll", CallingConvention = CallingConvention.Cdecl)\]
    public static extern void Decrypt(string key, string fileName, string fileSuffix);

调用，会在dll文件目录生成带指定后缀的加密后文件，如果后缀为空，替换文件
    
    private void btnDEcryptDLL_Click(object sender, EventArgs e)
    {
        //加密
        Encrypt(txtDEncryptKey.Text.Trim(), txtDEcryptDLL.Text.Trim(), txtEncryptSuffix.Text.Trim());
        System.Windows.Forms.MessageBox.Show("加密成功！");
    }

#### 三、编译mono.dll

1.下载[mono](https://github.com/Unity-Technologies/mono) 对应版本(当前项目是uniyt 5.5) 2.修改mono\_image\_open\_from\_data\_with\_name方法，添加解密代码,在加密的时候加个前缀，如果存在就解不存在不执行
    
    if (!data || !data_len) {
        if (status)
            *status = MONO\_IMAGE\_IMAGE_INVALID;
        return NULL;
    }
    ...................添加开始
    //maxx-m
        if (strstr(name, "Assembly-CSharp.dll")) {
            char* key = "demo2018";
            short size = strlen(key)*0.5;
            if(strncmp(key,data,size) == 0){
                size_t len;
                char* decryptData = (char *)DecryptData(key,data+size, data_len-size, &len);
                int i = 0;
                for (i = 0; i < len; ++i)
                {
                    data\[i\] = decryptData\[i\];
                }
                g_free(decryptData);
    
                data_len = len;
            }
        }
    ...................添加结束

原本计划写个静态库加解密方法写在一个文件中，方便维护（也是第一步制作dll的原因）。因技能能力有限，试了N次静态库链接有问题，遂放弃，将源码复制到mono项目中mono\\metadata\\crypt  crypt是创建的目录，将添加的所有源码复制到这个目录，添加到工程中 ![](http://www.le-more.com/wp-content/uploads/2018/06/u3d_dencrypt_02.png) 3.编译mono.dll 需要安装VS2010 编写脚本BuildMono.bat，编译32位和64位dll，执行成功后会在\\builds\\embedruntimes 生成win32和win64位目录，对应着32和64位的mono.dll
    
    @set path=C:\\Windows\\Microsoft.NET\\Framework\\v4.0.30319;%path%
    
    @cls
    msbuild.exe "mono-unity-5.5\\msvc\\mono.sln" /p:Configuration=Release_eglib /p:Platform=Win32  
    msbuild.exe "mono-unity-5.5\\msvc\\mono.sln" /p:Configuration=Release_eglib /p:Platform=x64
    pause;

在打包window项目时，用新的mono.dll替换打包后的XXX_Data\\Mono\\mono.dll文件,windows不是重点，不再描述。

#### 四、编译libmono.so

编译libmono.so比较麻烦，Window和macOS都需要配置环境，下面说一下windows 参考 [Unity防破解 —— 重新编译mono](https://www.cnblogs.com/lixiang-share/p/5978107.html) 1.下载[mingw-get-setup.exe](https://sourceforge.net/projects/mingw/files/) 安装到C盘或D盘根目录 在basic 全选安装 2.下载[Android NDK](http://dl.google.com/android/ndk/android-ndk-r10e-windows-x86_64.exe),解压到C:\\MinGW\\msys\\1.0\\home\\maxx\\android-ndk_auto-r10e 如果home\\maxx没有，创建即可,注意ndk的目录名称要一致 3.下载[zip](http://files.cnblogs.com/files/lixiang-share/zip.zip),解压到C:\\MinGW\\msys\\1.0\\bin 4.修改
    
    mono-unity-5.5\\external\\buildscripts\\PrepareAndroidSDK.pm
    
    elsif(lc $^O eq 'cygwin') 改成
    
    elsif(lc $^O eq 'cygwin' or lc $^O eq 'msys')

5.修改
    
    mono-unity-5.5\\external\\buildscripts\\build\_runtime\_android.sh
    KRAIT\_PATCH\_PATH="${CWD}/../../android\_krait\_signal_handler/build"
    改成
    KRAIT\_PATCH\_PATH="${CWD}/external/buildscripts/android\_krait\_signal_handler/build"
    
    将android\_krait\_signal_handler下载到当前目录

6.修改
    
    mono-unity-5.5\\external\\buildscripts\\build\_runtime\_android.sh
    
    -g 改成 -O2
    
    注释
    #clean\_build "$CCFLAGS\_ARMv5\_CPU" "$LDFLAGS\_ARMv5" "$OUTDIR/armv5"
    #clean\_build "$CCFLAGS\_ARMv6\_VFP" "$LDFLAGS\_ARMv5" "$OUTDIR/armv6_vfp"
    
    添加
    $ANDROID\_NDK\_ROOT/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-strip.exe libmono.so

7.修改
    
    mono-unity-5.5\\external\\buildscripts\\build\_runtime\_android_x86.sh
    
    -g 改成 -O2
    
    添加
    $ANDROID\_NDK\_ROOT/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-strip.exe libmono.so

8.执行
    
    打开MinGW\\msys\\1.0\\msys.bat
    cd到mono-unity-5.5目录;
    
    执行./external/buildscripts/build\_runtime\_android.sh
    
    执行会失败，但会下载文件mono-unity-5.5\\external\\buildscripts\\android\_krait\_signal_handler

9.修改
    
    mono-unity-5.5\\external\\buildscripts\\android\_krait\_signal_handler\\build\\PrepareAndroidSDK.pm
    elsif(lc $^O eq 'cygwin')
    改成
    elsif(lc $^O eq 'cygwin' or lc $^O eq 'msys')

10.修改
    
    mono-unity-5.5\\external\\buildscripts\\android\_krait\_signal_handler\\build\\build.pl
    注释
    #PrepareAndroidSDK::GetAndroidSDK(undef, undef, "r13b");
    
    改第一行成这个，不知道什么意思
    #!/usr/bin/perl -w

11.修改
    
    mono-unity-5.5\\external\\buildscripts\\android\_krait\_signal_handler\\build\\jni\\Application.mk
    
    注释
    #NDK\_TOOLCHAIN\_VERSION := clang

12.再次执行
   
    ./external/buildscripts/build\_runtime\_android.sh，成功后会在mono-unity-5.5\\builds\\embedruntimes\\android 生成armv7a和x86两个文件夹 五.替换libmono.so和加密后的Assembly-CSharp.dll 
    
下一篇会介绍[Unity开发源码的加解密二](http://www.le-more.com/?p=1326)