---
author: fireowl
banner_path: Attach/Image/demo-1.png
open_comment: 1
---
# 成本图障碍物扩展函数详解

## 概述

`costmap` 函数用于处理机器人的成本图（Costmap）数据，对障碍物进行膨胀/扩展处理，确保机器人与障碍物保持安全距离。
## 函数签名

```python
def costmap(data, width, height, resolution):
```
## 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `data` | list/numpy.array | 原始成本图数据（一维数组） |
| `width` | int | 成本图宽度（列数） |
| `height` | int | 成本图高度（行数） |
| `resolution` | float | 分辨率（每个格子代表的实际距离，单位：米/格） |
![[costmap_function_explained-2.png]]
## 函数流程

### 1. 数据重塑

```python
data = np.array(data).reshape(height, width)
```

将一维数据重塑为 `height × width` 的二维矩阵，便于进行空间操作。

### 2. 障碍物检测

```python
wall_mask = data == 100
```

创建一个布尔掩码，标记所有障碍物位置（值为100的格子）。

### 3. 障碍物扩展（核心算法）

```python
for i in range(-expansion_size, expansion_size + 1):
    for j in range(-expansion_size, expansion_size + 1):
        if i == 0 and j == 0:
            continue
        shifted_mask = np.roll(wall_mask, (i, j), axis=(0, 1))
        data[shifted_mask] = 100
```

#### 算法原理

- 使用**双层循环**遍历扩展范围内的每个偏移量 `(i, j)`
- `expansion_size` 定义了扩展半径（通常为3-5个格子）
- `np.roll()` 将障碍物掩码沿 `(i, j)` 方向平移
- 将平移后的掩码位置标记为障碍物（值100）

#### 图示

```
扩展前 (expansion_size=1):
    . . . . .
    . O O . .
    . O X O .
    . O O . .
    . . . . .
    
X = 原始障碍物
O = 扩展后的障碍物
. = 自由空间

扩展后 (expansion_size=2):
    . O O O O .
    O O O O O O
    O O X X O O
    O O X X O O
    O O O O O O
    . O O O O .
```

### 4. 分辨率转换

```python
data = data * resolution
```

将整数成本值转换为实际物理距离（米）。

## 可视化
### 动画演示

![成本图动画](costmap_animation.gif)

## 算法特点

### 优点

1. **简洁直观**：使用循环和 `np.roll` 实现，易于理解和维护
2. **无需显式循环遍历每个格子**：利用 NumPy 向量化操作
3. **逐步扩展**：可以观察障碍物膨胀的完整过程

### 缺点

1. **性能较低**：双重循环遍历 `(2×expansion_size+1)²` 次
2. **边界处理**：`np.roll` 会将移出边界的格子从另一侧移回，可能产生伪影
3. **可优化空间**：可考虑使用卷积操作或 scipy 的 `binary_dilation` 替代

## 性能分析

对于 `expansion_size = 3`：
- 循环次数：`(2×3+1)² - 1 = 48` 次
- 每次操作复杂度：O(height × width)

## 优化建议

```python
# 方案1：使用 scipy.ndimage.binary_dilation
from scipy.ndimage import binary_dilation
wall_mask = data == 100
structure = np.ones((2*expansion_size+1, 2*expansion_size+1))
expanded_mask = binary_dilation(wall_mask, structure=structure)
data[expanded_mask] = 100

# 方案2：使用卷积操作
from scipy.signal import convolve2d
kernel = np.ones((2*expansion_size+1, 2*expansion_size+1))
overlap = convolve2d(wall_mask.astype(int), kernel, mode='same')
data[overlap > 0] = 100
```

## 应用场景

- 机器人路径规划（ROS Navigation Stack）
- 自动化仓储系统避障
- 自动驾驶局部路径规划
- 无人机室内导航


我来总结一下这篇笔记的核心内容：

## 📋 核心要点

**函数功能**：对机器人成本图进行障碍物膨胀处理，确保安全距离

**关键参数**：
- `data`: 一维成本图数组
- `width/height`: 尺寸
- `resolution`: 物理分辨率（米/格）

**核心算法**：
使用双重循环 + `np.roll()` 向四周扩展障碍物（值为100的格子）

**主要优缺点**：

| 优点 | 缺点 |
|------|------|
| 简洁直观 | 性能较低（双重循环） |
| 向量化操作 | 边界处理有伪影 |
| 易于维护 | 未充分利用优化算法 |

**推荐优化**：使用 `scipy.ndimage.binary_dilation` 或卷积操作替代

**典型应用**：ROS导航、仓储避障、自动驾驶、无人机导航
