# -3D模型格式代码示例-
常用3D模型从创建到应用的完整指南
## 一、Python读写3D模型代码示例（.3DS） 
依赖库：py3d（需安装：pip install py3d）
相关代码片段：

```
import py3d

# 读取3DS文件
model = py3d.read_3ds("character.3ds")

# 提取顶点数据
vertices = model.vertices
print(f"顶点数: {len(vertices)}")

# 导出为OBJ格式
model.export("character_converted.obj")

```

## 二、Python读写3D模型代码示例（OBJ/glTF）
### 1.OBJ读取（Trimesh库）：

```
import trimesh

mesh = trimesh.load("model.obj")
print(f"表面积: {mesh.area:.2f}, 体积: {mesh.volume:.2f}")
mesh.export("model_optimized.stl")  # 导出为STL

```

### 2.glTF读取（PyGLTF2库）：

```
from pygltf2 import GLTF2

gltf = GLTF2.load("scene.glb")
print(f"包含 {len(gltf.meshes)} 个网格")

```

## 三、Python读写3D模型代码示例（PLY）

```
import numpy as np  
  
def parse_ply_header(file_path):  
    """解析PLY文件头部信息"""  
    header = {}  
    elements = {}  
    current_element = None  
      
    try:  
        with open(file_path, 'r') as f:  
            # 验证PLY文件签名  
            line = f.readline().strip()  
            if line != 'ply':  
                raise ValueError('无效的PLY文件签名')  
              
            # 解析格式和版本  
            line = f.readline().strip()  
            if line.startswith('format'):  
                parts = line.split()  
                header['format'] = parts[1]  
                header['version'] = parts[2]  
              
            # 解析元素和属性  
            for line in f:  
                line = line.strip()  
                if not line or line == 'end_header':  
                    break  
                  
                if line.startswith('element'):  
                    parts = line.split()  
                    elem_name = parts[1]  
                    count = int(parts[2])  
                    elements[elem_name] = {'count': count, 'properties': []}  
                    current_element = elem_name  
                elif line.startswith('property'):  
                    if current_element:  
                        prop_type = line.split()[1]  
                        prop_name = ' '.join(line.split()[2:])  
                        elements[current_element]['properties'].append((prop_type, prop_name))  
    except FileNotFoundError:  
        raise FileNotFoundError(f"文件 {file_path} 不存在")  
      
    return header, elements  
  
def read_ply_vertices(file_path):  
    """读取PLY文件中的顶点坐标"""  
    try:  
        header, elements = parse_ply_header(file_path)  
          
        # 检查是否存在顶点元素  
        if 'vertex' not in elements:  
            raise ValueError('文件中缺少顶点元素')  
          
        # 确定顶点属性索引  
        vertex_props = elements['vertex']['properties']  
        x_idx = next((i for i, (t, n) in enumerate(vertex_props) if n == 'x'), None)  
        y_idx = next((i for i, (t, n) in enumerate(vertex_props) if n == 'y'), None)  
        z_idx = next((i for i, (t, n) in enumerate(vertex_props) if n == 'z'), None)  
          
        if x_idx is None or y_idx is None or z_idx is None:  
            raise ValueError('顶点元素中缺少x, y, z属性')  
          
        # 读取顶点数据  
        vertices = []  
        with open(file_path, 'r') as f:  
            # 定位到数据部分开始位置  
            f.seek(0)  
            for line in f:  
                if line.strip() == 'end_header':  
                    break  
              
            # 读取顶点数据  
            for _ in range(elements['vertex']['count']):  
                line = f.readline().split()  
                if len(line) <= max(x_idx, y_idx, z_idx):  
                    continue  
                x = float(line[x_idx])  
                y = float(line[y_idx])  
                z = float(line[z_idx])  
                vertices.append([x, y, z])  
          
        return np.array(vertices)  
    except Exception as e:  
        raise RuntimeError(f"读取顶点数据时出错: {str(e)}")  
  
# 示例使用  
if __name__ == '__main__':  
    # 替换为实际可用的PLY文件路径  
    file_path = './example.ply'  # 需要替换为实际文件路径  
      
    try:  
        vertices = read_ply_vertices(file_path)  
        print(f"成功读取 {len(vertices)} 个顶点")  
        print(f"前三个顶点坐标：\n{vertices[:3]}")  
    except Exception as e:  
        print(f"处理文件时出错: {str(e)}")

```
