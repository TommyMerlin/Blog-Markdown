---
title: Win10系统常见问题及解决办法
date: 2020-09-22 10:18:27
tags:
- Win10
- 常见问题
categories:
- 教程
description: Win10系统常见问题及解决办法
sticky: 98
---

## 窗口无阴影

此电脑—属性—高级系统设置—高级—性能—设置—在窗口下显示阴影

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E7%AA%97%E5%8F%A3%E6%97%A0%E9%98%B4%E5%BD%B1-b66bcd.png" alt="窗口无阴影" width="40%" />

## Office 相关

### Word 相关

#### word中添加 Mathtype 公式行间距改变问题

在 mathtype 中编辑任何一个公式，将字体调整好，然后在公式编辑窗口中点 “Preference”-“Save to file”，将设置保存为一个文件，文件名任取。关闭公式编辑窗口，退回到Word窗口，然后在 Mathtype 菜单中点 “Format Equations”，再选择你定义的那个文件，勾选 “Whole Document”，确定。然后就可以看到状态栏上公式处理进度飞闪，一到两分钟左右（视文章长短）提示OK。

mathtype inline(内联)表示在一行文字中输入公式：

- 1.在 word 中点击 “**布局**” 菜单下的 “**页面设置**” 项。在 “**文档网格**” 标签页中的 “**网格**” 一栏，勾选 “**无网格**” 项。这样能很大程度上缓解行距不等的情况，然后再进行公式大小的微调，就能取得很好的显示效果。

- 2.两行公式行距问题：经研究，发现格式化公式，将公式设置为 tex 格式公式能解决这个问题，具体设置如下：菜单\mathtype\Format Equation 选选项 mathtype preference file，点击 browse 找到安装目录下的 preference 文件夹，选择 TeX Look 再确定，大功告成！

**[答疑]mathtype编辑数学公式的几点说明**

1.**公式内汉字显示**问题：请设置text字体为宋体，默认的方正大字库，有的汉字不能显示。当然最好不要在公式内写汉字，因为排版麻烦，且转化为方正书版后有一些乱码，校稿麻烦。

2.**设置尺寸时磅值与字号对应关系**

在 WORD 中字号越大，字符越小，而磅值越大，字符也越大的，两者具体的对应关系大
约如下吧： 
字号‘八号’对应磅值 5     字号‘七号’对应磅值 5.5 
字号‘小六’对应磅值 6.5    字号‘六号’对应磅值7.5 
字号‘小五’对应磅值 9     字号‘五号’对应磅值 10.5 
字号‘小四’对应磅值 12    字号‘四号’对应磅值14 
字号‘小三’对应磅值 15     字号‘三号’对应磅值 16 
字号‘小二’对应磅值 18     字号‘二号’对应磅值 22 
字号‘小一’对应磅值 24     字号‘一号’对应磅值 26 
字号‘小初’对应磅值 36     字号‘初号’对应磅值 42 

3.**如何批量处理公式大小？**

在用 Word编辑的数学试卷中，会有大量的公式存在。如果在文档编辑完成后，需要重新调整字号的大小，那么文档中的这些公式怎么办呢？ 
通常情况下，Word 文档中的这些公式都是用 MathType 编辑完成的，在 Word 中将它们当成图形对象来对待的。 我们不可能一个一个地选中图形然后拖动鼠标手工完成公式大小的调整。下面的办法可以让我们批量完成公式中字号大小的调整，从而达到调整公式大小的目
的。 
先运行 MathType，点击 “Size” 菜单中的 “Define” 命令。 打开 “Define Sizes” 对话框，我们可以在 “Full” 后的输入框中要调整的字号大小，公式中其它的元素会自动进行相应的调整的， 所以一般情况下可以不做其它改动。点击 “OK” 按钮，关闭对话框。再点击 “Preferences” 菜单中的 “Equation Preferences→Save to File” 命令，将我们设置好的选项保存成一个后缀名为 “eqp” 的文件。 现在回到Word 环境中，点击 “MathType” 菜单中的 “Format Equations” 命令。然后在打开的 “Format Equations” 对话框中选中中间的 “MathType preference file” 单选项，并点击 “Browse” 按钮，找到我们保存好的那个 eqp 文件并双击。然后再选中下方 “Range” 项目中 “Whole document” 单选项，点击 “OK” 按钮后稍候片刻。

#### 避免Word打字键入文字后替换删除后面原来的字

