
# Pods的使用
---

``` bash
gem sources -l                                         //查看当前已有的镜像地址
gem sources --remove https://rubygems.org/             //删除这个镜像地址
gem sources -a http://rubygems-china.oss.aliyuncs.com  //添加国内镜像
sudo gem install cocoapods                             //安装完成
```

## Podfile.lock文件

在执行`pod install`明后之后，会生成一个`Podfile.lock`文件，此文件用于保存已经安装的Pods依赖库的版本。在多人协作开发过程中，开发者
通过`check`下来的工程包含此文件，再执行`pod install`命令时，获取下来的Pod依赖库的版本将会保持一致。

可以通过更改`Podfile`文件，然后再执行`pod update`命令来实现更新依赖库的版本。

## Target
`Podfile`本质是用来描述`Xcode`工程中的`targets`用的。如果没有指定target，那么只有工程中第一个target能够使用Podfile中描述的Pods依赖库。

### 多个target使用相同的Pods依赖库:

```Text
link_with 'target1','target2'
platform :ios,'8.0'
pod 'A','~> 3.0.0'
```
### 多个target使用不同的Pods依赖库

```Text
target 'target1' do
platform :ios
pod 'A','~> 3.0.0'
do

target 'target2' do
pod 'B','~> 2.0.0'
end
```
## 依赖库版本说明

```Text
pod 'A'               //不显示指定版本，表示每次获取最新版本
pod ‘A’,'~> 1.0.2'    //使用大于1.0.2但小于1.1的版本
```

## 依赖库查询

```Text
pod search A
```

## Pod其他用法

```Text
pod 'A', :subspecs => ['B','C']  //用于安装指定库下的文件目录
pod 'A', :path => '~/local/A'    //用于安装本地的Pod
pod 'A', :git => 'https://github.com/xx/A.git' //使用master分支
pod 'A', :git => 'https://github.com/xx/A.git', :branch => 'dev' //使用dev分支
pod 'A', :git => 'https://github.com/xx/A.git', :tag => '0.7.0' //使用tag
pod 'A', :git => 'https://github.com/xx/A.git', :commit => '082f8319f' //使用提交记录
pod 'A', :podspec => 'https://example.com/A.podspec' //在一个不带podsepec的库里引用外部spec
```

屏蔽Coacoapods库里面的所有警告

> inhibit_all_warnings!

自定义屏蔽警告

```Text
`pod 'A', inhibit_warning => true` 
```

