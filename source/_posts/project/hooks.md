---
title: cordova 之 hooks
categories:
- project
---
hooks 是一些在 Cordova 执行命令时运行的特殊脚本，能够允许你拓展 Cordova 命令来适应你自己的需求。官方推荐三个地方定义 hooks，分别在工程的 config.xml、插件的 plugin.xml 和 hooks 文件夹，这里介绍的是第三个地方。任何一门编程语言都能够写 hooks，但是考虑到跨平台运行（即需要不同语言的运行环境），推荐使用 Node.js。
<!--more-->
### 一、hooks 应用场景
hooks 应用场景一：修改应用配置信息，我们在应用开发中，引入的插件可能有冲突之类的问题，这个时候原生开发人员会让我们在添加平台加一些配置信息。也就是说我们每次添加平台后都需要手动去改平台中代码，不太方便，程序员就要尽可能的懒起来，这个时候就可以用上 hooks 了；
hooks 应用场景二：环境切换，我们的 Cordova 工程经常有开发、uat、生产等不同环境。对应不同的环境，我们有不同的接口地址、不同的插件 API keys，甚至不同的应用图标和 Splash。对于不同接口地址、不同的应用图标和 Splash等，我们可以使用自定义 gulp 任务实现，但是对于插件中的接口地址、API keys 不太好做处理。
### 二、Cordova 支持的 hooks 类型
<img src="/assets/project/hooks-type.png">