如果用 word 时发现打字后把后面的字替换了，这很可能是启用了改写模式，启用改写模式的原因很可能是误按了键盘的 `Insert` 键，按这个键默认会让 word 启用和取消改写模式，启用改写模式后只要再按一次 `Insert` 键，就会取消改写模式。

#### 应用程序无法正常启动（0xc0000142）

在打开 Office 文档时，显示如下错误：

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-08-02-20200606151928742-687e67.png" alt="20200606151928742" style="width:50%;" />

解决方法：

- 1.开始菜单-运行-输入 services.msc
- 2.在打开的服务中找到Microsoft Office即点即用服务
- 3.启动类型改为禁用---点击应用---启动类型再改为自动---点击应用
- 4.点击确定
- 5.右击-重新启动 /启动  在打开 offfice 就没有问题了

#### 使用 wingdings2 字体快速输入打勾，打叉符号

- 1.字体选中Wingdings 2。
- 在空白单元格，输入大写P，就是打勾，大写O，就是打叉，输入大写R，带框勾，输入大写Q，带框叉。

#### Endnote 参考文献格式

样式库内更改 `EndNote Bibliography` 样式。

[视频教程](https://www.bilibili.com/video/av289791444/?t=03m20s)

### Excel 相关

#### Excel 小写金额转大写

公式 1：

```
=IFS(ISERROR(FIND(".",G4,1)),NUMBERSTRING(INT(G4),2)&"元整",AND(INT(G4)=0,(LEN(G4)-FIND(".",G4,1)=1)),NUMBERSTRING(RIGHT(G4,1),2)&"角",AND(INT(G4)=0,(LEN(G4)-FIND(".",G4,1)=2)),NUMBERSTRING(MID(G4,FIND(".",G4,1)+1,1),2)&"角"&NUMBERSTRING(RIGHT(G4,1),2)&"分",AND(INT(G4)>0,ISNUMBER(FIND(".",G4,1)),(LEN(G4)-FIND(".",G4,1)=1)),NUMBERSTRING(INT(G4),2)&"元"&NUMBERSTRING(RIGHT(G4,1),2)&"角",AND(INT(G4)>0,ISNUMBER(FIND(".",G4,1)),(LEN(G4)-FIND(".",G4,1)=2)),NUMBERSTRING(INT(G4),2)&"元"&NUMBERSTRING(MID(G4,FIND(".",G4,1)+1,1),2)&"角"&NUMBERSTRING(RIGHT(G4,1),2)&"分")
```

公式 2：

```
=SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(SUBSTITUTE(TEXT(INT(ABS(G4)),"[DBNum2]")&"元"&TEXT(RIGHT(TEXT(G4,".00"),2),"[DBNum2]0角0分"),"零角零分","整"),"零分","整"),"零角","零"),"零元零","")
```

[原帖](https://www.52pojie.cn/thread-1347549-1-1.html)

### PPT 相关

#### PPT绘图导出高分辨率图

* 1.首先将绘制的图`组合`成为一个图。

* 2.右键单击图，另存为图片，格式为 `emf`。

* 3.在 `visio` 中插入 emf 图片。

* 4.导出 `tif` 格式图片，并设置分辨率。

### 其他

#### 网上下载的 PPT、Excel 等无法打开

点左上角：文件---选项（最后一个）--- 信任中心（最后一个）--- 信任中心设置 --- 受保护的视图，然后将前面两个勾去掉即可。



## Origin提示许可证过期

**提示错误**：

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-Origin%E6%97%A0%E6%B3%95%E6%BF%80%E6%B4%BB-45a4ce.png" alt="Origin无法激活" style="width:450px;" />

**解决方法**：

打开 `C:\ProgramData\OriginLab` 文件夹，删除 `Regid.lic` 文件和 `License` 文件夹，使用**管理员身份**重新激活 Origin。

## 修改 Gitee/GitHub 本地密码

**1.**
<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E4%BF%AE%E6%94%B9%20Gitee%20GitHub%20%E6%9C%AC%E5%9C%B0%E5%AF%86%E7%A0%811-59f133.png" alt="修改 Gitee GitHub 本地密码1" style="width:700px;" />

**2.**
<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E4%BF%AE%E6%94%B9%20Gitee%20GitHub%20%E6%9C%AC%E5%9C%B0%E5%AF%86%E7%A0%812-e96f0a.png" alt="修改 Gitee GitHub 本地密码2" style="width:700px;" />