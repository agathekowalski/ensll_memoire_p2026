---
layout: default
title: Simulations acoustiques

---
# Simulations acoustiques du théâtre d'Argentomagus 

## Calibrations des coefficients d'absorption et de diffusion des matériaux 
Une campagne de mesure menée dans le théâtre analogue de Syracuse, permet de viser un temps
de réverbération moyen pour le théâtre d'Argentomagus. 

L'estimation du volume du modèle 3D du théâtre et la surface occupée par chacun de ses matériaux
est possible dans _Blender_ au moyen d'un script _Python_ : 

### Calucl du volume total du modèle (appelé _mesh_) : 

```python
import bpy
import bmesh
from mathutils import Vector

def mesh_volume(obj):
    # Get evaluated mesh (includes modifiers)
    dg = bpy.context.evaluated_depsgraph_get()
    eval_obj = obj.evaluated_get(dg)
    mesh = eval_obj.to_mesh()

    # Ensure triangulation for accurate calculation
    bm = bmesh.new()
    bm.from_mesh(mesh)
    bmesh.ops.triangulate(bm, faces=bm.faces)
    bm.to_mesh(mesh)
    bm.free()

    volume = 0.0
    verts = mesh.vertices

    for poly in mesh.polygons:
        # Poly is now always a triangle thanks to triangulation
        v1, v2, v3 = (verts[i].co for i in poly.vertices)
        volume += v1.cross(v2).dot(v3)

    eval_obj.to_mesh_clear()

    return abs(volume) / 6.0

obj = bpy.context.active_object
print("Volume:", mesh_volume(obj))

```

### Calucl des surfaces occupées par chaque matériau (appelés _material_) 

```python

import bpy
import bmesh
from mathutils import Vector

obj = bpy.context.active_object
mesh = obj.data

bm = bmesh.new()
bm.from_mesh(mesh)
bm.faces.ensure_lookup_table()

material_areas = {}

for face in bm.faces:
    mat_index = face.material_index
    area = face.calc_area()
    material_areas[mat_index] = material_areas.get(mat_index, 0) + area

bm.free()

print("Surface area by material index:")
for mat_index, area in material_areas.items():
    mat_name = mesh.materials[mat_index].name if mesh.materials[mat_index] else "None"
    print(f"{mat_name}: {area:.4f} m2")
```

## Essai audio

<audio controls>
  <source src="{{ '/simulations/extrait1.mp3' | relative_url }}" type="audio/mpeg">
</audio>
