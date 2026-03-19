---
name: meshy-symmetric-model
description: >-
  通过本地直接调用 Meshy.ai REST API 从参考图片生成对称低模 3D 角色。
  覆盖 Image-to-3D 全参数调用、轮询、GLB 下载、Blender 导入、
  bisect+mirror 拓扑对称强制、纹理压缩、对称验证、模型导出。
  当需要通过 Meshy API 生成 3D 模型、处理模型对称性、
  或批量生产标准化低模资产时使用。
---

# Meshy.ai 对称模型生成流程

## 前置条件

- **Blender 4.0+** 已安装并运行，且启用了 [Blender MCP Addon](https://github.com/ahujasid/blender-mcp)
- **Cursor MCP** 已连接到 Blender MCP Server
- **项目根目录 `.env`** 中已配置 `MESHY_API_KEY`（参见 `.env.example` 模板）
- 安装说明详见同目录下 `README.md`

## 流程概览

```
参考图片 → 本地直调 Meshy REST API（生成+减面一步完成）
→ 轮询等待 → 下载 GLB → 导入 Blender
→ bisect+mirror 拓扑对称 → 纹理压缩 → 验证 → 导出
```

## API 基础信息

- 基地址：`https://api.meshy.ai/openapi/v1`
- 认证：`Authorization: Bearer {API_KEY}`
- API Key 存储在项目根目录 `.env` 文件中：`MESHY_API_KEY=msy_xxx`
- 读取方式（优先级）：
  1. 项目 `.env` 文件（推荐）
  2. 系统环境变量 `MESHY_API_KEY`
  3. Blender 场景属性 `bpy.context.scene.blendermcp_meshy_api_key`（仅限 Blender 内执行时）

---

## Step 1: 创建 Image-to-3D 任务

**端点**：`POST /openapi/v1/image-to-3d`

### 完整参数表

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `image_url` | string | ✅ | - | 图片 URL 或 `data:image/png;base64,...` |
| `ai_model` | string | | `"latest"` | `"meshy-6"` / `"meshy-5"` / `"latest"` |
| `model_type` | string | | `"standard"` | `"standard"` / `"lowpoly"`（lowpoly 时忽略 remesh 相关参数） |
| `should_remesh` | bool | | meshy-6: `false` | **设为 `true` 启用内置减面**，省去单独 Remesh API |
| `target_polycount` | int | | 30000 | 目标面数 100-300000，需 `should_remesh: true` |
| `topology` | string | | `"triangle"` | `"triangle"` / `"quad"`，需 `should_remesh: true` |
| `symmetry_mode` | string | | `"auto"` | `"on"` / `"auto"` / `"off"` 形状对称控制 |
| `pose_mode` | string | | `""` | `"t-pose"` / `"a-pose"` / `""` |
| `should_texture` | bool | | `true` | 是否生成纹理（false 省 10 credits） |
| `enable_pbr` | bool | | `false` | 生成 PBR 贴图（metallic/roughness/normal） |
| `texture_prompt` | string | | `""` | 纹理文字引导，≤600 字符 |
| `texture_image_url` | string | | - | 纹理参考图（与 texture_prompt 二选一） |
| `save_pre_remeshed_model` | bool | | `false` | 保留 remesh 前高模 GLB |
| `image_enhancement` | bool | | `true` | 输入图片优化（仅 meshy-6） |
| `remove_lighting` | bool | | `true` | 移除贴图光照（仅 meshy-6） |
| `target_formats` | string[] | | 全部 | 如 `["glb"]` 可加速生成 |
| `moderation` | bool | | `false` | 内容审核 |

### 推荐参数组合（游戏低模角色）

```python
payload = {
    "image_url": image_data_uri,       # data:image/png;base64,...
    "ai_model": "meshy-6",
    "should_remesh": True,
    "target_polycount": 1500,
    "topology": "triangle",
    "symmetry_mode": "on",
    "pose_mode": "t-pose",
    "enable_pbr": False,
    "target_formats": ["glb"],
}
```

### 调用示例

```python
import urllib.request, json, base64, os

def load_api_key():
    """从 .env 文件或环境变量读取 Meshy API Key"""
    # 1. 尝试项目 .env
    env_path = os.path.join(os.path.dirname(__file__), '..', '.env')
    for candidate in [env_path, os.path.join(os.getcwd(), '.env')]:
        if os.path.isfile(candidate):
            with open(candidate) as f:
                for line in f:
                    line = line.strip()
                    if line.startswith('MESHY_API_KEY='):
                        return line.split('=', 1)[1].strip()
    # 2. 环境变量
    key = os.environ.get('MESHY_API_KEY')
    if key:
        return key
    raise RuntimeError("未找到 MESHY_API_KEY，请在 .env 文件或环境变量中设置")

API_KEY = load_api_key()
BASE_URL = "https://api.meshy.ai/openapi/v1"

# 读取图片转 base64
with open(image_path, 'rb') as f:
    data_uri = f'data:image/png;base64,{base64.b64encode(f.read()).decode()}'

payload = {
    "image_url": data_uri,
    "ai_model": "meshy-6",
    "should_remesh": True,
    "target_polycount": 1500,
    "topology": "triangle",
    "symmetry_mode": "on",
    "pose_mode": "t-pose",
    "enable_pbr": False,
    "target_formats": ["glb"],
}

req = urllib.request.Request(
    f"{BASE_URL}/image-to-3d",
    data=json.dumps(payload).encode(),
    method='POST'
)
req.add_header('Authorization', f'Bearer {API_KEY}')
req.add_header('Content-Type', 'application/json')

with urllib.request.urlopen(req) as resp:
    task_id = json.loads(resp.read().decode())['result']
```

### 返回

```json
{ "result": "019d03ee-cffb-7bb2-86f0-876d6780b97c" }
```

---

## Step 2: 轮询任务状态

**端点**：`GET /openapi/v1/image-to-3d/{task_id}`

```python
import time

def poll_meshy_task(task_id, api_key, interval=30, timeout=300):
    url = f"{BASE_URL}/image-to-3d/{task_id}"
    elapsed = 0
    while elapsed < timeout:
        req = urllib.request.Request(url)
        req.add_header('Authorization', f'Bearer {api_key}')
        with urllib.request.urlopen(req) as resp:
            data = json.loads(resp.read().decode())
        
        status = data['status']
        progress = data.get('progress', 0)
        
        if status == 'SUCCEEDED':
            return data
        elif status in ('FAILED', 'CANCELED'):
            raise Exception(f"Task {status}: {data.get('task_error', {}).get('message', '')}")
        
        time.sleep(interval)
        elapsed += interval
    
    raise TimeoutError(f"Task not completed within {timeout}s")
```

### 状态值

| status | 含义 |
|--------|------|
| `PENDING` | 排队中 |
| `IN_PROGRESS` | 生成中（检查 `progress` 字段 0-100） |
| `SUCCEEDED` | 完成，`model_urls` 可用 |
| `FAILED` | 失败，检查 `task_error.message` |
| `CANCELED` | 已取消 |

### 成功响应关键字段

```json
{
  "status": "SUCCEEDED",
  "model_urls": {
    "glb": "https://assets.meshy.ai/.../model.glb?...",
    "pre_remeshed_glb": "..."
  },
  "texture_urls": [{
    "base_color": "...",
    "metallic": "...",
    "normal": "...",
    "roughness": "..."
  }],
  "thumbnail_url": "..."
}
```

典型生成时间：60-120 秒。建议 30 秒间隔轮询。

---

## Step 3: 下载 GLB

```python
import tempfile, os

glb_url = result_data['model_urls']['glb']
tmp_path = os.path.join(tempfile.gettempdir(), f'meshy_{task_id}.glb')
urllib.request.urlretrieve(glb_url, tmp_path)
```

---

## Step 4: 导入 Blender 并定位

```python
import bpy, mathutils

bpy.ops.import_scene.gltf(filepath=tmp_path)
obj = bpy.context.selected_objects[0]
bpy.context.view_layer.objects.active = obj

# 按目标高度缩放，底部对齐地面
coords = [mathutils.Vector(v) for v in obj.bound_box]
height = max(c.z for c in coords) - min(c.z for c in coords)
z_min = min(c.z for c in coords)
scale = TARGET_HEIGHT / height
obj.scale = (scale, scale, scale)
obj.location = (TARGET_X, 0.0, -z_min * scale)
bpy.ops.object.transform_apply(location=False, rotation=False, scale=True)
```

---

## Step 5: 强制拓扑对称 — bisect + mirror

> **关键发现**：Meshy API 的 `symmetry_mode: "on"` 仅保证 3D 形状视觉对称，
> remesh 后的顶点拓扑不对称（实测 < 1%）。必须后处理。

```python
import bmesh

# 裁切 -X 侧
bpy.ops.object.mode_set(mode='EDIT')
bm = bmesh.from_edit_mesh(obj.data)
bmesh.ops.bisect_plane(
    bm,
    geom=bm.verts[:] + bm.edges[:] + bm.faces[:],
    plane_co=(0, 0, 0),
    plane_no=(-1, 0, 0),
    clear_inner=False,
    clear_outer=True
)
bmesh.update_edit_mesh(obj.data)
bm.free()
bpy.ops.object.mode_set(mode='OBJECT')

# Mirror 修改器
mod = obj.modifiers.new(name='Mirror', type='MIRROR')
mod.use_axis[0] = True
mod.use_clip = True
mod.use_mirror_merge = True
mod.merge_threshold = 0.001
bpy.ops.object.modifier_apply(modifier='Mirror')
```

UV 映射在 bisect+mirror 后保留，无需重新烘焙纹理。

---

## Step 6: 纹理压缩

```python
for node in obj.data.materials[0].node_tree.nodes:
    if node.type == 'TEX_IMAGE' and node.image:
        node.image.scale(512, 512)
```

---

## Step 7: 对称性验证

```python
def check_symmetry(obj, tolerance=0.002):
    verts = [v.co.copy() for v in obj.data.vertices]
    matched = unmatched = on_center = 0
    for v in verts:
        if abs(v.x) < tolerance:
            on_center += 1
            continue
        mirror = mathutils.Vector((-v.x, v.y, v.z))
        if any((v2 - mirror).length < tolerance for v2 in verts):
            matched += 1
        else:
            unmatched += 1
    total = matched + unmatched
    return (matched / total * 100) if total > 0 else 100.0
```

bisect+mirror 后对称率应为 **100.0%**。

---

## Step 8: 导出

```python
bpy.ops.object.select_all(action='DESELECT')
obj.select_set(True)
bpy.context.view_layer.objects.active = obj
bpy.ops.export_scene.gltf(
    filepath=output_path,
    use_selection=True,
    export_format='GLB',
    export_apply=True
)
```

---

## 已知问题

1. **`symmetry_mode: "on"` 不保证拓扑对称**：形状对称 ≠ 顶点对称，必须 bisect+mirror
2. **`should_remesh` 对 meshy-6 默认 false**：必须显式设为 `true`，否则返回高模（20万+面）
3. **bisect+mirror 后面数有 ±10% 偏差**：中缝切割/合并导致面数偏离 target_polycount
4. **Blender Decimate 不适合 AI 模型**：极端 ratio 导致碎裂，减面应交给 Meshy API
5. **Voxel Remesh 质量差**：方块感重、丢细节、破坏 UV，不推荐
6. **credits 消耗**：meshy-6 生成 20 credits + 纹理 10 credits = 30 credits/次

## 其他可用端点（参考）

| 端点 | 用途 |
|------|------|
| `POST /openapi/v1/image-to-3d` | 图生 3D（本流程主用） |
| `GET /openapi/v1/image-to-3d/{id}` | 查询任务状态 |
| `DELETE /openapi/v1/image-to-3d/{id}` | 删除任务 |
| `GET /openapi/v1/image-to-3d?page_num=1&page_size=10` | 列出历史任务 |
| `POST /openapi/v2/text-to-3d` | 文字生 3D（preview 阶段） |
| `POST /openapi/v2/text-to-3d` | 文字生 3D Refine（添加纹理） |
| `POST /openapi/v1/remesh` | 独立 Remesh（本流程不需要） |
| `GET /openapi/v1/balance` | 查询 credits 余额 |
