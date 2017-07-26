# XcodeAutoBuild
iOS自动打包脚本制作

在工程根目录下新建两个文件


![项目文件夹](http://upload-images.jianshu.io/upload_images/953487-f6eae1d35daa4135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


`DevelopmentExportOptionsPlist.plist`用来指定打包的类型，`xcodebuild.sh`是打包执行的`shell`脚本文件。


![DevelopmentExportOptionsPlist.plist 文件内容](http://upload-images.jianshu.io/upload_images/953487-c1e9af9e286e5f28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

method的类型为`String`，用来指定打包`ipa`的类型，有以下可选项`app-store、enterprise、ad-hoc、development`，默认为`development`


`xcodebuild.sh`文件中包括一些需要配置的参数，`工程名project_name、打包模式development_mode、scheme名scheme_name`。

打包的工程文件分两种(是否包含cocoapods)：`project`和`workspace`，如果需要打包的是`project`，则在`archieve`中将`-workspace`替换为`-project`，将`.xcworkspace`的路径替换为`.xcodeproj`的路径

```
#工程名（自定义）
project_name=eHRmobile

#打包模式 Debug/Release（自定义）
development_mode=Release

#scheme名（自定义，一般与工程名相同）
scheme_name=eHRmobile

#plist文件所在路径
exportOptionsPlistPath=./DevelopmentExportOptionsPlist.plist

#导出.ipa文件所在路径
exportFilePath=~/Desktop/$project_name-ipa

echo '*** 正在 清理工程 ***'
xcodebuild \
clean -configuration ${development_mode} -quiet  || exit 
echo '*** 清理完成 ***'


echo '*** 正在 编译工程 For '${development_mode}
xcodebuild \
archive -workspace ${project_name}.xcworkspace \
-scheme ${scheme_name} \
-configuration ${development_mode} \
-archivePath build/${project_name}.xcarchive -quiet  || exit
echo '*** 编译完成 ***'


echo '*** 正在 打包 ***'
xcodebuild -exportArchive -archivePath build/${project_name}.xcarchive \
-configuration ${development_mode} \
-exportPath ${exportFilePath} \
-exportOptionsPlist ${exportOptionsPlistPath} \
-quiet || exit

# 删除build包
if [[ -d build ]]; then
    rm -rf build -r
fi

if [ -e $exportFilePath/$scheme_name.ipa ]; then
    echo "*** .ipa文件已导出 ***"
    cd ${exportFilePath}
    echo "*** 开始上传.ipa文件 ***"
    #此处上传分发应用
    echo "*** .ipa文件上传成功 ***"
else
    echo "*** 创建.ipa文件失败 ***"
fi
echo '*** 打包完成 ***'

```

自动打包（需在项目中配置好描述文件、开发者证书）
使用方式:
在终端中进入`*.xcodeproj`上级目录
输入`./xcodebuild.sh`即可自动打包、如无执行权限则先执行`chmod +x xcodebuild.sh`

最后会在桌面上生成打包完成的ipa文件夹。


-------
参考文章：

[iOS 制作自动打包脚本 Xcode8.3.2](http://www.cnblogs.com/ficow/p/6823962.html)
[自动打包脚本](https://github.com/PurpleSweetPotatoes/AutoBuild-ipa)


