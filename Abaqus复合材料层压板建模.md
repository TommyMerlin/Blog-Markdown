---
title: Abaqus复合材料层压板建模
date: 2020-06-10 11:25:35
tags:
- Abaqus
- 有限元
- 复合材料
categories:
- 复合材料
mathjax: true
---

**学习资料**

- 1.《ABAQUS分析之美》江丙云,2020,人民邮电出版社
- 2.《复合材料力学与结构设计》王耀先,2012,华东理工大学出版社
- 3.[Abaqus复合材料建模](https://space.bilibili.com/381464575/video)
- 4.[Abaqus中如何创建变厚度复合材料层板有限元模型](https://mp.weixin.qq.com/s?__biz=MzI3MTE3OTgzNA==&mid=2649180241&idx=1&sn=e78447652fe81f9dc9ee8771d63bbd79&chksm=f2d62104c5a1a81274638d4a7c78906f0f4cc7bb1c1324c5c229ac441957b452afc2c87f2507&scene=21#wechat_redirect)

<!-- more -->

## 1.网格划分

### 1.1指派网格控制属性

- **中性轴算法**—将目标区域划分为一些简单区域
  - 形状规则
  - 网格节点与种子的位置吻合差
- **进阶算法**—现在目标区域边界生成四边形网格，然后逐步向区域内部扩展
  - 易获得大小均匀的网格
  - 网格节点与种子位置吻合较好
  - 易出现网格形状歪斜、扭曲的问题

### 1.2 分割部件

设定工具栏颜色编码模式为 `Mesh defaults`

- **橙色**：无法划分，需进一步对几何体进行分割
- **黄色**：扫掠网格
- **粉红色**：自由网格
- **绿色**：结构化网格

**注意**：结构化网格和扫掠网格采用四边形（二维）和六面体（三维），其分析精度较高，应优先选择。

---

## 2.胶接特性模拟(1-P94)

- 使用 **Cohesive 单元**模拟胶接
  - 定义材料属性/截面，给 Cohesive 单元赋 Cohesive 截面属性，以此模拟胶层
- 使用**接触（Contact）**模拟胶接
  - 在需要胶接的面之间创建接触，接触特性选用 Cohesive Behavior

---

## 3.层压板有限元分析

### 3.1 复合材料本构关系

#### 3.1.1 Isotropic(各向同性材料)

$$
\left\{\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}=\left[\begin{array}{cccccc}
1 / E & -\nu / E & -\nu / E & 0 & 0 & 0 \\
-\nu / E & 1 / E & -\nu / E & 0 & 0 & 0 \\
-\nu / E & -\nu / E & 1 / E & 0 & 0 & 0 \\
0 & 0 & 0 & 1 / G & 0 & 0 \\
0 & 0 & 0 & 0 & 1 / G & 0 \\
0 & 0 & 0 & 0 & 0 & 1 / G
\end{array}\right]\left\{\begin{array}{l}
\sigma_{11} \\
\sigma_{22} \\
\sigma_{33} \\
\sigma_{12} \\
\sigma_{13} \\
\sigma_{23}
\end{array}\right\}
$$

$$
G=E/2(1+\mu)
$$

**2个**独立的工程常数：杨氏模量、泊松比

#### 3.1.2 Engineering Constant(三维工程常数)

$$
\left\{\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}=\left[\begin{array}{cccccc}
1 / E_{1} & -\nu_{21} / E_{2} & -\nu_{31} / E_{3} & 0 & 0 & 0 \\
-\nu_{12} / E_{1} & 1 / E_{2} & -\nu_{32} / E_{3} & 0 & 0 & 0 \\
-\nu_{13} / E_{1} & -\nu_{23} / E_{2} & 1 / E_{3} & 0 & 0 & 0 \\
0 & 0 & 0 & 1 / G_{12} & 0 & 0 \\
0 & 0 & 0 & 0 & 1 / G_{13} & 0 \\
0 & 0 & 0 & 0 & 0 & 1 / G_{23}
\end{array}\right]\left\{\begin{array}{c}
\sigma_{11} \\
\sigma_{22} \\
\sigma_{33} \\
\sigma_{12} \\
\sigma_{13} \\
\sigma_{23}
\end{array}\right)
$$

材料稳定性条件：
$$
\begin{array}{l}
E_{1}, E_{2}, E_{3}, G_{12}, G_{13}, G_{23}>0 \\
\left|\nu_{12}\right|<\left(E_{1} / E_{2}\right)^{1 / 2} \\
\left|\nu_{13}\right|<\left(E_{1} / E_{3}\right)^{1 / 2} \\
\left|\nu_{23}\right|<\left(E_{2} / E_{3}\right)^{1 / 2} \\
1-\nu_{12} \nu_{21}-\nu_{23} \nu_{32}-\nu_{31} \nu_{13}-2 \nu_{21} \nu_{32} \nu_{13}>0
\end{array}
$$
**9个**独立的工程常数。

用于**体单元**。

#### 3.1.3 Lamina(二维)

**平面应力**状态下的**正交各向异性**材料
$$
\left\{\begin{array}{c}
\varepsilon_{1} \\
\varepsilon_{2} \\
\gamma_{12}
\end{array}\right\}=\left[\begin{array}{ccc}
1 / E_{1} & -\nu_{12} / E_{1} & 0 \\
-\nu_{12} / E_{1} & 1 / E_{2} & 0 \\
0 & 0 & 1 / G_{12}
\end{array}\right]\left\{\begin{array}{c}
\sigma_{11} \\
\sigma_{22} \\
\tau_{12}
\end{array}\right\}
$$
Abaqus 还需输入 $G_{23},G_{13}$ 用于计算壳的横向剪切变形。

用于**壳单元**。

#### 3.1.4 Orthotropic(正交各向异性)

$$
\left\{\begin{array}{l}
\sigma_{11} \\
\sigma_{22} \\
\sigma_{33} \\
\sigma_{12} \\
\sigma_{13} \\
\sigma_{23}
\end{array}\right)=\left[\begin{array}{cccccc}
D_{1111} & D_{1122} & D_{1133} & 0 & 0 & 0 \\
& D_{2222} & D_{2233} & 0 & 0 & 0 \\
& & D_{3333} & 0 & 0 & 0 \\
& & & D_{1212} & 0 & 0 \\
& s y m & & & D_{1313} & 0 \\
& & & & &D_{2323}
\end{array}\right]\left\{\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}=\left[D^{e l}\right]\left\{\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}
$$

三维正交各向异性材料：三个对称面，直接指定弹性矩阵，共 9 个独立弹性系数。
$$
\begin{array}{l}
D_{1111}=E_{1}\left(1-\nu_{23} \nu_{32}\right) \Upsilon \\
D_{2222}=E_{2}\left(1-\nu_{13} \nu_{31}\right) \Upsilon \\
D_{3333}=E_{3}\left(1-\nu_{12} \nu_{21}\right) \Upsilon \\
D_{1122}=E_{1}\left(\nu_{21}+\nu_{31} \nu_{23}\right) \Upsilon=E_{2}\left(\nu_{12}+\nu_{32} \nu_{13}\right) \Upsilon \\
D_{1133}=E_{1}\left(\nu_{31}+\nu_{21} \nu_{32}\right) \Upsilon=E_{3}\left(\nu_{13}+\nu_{12} \nu_{23}\right) \Upsilon \\
D_{2233}=E_{2}\left(\nu_{32}+\nu_{12} \nu_{31}\right) \Upsilon=E_{3}\left(\nu_{23}+\nu_{21} \nu_{13}\right) \Upsilon \\
D_{1212}=G_{12} \\
D_{1313}=G_{13} \\
D_{2323}=G_{23}
\end{array}
$$

$$
\Upsilon=\frac{1}{1-\nu_{12} \nu_{21}-\nu_{23} \nu_{32}-\nu_{31} \nu_{13}-2 \nu_{21} \nu_{32} \nu_{13}}
$$

**注意**：**单层板**使用 Engineering Constant 或 Orthotropic 结果相同，Engineering Constant 可以做**多层板**的分析，可将等效的工程常数代入进行计算。

#### 3.1.5 Anisotropic(完全各向异性)

$$
\left\{\begin{array}{l}
\sigma_{11} \\
\sigma_{22} \\
\sigma_{33} \\
\sigma_{12} \\
\sigma_{13} \\
\sigma_{23}
\end{array}\right\}=\left[\begin{array}{llllll}
D_{1111} & D_{1122} & D_{1133} & D_{1112} & D_{1113} & D_{1123} \\
& D_{2222} & D_{2233} & D_{2212} & D_{2213} & D_{2223} \\
&&D_{3333} & D_{3312} & D_{3313} & D_{3323} \\
&&&D_{1212} & D_{1213} & D_{1223} \\
&sym&&&D_{1313} & D_{1323} \\
&&&&&D_{2323}
\end{array}\right]\left\{\begin{array}{l}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}=\left[D^{e l}\right]\left\{\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\gamma_{12} \\
\gamma_{13} \\
\gamma_{23}
\end{array}\right\}
$$

三维完全各向异性材料：直接指定弹性矩阵，无对称面，共 21 个独立弹性系数。

#### 3.1.6 Transversely Isotropic(横观各向同性)

$$
\left\{\begin{array}{l}
\sigma_{11} \\
\sigma_{22} \\
\sigma_{33} \\
\sigma_{23} \\
\sigma_{31} \\
\sigma_{12}
\end{array}\right\}=\left[\begin{array}{cccccc}
C_{11} & C_{12} & C_{12} & 0 & 0 & 0 \\
C_{21} & C_{22} & C_{23} & 0 & 0 & 0 \\
C_{21} & C_{23} & C_{22} & 0 & 0 & 0 \\
0 & 0 & 0 & \frac{1}{2}\left(C_{22}-C_{23}\right) & 0 & 0 \\
0 & 0 & 0 & 0 & C_{66} & 0 \\
0 & 0 & 0 & 0 & 0 & C_{66}
\end{array}\right]\left[\begin{array}{c}
\varepsilon_{11} \\
\varepsilon_{22} \\
\varepsilon_{33} \\
\varepsilon_{23} \\
\varepsilon_{31} \\
\varepsilon_{12}
\end{array}\right]
$$

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%A8%AA%E8%A7%82%E5%90%84%E5%90%91%E5%90%8C%E6%80%A7-789aac.png" alt="横观各向同性" width="30%" />

可指定为 Engineering Constant。

### 3.2 层压板建模方法(1-P103)

#### 3.2.1 由单元不同，分为4个方案

- 壳模型、壳单元
- 实体模型、连续壳单元、厚度方向划分一层单元
- 实体模型、体单元、厚度方向划分一层单元
- 实体模型、体单元、厚度方向划分多层单元

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E5%B1%82%E5%8E%8B%E6%9D%BF%E5%BB%BA%E6%A8%A1%E5%8D%95%E5%85%83%E6%A8%A1%E5%9E%8B-b358b8.png" alt="横观各向同性" width="40%" />

#### 3.2.2 普通壳单元与连续壳单元比较

|              | 普通壳单元       | 连续壳单元       |
| ------------ | ---------------- | ---------------- |
| 依附几何     | 面               | 实体             |
| 厚度确定方式 | 截面属性         | 通过单元节点确定 |
| 节点自由度   | 平动及转动自由度 | 仅平动自由度     |

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%99%AE%E9%80%9A%E5%A3%B3%E5%92%8C%E8%BF%9E%E7%BB%AD%E5%A3%B3-73ab1e.png" alt="横观各向同性" width="30%" />

- 对于薄板，普通壳单元精度会更高一些。
- 对于接触问题，连续壳单元可以在接触中考虑双面接触及厚度的变化，精度更高。

### 3.3 普通壳单元建模方法

[教程视频](https://www.bilibili.com/video/BV1wf4y1m7jg)

**流程**

> Part①➡Material②➡Section③➡Assign section➡Assign material orientation④➡Create mesh➡Assign element type➡Assembly➡Step⑤➡output➡Interaction➡Load➡Job➡Visualization

**①Part**：3D—Deformable—Shell—Planar

**②Property / Material**：Elasticity—Elastic—Lamina

**③Property / Section**：Shell—Composite

- During analysis
  - 在积分过程中，每个增量步重新计算截面属性，计算量较大，准确性较高，当材料有明显非线性（塑性、损伤后刚度退化等）时，建议使用。
- Before analysis
  - 在计算前对截面属性进行预积分，分析过程中截面属性不再变化，计算量小。

**④Assign material orientation**

- Create Datum Coordinate：Rectangular 建立材料坐标系
- Assign material orientation：指派材料方向

**⑤Step**：Field Output

- 输出每一层节点的数据

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-field%20output-7c72df.png" alt="横观各向同性" width="40%" />

- 确定节点编号

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-section%20points-4121f2.png" alt="横观各向同性" width="30%" />

### 3.4 连续壳单元建模方法

[教程视频](https://www.bilibili.com/video/BV1ia4y1v7S8)

**流程**

> Part①➡Material②➡Section③➡Assign section➡Assign material orientation➡Create mesh④➡Assign element type⑤➡Assembly➡Step⑥➡output➡Interaction➡Load➡Job➡Visualization

**①Part**：3D—Deformable—Solid

**②Property / Material**：Elasticity—Elastic—Lamina

**③Property / Section**：Shell—Composite

**④Assign Mesh Controls**：Element Shape—Hex，Technique—Sweep

**⑤Assign Element Type**：Family 中改为 Continuum Shell：SC8R

### 3.5 多层实体单元建模方法

**①Part**：3D—Deformable—Solid

**②Property / Material**：Elasticity—Elastic—Engineering Constants

**③Property / Section**：Solid—Composite

**④Assign Mesh Controls**：Element Shape—Hex，Technique—Sweep

**⑤Assign Element Type**：C3D8R

**⑥Step**：Field Output

- 输出每一层节点的数据

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-1-0193c3.png" alt="横观各向同性" width="40%" />

- 只需输入 1,2,3,4 即可，因为每个铺层的积分点为 1

### 3.6 小结

|               | 普通壳          | 连续壳          | 多层实体              |
| ------------- | --------------- | --------------- | --------------------- |
| Part          | Shell—Planar    | Solid           | Solid                 |
| Material      | Lamina          | Lamina          | Engineering Constants |
| Section       | Shell—Composite | Shell—Composite | Solid—Composite       |
| Mesh Controls | Quad，Free      | Hex，Sweep      | Hex，Sweep            |
| Element Type  | S4R             | SC8R            | C3D8R                 |

