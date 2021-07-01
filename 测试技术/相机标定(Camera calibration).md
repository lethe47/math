---
title:  相机标定(Camera calibration)
tags: 学习笔记,测试技术,计算机视觉
renderNumberedHeading: true
grammar_cjkRuby: true
---
# 为什么要相机标定？
在图像测量过程以及机器视觉应用中，为确定空间物体表面某点的三维几何位置与其在图像中对应点之间的相互关系，必须建立相机成像的几何模型，这些几何模型参数就是相机参数。  

通过相机标定可以做两件事
* 由于每个镜头的畸变程度各不相同，通过相机标定可以校正这种镜头畸变矫正畸变，生成矫正后的图像
* 根据获得的图像重构三维场景
# 相机标定方法有哪些？  
本次实验使用的应该是张氏标定法
## 传统相机标定法
传统相机标定法需要使用尺寸已知的标定物，通过建立标定物上坐标已知的点与其图像点之间的对应，利用一定的算法获得相机模型的内外参数。根据标定物的不同可分为三维标定物和平面型标定物。
三维标定物可由单幅图像进行标定，标定精度较高，但高精密三维标定物的加工和维护较困难。
平面型标定物比三维标定物制作简单，精度易保证，但标定时必须采用两幅或两幅以上的图像。传统相机标定法在标定过程中始终需要标定物，且标定物的制作精度会影响标定结果。同时有些场合不适合放置标定物也限制了传统相机标定法的应用。
## 基于主动视觉的相机标定法
该方法不需要标定物，但需要控制相机做某些特殊运动，利用这种运动的特殊性可以计算出相机内部参数。基于主动视觉的相机标定法的优点是算法简单，往往能够获得线性解，故鲁棒性较高，缺点是系统的成本高、实验设备昂贵、实验条件要求高，而且不适合于运动参数未知或无法控制的场合。
## 相机自标定法
目前出现的自标定算法中主要是利用相机运动的约束。相机的运动约束条件太强，因此使得其在实际中并不实用。利用场景约束主要是利用场景中的一些平行或者正交的信息。其中空间平行线在相机图像平面上的交点被称为消失点，它是射影几何中一个非常重要的特征，所以很多学者研究了基于消失点的相机自标定方法。自标定方法灵活性强，可对相机进行在线定标。但由于它是基于绝对二次曲线或曲面的方法，其算法鲁棒性差。
# 常用术语
内参矩阵: Intrinsic Matrix
焦距: Focal Length
主点: Principal Point
径向畸变: Radial Distortion
切向畸变: Tangential Distortion
旋转矩阵: Rotation Matrices
平移向量: Translation Vectors
平均重投影误差: Mean Reprojection Error
重投影误差: Reprojection Errors
重投影点: Reprojected Points
# 相机成像原理
![摄像机成像坐标系](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625115778910.png)
## 相机坐标系和世界坐标系转换
$$
\left[\begin{array}{c}
x_{c} \\
y_{c} \\
z_{c} \\
1
\end{array}\right]=\left[\begin{array}{cc}
\mathbf{R} & \mathbf{t} \\
0 & 1
\end{array}\right]\left[\begin{array}{c}
x_{w} \\
y_{w} \\
z_{w} \\
1
\end{array}\right]
$$
$\left[\begin{array}{cc}\mathbf{R} & \mathbf{t} \\ 0 & 1\end{array}\right]$即为相机的外参数矩阵
这个$R$矩阵就是一个$3*3$ 的旋转矩阵,由三个坐标方向的二维旋转转换相乘得来
![旋转矩阵](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625118885435.png)
$T$是$3*1$的平移矩阵![平移矩阵](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625118951293.png)
## 像素坐标系与图像坐标系
像素坐标系横纵坐标轴是以像素为单位,数值为正数
因像素坐标系不利于坐标变换,另建立图像坐标系,图像坐标系的坐标轴单位为毫米.
![像素,图像坐标系转换](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625119841169.png)
其中dx和dy表示每一列和每一行分别代表多少mm，即1pixel=dx mm
## 相机坐标系与图像坐标系
相机与图像坐标系变换依据如下图公式
![相机,图像坐标系](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625120117339.png)
## 世界坐标系与像素坐标系
![世界坐标与像素坐标的转换](https://raw.githubusercontent.com/lethe47/story-picture/main/小书匠/1625124678667.png)
由此公式可以看出相机内参合相机外参的定义,此为坐标变换
# 张氏变换
张氏变换为张正友博士发表的论文中提出的使用棋盘做相机参数标定的方法,好用
## 单映性矩阵
世界坐标系与像素坐标系转化公式简化为
$$
\left[\begin{array}{l}
u \\
v \\
1
\end{array}\right]=s\left[\begin{array}{ccc}
f_{x} & \gamma & u_{0} \\
0 & f_{y} & v_{0} \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{lll}
r_{1} & r_{2} & t
\end{array}\right]\left[\begin{array}{c}
x_{W} \\
y_{W} \\
1
\end{array}\right]
$$
令H为单应性矩阵
$$
H=s\left[\begin{array}{ccc}
f_{x} & \gamma & u_{0} \\
0 & f_{y} & v_{0} \\
0 & 0 & 1
\end{array}\right]\left[\begin{array}{lll}
r_{1} & r_{2} & t
\end{array}\right]=s M\left[\begin{array}{lll}
r_{1} & r_{2} & t
\end{array}\right]
$$
其中M为内参矩阵
## 矩阵求解
张氏变换通过标定棋盘中点与点的距离对H矩阵进行求解,求解过程看
https://mp.weixin.qq.com/s/0olxrct8GWUXJ35549bioA
https://mp.weixin.qq.com/s/aGNgpL_Df05KC-uxoyQOUA
中介绍
## Matlab
matlab标定得到的相机内参矩阵（IntrinsicMatrix)
matlab标定得到的旋转矩阵（RotationMatrices )
matlab标定得到的平移向量（TranslationVectors)


&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;
2021-07-01 12:48:56