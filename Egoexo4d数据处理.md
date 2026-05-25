### 原始关键点数据格式

一个人体实例里面通常包含两份“同一个人”的信息：

- `annotation3D`：这个人在三维空间里的关节点位置
- `annotation2D`：这个人在每个相机画面，也就是二维图像里的关节点位置

---

##### 1. `annotation3D` 是什么：3D 骨架

`annotation3D` 里面按照关节名存储数据，例如：

- `nose`
- `left-shoulder`
- `right-knee`
- `left-wrist`
- `right-ankle`

常见人体骨架通常包含 17 个关节。

###### 示例

```json
"left-wrist": {
  "x": 2.8215,
  "y": 0.9125,
  "z": -0.5461,
  "num_views_for_3d": 4
}
```

##### 2. `annotation2D` 是什么：各相机下的 2D 关节点

`annotation2D` 首先按照相机分组，例如：

- `aria01`
- `cam01`
- `cam02`
- `cam03`
- `cam04`
- `cam05`

每个相机下面再按照关节名存储对应的二维像素点。

###### 示例

```bash
"cam05": {
  "left-wrist": {
    "x": 2182.28,
    "y": 1081.78,
    "placement": "manual"
  },
  "nose": {
    "x": 2527.95,
    "y": 890.89,
    "placement": "auto"
  }
}
```

字段含义

| 字段        | 含义                                 |
| ----------- | ------------------------------------ |
| `x`         | 该关节在当前相机图像中的横向像素坐标 |
| `y`         | 该关节在当前相机图像中的纵向像素坐标 |
| `placement` | 该 2D 点的标注来源                   |

`placement` 的含义

| 取值     | 含义                 |
| -------- | -------------------- |
| `manual` | 人工标注或人工修正   |
| `auto`   | 算法自动生成的关节点 |

需要注意的是，`annotation2D` 里的 `x, y` 是图像像素坐标，不是三维空间坐标。

------

##### 3. `annotation3D` 和 `annotation2D` 的关系

对于同一个关节，例如 `nose`，它会同时存在两种表示：

- 在 `annotation3D` 中：它是一个三维空间点
- 在 `annotation2D` 中：它是在不同相机图像上的二维投影点

可以理解为：

> `annotation3D` 表示这个关节在真实空间中的位置；
>  `annotation2D` 表示这个三维位置在不同摄像头画面中看到的像素位置。

也就是说：

```
同一个 3D 关节点
        ↓ 投影到不同相机图像
cam01 中有一个 2D 点
cam02 中有一个 2D 点
cam03 中有一个 2D 点
...
```

------

##### 4. 一个完整的人体实例示例

```python
{
  "annotation3D": {
    "nose": {
      "x": 2.10,
      "y": 1.45,
      "z": -0.30,
      "num_views_for_3d": 4
    },
    "left-shoulder": {
      "x": 1.95,
      "y": 1.30,
      "z": -0.25,
      "num_views_for_3d": 4
    }
  },
  "annotation2D": {
    "cam01": {
      "nose": {
        "x": 1899.07,
        "y": 806.67,
        "placement": "manual"
      },
      "left-shoulder": {
        "x": 2008.11,
        "y": 854.44,
        "placement": "manual"
      }
    },
    "cam05": {
      "nose": {
        "x": 2527.95,
        "y": 890.89,
        "placement": "auto"
      },
      "left-shoulder": {
        "x": 2357.82,
        "y": 1171.32,
        "placement": "manual"
      }
    }
  }
}
```

### 原始关键点特征包含

```bash
nose
left-eye
right-eye
left-ear
right-ear
left-shoulder
right-shoulder
left-elbow
right-elbow
`left-wrist`
`right-wrist`
`left-hip
`right-hip
left-knee
right-knee
`left-ankle
`right-ankle
```

### 清洗

5 个 3D 点：

- `left-wrist`
- `right-wrist`
- `left-ankle`
- `right-ankle`
- `pelvis-mid`（由 `left-hip` 和 `right-hip` 取中点）





![0a6720a4-e840-4e60-9e7b-77a0743d5e32](assets/0a6720a4-e840-4e60-9e7b-77a0743d5e32.gif)

![vis_0a6720a4_5kp](assets/vis_0a6720a4_5kp.gif)



### Retargeting

















































