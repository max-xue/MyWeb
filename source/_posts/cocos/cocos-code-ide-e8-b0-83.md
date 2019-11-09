---
title: quick-3.5 绑定自定义C++类到Lua并使用cocos code ide 调式
categories:
- Cocos
url: 56.html
id: 56
date: 2016-09-01 18:27:41
tags:
---

在这里主要记录怎么绑定自定义C++类到Lua 参考文章在这里[【绑定自定义类至Lua】](http://www.cocos.com/doc/tutorial/show?id=2518)，在我操作过程中略有不同，现在把步骤记录下来。 1.打开引擎目录下\\tools\\tolua\\README.mdown，安装相应软件 2.项目目录\\frameworks\\cocos2d-x\\tools\\tolua,复制cocos2dx_cocosbuilder->custom.ini,genbindings->custom-genbindings.py 3.修改复制后的文件 custion.ini
    
    \[custom_socket\]
    \# the prefix to be added to the generated functions. You might or might not use this in your own
    \# templates
    prefix = custom_socket
    
    \# create a target namespace (in javascript, this would create some code like the equiv. to \`ns = ns || {}\`)
    \# all classes will be embedded in that namespace
    target_namespace =
    
    android_headers = -I%(androidndkdir)s/platforms/android-14/arch-arm/usr/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.7/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.7/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/libs/armeabi-v7a/include -I%(androidndkdir)s/sources/cxx-stl/gnu-libstdc++/4.8/include
    android\_flags = -D\_SIZE\_T\_DEFINED_ 
    
    clang_headers = -I%(clangllvmdir)s/lib/clang/3.3/include 
    clang\_flags = -nostdinc -x c++ -std=c++11 -U \_\_SSE__
    
    cocos_headers = -I%(cocosdir)s -I%(cocosdir)s/cocos -I%(cocosdir)s/cocos/editor-support -I%(cocosdir)s/cocos/platform/android
    
    cocos_flags = -DANDROID
    
    cxxgenerator_headers = 
    
    \# extra arguments for clang
    extra\_arguments = %(android\_headers)s %(clang\_headers)s %(cxxgenerator\_headers)s %(cocos\_headers)s %(android\_flags)s %(clang\_flags)s %(cocos\_flags)s %(extra_flags)s 
    
    \# what headers to parse
    headers = %(cocosdir)s/../runtime-src/Classes/net/SocketModule.h
    
    \# what classes to produce code for. You can use regular expressions here. When testing the regular
    \# expression, it will be enclosed in "^$", like this: "^Menu*$".
    classes = ClientSocket.* CMD_Command.* ConnectHandler.* RecvHandler.*
    
    \# what should we skip? in the format ClassName::\[function function\]
    \# ClassName is a regular expression, but will be used like this: "^ClassName$" functions are also
    \# regular expressions, they will not be surrounded by "^$". If you want to skip a whole class, just
    \# add a single "*" as functions. See bellow for several examples. A special class name is "*", which
    \# will apply to all class names. This is a convenience wildcard to be able to skip similar named
    \# functions from all classes.
    
    skip = 
    
    rename_functions = 
    
    rename_classes = 
    
    \# for all class names, should we remove something when registering in the target VM?
    remove_prefix = 
    
    \# classes for which there will be no "parent" lookup
    classes\_have\_no_parents = 
    
    \# base classes which will be skipped when their sub-classes found them.
    base\_classes\_to_skip =
    
    \# classes that create no constructor
    \# Set is special and we will use a hand-written constructor
    abstract_classes =
    
    \# Determining whether to use script object(js object) to control the lifecycle of native(cpp) object or the other way around. Supported values are 'yes' or 'no'.
    script\_control\_cpp = no

主要修改： 第1行：\[custom_socket\] 第4行：prefix = custom_socket 第8行：target_namespace = 第26行：headers = %(cocosdir)s/../runtime-src/Classes/net/SocketModule.h 第30行：classes = ClientSocket.* CMD_Command.* ConnectHandler.* RecvHandler.* 其余根据具体来设置，主要说一下第26行是创建一个头文件，包含导出类的相关头文件，30行是导出的类定义 custom-genbindings.py
    
    #!/usr/bin/python
    
    \# This script is used to generate luabinding glue codes.
    \# Android ndk version must be ndk-r9b.
    
    
    import sys
    import os, os.path
    import shutil
    import ConfigParser
    import subprocess
    import re
    from contextlib import contextmanager
    
    
    def \_check\_ndk\_root\_env():
        ''' Checking the environment NDK_ROOT, which will be used for building
        '''
    
        try:
            NDK\_ROOT = os.environ\['NDK\_ROOT'\]
        except Exception:
            print "NDK\_ROOT not defined. Please define NDK\_ROOT in your environment."
            sys.exit(1)
    
        return NDK_ROOT
    
    def \_check\_python\_bin\_env():
        ''' Checking the environment PYTHON_BIN, which will be used for building
        '''
    
        try:
            PYTHON\_BIN = os.environ\['PYTHON\_BIN'\]
        except Exception:
            print "PYTHON_BIN not defined, use current python."
            PYTHON_BIN = sys.executable
    
        return PYTHON_BIN
    
    
    class CmdError(Exception):
        pass
    
    
    @contextmanager
    def _pushd(newDir):
        previousDir = os.getcwd()
        os.chdir(newDir)
        yield
        os.chdir(previousDir)
    
    def \_run\_cmd(command):
        ret = subprocess.call(command, shell=True)
        if ret != 0:
            message = "Error running command"
            raise CmdError(message)
    
    def main():
    
        cur_platform= '??'
        llvm_path = '??'
        ndk\_root = \_check\_ndk\_root_env()
        # del the " in the path
        ndk\_root = re.sub(r"\\"", "", ndk\_root)
        python\_bin = \_check\_python\_bin_env()
    
        platform = sys.platform
        if platform == 'win32':
            cur_platform = 'windows'
        elif platform == 'darwin':
            cur_platform = platform
        elif 'linux' in platform:
            cur_platform = 'linux'
        else:
            print 'Your platform is not supported!'
            sys.exit(1)
    
        if platform == 'win32':
            x86\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.3/prebuilt', '%s' % cur\_platform))
            if not os.path.exists(x86\_llvm\_path):
                x86\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.4/prebuilt', '%s' % cur\_platform))
        else:
            x86\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.3/prebuilt', '%s-%s' % (cur\_platform, 'x86')))
            if not os.path.exists(x86\_llvm\_path):
                x86\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.4/prebuilt', '%s-%s' % (cur\_platform, 'x86')))
    
        x64\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.3/prebuilt', '%s-%s' % (cur\_platform, 'x86_64')))
        if not os.path.exists(x64\_llvm\_path):
            x64\_llvm\_path = os.path.abspath(os.path.join(ndk\_root, 'toolchains/llvm-3.4/prebuilt', '%s-%s' % (cur\_platform, 'x86_64')))
    
        if os.path.isdir(x86\_llvm\_path):
            llvm\_path = x86\_llvm_path
        elif os.path.isdir(x64\_llvm\_path):
            llvm\_path = x64\_llvm_path
        else:
            print 'llvm toolchain not found!'
            print 'path: %s or path: %s are not valid! ' % (x86\_llvm\_path, x64\_llvm\_path)
            sys.exit(1)
    
        project\_root = os.path.abspath(os.path.join(os.path.dirname(\_\_file__), '..', '..'))
        cocos\_root = os.path.abspath(os.path.join(project\_root, ''))
        cxx\_generator\_root = os.path.abspath(os.path.join(project_root, 'tools/bindings-generator'))
    
        # save config to file
        config = ConfigParser.ConfigParser()
        config.set('DEFAULT', 'androidndkdir', ndk_root)
        config.set('DEFAULT', 'clangllvmdir', llvm_path)
        config.set('DEFAULT', 'cocosdir', cocos_root)
        config.set('DEFAULT', 'cxxgeneratordir', cxx\_generator\_root)
        config.set('DEFAULT', 'extra_flags', '')
    
        # To fix parse error on windows, we must difine \_\_WCHAR\_MAX__ and undefine \_\_MINGW32\_\_ .
        if platform == 'win32':
            config.set('DEFAULT', 'extra\_flags', '-D\_\_WCHAR\_MAX\_\_=0x7fffffff -U\_\_MINGW32\_\_')
    
        conf\_ini\_file = os.path.abspath(os.path.join(os.path.dirname(\_\_file\_\_), 'userconf.ini'))
    
        print 'generating userconf.ini...'
        with open(conf\_ini\_file, 'w') as configfile:
          config.write(configfile)
    
    
        # set proper environment variables
        if 'linux' in platform or platform == 'darwin':
            os.putenv('LD\_LIBRARY\_PATH', '%s/libclang' % cxx\_generator\_root)
        if platform == 'win32':
            path_env = os.environ\['PATH'\]
            os.putenv('PATH', r'%s;%s\\libclang;%s\\tools\\win32;' % (path\_env, cxx\_generator\_root, cxx\_generator_root))
    
    
        try:
    
            tolua\_root = '%s/tools/tolua' % project\_root
            output\_dir = '%s/../runtime-src/Classes/lua-bindings/auto' % project\_root
    
            cmd\_args = {'custom.ini' : ('custom\_socket', 'lua\_custom\_socket_auto'),\
                                    }
            target = 'lua'
            generator\_py = '%s/generator.py' % cxx\_generator_root
            for key in cmd_args.keys():
                args = cmd_args\[key\]
                cfg = '%s/%s' % (tolua_root, key)
                print 'Generating bindings for %s...' % (key\[:-4\])
                command = '%s %s %s -s %s -t %s -o %s -n %s' % (python\_bin, generator\_py, cfg, args\[0\], target, output_dir, args\[1\])
                \_run\_cmd(command)
    
            print '---------------------------------'
            print 'Generating lua bindings succeeds.'
            print '---------------------------------'
    
        except Exception as e:
            if e.\_\_class\_\_.\_\_name\_\_ == 'CmdError':
                print '---------------------------------'
                print 'Generating lua bindings fails.'
                print '---------------------------------'
                sys.exit(1)
            else:
                raise
    
    
    \# -------------- main --------------
    if \_\_name\_\_ == '\_\_main\_\_':
        main()
    
    主要修改： 第134行： output\_dir = '%s/../runtime-src/Classes/lua-bindings/auto' % project\_root 第136行： cmd\_args = {'custom.ini' : ('custom\_socket', 'lua\_custom\_socket_auto'),\ } 修改好后，在命令行执行：custom-genbindings.py 如果没有错误的话会生成lua\_custom\_socket\_auto.hpp和lua\_custom\_socket\_auto.cpp 接下来注册自定义模块： 1.添加到工程 ![](http://img.blog.csdn.net/20150914172814416?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  2.引入到AppDelegate ![](http://img.blog.csdn.net/20150914172852765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  3.注册模块 ![](http://img.blog.csdn.net/20150914172932866?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  使用cocos code IDE 1.2.0断点调试，由于使用调试的模拟器是另外一个程序quick-3.5/tools/simulator，所以需要将脚本生成的文件在simulator工程再添加一遍： ![](http://img.blog.csdn.net/20150914173525975?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  生成新的模拟器，然后在lua中测试，就可以调用自定义的C++类了 
    
     --test 
        self.socket_ = ClientSocket:new()
        self.socket_:retain()
        
        print("create success")

QQ群：239759131 cocos 技术交流 欢迎您