## 简介
[Qt Installer Framework](http://doc.qt.io/qtinstallerframework/)是一个开源的打包工具，Qt的安装包就是该工具生成的。它有下面几个特性
- 基于命令行
- 跨平台，支持windows、linux和苹果
- 安装流程可用js脚本控制，也可以自定义显示界面
- 有包管理功能，安装完成后，可以通过自动生成的维护工具，安装、卸载程序的某一部件

## 下载和安装

带源码的Qt安装包有Qt Installer Framework的选项(如下图)，不过需要注册一个Qt账户；如果不使用Qt的话，不推荐用这个方法，因为Qt安装包比较大，下载比较花时间。

另一个方法是单独下载Qt Installer Framework安装包，链接[Qt Installer Framework](https://download.qt.io/official_releases/qt-installer-framework/)

![](https://github.com/codinganywhere/mynotes/blob/master/Qt_Installer_Framework/images/source_install.png)

##使用说明

IFW(Qt Installer Framework)的打包工具是binarycreator.exe，输入待打包的配置信息，生成一个exe文件。把需要打包的程序及相关资源统一放在一个文件夹，例如tutorial；在tutorial下面必须包含config和packages两个子文件夹，config文件夹是打包的配置信息config.xml。编写好必要的配置信息后，就可以使用下面的命令**binarycreator.exe -c config\config.xml -p packages tutorial.exe**，注意命令的当前路径是tutorial，打包生成了tutotial.exe。

把[tutorial 代码](https://github.com/codinganywhere/mynotes.git)clone到本地，对照着下面的介绍学习。

1. 配置文件config.xml

    ```xml
<?xml version="1.0" encoding="UTF-8"?>
<Installer>
        <Name>Your application</Name>
        <Version>1.0.0</Version>
        <Title>Your application Installer</Title>
        <Publisher>Your vendor</Publisher>
        <StartMenuDir>Super App</StartMenuDir>
        <TargetDir>@HomeDir@/InstallationDirectory</TargetDir>
</Installer>
    ```

    效果如图 

    ![](http://doc.qt.io/qtinstallerframework/images/ifw-tutorial-introduction-page.png)

    \<TargetDir\>元素指定了软件的默认安装目录，其中@HomeDir@是IFW的预定义变量，更多的参考[预定义变量列表](http://doc.qt.io/qtinstallerframework/scripting.html#predefined-variables)。

    配置文件还可以指定其他属性，类似的参考[配置文件元素列表](http://doc.qt.io/qtinstallerframework/ifw-globalconfig.html)

2. 创建一个package

    packages文件夹下面可以有多个package，我们以创建一个package为例，更多package的创建方法类似。packages典型结构如下

        -packages
            - com.vendor.root
                - data
                - meta
            - com.vendor.root.component1
                - data
                - meta
            - com.vendor.root.component1.subcomponent1
                - data
                - meta
            - com.vendor.root.component2
                - data
                - meta
        
    1. 在packages文件夹下面创建文件夹com.vendor.example
    2. 在com.vendor.example下面创建data和meta两个子文件夹
    3. 在meta下面创建文件package.xml

    package.xml的基本格式如下

    ```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package>
        <DisplayName>The root component</DisplayName>
        <Description>Install this example.</Description>
        <Version>0.1.0-1</Version>
        <ReleaseDate>2017-02-17</ReleaseDate>
        <Default>true</Default>
        <Script>installscript.qs</Script>
</Package>
    ```

    \<Default\>指明该package是否默认选中，true选中，false未选中。一个安装包可能有多个package，有的是必须的部件，而有的是非必须的。
    \<Script\>指定了安装该package时的控制脚本，在meta文件夹下。具体的语法见下面的介绍。
    有关package.xml的更多信息参考[包配置文件](http://doc.qt.io/qtinstallerframework/ifw-component-description.html#package-information-file-syntax)

3. 脚本

    使用javascript脚本，最简单的格式如下

    ```javascript
	function Component()
	{
	
	}
	Component.prototype.createOperations = function()
	{
		// call default implementation
		component.createOperations();
		// ... add custom operations
		component.addOperation("Execute", "cmd.exe", "/C start /WAIT mspaint");
	}
    ```

    **注意**Component是必须要有的。添加的operation是在安装该package时执行的命令，除了启动外部程序外，
    还可以新建、删除、复制文件和文件夹，创建快捷方式，解压文件等，更多的命令参考[Operations](http://doc.qt.io/qtinstallerframework/operations.html)

    **BUG**: IFW在windows下添加"Execute"命令，如果只指定了一个参数，就会提示"No program defined"(见下图)。在源码中已经定位了问题代码，等待修复...所以使用Execute时，即便只需要程序路径，也要传一个多余的参数。也可以使用上面示例的方式，这一块儿问题还有点儿多，先按照这种可用的方式来，日后有更好的方法再添加。

    ![](https://github.com/codinganywhere/mynotes/blob/master/Qt_Installer_Framework/images/program_not_defined.PNG)

    问题代码(在installer/elevatedexecuteoperation.cpp)

    ```c++
	#ifdef Q_OS_WIN
		if (args.count() == 1) {
			process->setNativeArguments(args.front());
			qDebug() << "ElevatedExecuteOperation setNativeArguments to start:" << args.front();
			process->start(QString(), QStringList());
		} else
	#endif
		{
			process->start(args.front(), args.mid(1));
		}
    ```

    更多关于脚本的用法，参考[Script](http://doc.qt.io/qtinstallerframework/scripting.html)

---
**这篇文章最初发布在本人的**[github](https://github.com/codinganywhere/mynotes/wiki/QT-Installer-Framework-tutorial)**上**