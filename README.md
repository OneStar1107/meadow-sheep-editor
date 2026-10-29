# 草原绵羊编辑器 · Meadow Sheep Editor

> 单文件 three.js 立体草原：可编辑的羊群、随机风场、流动河水和成熟稻田。
>
> A single-file three.js meadow diorama with an editable flock, random wind, flowing water and a ripe rice paddy.

## 这是什么 / What it is

一个用 **three.js 0.160** 写的交互式微缩草原场景，全部内容放在一个 **index.html** 里，双击即开。画面是手绘纸艺/软陶质感的圆角草皮岛：11 只黑脸绵羊、一棵紫色树、一条会流动的河、一条土路、一块成熟稻田，以及会随机起风的天空。

An interactive miniature meadow built with **three.js 0.160**, shipped as one **index.html** you can open by double-clicking. A hand-painted, paper-diorama style turf island with 11 black-faced sheep, a purple tree, a flowing river, a dirt path, a ripe rice paddy, and a randomly gusting sky.

## 运行 / Run

1. 下载或克隆本仓库，找到 index.html。
2. 双击用浏览器打开（Chrome / Edge / Safari 均可）。
3. 页面需要联网，从 unpkg 载入 three.js 0.160 与 OrbitControls；没有构建步骤、没有 npm 安装。

1. Download or clone this repository and locate index.html.
2. Double-click it in a browser (Chrome / Edge / Safari).
3. It needs internet access to load three.js 0.160 and OrbitControls from unpkg. No build step, no npm install.

## 操作 / Controls

| 操作 | 效果 |
| --- | --- |
| 左键点击绵羊 | 选中：出现 8 个控制点、尺寸标签和行为工具栏 |
| 拖动蓝色选择框 | 平移绵羊（相机不转） |
| 在草地上按住左键画圈 | 松开后在该区域生成一只新绵羊，不会和已有羊重叠 |
| 空白处左键拖 / 右键拖 | 旋转视角 |
| 滚轮 | 缩放 |
| 工具栏 | 待机 / 吃草 / 行走 / 跳跃 · 撤销 / 重做 / 复制 / 删除 |
| Ctrl/Cmd + Z | 撤销；加 Shift 为重做 |

## 场景里有什么 / What's inside

- **草皮岛**：44 × 29 m 圆角矩形，Canvas 程序化 alpha 毛边遮罩，奶油色边缘 + 深绿色薄侧壁，像一块浮起来的草皮。
- **羊群**：11 只，奶油色毛球、黑脸、短腿；层级为 root → bob → body / neck / legs[4] / tail；整体缩放 1.25。
- **行为**：待机、吃草、行走、跳跃。跳跃总时长约 0.82 秒，世界高度顶点约 0.62 m；只有身体做 squash/stretch，腿在腾空时收起。
- **河流**：自定义 ShaderMaterial，两层噪声沿河道向下游滚动，带淡蓝 glint、岸边泡沫和低强度太阳反光；用岛屿遮罩裁掉河岸外的部分。风会让水面起伏、亮纹和泡沫略有变化，但保持连续顺滑。
- **稻田**：低矮田埂 + 约 3200 株实例化稻丛；每株 8 片绿叶 + 3 个交叉 ribbon 稻穗，稻穗向上拱起再下垂；顶点色分层（叶绿、穗琥珀金），随风摆动。羊的漫游目标会避开稻田。
- **树与云**：8 棵树（其中一棵紫色）、3 块石头、3 朵云；树冠和树干在风中的响应不同。
- **光照**：暖色平行光 + 半球光 + ACES 色调映射 + VSM 软阴影 + 页面暖色暗角。
- **界面**：全部简体中文；选中标签显示“宽 × 高 米 · 距离 米”。

## 风、水、稻田 / Wind, water, rice

风是一个全局风场：基础强度每 2–6 秒随机换一次（约 0.05–1.2），方向缓慢摆动，再叠加阵风噪声。轻的物体（树冠、稻叶、稻穗）始终随风动；重的物体（树干）只在风力超过 0.5 时轻微倾斜（最大约 3.4°），小风时保持完全静止。稻子用顶点着色器摆动，叶片和稻穗顺风弯，并有从田这头传到那头的行波；河面用风 uniform 调整起伏、流速与亮纹强度，但不会把水纹撕成白色碎线。

## 代码结构 / How it works

全部逻辑在 index.html 内，按 0–12 分节：

- 0 · helpers：数学工具、全局风场
- 1 · island metrics：岛屿尺寸、圆角矩形轮廓
- 2 · renderer / scene / camera：渲染器、轨道相机、ACES
- 3 · lighting：平行光、半球光、VSM 软阴影
- 4 · textures：草地、田土、岛屿毛边遮罩等 Canvas 纹理
- 5 · island mesh：圆角草皮网格与侧壁
- 6 · rivers / road / wheat：河流、道路、水面 shader、田埂与实例化稻丛
- 7 · trees / rocks / clouds：树、石头、云
- 8 · sheep：程序化羊模型、层级、材质
- 9 · behaviour：待机 / 吃草 / 行走 / 跳跃状态机
- 10 · selection UI + history：8 个控制点、尺寸标签、工具栏、撤销/重做/复制/删除
- 11 · picking：拾取、拖动、圈选生成、轨道控制切换
- 12 · loop：风场更新、树/稻/水 uniform、渲染循环

关键实现：Canvas 程序化纹理；岛屿 mask 通过 onBeforeCompile 注入到草地/河流/道路 shader；稻丛用 InstancedMesh + onBeforeCompile 风摆；屏幕空间 AABB 做选择框；射线与地面求交做拖动和圈选；圈选用 shoelace 面积 + 点在多边形内判断。

## 文件 / Files

- index.html — 全部场景、样式与逻辑（约 74 KB）
- docs/PROMPT.zh.md — 中文 AI 复刻提示词
- docs/PROMPT.en.md — English AI prompt
- .gitignore — 忽略本地记忆库与测试产物

## AI 提示词 / AI prompts

- 中文：[docs/PROMPT.zh.md](docs/PROMPT.zh.md)
- English: [docs/PROMPT.en.md](docs/PROMPT.en.md)

这两份提示词是根据最终成品整理的“完整复刻提示词”，可以直接粘给 AI，让它从零生成同款单文件场景。中英内容一一对应。

## 已知限制 / Limits

- 需要联网从 unpkg 载入 three.js；离线打开会显示中文兜底提示。
- 需要支持 WebGL2 的浏览器；移动端可打开，但手势与性能未专门优化。
- 单文件约 74 KB，所有纹理都是运行时 Canvas 生成，没有外部图片/模型资源。
- 羊的尺寸标签是世界空间 AABB，会随羊的朝向略有变化。
- 未附加开源许可证；如需 MIT 等许可证，请仓库作者补充 LICENSE。

## 许可 / License

No license file is included yet. Add one (for example MIT) if you want others to reuse this project.
