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

### Calucl des surfaces occupées par chaque _material_ 

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
