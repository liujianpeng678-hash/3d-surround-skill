# 3D环绕工作参考

## 全景图提示词结构

```text
Create a wide equirectangular 360° panorama for an interactive WebGL viewer.
Theme: [场景主题]. Camera at the center of the space, strong foreground,
midground and distant landmarks, left-right seamless continuity, no text,
no labels, no interface, no frame, no fisheye distortion, cinematic lighting,
rich spatial depth, high detail, 2:1 panorama composition.
```

生成后先检查：画面左右边缘是否能接上、地平线是否稳定、主要地标是否分布在不同方向、是否出现文字或 UI。若用户要网页内置图片，保留一份原始文件并记录其路径。

## Three.js 场景骨架

```js
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(70, width / height, 0.1, 1100);
camera.position.set(0, 0, 0.01);

const geometry = new THREE.SphereGeometry(500, 64, 32);
geometry.scale(-1, 1, 1); // 把球面翻到内侧
const texture = await new THREE.TextureLoader().loadAsync(panoramaUrl);
const material = new THREE.MeshBasicMaterial({ map: texture });
scene.add(new THREE.Mesh(geometry, material));
```

将拖动得到的经纬度转换为相机旋转，并将垂直角度限制在约 `[-85°, 85°]`。窗口变化时同步更新相机宽高比和 renderer 尺寸；组件卸载时释放纹理、几何体和 renderer。

## 热点数据

用数据驱动热点，不把文案散落在渲染逻辑里：

```js
{
  id: "crystal-lake",
  title: "水晶中庭",
  description: "湖面汇聚洞窟的蓝色光源。",
  yaw: 18,
  pitch: -6
}
```

热点的 `yaw`、`pitch` 应对应全景图中的真实方位。选中后显示标题、说明和关闭按钮；点击空白区域关闭说明。

## 离线交付

将构建后的 JS、CSS 和全景图合成一个 HTML：CSS 放入 `<style>`，应用脚本放入 `<script type="module">`，图片使用 `data:image/...`，Three.js 模块使用可解析的 data URL。去掉在线字体的强制依赖并保留系统字体回退。再提供 ZIP 和一份中文说明，写明手机用浏览器打开 HTML；微信内直接预览可能拦截本地脚本。

