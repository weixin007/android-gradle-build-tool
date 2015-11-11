# Android Gradle Build Tool

## 说明
一个工具脚本，帮助 Android 打包，支持 application 项目与 library 项目。

功能列表：
- 根据项目版本文件，自动更改和升级版本。
- 更改打包后产物的名称。
- 收集 Android 打包产物（apk、 aar、proguard 文件等）。

以上所有的功能，默认的，只在执行 `assemble` 任务时才会执行。同时，本脚本的功能以及目标比较死板，使用者可以根据自己项目的具体情况，进行修改。

## 使用方式

将 `tool.gradle` 脚本追加至 module 的 `build.gradle` 中，可以直接复制 `tool.gradle` 中的内容，或者使用 `apply` 方法。

```
apply from: 'path/to/tool.gradle'
```

所有功能，支持以参数形式进行开关，可以在 `gradle.properties` 文件中配置，也可以使用 Gradle 执行参数。

所有功能中，如果 **必传参数** 没有设置，或者设置的值为空，则表示不开启该功能（无需参数值的除外）。

### 自动升级版本号
参数名称： `verFile` ，参数值指向版本信息文件，如果文件不存在，会自动创建。**必传参数**。

在此脚本中，版本信息文件，有 3 个字段，`hVerName` ，`lVerName` ，`verCode` ，分别对应版本名称高位，版本名称低位，版本号。
最终生成的 release 产物的版本名称为 `hVerName` ，版本号为 `verCode` ； debug 产物的版本名称为 `hVerName.lVerName` ，版本号为 `verCode` 。
一次打包完成之后，`lVerName` ，`verCode` 字段均会自动增加 1 ，而从实现版本自动增加的功能。

例如对于一个版本文件信息如下：

```
hVerName=1.2.3
lVerName=2
verCode=226
```

release 产物的版本名称为 `1.2.3` ，版本号为 `226` ； debug 产物的版本名称为 `1.2.3.2` ，版本号为 `226` 。
完成这次打包后，`hVerName` 值为 `1.2.3` ；`lVerName` 值为 `3` ；`verCode` 值为 `227` 。

配置方式：
- 方法1：`gradle.properties` 文件中添加以下内容：

  ```
  verFile=path/to/your/version/file
  ```

- 方法2：Gradle 执行时添加参数：

  ```
  -PverFile=path/to/your/version/file
  ```

### 更改打包产物的文件名称
参数名称：`changeOutputName` ，无需参数值。**必传参数**。
参数名称：`proName` ，参数值为该项目的名称，如果没有设置该参数，则默认使用项目名称。

默认的 Android Gradle 打包时，生成 apk 或者 aar 文件的名称为 `{projectName}-{flavor}-{buildType}.apk(aar)` 。
现在此基础上，添加了版本名称和版本号的信息，最终生成的文件名称为 `{proName}-{flavor}-{buildType}-{versionName}-{versionCode}.apk(aar)` 。

配置方式：
- 方法1：`gradle.properties` 文件中添加以下内容：

  ```
  changeOutputName
  proName=your_project_name
  ```

- 方法2：Gradle 执行时添加参数：

  ```
  -PchangeOutputName -PproName=your_project_name
  ```

### 收集打包产物
参数名称：`outputs` ，参数值生成产物存放的目录，如果目录不存在，会自动创建。**必传参数**。
参数名称：`proName` ，同 [更改打包产物的文件名称](###更改打包产物的文件名称) 中的 `proName` 意义和作用相同。

默认的 Android Gradle 打包时，生成的 apk 等文件，存放在项目的 build 文件夹下，每次正式打包后，需要手动复制存放，太麻烦。
此功能，在打包完成后，自动收集有用的文件，存放到对应的目录。

所谓的对应的目录如下：
- apk(aar)：`{outputs}/{proName}/{date-versionName-versionCode}/{flavor}-{buildType}/apk(aar)/`
- dex：`{outputs}/{proName}/{date-versionName-versionCode}/{flavor}-{buildType}/dex/`
- proguard 混淆日志文件：`{outputs}/{proName}/{date-versionName-versionCode}/{flavor}-{buildType}/mapping/`
- proguard 混淆生成的 jar 文件：`{outputs}/{proName}/{date-versionName-versionCode}/{flavor}-{buildType}/jar/`
- manifest merge 后的 AndroidManifest 文件：`{outputs}/{proName}/{date-versionName-versionCode}/{flavor}-{buildType}/manifest/`

配置方式：
- 方法1：`gradle.properties` 文件中添加以下内容：

  ```
  outputs=path/to/your/outputs/folder
  proName=your_project_name
  ```

- 方法2：Gradle 执行时添加参数：

  ```
  -Poutputs=path/to/your/outputs/folder -PproName=your_project_name
  ```