### 三、hooks 执行顺序
根据以上表格可以推导出不同类型 hooks 执行顺序，以 cordova platform add 和 cordova build 为例：
```
cordova platform add：
    before_platform_add
    before_prepare
    after_prepare
    before_plugin_install
    after_plugin_install
    after_platform_add
    
cordova build：
    before_build
    before_prepare
    after_prepare
    before_compile
    after_compile
    after_build
```
### 四、应用实例
#### 4.1、以我现在正在开发的 xxx 项目为例，添加平台后修改平台内配置信息（cordova platform add）
1、在应用根目录下新建 hooks 文件夹（如果先前有可删除），在 hooks 文件夹内新建 after_platform_add 文件夹，内新建一个 js 文件，命名随意；  
2、读写平台中的 AndroidManifest.xml 文件，修改配置信息；
```
一、解决方法数越界       
1. build.gradle 配置
defaultConfig{
	multiDexEnabled true
}  
2. dependencies{
	compile "com.android.support:multidex:1.0.0"
}
3. AndroidManifest.xml 中 Application 添加属性
android:name="android.support.multidex.MultiDexApplication"
    
二、解决条码扫描插件最低版本冲突 
1. AndroidManifest.xml 中 Manifest 添加属性  
xmlns:tools="http://schemas.android.com/tools"     
2. Manifest 中添加节点
<uses-sdk android:minSdkVersion="16" tools:overrideLibrary="org.xwalk.core" />
```
3、Android 打包配置需求如上；
```
var fs = require('fs');
var path = require('path');
var rootdir = process.argv[2];

//正则表达式替换文本
function replace_string_in_file(filename, to_replace, replace_with) {
  var data = fs.readFileSync(filename, 'utf8');
  var result = data.replace(to_replace, replace_with);
  fs.writeFileSync(filename, result, 'utf8');
}

if (rootdir) {
  //获取平台信息数组
  var platforms = (process.env.CORDOVA_PLATFORMS ? process.env.CORDOVA_PLATFORMS.split(',') : []);
  for (var x = 0; x < platforms.length; x += 1) {
    try {
      var platform = platforms[x].trim().toLowerCase();
      //如果是android平台
      if (platform === 'android') {
        //替换平台内AndroidManifest.xml文件内容
        var fullfilename = path.join(rootdir, ‘platforms/android/AndroidManifest.xml');
        //判断AndroidManifest.xml文件是否存在
        if (fs.existsSync(fullfilename)) {
          replace_string_in_file(fullfilename, "<manifest", '<manifest xmlns:tools="http://schemas.android.com/tools"');
          replace_string_in_file(fullfilename, "<supports-screens", '<uses-sdk android:minSdkVersion="16" tools:overrideLibrary="org.xwalk.core" /><supports-screens');
          replace_string_in_file(fullfilename, "<application", '<application android:name="android.support.multidex.MultiDexApplication"');
        } else {
          console.log("missing: " + fullfilename);
        }
        //替换build.gradle文件内容
        var fullfilename2 = path.join(rootdir, 'platforms/android/build.gradle');
        if (fs.existsSync(fullfilename2)) {
          replace_string_in_file(fullfilename2, /defaultConfig\s*\{/gi, 'defaultConfig {\n multiDexEnabled true');
          replace_string_in_file(fullfilename2, /}\s*dependencies\s*\{/gi, '}\n dependencies {\n compile "com.android.support:multidex:1.0.0"');
        } else {
          console.log("missing: " + fullfilename2);
        }
      }
    } catch (e) {
      process.stdout.write(e);
    }
  }
}
```
4、附上 hooks 源代码(xxx/after_platform_add/xxx.js)。
#### 4.2、修改插件配置信息（这里修改 xxx 插件 API_KEY 为例）
1、在应用根目录下新建 hooks 文件夹（如果先前有可删除），在 hooks 文件夹内新建 before_build 和 before_compile 文件夹，内分别新建一个 js 文件，命令随意；
```
1、修改 android 平台内 AndroidManifest.xml，配置 RONG_CLOUD_APP_KEY（<meta-data android:name=“RONG_CLOUD_APP_KEY" android:value="" />）
2、修改 ios 平台的 plist文件，配置参数（<config-file target="*LBXIMConfig.plist"  parent="RongCloudKey"><string>$RONGCLOUD_KEY</string></config-file>）
```
2、需求如上（需在原生开发人员配合下修改，由原生开发人员提出修改需求）
```
var fs = require('fs');
var path = require('path');
var rootdir = process.argv[2];

function replace_string_in_file(filename, to_replace, replace_with) {
  var data = fs.readFileSync(filename, 'utf8');
  var result = data.replace(data.match(to_replace)[0], replace_with);
  fs.writeFileSync(filename, result, 'utf8');
}

if (rootdir) {
  // go through each of the platform directories that have been prepared
  var platforms = (process.env.CORDOVA_PLATFORMS ? process.env.CORDOVA_PLATFORMS.split(',') : []);
  for (var x = 0; x < platforms.length; x += 1) {
    try {
      var platform = platforms[x].trim().toLowerCase();
      if (platform === 'android') {
      //替换AndroidManifest.xml文件内容，删除RONG_CLOUD_APP_KEY
        var fullfilename = path.join(rootdir, 'platforms/android/AndroidManifest.xml');
        replace_string_in_file(fullfilename, '<meta-data\.\*android:name="RONG_CLOUD_APP_KEY"\.\*\/>', '');
      }
    } catch (e) {
      process.stdout.write(e);
    }
  }
}
```
3、考虑到 android 在 prepare中 会添加 <meta-data android:name="RONG_CLOUD_APP_KEY" android:value="" /> 到AndroidManifest.xml 中，所以在 before_build 文件夹中写如下代码，删除相应配置信息，代码如上(xxx/before_build/xxx.js)。
```
var fs = require('fs');
var path = require('path');
var rootdir = process.argv[2];
function replace_string_in_file(filename, to_replace, replace_with) {
  var data = fs.readFileSync(filename, 'utf8');
  var result = data.replace(data.match(to_replace)[0], replace_with);
  fs.writeFileSync(filename, result, 'utf8');
}
//获取应用名（因为在ios平台中plist配置文件在应用名文件夹目录下）
function get_data_in_configxmlfile(filename, to_replace) {
  var data = fs.readFileSync(filename, 'utf8');
  return data.match(to_replace)[1];
}
//替代plist文件内的配置信息
function replace_data_in_plistfile(filename, to_replace, replace_with) {
  var configPlist = fs.readFileSync(filename, 'utf8');

  var result = configPlist.replace(to_replace, replace_with);
  fs.writeFileSync(filename, result, 'utf8');
}

if (rootdir) {
  // go through each of the platform directories that have been prepared
  var platforms = (process.env.CORDOVA_PLATFORMS ? process.env.CORDOVA_PLATFORMS.split(',') : []);
  var ourConfigFile = path.join(rootdir, "app/config/config.json");
  if (fs.existsSync(ourConfigFile)) {
    var configObj = JSON.parse(fs.readFileSync(ourConfigFile, 'utf8'));
  } else {
    console.log("app/config/config.json not exist!!!");
    return;
  }

  for (var x = 0; x < platforms.length; x += 1) {
    try {
      var platform = platforms[x].trim().toLowerCase();
      if (platform === 'android') {
     //替换AndroidManifest.xml文件内容，配置RONG_CLOUD_APP_KEY
        var fullfilename = path.join(rootdir, 'platforms/android/AndroidManifest.xml');
        replace_string_in_file(fullfilename, 'android:name="RONG_CLOUD_APP_KEY"\.\*\/>',
          'android:name="RONG_CLOUD_APP_KEY" android:value="' + configObj["baseConfig"]["RONG_CLOUD_APP_KEY"] + '" />');
      } else {
        //添加config.xml，读取文件内的应用名
        var ourIosConfigFile = path.join(rootdir, "config.xml");
        //获取应用名
        var appName = get_data_in_configxmlfile(ourIosConfigFile, "<name>(\.\*)<\/name>");
        //修改im配置plist文件
        var imConfigFile = 'platforms/ios/' + appName + '/Resources/LBXIMConfig.plist';
        if (fs.existsSync(imConfigFile){
	replace_data_in_plistfile(imConfigFile, /<key>RongCloudKey<\/key>\s*<string>\s*\S*<\/string>/gi, '<key>RongCloudKey</key>\n\t<string>' + configObj["baseConfig"]["RONG_CLOUD_APP_KEY"] + '<\/string>');
        } 
      }
    } catch (e) {
      process.stdout.write(e);
    }
  }
}
```
4、在平台编译之前加上参数。读取配置文件（自定义）内的 RONG_CLOUD_APP_KEY，把参数信息写入平台内部的配置文件中（xxx/before_compile/xxx.js）。