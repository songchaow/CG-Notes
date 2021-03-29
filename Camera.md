

# Camera

此节的讨论是平台无关的。我们先仅仅讨论矩阵的数学表示，暂忽略这些矩阵是怎么存储的。

##  Camera State

The final purpose of a camera in real time rendering, is to provide two transforms: `world2cam` and `cam2ndc`. Both transforms are expressed in Matrix4x4 form.

However, to support user interaction, a camera state is usually maintained internally. 

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

$\begin{bmatrix}\end{bmatrix}$

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

