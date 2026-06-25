# 3D 照片丝带环绕旋转效果

## 项目概述
单个 HTML 文件，使用 Three.js（CDN 引入）实现 3D 照片丝带环绕旋转效果——照片排列成椭圆形轨道，绕 Y 轴旋转，中心有一根发光柱子。

## 技术栈
- Three.js（CDN import map 方式引入）
- 纯 HTML/CSS/JS，单文件，零依赖

## 核心功能

### 3D 场景
- 24 张占位图片（使用 `https://picsum.photos/300/200?random=N`，N 为 1~24）
- 照片沿 Y 轴排列成椭圆形轨道
- 每张照片与轨道切线方向一致，形成丝带环绕效果（照片面朝外、沿轨道切向排列）
- 照片间距均匀分布（360° / 照片数量）
- 整体缓慢平滑地自动旋转

### 交互控制
- **OrbitControls**：鼠标拖拽自由旋转视角
- **滚轮**：拉近拉远
- **悬停暂停**：鼠标移到画布上时，自动旋转暂停

### 视觉风格
- 深色渐变背景（深海军蓝到近乎黑色）
- 雾效增加深度感/氛围感
- 中心半透明发光柱子（圆柱体 + 自发光材质或自定义着色器）
- 照片带圆角效果
- 照片带轻微投影阴影
- 鼠标悬停单张照片时略微放大（光线投射检测）
- 整体偏科技感 / 梦幻感

## 可调参数（放在脚本顶部）

```js
const CONFIG = {
  radius: 5,            // 轨道半径
  count: 24,            // 照片数量
  speed: 0.3,           // 自动旋转速度（弧度/秒）
  photoWidth: 1.2,      // 照片宽度
  photoHeight: 0.8,     // 照片高度
  pillarHeight: 8,      // 中心柱子高度
  pillarRadius: 0.15,   // 中心柱子半径
  fogNear: 8,           // 雾效近平面
  fogFar: 25,           // 雾效远平面
};
```

## 文件结构
- 单文件：`index.html`，放在项目根目录
- 所有 CSS 内联在 `<style>` 标签中
- 所有 JS 内联在 `<script type="module">` 标签中

## 关键实现细节

### 纹理加载
- 使用 `picsum.photos` 的 URL，带上随机种子参数
- 用 `THREE.TextureLoader` 预加载所有纹理，加载完后再启动场景
- 加载失败时优雅降级（回退到纯色材质）

### 照片网格
- 每张照片使用 `THREE.PlaneGeometry`
- 材质：`THREE.MeshStandardMaterial` 或 `THREE.MeshPhongMaterial`，贴上纹理
- 设置 `material.side = THREE.DoubleSide`，使照片双面可见

### 轨道运动逻辑
- 每张照片在椭圆上的位置计算：`x = radius * cos(angle)`，`z = radius * sin(angle)`
- 每帧按 `speed * deltaTime` 递增角度
- **关键**：照片平面沿轨道切向排列（而非面向圆心），面朝外，形成丝带效果
- 实现方式：计算轨道切向 `tangent = (-sin(θ), 0, cos(θ))`，将照片的宽度方向对齐切向，法线朝外 `radial = (cos(θ), 0, sin(θ))`，用 `lookAt` 指向径向朝外的点即可

### 中心柱子
- `THREE.CylinderGeometry` + 半透明材质
- 可选：再加一个稍大、透明度更低的圆柱体做光晕
- 可选：在中心放一个点光源增加照明

### 光线投射（悬停检测）
- 使用 `THREE.Raycaster`，在 `pointermove` 事件中检测
- 鼠标悬停到照片上时：缩放到 1.15 倍
- 鼠标移开时：缩放回 1.0 倍
- 使用简单线性插值实现平滑过渡

### 自动旋转暂停
- 用 `isHovering` 布尔变量追踪鼠标是否在画布上
- `mouseenter` → 暂停自动旋转，`mouseleave` → 恢复
- 暂停的只是轨道自转，OrbitControls 不受影响（用户仍可拖拽）

### 雾效
- `scene.fog = new THREE.Fog(0x0a0a1a, CONFIG.fogNear, CONFIG.fogFar)`

### 灯光
- 环境光提供基础照明
- 方向光增加立体感/阴影
- 可选：在柱子内部放点光源做发光效果

## CDN 引入方式
```html
<script type="importmap">
{
  "imports": {
    "three": "https://unpkg.com/three@0.170.0/build/three.module.js",
    "three/addons/": "https://unpkg.com/three@0.170.0/examples/jsm/"
  }
}
</script>
```

## 响应式
- Canvas 铺满视口
- 监听窗口大小变化：同步更新相机宽高比和渲染器尺寸
