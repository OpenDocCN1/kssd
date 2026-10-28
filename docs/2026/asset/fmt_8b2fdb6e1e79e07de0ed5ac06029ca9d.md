
```python
def change_size(self, file):
    image = cv2.imread(file, 1)  # 读取图片 image_name应该是变量
    img = cv2.medianBlur(image, 5)  # 中值滤波，去除黑色边际中可能含有的噪声干扰
    b = cv2.threshold(img, 15, 255, cv2.THRESH_BINARY)  # 调整裁剪效果
    binary_image = b[1]  # 二值图--具有三通道
    binary_image = cv2.cvtColor(binary_image, cv2.COLOR_BGR2GRAY)
    x, y = binary_image.shape
    edges_x = []
    edges_y = []
    for i in range(x):
        for j in range(y):
            if binary_image[i][j] == 255:
                edges_x.append(i)
                edges_y.append(j)
    left = min(edges_x)  # 左边界
    right = max(edges_x)  # 右边界
    width = right - left  # 宽度
    bottom = min(edges_y)  # 底部
    top = max(edges_y)  # 顶部
    height = top - bottom  # 高度
    pre1_picture = image[left:left + width, bottom:bottom + height]  # 图片截取
    return pre1_picture  # 返回图片数据
```

# 算出缺口的位置

```python
def match(self, target, template):
    img_gray = cv2.imread(target, 0)
    img_rgb = self.change_size(template)
    template = cv2.cvtColor(img_rgb, cv2.COLOR_BGR2GRAY)
    res = cv2.matchTemplate(img_gray, template, cv2.TM_CCOEFF_NORMED)
    run = 1
    # 使用二分法查找阈值的精确值
    L = 0
    R = 1
    while run < 20:
        run += 1
        threshold = (R + L) / 2
        if threshold < 0:
            print('Error')
            return None
        loc = np.where(res >= threshold)
        if len(loc[1]) > 1:
            L += (R - L) / 2
        elif len(loc[1]) == 1:
            break
        elif len(loc[1]) < 1:
            R -= (R - L) / 2
    return loc[1][0]
```

# 模拟变加速

```python
def get_tracks(self, distance, seconds, ease_func):
    distance += 20
    tracks = [0]
    offsets = [0]
    for t in np.arange(0.0, seconds, 0.1):
        ease = ease_func
        offset = round(ease(t / seconds) * distance)
        tracks.append(offset - offsets[-1])
        offsets.append(offset)
    tracks.extend([-3, -2, -3, -2, -2, -2, -2, -1, -0, -1, -1, -1])
    return tracks

def ease_out_quart(self, x):
    return 1 - pow(1 - x, 4)
```

# 完整代码
