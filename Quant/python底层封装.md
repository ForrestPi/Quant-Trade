#python底层封装

##python封装C++API设计

本篇将会包含的内容：

1. python2.7版本，建议安装Anaconda 
2. Visual Studio 2012
3. 编译Boost库，建议直接下载预编译版本库
4. 本机编译环境为Win7 32位系统，建议选择合适的版本库

###创建VS项目

打开VS2012后，点击菜单栏的“文件” -> “新建” -> “项目”， 在弹出的窗口中的左侧选择“模板” -> “Visual C++” -> “Win32” -> “Win32项目”->应用程序类型选择为“DLL”，点击“完成”。


### 配置项目

点击上方工具栏中的解决方案配置选项框（默认显示应该是“Debug”），选择“Release”

###添加包含目录以及库目录
在包含目录中添加：

    D:\boost_1_61_0
    D:\Anaconda\include

然后库目录中添加：

    D:\boost_1_61_0\stage\lib
    D:\Anaconda\libs
    
添加库文件：
    Python27.lib;boost-python.lib

