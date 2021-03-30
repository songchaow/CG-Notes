

# 旋转的表示

## 欧拉角

- 按轴的分类，有两种习惯：
  - 沿物体内在的轴旋转 （intrinsic rotations）
  - 沿固定的坐标系轴旋转 （extrinsic rotations）

  似乎第二种通用一些。

- 按旋转次序分类，有三种习惯：

  **无论intrinsic还是extrinsic，都是旋转三次，但只用到了两个轴，就把一个物体旋转到所有可能的状态!**

  - x-y-x

  - y-x-y

  - ...

    

### Gimbal lock（万向锁）

https://en.wikipedia.org/wiki/Gimbal_lock

在三轴陀螺仪中，如两个gimbal旋转到对齐的位置，此时最外轴与最内轴的改变，所造成的影响是相同的。即损失了一个旋转自由度。

> 无论是何种表示，都至少有3个float去描述旋转，因为旋转的自由度为3。

**具体到实时渲染应用中，体现的问题**

如果我们使用这3个float去描述一个物体的旋转状态。则会存在某些状态下，我们对这些状态的修改只能在两个维度上进行，少了一个旋转维度。当然，我们可以先在两个维度上把状态修改了（解除锁定了），再去达到我们要的状态。

用另一种方式去解释：我们在锁的状态下，没办法在该点为某些方向的旋转求梯度。



# Camera

## 摄像机与旋转

摄像机会用到旋转，但只用到很有限的旋转：只旋转了view vector。

摄像机的旋转操作只含有2个自由度。

此节的讨论是平台无关的。我们先仅仅讨论矩阵的数学表示，暂忽略这些矩阵是怎么存储的。

##  摄像机的状态表示

The final purpose of a camera in real time rendering, is to provide two transforms: `world2cam` and `cam2ndc`. Both transforms are expressed in Matrix4x4 form.

However, to support user interaction, a camera state is usually maintained internally. 

实时渲染中摄像机的最终目的，是提供变换：`world2cam` and `cam2ndc`. 它们都是Matrix4x4形式。

如果直接使用矩阵作为状态，并通过矩阵乘法对该状态加以修改，该旋转矩阵会迅速失真，因为有数值误差。所以需要有不失真的状态，再从该状态表示成Matrix4x4。

欧拉角、四元数都是描述旋转操作（或者旋转状态）的不失真的好的表示。对于摄像机而言，完全可以直接用正则化后的view vector作为状态。

> 四元数适合用于描述增量的一个”旋转操作“。而描述view vector的状态，四元数其实不是很直观，大材小用、冗余了。
>
> 在其他三个自由度的旋转情形，四元数用来描述旋转状态还是很合适的。

User interaction modifies the state, before the two transform matrices can be converted from the camera state.

A camera state typically includes:

```c++
float pos[3];
float view_vec[3];
float up_vec[3];
```

首先需要计算摄像机的三个轴向量在world space中对应的方向向量：

$Z_c=view\_vec$ （左手系）

$Z_c=-view\_vec$ （右手系）

$Y_c=up\_vec$

$X_c=-Y_c \times Z_c$ （左手系）

$X_c=Y_c\times Z_c$ （右手系）

所以使用左手系的初衷，就是希望能使z轴的值能直接反映出深度，符合自觉。使用右手系的初衷，就是希望与数学中常用的坐标系相符合。

## `world2cam` calculation from axis vectors

$M_{w2c}X_{world}=>X_{cam}$

$\begin{bmatrix}X_{cx}&X_{cy}&X_{cz}&-Dot(\textbf{P},\textbf{X}_c)\\Y_{cx}&Y_{cy}&Y_{cz}&-Dot(\textbf{P},\textbf{Y}_c)\\Z_{cx}&Z_{cy}&Z_{cz}&-Dot(\textbf{P},\textbf{Z}_c)\\0&0&0&1\end{bmatrix}\begin{pmatrix}x_{pos}\\y_{pos}\\z_{pos}\\1\end{pmatrix}$

$\textbf{P}$ is the position of  the camera.

## `cam2world` calculation from axis vectors

$M_{c2w}X_{cam}=>X_{world}$

$\begin{bmatrix}X_{cx}&Y_{cx}&Z_{vx}&P_x\\X_{cy}&Y_{cy}&Z_{cx}&P_y\\X_{cz}&Y_{cz}&Z_{cz}&P_z\\0&0&0&1\end{bmatrix}\begin{pmatrix}x_{cam}\\y_{cam}\\z_{cam}\\1\end{pmatrix}$



# Camera Space Specification & Practice

- Camera Space

  应该可以由开发者自己决定。

  OpenGL常用的应是**右手系**：x指向右边，y指向头顶，z指向我们。

  违背常识的一点：**z轴并不是指向摄像机瞄准的方向，而是相反的方向**

- OpenGL/Direct3D NDC Space

  左手系

  **z轴指向屏幕内**

- Vulkan NDC Space

  **右手系**

# Matrix Storage and Calculation Procedure

## 乘法形式

从数学表示上说，Direct3D使用的矩阵是上面描述的矩阵的转置，因为多数D3D的代码使用$x\textbf{W}$的形式做乘法。

> 似乎，并没有强制规定，一定要是左乘还是右乘？

似乎多数实现会使用$x\textbf{W}$的形式。

## 内存布局

- Direct3D

  Shader storage layout:

  > Matrix packing order for uniform parameters is set to column-major by default.

  from https://docs.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-per-component-math

  如果我们同时使用$x\textbf{W}$的形式的矩阵，则喂进shader的内存布局，与row-major、$\textbf{W}x$的形式的矩阵对应的内存布局完全一样。