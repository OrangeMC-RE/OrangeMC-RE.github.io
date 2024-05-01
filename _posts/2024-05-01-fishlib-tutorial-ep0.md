---
layout: post
title: "fishlib用法-第零章"
category: [minecraft]
---


# FishLib - 鱼子库

一款我自己使用的库, 但考虑到可能也会有别人来用, 所以我先在这写上文档, 万一有人要用呢

将不定时更新, 如果太久没更新随时催一下老鱼

[资源地址](https://www.minebbs.com/resources/fishlib.7901/) [源代码在这](https://github.com/astro-angelfish/fishlib)

## 安装

需求:
  - Spigot/Paper 1.13+
  - Java 17+
  - 任意操作系统

## 开发预备

目前老鱼我没钱, 之前蹭过别人的maven仓库但是随着服务器关闭而无家可归了

你需要在[上面的资源地址](#FishLib - 鱼子库)下载这个资源

<details><summary>如果你是纯手动创建项目的话, 按照这里的走</summary>

直接将jar文件依赖引入到你的项目里头即可, 但要保证fishlib不会因为其他原因被意外删掉即可

</details>

<details><summary>如果你使用的是maven, 按照这里的走</summary>

你需要在你的`pom.xml`中找到`dependencies`标签, 加入以下标签内容

```xml
<dependency>
    <groupId>moe.orangemc</groupid>
    <artifactId>fishlib</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <scope>system</scope>
    <systemPath>fishlib住的地方</systemPath>
</dependency>
```

具体情况见[这个(英文警告)](https://stackoverflow.com/questions/4955635/how-to-add-local-jar-files-to-a-maven-project)

当然你也可以使用`mvn install`指令来直接全局加入, 等老鱼什么时候有钱买maven仓库服务器罢

</details>

<details><summary>如果你使用的是gradle, 按照这里的走</summary>

你需要在你的`build.gradle`下找到`dependencies`块, 加入以下内容

```groovy
implementation(files("fishlib住的地方"))
```

</details>

## 开发文档目录

正在放鸽子.jpg

 - [ ] 指令模块
 - [ ] 物品堆创建
 - [ ] 基于物品栏的交互界面
 - [ ] 国际化
 - [ ] 基于地图的交互界面
 - [ ] 反射(其实代码都没写完)
 - [ ] 记分板展示

## Javadoc

~~来个人行行好给咱整台服务器放javadoc罢~~

