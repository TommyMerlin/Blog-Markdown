---
title: Abaqus中内部接触的创建方法
date: 2020-08-19 14:45:35
tags:
- Abaqus
- 有限元
- 复合材料
categories:
- 复合材料
mathjax: true
---

**学习资料**

- 1.[Abaqus中内部接触的创建方法（一）](https://mp.weixin.qq.com/s?__biz=MzI3MTE3OTgzNA==&mid=2649176107&idx=1&sn=eb19f5400d7e8d564450f889b0a5e59f&chksm=f2d6517ec5a1d8683ec3c4a951150e5d8451ca528c7317871d028be44c5543232533325c0326&scene=21#wechat_redirect)
- 2.[Abaqus中内部接触的创建方法（二）](https://mp.weixin.qq.com/s?__biz=MzI3MTE3OTgzNA==&mid=2649176122&idx=1&sn=b2405deac5548cc84e447f993f04504e&chksm=f2d6516fc5a1d87988fee55f67870a2aefdbfbb823c139da573c4c45229dff609459d7b64081&scene=21#wechat_redirect)
- 3.[复合材料渐进损伤分析中内部接触的三种创建方法](https://mp.weixin.qq.com/s?__biz=MzI3MTE3OTgzNA==&mid=2649176160&idx=1&sn=2408f6b19b575aaaed0d0a9efd3acab8&chksm=f2d65135c5a1d823ad276a7d26695ed11208708cb629f91060e714eafc0b99e2d50f25f0207d&scene=21#wechat_redirect)

<!-- more -->

## 1.必要性

在做复合材料冲击、压溃、切削、钻削等问题仿真分析时，随着分层或者面内损伤的扩展，部分单元会逐渐被删除，**局部单元删除后，与被删除单元相邻的单元上共用的内部单元面便会裸露出来，而这些内部单元面默认是不会被考虑在接触范围之内的**。

以下图所示的冲击损伤为例，当脆性材料开裂以后，如果不考虑内部接触，开裂产生的新的内部单元面与底板之间无接触关系，则破碎单元将会穿透底板，达不到预期的效果。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-1-bdf93c.gif)

同样的，下面的复合材料层压板压溃分析中，也有类似的问题，当分层产生以后，层间 cohesive 单元被删除，cohesive 单元两侧的层板单元面裸露出来，如果不考虑内部接触，两侧单元会互相穿透。另外，当层板单元被删除后，如果不考虑内部接触，加载板/支撑板与层压板之间也将失去接触关系，使得载荷施加不上去，加载板/支撑板直接穿过层板网格。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-2-ddb986.gif)

## 2.内部接触创建方法

### 2.1 方法一

该方法为**官方**正式方法，**基于单元集合创建内部面，然后将内部面包含在通用接触中**。具体实施步骤如下：

- 1.基于 Part 创建单元集合，该单元集合须包含所有可能进入内部接触区域的单元，并对单元集合命名，例如命名为 “ALL_ELEMS”。

- 2.同样在 Part 层面创建一个 Surface，类别为基于单元类型，在视图中选择对象时可以选择某一个单元的某一个面作为一个假面（并非真正参与内部接触的面），并进行命名，例如命名为 “inner_surf”。

- 3.在 interaction 模块创建通用接触，通用接触中将上一步创建的内面 “inner_surf” 包含进去，注意，此时的内面为假面。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-3-af35f6.png)

- 4.进入 Job 模块，写出 inp 文件，并用文本编辑器将 inp 文件打开，通过关键字搜索找到定义名为 “inner_surf” 的位置，一般为以下形式：

  ```python
  *Surface, type=ELEMENT, name=inner_surf
  _inner_surf_S3, S3
  ```

  将上述关键字按照以下形式进行修改并保存：

  ```python
  *Surface, type=ELEMENT, name=inner_surf
  ALL_ELEMS, interior
  ```

  其中，ALL_ELEMS 即为第一步创建的单元集合，interior 关键字代表内部面。至此内部单元面及内部接触就创建完成了。

  保存 inp 以后，再次提交任务时，可以用 Abaqus Command 来提交，也可以在 CAE Job 模块提交任务，后者提交任务时，切记不要在原模型任务中直接提交，否则将覆盖掉刚刚修改过的 inp，正确的方式为创建一个新的 Job，Job 类型选择 Input file 而不是默认的Model：

  ![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-4-1ea673.png)

### 2.2 方法二

该方法基于**单元节点集合创建内部面，然后将内部面包含在面面接触中，而不是通用接触中**。具体实施步骤如下：

- 1.基于 Part 创建节点集合，该集合须包含所有可能进入内部接触区域的节点，并对节点集合命名，例如命名为 “ALL_NODES”。

- 2.在 interaction 模块创建面面接触，注意不是通用接触，然后选择个主面，再选择一个从面，从面类型为 “node region”，并指向所创建的节点集合 “ALL_NODES”。由此可见，基于 node 的内部面仅能用于面面接触中的从面。

当基于节点的内面生成以后，inp 文件中会多出一行，

```python
*SURFACE, NAME=name, TYPE=NODE
node number or node set, area
```

举个例子：

```python
*Surface, type=NODE, name=surf_node
plate-mesh-1.all_node, 1.
```

第二行的第一个参数即为**所定义的单元集合**，也可以直接写单元节点号；

第二行的第二个参数为**节点对应的接触面积**，Abaqus/Explicit 中默认1.0，也可以自己指定。还需要注意的是，默认接触面积是 1.0 时，所输出的接触压力实际上就是节点上的接触力。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-5-f864e0.png)

这种方法操作简单，不需要修改关键字，**但是也存在一些问题**：

- 1.基于节点的内部面只能做从面。
- 2.当对接触应力的精度要求较高时，慎用此方法。
- 3.复合材料渐进损伤分析中，无法考虑层间损伤以后，层与层之间的接触。

### 2.3 方法三

该方法与第一种方法类似，**基于单元面创建内部接触面，然后将内部面包含在通用接触中，而不是面面接触中**。具体实施步骤如下：

- 1.切换到 mesh 模块下，对象也切换到 part 层面，选择需要创建内面的 part。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-6-e791bf.png" alt="6" width="60%" />

- 2.点击菜单 Tools→Surface→Create，创建曲面，入股你的模型中有 cohesive 单元，请先将 cohesive 单元隐藏掉在创建曲面。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-7-0bf3ba.png)

- 3.选择 Surfaace 的类型为 Mesh，如果你的模型中没有几何，只有孤立网格，此步跳过。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-8-19b0dd.png)

- 4.**关键的一步**：长按下图红框标注的对象选择按钮，保持三秒。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-9-873a52.png)

这时候会弹出三个按钮，如下图所示，第一个表示内部外部对象都选，第二个只选择内部对象，第三个只选择外部对象，默认的就是第三个，只选择外部对象。而我们现在要将其切换到第一个或者第二个，只要能选中内部单元面即可。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-10-1fa663.png)

- 5.到视图中去框选有可能会出现单元删除的区域的单元面，如下图所示。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-11-1690a6.png)

然后点击鼠标中键或者 Done 确认，此时，视图下方会出现以下提示。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-12-616fe8.png)

选择 Both sides 即可完成内面的创建。

这个时候在你的 inp 文件中可以看到所创建的面包含了所有被选单元的所有面。

- 6.创建通用接触。其方法和第一种内部接触创建方法一样。

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-13-ec5b92.png)