---
layout: default
title: Simulations acoustiques

---
# Simulation acoustique du théâtre d'Argentomagus 

## Calibrations des coefficients d'absorption et de diffusion des matériaux 
Une campagne de mesure menée dans le théâtre analogue de Syracuse, permet de viser un temps
de réverbération moyen pour le théâtre d'Argentomagus. 

L'estimation du volume du modèle 3D du théâtre et la surface occupée par chacun de ses matériaux
est possible dans _Blender_ au moyen d'un script _Python_ : 

### Calcul du volume total du modèle (appelé _mesh_) : 

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

On obtient alors une valeur en m³ : 

```python
Volume: 18 179.68
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

On obtient alors des valeurs en m² : 

```python
Surface area by material index:
100% absorbent: 3 465,0182 m2
Pierre Poreuse (Argentomagus): 1391.9365 m2
Mur de scene (Argentomagus): 27.3457 m2
Marbre frons pulpiti (Argentomagus): 29.4726 m2
Pilier (Argentomagus): 32.2453 m2
CR4_parquet (BRAS): 275.8170 m2
Audience : 2932.8210
```


### Estimation du temps de réverbération moyen par bande de fréquences à partir des coefficients d'absorption définis pour chaque matériau : 

On calcule alors un temps de réverbération théorique du modèle 3D, en fonction des dimensions obtenues et des coefficients d'absorption
appliqués aux matériaux :

```python
import pyroomacoustics as pra 
from pyroomacoustics.parameters import air_absorption_table
import pandas as pd
import numpy 
import os

scaling_factor = 1.4 
# différence de diamètre entre Syracuse et Argentomagus 

m = air_absorption_table["20C_30-50%"]

script_dir = os.path.dirname(os.path.abspath(__file__))
csv_path = os.path.join(script_dir, "surfaces_materiaux_calibration.csv")

df = pd.read_csv(csv_path, sep=";")
#print(df)

frequencies = ['125Hz', '250Hz', '500Hz', '1000Hz', '2000Hz', '4000Hz', '8000Hz']
surfaces = df['Surface occupee (m2)']

# a*S par bande 
a_S = df[frequencies].multiply(surfaces, axis='index')

# somme des a*S
sum_a_S = a_S.sum()

S_total = surfaces.sum()

V_total = 18179.68 
#V_total = 23000


# a par bande
a_coefficient = sum_a_S.values / S_total

for i, f in enumerate(frequencies) :
    eyring_syracuse = pra.rt60_eyring(S_total*(scaling_factor)**2,V_total*(scaling_factor)**3,a_coefficient[i],m[i],343)
    print(f, eyring_syracuse)
```

On obtient un temps de réverbération théorique en secondes, par bande de fréquences : 

```python
125Hz 0.7543352292238422
250Hz 0.7480709075080848
500Hz 0.7444873103403937
1000Hz 0.7404742688208299
2000Hz 0.7820422372747707
4000Hz 0.840997203145685
8000Hz 1.2119304859605542
```

Grâce au facteur multiplicateur appliqué, on peut alors prévoir une surestimation ou non du temps de réverbération, 
et ajuster en fonction les coefficients de d'absorption et de diffusion des matériaux. 

## Essai audio

<audio controls>
  <source src="{{ '/simulations/extrait1.mp3' | relative_url }}" type="audio/mpeg">
</audio>
